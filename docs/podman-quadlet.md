# Podman rootless y Quadlet

## Idea
Quadlet convierte archivos `.container` y `.network` en servicios de systemd. Los servicios de usuario no pueden depender de unidades del sistema (por ejemplo `wg-quick@wg0`), así que cada quadlet lleva `Restart=on-failure` y `RestartSec=5`.

## Rutas
| Ruta | Dueño | Uso |
|---|---|---|
| `/etc/containers/systemd/users/1001/` | root | Quadlets del usuario de servicio |
| `/srv/minecraft/<servidor>/` | usuario de servicio | Datos de cada servidor |

Nombres generados: `mc.network` produce `mc-network.service`; `proxy.container` produce `proxy.service`.

## Ciclo de trabajo
```bash
sudo nano /etc/containers/systemd/users/1001/proxy.container
sudo systemctl --user -M usr-mc@ daemon-reload
sudo systemctl --user -M usr-mc@ start proxy.service
sudo systemctl --user -M usr-mc@ status proxy.service
```

## Rollback de un servicio
```bash
sudo systemctl --user -M usr-mc@ stop proxy.service
sudo rm /etc/containers/systemd/users/1001/proxy.container
sudo systemctl --user -M usr-mc@ daemon-reload
```
Los datos no se borran.

## Plantilla: añadir un servidor detrás de Velocity
1. Crear el quadlet (red `mc`, sin puertos publicados, límites de memoria).
2. Primer arranque para que genere sus archivos de configuración.
3. Configurar el reenvío moderno en Paper con el secreto del proxy.
4. Registrar el servidor en el proxy.
5. Reiniciar lo que cambió, con `systemctl`.
6. Probar con un cliente real y comprobar que el UUID recibido es el real (en un UUID real, el tercer bloque empieza por 4; empezar por 3 indica modo offline y reenvío mal configurado).

Los quadlets de ejemplo están en [`quadlets/`](../quadlets/).
