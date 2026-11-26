# Defensa del borde

El nodo de acceso es el único equipo con exposición pública, por lo que concentra las medidas de contención del tráfico hostil antes de que llegue al servidor doméstico.

## Capa 1: límites por IP con nftables (`mcguard`)

Una tabla propia de nftables limita el abuso por conexión en el tráfico reenviado desde la interfaz pública (`ens3`) hacia el túnel (`wg0`). Como la regla actúa antes del DNAT hacia el servidor doméstico, se ve la IP real del cliente.

| Tráfico | Puerto | Límite por IP |
|---|---|---|
| Java (TCP) | 25565 | 5 conexiones simultáneas; 10 nuevas por segundo (ráfaga de 20) |
| Bedrock y Geyser (TCP) | 19132 y 19134 | 40 conexiones simultáneas; 30 nuevas por segundo (ráfaga de 60) |
| Bedrock y Geyser (UDP) | 19132 y 19134 | 30 flujos nuevos por segundo (ráfaga de 60) y 400 paquetes por segundo (ráfaga de 800) |
| Cualquiera | todos | Se descartan los paquetes con estado de conexión inválido |

### Parámetros del kernel

Archivo `/etc/sysctl.d/99-mcguard.conf`:

```
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.default.rp_filter = 2
net.netfilter.nf_conntrack_max = 131072
```

```bash
sudo sysctl --system
```

Las cookies SYN y el backlog ampliado atenuan las inundaciones de SYN; `rp_filter` en modo laxo (2) evita descartes por rutas asimétricas; y `nf_conntrack_max` fija el tamaño de la tabla de seguimiento de conexiones, de la que dependen los contadores por IP.

### Reglas

Archivo `/etc/nftables-mcguard.nft`. Las dos primeras líneas crean la tabla y la borran, de modo que cada recarga empieza desde cero y el archivo es repetible (ver la lección 3 en [lessons-learned.md](lessons-learned.md)).

```
table inet mcguard
delete table inet mcguard

table inet mcguard {
  set tcp_conns { type ipv4_addr; flags dynamic; size 65535; }
  set tcp_rate  { type ipv4_addr; flags dynamic,timeout; timeout 1m; size 65535; }
  set udp_new   { type ipv4_addr; flags dynamic,timeout; timeout 1m; size 65535; }
  set udp_pps   { type ipv4_addr; flags dynamic,timeout; timeout 1m; size 65535; }
  set bt_conns  { type ipv4_addr; flags dynamic; size 65535; }
  set bt_rate   { type ipv4_addr; flags dynamic,timeout; timeout 1m; size 65535; }

  chain forward {
    type filter hook forward priority -10; policy accept;
    iifname != "ens3" return
    oifname != "wg0" return

    ct state invalid counter drop

    # Java: máx. 5 conexiones simultáneas y 10 nuevas/s por IP
    tcp dport 25565 ct state new add @tcp_conns { ip saddr ct count over 5 } counter drop
    tcp dport 25565 ct state new add @tcp_rate { ip saddr limit rate over 10/second burst 20 packets } counter drop

    # Bedrock/Geyser (TCP): máx. 40 conexiones simultáneas y 30 nuevas/s por IP
    tcp dport { 19132, 19134 } ct state new add @bt_conns { ip saddr ct count over 40 } counter drop
    tcp dport { 19132, 19134 } ct state new add @bt_rate { ip saddr limit rate over 30/second burst 60 packets } counter drop

    # Bedrock/Geyser (UDP): flujos nuevos y paquetes por segundo por IP
    udp dport { 19132, 19134 } ct state new add @udp_new { ip saddr limit rate over 30/second burst 60 packets } counter drop
    udp dport { 19132, 19134 } add @udp_pps { ip saddr limit rate over 400/second burst 800 packets } counter drop
  }
}
```

### Servicio de systemd

Archivo `/etc/systemd/system/mcguard.service`, para que las reglas se carguen en cada arranque:

```
[Unit]
Description=Limites por IP para Minecraft (nftables)
After=network-online.target ufw.service
Wants=network-online.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/sbin/nft -f /etc/nftables-mcguard.nft
ExecStop=/usr/sbin/nft delete table inet mcguard

[Install]
WantedBy=multi-user.target
```

```bash
sudo nft -c -f /etc/nftables-mcguard.nft && echo "sintaxis OK"
sudo systemctl daemon-reload
sudo systemctl enable --now mcguard.service
```

### Verificación

Se abrieron 8 conexiones simultáneas desde el nodo de control al puerto 25565. Los contadores de la tabla muestran las que se descartaron:

```bash
sudo nft list table inet mcguard | grep -B1 -E 'packets [1-9]'
```

```
ct state invalid counter packets 0 bytes 0 drop
tcp dport 25565 ct state new add @tcp_conns { ip saddr ct count over 5 } counter packets 3 bytes 180 drop
```

Resultado: se descartaron 3 conexiones, como corresponde al límite de 5.

### Ajuste y reversión

- **Un grupo legítimo no puede entrar** (varios jugadores detrás de la misma IP): subir `over 5` a 10 o 15, validar con `sudo nft -c -f /etc/nftables-mcguard.nft` y recargar con `sudo systemctl restart mcguard.service`.
- **Revertir de inmediato:** `sudo nft delete table inet mcguard`.

## Capa 2: bloqueo automático con CrowdSec

CrowdSec analiza el comportamiento y publica decisiones de bloqueo que aplica un *bouncer* de nftables sobre el tráfico reenviado. Incluye la lista de la comunidad. Se eligió en lugar de Fail2Ban para tener un único sistema de baneos y alertas.

#### Instalación

Descargar el instalador, revisarlo y ejecutarlo:

```bash
curl -fsSL https://install.crowdsec.net -o crowdsec-install.sh
less crowdsec-install.sh
sudo sh crowdsec-install.sh
```

Comprobar si el paquete está disponible y, si hace falta, instalarlo desde los repositorios:

```bash
apt list crowdsec
sudo apt install crowdsec
```

#### Comprobaciones

```bash
sudo systemctl is-active crowdsec
sudo cscli capi status
sudo cscli collections list
sudo cscli metrics
```

#### Instalación del bouncer de firewall

```bash
sudo apt install crowdsec-firewall-bouncer-nftables
sudo nano /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml
```

Después de revisar y guardar la configuración:

```bash
sudo systemctl restart crowdsec-firewall-bouncer
sudo systemctl is-active crowdsec-firewall-bouncer
```

## Hallazgo: Bedrock y Geyser entran por TCP

Los contadores del DNAT mostraron cientos de paquetes TCP y casi ningún paquete UDP en los puertos 19132 y 19134. NetherNet usa TCP para la señalización, así que esas reglas deben mantenerse en el servidor doméstico (origen: nodo de acceso, por `wg0`). Al retirarlas, Bedrock dejó de funcionar. Ver la lección 11 en [lessons-learned.md](lessons-learned.md).

## Limitaciones conocidas

- Velocity aún no recibe la IP real del jugador: el reenvío con enmascaramiento la oculta. Está planificado el uso de PROXY protocol (ver [roadmap](roadmap.md)).
- Sin IP real en los registros, CrowdSec no puede detectar ataques propios de Minecraft, solo los de su lista comunitaria y los patrones genéricos.
- Ninguna de estas capas mitiga un ataque volumétrico que sature el enlace.
