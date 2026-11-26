# Arquitectura

## Nodos
| Rol | Sistema | IP del túnel | Función |
|---|---|---|---|
| Nodo de control | Fedora | 10.10.10.1 | Administración por SSH a través del túnel |
| Servidor doméstico | Debian 13 | 10.10.10.2 | Ejecuta los contenedores (Podman rootless) |
| Nodo de almacenamiento | Arch | 10.10.10.3 | Almacenamiento y monitor secundario por VNC |
| Nodo de acceso | Ubuntu (nube) | 10.10.10.4 | Único nodo con puerto público; reenvía tráfico al servidor doméstico |

## Red WireGuard
- Interfaz `wg0`, red 10.10.10.0/24, puerto 51820/udp.
- Los nodos de casa inician la conexión hacia el nodo de acceso con keepalive de 25 s: el servidor doméstico no necesita puertos abiertos en el router.
- Entre nodos de la misma LAN, el puerto 51820 solo acepta las IP de los demás nodos.
- El acceso SSH está limitado por UFW a la interfaz del túnel y a las IP del nodo de control; `sshd` restringe además los usuarios con `AllowUsers`. Pendiente: limitar también la dirección de escucha de `sshd` a la IP del túnel (ver [roadmap](roadmap.md)).

## Flujos de tráfico
- **Java:** jugador, nodo de acceso (TCP 25565), DNAT, `wg0`, servidor doméstico (25565), Velocity, backend por la red `mc`.
- **Bedrock dedicado:** jugador, nodo de acceso (TCP y UDP 19132), DNAT, `wg0`, servidor doméstico, contenedor Bedrock.
- **Juego cruzado (Geyser):** jugador Bedrock, nodo de acceso (TCP y UDP 19134), DNAT, `wg0`, Geyser dentro de Velocity, backend Java.

## Puertos
| Servicio | Protocolo | Puerto |
|---|---|---|
| Velocity (Java) | TCP | 25565 |
| Bedrock dedicado (NetherNet) | TCP y UDP | 19132 |
| Geyser (NetherNet) | TCP y UDP | 19134 |

NetherNet usa señalización por TCP y juego por UDP en el mismo número de puerto, que debe coincidir en ambos lados del reenvío.

## Contenedores
| Contenedor | Imagen | Red | Puertos publicados | Memoria |
|---|---|---|---|---|
| proxy | `itzg/mc-proxy` (Velocity, Geyser, Floodgate) | mc | 25565 y 19134 en la IP del túnel | 768 MB |
| lobby | `itzg/minecraft-server:java25` (Paper) | mc | ninguno | 2 GB |
| java | `itzg/minecraft-server:java25` (Paper) | mc | ninguno | 1,5 GB |
| survival | `itzg/minecraft-server:java25` (Paper) | mc | ninguno | 2,5 GB |
| bedrock | `itzg/minecraft-bedrock-server` | por defecto | 19132 en la IP del túnel | 1 GB |

Los puertos se publican solo en la IP de `wg0`, nunca en todas las interfaces.
