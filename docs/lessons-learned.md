# Lecciones aprendidas

| # | Síntoma | Causa | Arreglo | Lección |
|---|---|---|---|---|
| 1 | Un contenedor no resolvía nombres (`Failed to resolve ...`) pero el host sí | El DNS interno de la red de Podman quedó obsoleto tras un reinicio: se creó antes de que el DHCP escribiera `resolv.conf` | Parar y arrancar todos los contenedores de la red | Comparar host y contenedor aísla la capa; probar tras cada reinicio |
| 2 | Paper se cerraba con "requires Java 25" | La imagen `java21` no cumplía con la versión de Minecraft | Cambiar a la etiqueta `java25` | La versión de Java depende de la de Minecraft |
| 3 | Reglas NAT repetidas en cada recarga de UFW | UFW no vacía las cadenas `nat` al recargar | Añadir `-F` al bloque y probar con dos recargas | Probar que una configuración es repetible, no solo que funciona una vez |
| 4 | Un puerto no llegaba desde internet | Faltaba la regla de la capa de nube | Escalera de diagnóstico por capas | Con tres capas de firewall se revisa en orden, con contadores y `tcpdump` |
| 5 | La prueba de ping al servidor Bedrock agotaba el tiempo | La prueba asumía RakNet; el servidor usa NetherNet | Publicar TCP y UDP con el mismo puerto | Verificar el protocolo antes de diseñar la prueba |
| 6 | Reset TCP justo después del SYN-ACK en el puerto de Geyser | El plugin seguía escuchando en el puerto por defecto, no en el publicado | Corregir el puerto en la configuración | Un reset sin datos significa que el puerto existe pero nadie escucha dentro |
| 7 | `podman restart` fallaba con "address already in use" | Systemd y el comando manual peleaban por el mismo puerto | Gestionar siempre con `systemctl --user` | Un solo gestor de ciclo de vida |
| 8 | `chown .../*.jar` no encontraba archivos | La shell exterior expande el comodín antes de entrar al espacio de nombres | `podman unshare sh -c '...'` con comillas simples | Saber dónde se expande cada comodín |
| 9 | Desconexión inmediata al entrar a un backend nuevo | Sin reenvío moderno, el UUID era de modo offline (tercer bloque en 3) | Configurar el secreto en Paper | El UUID recibido es una prueba rápida del reenvío |
| 10 | Una regla de firewall quedó en el equipo equivocado | Se pegó un comando sin mirar el prompt | Borrarla y comprobar el prompt antes de ejecutar | Los hábitos de verificación importan tanto como los comandos |
