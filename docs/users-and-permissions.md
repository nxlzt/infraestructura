# Usuarios y permisos

## Quién es quién
| Usuario | Dónde | Función | Cómo se entra |
|---|---|---|---|
| Administrador | Servidor doméstico | Administra el sistema (`sudo`) | SSH desde el nodo de control, solo por el túnel |
| `usr-mc` | Servidor doméstico | Opera los contenedores. Sin `sudo`, sin contraseña, sin SSH, shell `nologin` | `sudo machinectl shell usr-mc@ /bin/bash` desde el administrador |
| `root` | Todos | No se usa directamente | `sudo` puntual |

## Qué ejecuta cada usuario
| Tarea | Usuario |
|---|---|
| Firewall, WireGuard, paquetes, quadlets (propiedad de root) | Administrador con `sudo` |
| Controlar servicios de `usr-mc` sin entrar | Administrador: `sudo systemctl --user -M usr-mc@ <acción> <servicio>` |
| `podman ps`, `podman logs`, editar archivos de los servidores | `usr-mc` |

## Configuración necesaria para rootless
- Rangos `subuid` y `subgid` para `usr-mc`.
- `loginctl enable-linger usr-mc`: sus servicios arrancan sin sesión abierta.
- Delegación de cgroups en `user@.service` (`Delegate=cpu cpuset io memory pids`) para aplicar límites de memoria.

## Propiedad de archivos con `:U`
Con `Volume=...:U`, Podman cambia el dueño de la carpeta al usuario interno del contenedor. Desde el host se ve un UID numérico del rango subuid. Para editar esos archivos se entra al mismo espacio de nombres: `podman unshare ...`.

## Reglas de oro
1. Nunca `sudo podman`: crea contenedores de root y sus puertos pueden saltarse UFW.
2. Nunca `chmod -R 777`.
3. El ciclo de vida de los servicios se gestiona siempre con `systemctl --user`, no con `podman start/stop/restart`.
4. Los comodines con `podman unshare` van dentro de `sh -c '...'` con comillas simples.
5. Antes de pegar un comando, comprobar el prompt: usuario y equipo correctos.
