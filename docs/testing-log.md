# Bitácora de pruebas

Cada fase terminó con una prueba verificable. Resumen:

| Fase | Qué se hizo | Prueba | Resultado |
|---|---|---|---|
| 0 | Podman rootless, subuid, linger, delegación de cgroups | `podman info` rootless; `memory.max` aplicado en un contenedor de prueba | Correcto |
| 1 | Redes Quadlet, Cockpit fuera, SSH restringido | `nmap` por el túnel: solo SSH abierto; `ss -tulpn` sin escuchas inesperadas | Correcto |
| 2 | Velocity solo | Conexión al puerto desde el nodo de acceso abierta y desde el de control agotada; reinicio del servidor | Correcto |
| 3 | Servidor `java` con reenvío moderno | Entrada con cliente real; UUID real recibido | Correcto |
| 4 | Servidor Bedrock dedicado | Puerto TCP abierto con regla y cerrado sin ella | Correcto |
| 5 | `survival`, Geyser y Floodgate | Cliente Bedrock llega a Velocity y al backend; cambio de servidor con `/server` | Correcto |
| 6 | Reenvío por el nodo de acceso | Prueba de extremo a extremo desde internet (Java, Bedrock y Geyser) | Correcto |
| 7 | Límites por IP en el nodo de acceso (`mcguard`) | 8 conexiones simultáneas desde el nodo de control al puerto 25565 | Correcto: se descartaron 3 |

## Comandos de las pruebas clave
```bash
# Mínimo privilegio: debe abrir desde el nodo de acceso y agotarse desde el de control
timeout 3 bash -c '</dev/tcp/10.10.10.2/25565' && echo abierto || echo cerrado

# Superficie expuesta del servidor
sudo ss -tulpn
nmap -Pn -p- 10.10.10.2          # desde el nodo de control

# Límites de memoria realmente aplicados
podman run --rm --memory=64m docker.io/library/alpine cat /sys/fs/cgroup/memory.max
```
