# Infraestructura de servidores Minecraft con Podman rootless, WireGuard y seguridad por capas

Laboratorio de infraestructura diseñado para alojar servidores de Minecraft (Java y Bedrock, con juego cruzado) mediante **contenedores Podman rootless administrados con Quadlet y systemd**. La infraestructura utiliza una **red privada WireGuard de cuatro nodos** y una instancia en la nube como **punto de acceso seguro**.

El proyecto busca poner en práctica conocimientos de administración de sistemas, redes y seguridad informática mediante principios como el mínimo privilegio, la segmentación de servicios, la defensa en profundidad y la validación de cada medida de seguridad. Todo ello acompañado de documentación técnica clara y reproducible.

> **Estado:** proyecto en desarrollo. La sección Estado del proyecto distingue entre las funciones implementadas y verificadas y las que aún están pendientes. Los nombres de los equipos, las direcciones IP públicas y otros identificadores sensibles se han omitido o reemplazado.

---

## Qué demuestra este proyecto

- **Administración de Linux:** trabajo con Debian, Ubuntu, Fedora y Arch Linux.
- **Redes y conectividad:** implementación de VPN con WireGuard, cortafuegos en tres capas (nube, nodo de acceso y servidor), reglas DNAT y NAT, y diagnóstico de tráfico mediante `tcpdump`.
- **Contenedores sin privilegios:** uso de Podman rootless, Quadlet y systemd, cuentas de servicio sin acceso a `sudo`, configuración de subuid/subgid y administración de cgroups v2.
- **Seguridad informática:** aplicación del principio de mínimo privilegio, reducción de la superficie de exposición, gestión de secretos, análisis de amenazas y validación de controles de seguridad.
- **Documentación técnica:** registro de configuraciones, comandos, pruebas, problemas encontrados y soluciones aplicadas.

---

## Arquitectura

mermaid

```mermaid
graph LR
  P[Jugadores] -->|Internet| E[Nodo de acceso: Ubuntu en la nube]
  E -->|WireGuard| V[Velocity, Geyser y Floodgate]
  E -->|WireGuard| B[Bedrock dedicado]
  V --> L[lobby]
  V --> J[java]
  V --> S[survival]
  C[Nodo de control: Fedora] -.->|SSH por WireGuard| E
  C -.->|SSH por WireGuard| H[Servidor doméstico: Debian 13]
  C -.->|SSH por WireGuard| A[Nodo de almacenamiento: Arch]
```

Todos los contenedores se ejecutan en el servidor doméstico. El único equipo con exposición pública es el nodo de acceso, que mantiene la conectividad mediante WireGuard. El servidor doméstico no publica servicios directamente en internet.

Más detalles en docs/architecture.md.

---

## Decisiones de diseño y seguridad

| Decisión | Justificación |
| --- | --- |
| Acceso SSH únicamente mediante WireGuard, sin autenticación por contraseña ni acceso directo como root, con `AllowUsers` | Evitar la exposición de servicios de administración a internet |
| Reglas UFW basadas en interfaz y dirección IP de origen | Aplicar el principio de mínimo privilegio en cada nodo |
| Podman rootless con una cuenta de servicio sin `sudo`, contraseña ni acceso SSH | Reducir el impacto de una posible vulneración de un contenedor sobre el sistema anfitrión |
| Quadlets administrados por root | Impedir que la cuenta de servicio modifique sus propias definiciones de contenedores |
| Redes de contenedores independientes (`mc`, `monitoring`) | Separar los servicios y limitar la comunicación entre ellos |
| Backends Java sin puertos publicados y accesibles únicamente mediante el proxy | Evitar la exposición directa de los servidores que utilizan `online-mode=false` |
| Reenvío moderno de Velocity con un secreto compartido | Proteger la comunicación entre el proxy y los servidores backend |
| Límites de memoria por contenedor | Evitar que un servidor consuma todos los recursos disponibles del host |
| Cortafuegos en tres capas: nube, nodo de acceso y servidor | Aplicar defensa en profundidad y facilitar el diagnóstico de problemas de red |

Más información en docs/security.md.

---

## Estado del proyecto

### Implementado y verificado

- Red privada WireGuard de cuatro nodos, con reglas UFW basadas en interfaz y dirección IP de origen.
- Acceso SSH restringido al túnel WireGuard. Los escaneos internos con Nmap únicamente detectan el puerto SSH.
- Podman 5.4 rootless integrado con Quadlet y systemd. Inicio automático de los servicios comprobado después de reiniciar el servidor.
- Velocity configurado con reenvío moderno y servidores Paper detrás del proxy (`java`, `survival` y `lobby`, actualmente en integración).
- Geyser y Floodgate con NetherNet para habilitar el juego cruzado entre Minecraft Java y Bedrock.
- Servidor dedicado de Minecraft Bedrock con NetherNet.
- Reenvío de puertos TCP y UDP mediante el nodo de acceso, con pruebas de conectividad de extremo a extremo.

### Pendiente de implementación

- **Monitoreo y alertas:** integración de Prometheus, Grafana y Alertmanager.
- **Copias de seguridad:** implementación de respaldos cifrados y pruebas de restauración.
- **Protección del perímetro:** medidas adicionales para controlar y proteger el tráfico público.
- **Control del tráfico saliente:** filtrado de conexiones iniciadas desde los contenedores.
- **Exposición pública gradual:** habilitación progresiva de los servicios, acompañada de pruebas de seguridad.

Hoja de ruta completa en docs/roadmap.md.

---

## Lecciones aprendidas

Algunos de los problemas técnicos más relevantes encontrados durante el desarrollo:

1. **DNS interno de Podman:** los registros pueden quedar desactualizados después de un reinicio. Comparar la resolución DNS desde el host y desde el contenedor permite identificar dónde se encuentra el problema.
2. **Compatibilidad de Java:** la versión de Java necesaria depende de la versión de Minecraft. Es importante seleccionar una imagen compatible con los requisitos del servidor.
3. **Reglas NAT de UFW:** las reglas pueden duplicarse después de cada recarga si el bloque de configuración no comienza con `-F`.
4. **Diagnóstico de firewalls:** cuando existen tres capas de filtrado, es necesario revisar cada una por separado mediante contadores de paquetes y capturas con `tcpdump`.
5. **Compatibilidad de NetherNet:** la implementación moderna para Bedrock requiere tráfico TCP y UDP en el mismo puerto en ambos extremos. Las pruebas diseñadas exclusivamente para RakNet no son suficientes.

Las diez lecciones documentadas, con sus respectivos síntomas, causas y soluciones, se encuentran en docs/lessons-learned.md.

---

## Estructura del repositorio

text

```text
README.md
docs/
├── architecture.md
├── security.md
├── users.md
├── podman-quadlet.md
├── network.md
├── tests.md
├── lessons-learned.md
└── roadmap.md

quadlets/
└── Definiciones de contenedores y redes
    (ejemplos sin información sensible)
```

---

## Tecnologías utilizadas

- **Sistemas operativos:** Debian 13, Ubuntu, Fedora y Arch Linux.
- **Redes y seguridad:** WireGuard, UFW, iptables y NAT.
- **Contenedores:** Podman 5.4 rootless, Quadlet y systemd.
- **Minecraft:** Velocity, Paper, Geyser, Floodgate y Bedrock Dedicated Server.
- **Diagnóstico y administración:** tcpdump, Nmap y SSH.

---

**Proyecto en desarrollo.** La infraestructura se amplía y documenta progresivamente, priorizando la seguridad, la estabilidad y la reproducibilidad de cada implementación.
