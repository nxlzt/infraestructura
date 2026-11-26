kk# Seguridad

## Modelo de amenazas (resumen)

| Activo | Amenaza | Control |
|---|---|---|
| Servidor doméstico | Acceso remoto no autorizado | SSH solo por el túnel, sin contraseña, `AllowUsers`, sin puertos innecesarios |
| Contenedores | Escape o abuso desde un servidor comprometido | Rootless, usuario sin `sudo`, quadlets de root, ... |
| Backends `online-mode=false` | Conexión directa suplantando jugadores | Sin puertos publicados; solo el proxy ... |
| Nodo de acceso | Servicio expuesto a internet | Solo 51820/udp propio y puertos de juego reenviados; SSH solo ... |
| Secretos (claves WireGuard, secreto de reenvío, clave de Floodgate) | Fuga | Modo 600, fuera de Git, sin variables ... |
| Nodo de control | Es el punto único de compromiso | Llaves con passphrase; llave hardware como mejora |

## Controles verificados

- Escaneo `nmap` interno: solo el puerto de SSH abierto.
- `ss -tulpn` en el servidor: solo SSH, WireGuard y el cliente DHCP, más los puertos publicados a propósito.
- Prueba de mínimo privilegio: un puerto del proxy responde al nodo de acceso y se agota al nodo de control.
- Políticas por defecto de UFW: denegar entrada.
- Reinicio del servidor: los servicios vuelven solos.

## Riesgos residuales conocidos

- **IP real del jugador:** se pierde en el enmascaramiento y en el reenvío de puertos de Podman.
- **Salida de contenedores sin filtrar:** un contenedor comprometido podría intentar alcanzar la LAN y el túnel.
- **Protección contra DDoS volumétrico:** la instancia gratuita no la incluye.
- **Plugins en versiones de desarrollo (Geyser y Floodgate):** requieren actualización frecuente.
- **Dirección anunciada por NetherNet:** por verificar que no revele la IP doméstica en los candidatos ICE.

## Gestión de secretos

| Secreto | Dónde vive |
|---|---|
| Claves privadas de WireGuard | `/etc/wireguard/` de cada nodo, modo 600 |
| Secreto de reenvío de Velocity | Carpeta de datos del proxy y configuración de los backends |
| Clave de Floodgate | Carpeta de plugins del proxy |
| Credenciales (monitoreo, backups) | Gestor de contraseñas; si van en Git, cifradas con `sops` y `age` |

## Antes de publicar cualquier archivo de este repositorio

- Sin IP públicas, ni en capturas de pantalla.
- Sin claves ni configuración de WireGuard.
- Sin identificadores de jugadores (UUID, gamertags).
- `gitleaks detect` sin hallazgos.

## CrowdSec

### Instalación

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

### Comprobaciones

```bash
sudo systemctl is-active crowdsec
sudo cscli capi status
sudo cscli collections list
sudo cscli metrics
```

### Instalación del bouncer de firewall

```bash
sudo apt install crowdsec-firewall-bouncer-nftables
sudo nano /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml
```

Después de revisar y guardar la configuración:

```bash
sudo systemctl restart crowdsec-firewall-bouncer
sudo systemctl is-active crowdsec-firewall-bouncer
```


