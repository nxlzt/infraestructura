# Hoja de ruta

## Etapa 1: completar este proyecto

Elementos necesarios para operar la infraestructura con seguridad antes de la apertura pública.

- [x] **Defensa del borde (primera parte):** límites por IP con nftables y bloqueo automático con CrowdSec. Ver [edge-defense.md](edge-defense.md).
- [ ] **Defensa del borde (continuación):** PROXY protocol hacia Velocity para recuperar la IP real del jugador y registros útiles para CrowdSec.
- [ ] **Monitoreo:** Prometheus y Grafana desplegados en la red `monitoring`, con exportadores del host, de Podman y de los servidores de juego.
- [ ] **Alertamiento:** Alertmanager integrado con un canal de mensajería. El token se gestiona como secreto de Podman y nunca se incluye en un quadlet.
- [ ] **Respaldos:** copias cifradas hacia el nodo de almacenamiento, con restauración verificada y resguardo de las claves de los plugins.
- [ ] **Contención del tráfico saliente:** filtrado con nftables de las conexiones iniciadas por el usuario de servicio.
- [ ] **Endurecimiento de SSH:** escucha exclusiva en la IP del túnel, con arranque ordenado tras WireGuard.
- [ ] **Apertura pública por etapas:** primero Java durante varios días, para caracterizar el tráfico legítimo; después Bedrock y Geyser. Se documenta un interruptor de emergencia: retirar la regla de entrada en la nube.

## Etapa 2: defensa de infraestructura en tiempo real

Proyecto complementario que utiliza esta infraestructura como entorno de práctica.

1. **Modelo de amenazas:** quién ataca, qué obtiene y qué lo detiene.
2. **Línea base:** evaluación inicial medible del estado de seguridad de cada nodo.
3. **Detección:** qué alerta se dispara ante cada ataque y por qué.
4. **Simulacros contra el propio laboratorio:** barrido de puertos, inundación de conexiones, bots y prueba del interruptor de emergencia.
5. **Métricas:** tiempos de detección y de recuperación, respaldados con evidencia.
6. **Runbooks:** procedimientos de respuesta para cada incidente.
7. **Informe de auditoría:** hallazgos clasificados por severidad, con su remediación y la evidencia de verificación.
