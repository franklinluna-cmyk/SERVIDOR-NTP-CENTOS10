# Práctica: Servidor NTP con Chrony en CentOS Stream 10

Documentación del procedimiento para configurar un servidor de tiempo local (NTP) en CentOS Stream 10 usando `chrony` y dar acceso a los equipos de la red.

---

## 1. Instalación y Configuración

Primero se instala el paquete chrony:


sudo dnf install -y chrony
Luego se edita el archivo /etc/chrony.conf:sudo nano /etc/chrony.conf
Dentro del archivo se mantuvieron los pools públicos de tiempo y se agregó la instrucción allow para permitir consultas desde la red local:pool 2.centos.pool.ntp.org iburst
allow 192.168.1.0/24
2. Servicio y Firewall
Para iniciar el servidor y hacer que arranque automáticamente con el sistema:sudo systemctl enable --now chronyd
Se abre el puerto NTP (UDP 123) en el firewall para que los clientes puedan conectarse:sudo firewall-cmd --add-service=ntp --permanent
sudo firewall-cmd --reload
3. Verificación
Para comprobar que el servidor está respondiendo y sincronizado con las fuentes externas:chronyc sources
chronyc tracking
## Justificación de los Pasos y Cambios Realizados

* **Instalación de Chrony (`dnf install chrony`):** Instalé este paquete porque CentOS no incluye un servicio de tiempo activo por defecto. Necesitaba disponer de la herramienta encargada de gestionar y responder las peticiones NTP.

* **Configuración del `pool`:** Mantuve y verifiqué los servidores del *pool* para que mi equipo tenga una fuente de referencia exacta desde Internet, ya que el servidor no puede distribuir la hora a la red si no la obtiene primero de relojes de referencia externos.

* **Inclusión de `allow 192.168.1.0/24`:** Agregué esta directiva porque Chrony viene bloqueado por defecto para evitar accesos no autorizados. Tuve que dar acceso explícito al segmento de red local para que los equipos clientes puedan consultar la hora.

* **Uso de `chronyd` con `enable --now`:** Ejecuté el comando sobre el nombre exacto del demonio (`chronyd`). Usé la opción `--now` para activar el servicio de inmediato sin reiniciar el sistema, y `enable` para asegurar que arranque automáticamente al encender el equipo.

* **Apertura del Firewall (`firewall-cmd`):** Habilité el servicio NTP (puerto UDP 123) en el cortafuegos debido a que el sistema operativo bloquea todo el tráfico entrante por seguridad. Sin esta regla, las peticiones de los clientes serían descartadas en la entrada.

* **Verificación con `chronyc`:** Ejecuté los comandos de diagnóstico para comprobar el estado del servicio y certificar que el servidor estuviera sincronizado con una fuente confiable antes de ofrecer la hora a la red.
