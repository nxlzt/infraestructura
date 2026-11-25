# Hoja de ruta

## Pendiente en este proyecto
- [ ] **Monitoreo:** Prometheus y Grafana en la red `monitoring`, con exportadores del host, de Podman y de los servidores.
- [ ] **Alertas:** Alertmanager hacia un canal de mensajería; el token como secreto de Podman, nunca en un quadlet.
- [ ] **Backups:** cifrados hacia el nodo de almacenamiento, con una restauración probada y respaldo de las claves de los plugins.
- [ ] **Defensa del borde:** servicio de protección para el tráfico Java, límites por IP en el nodo de acceso y límites de ritmo para el tráfico UDP.
- [ ] **Salida de contenedores:** filtrar el tráfico saliente del usuario de servicio con nftables.
- [ ] **Apertura pública por etapas:** primero Java durante unos días para ver el tráfico normal, luego Bedrock y Geyser. Un interruptor de emergencia documentado (retirar la regla de entrada de la nube).
- [ ] **Endurecimiento de SSH:** escuchar solo en la IP del túnel, con arranque ordenado después de WireGuard.

## Segundo proyecto: defender una infraestructura en tiempo real
1. Modelo de amenazas: quién ataca, qué gana, qué lo detiene.
2. Detección: qué alerta salta con cada ataque y por qué.
3. Simulacros contra el propio laboratorio: barrido de puertos, inundación de conexiones, bots, prueba del interruptor de emergencia.
4. Métricas: tiempo de detección y de recuperación, con evidencia.
5. Runbooks para cada incidente.
