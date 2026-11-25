# Red y firewall

## Tres capas
1. **Nube:** Security List o NSG de la instancia. Solo los puertos necesarios y, durante el desarrollo, solo desde la IP del administrador.
2. **Nodo de acceso:** DNAT, enmascaramiento y reglas `ufw route` explícitas.
3. **Servidor doméstico:** UFW por interfaz y por IP de origen.

Si un puerto no llega, se revisa en ese orden.

## Reenvío en el nodo de acceso
Bloque `*nat` de `/etc/ufw/before.rules`:
```
*nat
:PREROUTING ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-F PREROUTING
-F POSTROUTING
-A PREROUTING -i <if-publica> -p tcp --dport 25565 -j DNAT --to-destination 10.10.10.2:25565
-A PREROUTING -i <if-publica> -p tcp --dport 19132 -j DNAT --to-destination 10.10.10.2:19132
-A PREROUTING -i <if-publica> -p udp --dport 19132 -j DNAT --to-destination 10.10.10.2:19132
-A PREROUTING -i <if-publica> -p tcp --dport 19134 -j DNAT --to-destination 10.10.10.2:19134
-A PREROUTING -i <if-publica> -p udp --dport 19134 -j DNAT --to-destination 10.10.10.2:19134
-A POSTROUTING -o wg0 -d 10.10.10.2 -p tcp -m multiport --dports 25565,19132,19134 -j MASQUERADE
-A POSTROUTING -o wg0 -d 10.10.10.2 -p udp -m multiport --dports 19132,19134 -j MASQUERADE
COMMIT
```
Además: `net.ipv4.ip_forward=1`, `DEFAULT_FORWARD_POLICY="DROP"` y una regla `ufw route allow in on <if-publica> out on wg0 to 10.10.10.2 port <puerto> proto <tcp|udp>` por cada puerto.

Las líneas `-F` hacen que cada `ufw reload` empiece con las cadenas vacías: sin ellas, las reglas se duplican en cada recarga y, si se quita un puerto del bloque, su regla antigua sigue activa en vivo.

## Escalera de diagnóstico
1. Contadores del DNAT: `iptables -t nat -L PREROUTING -n -v`. Si están a 0, el tráfico no llega al nodo de acceso (capa de nube).
2. `tcpdump -ni wg0 'port <puerto>'` en el nodo de acceso: ¿sale tráfico por el túnel?
3. `tcpdump` en el servidor doméstico: ¿llega?
4. Registro de UFW en el servidor (`journalctl -k | grep UFW`).
5. Prueba desde otro contenedor de la misma red (`/dev/tcp/<contenedor>/<puerto>`): ¿escucha algo dentro?

Una conexión TCP que recibe SYN-ACK y luego un reset sin datos indica que el puerto publicado existe pero dentro del contenedor no escucha nadie.

## Servidor doméstico
UFW con denegación de entrada por defecto. Reglas por interfaz `wg0` y por origen: SSH solo desde el nodo de control; puertos de juego solo desde el nodo de acceso.
