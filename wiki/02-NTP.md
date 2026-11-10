# 02-NTP

## 1. Arquitectura y Requisitos

El servicio NTP distribuye la hora del servidor CentOS Stream 10 a los equipos autorizados de la red. Se utiliza Chrony, cuyo proceso de servicio se llama chronyd.

- **Servidor NTP:** máquina virtual CentOS Stream 10.
- **Clientes:** equipos de la misma red del laboratorio.
- **Servicio:** chronyd.
- **Puerto:** UDP 123.
- **Requisitos:** conectividad de red y, para sincronizar con fuentes externas, acceso a Internet y resolución DNS.

Identificar la IP, la interfaz y la red real antes de escribir la regla allow:

```bash
ip -br -4 addr
ip route
hostname -I
```

La directiva allow debe contener la subred real del laboratorio en formato CIDR. No usar 192.168.1.0/24 por costumbre: ese valor solo es correcto si coincide con la red real y su máscara.

## 2. Despliegue y Configuración

Instalar Chrony:

```bash
sudo dnf install -y chrony
```

Guardar una copia del archivo antes de editarlo:

```bash
sudo cp -a /etc/chrony.conf /etc/chrony.conf.bak
sudo nano /etc/chrony.conf
```

Conservar las fuentes pool/server que ya vienen configuradas. Agregar una directiva allow con la red real del laboratorio. Ejemplo únicamente ilustrativo:

```text
allow <RED_LOCAL_CIDR>
```

Si la red se comprobara como 10.21.165.0/24, la línea correspondiente sería allow 10.21.165.0/24. No utilizar ese ejemplo sin confirmar la máscara con ip route o la configuración de red.

Guardar el archivo y activar Chrony:

```bash
sudo systemctl enable --now chronyd
sudo systemctl restart chronyd
systemctl is-active chronyd
```

Permitir NTP en Firewalld:

```bash
sudo firewall-cmd --permanent --add-service=ntp
sudo firewall-cmd --reload
sudo firewall-cmd --list-services
```

Comprobar la sincronización y el puerto:

```bash
chronyc sources -v
chronyc tracking
sudo ss -lunp | grep ':123'
```

La marca de selección de una fuente en chronyc sources suele aparecer como ^* cuando esa fuente está seleccionada. La salida de tracking debe mostrar el estado de referencia y los datos de sincronización. Si aún está arrancando, esperar a que Chrony obtenga muestras y volver a consultar.

## 3. Aseguramiento Seguridad SELinux/Firewall

Mantener SELinux en modo Enforcing:

```bash
getenforce
sestatus
```

Permitir únicamente el servicio NTP en Firewalld y limitar Chrony a la subred autorizada con allow. No configurar allow 0/0 ni abrir UDP 123 a todas las redes salvo que exista una necesidad aprobada.

Para revisar la zona y las reglas activas:

```bash
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
```

En la configuración estándar de Chrony no se necesita desactivar SELinux. Si aparecen denegaciones, revisar primero los registros:

```bash
sudo ausearch -m AVC -ts recent
```

No utilizar setenforce 0 ni desactivar SELinux como solución.

## 4. Pruebas de Aceptación y Troubleshooting

**Criterios de aceptación**

| Prueba | Comando o acción | Resultado esperado |
|---|---|---|
| Servicio | systemctl is-active chronyd | active |
| Fuentes | chronyc sources -v | se muestran las fuentes configuradas |
| Estado | chronyc tracking | se muestran los datos de seguimiento |
| Puerto | ss -lunp | grep :123 | chronyd escucha en UDP 123 cuando se permite servir clientes |
| Firewall | firewall-cmd --list-services | aparece ntp |
| Cliente | consultar el estado NTP desde un equipo de la red | el cliente utiliza la IP del servidor |

En otro equipo Linux con Chrony se puede comprobar la conexión con:

```bash
chronyc -n sources
```

En Windows, la comprobación se puede hacer desde CMD como administrador, apuntando temporalmente el cliente al servidor:

```powershell
w32tm /query /status
w32tm /stripchart /computer:<IP_CENTOS> /dataonly /samples:5
```

**Errores frecuentes**

1. **El cliente no sincroniza:** confirmar la IP y máscara del cliente, la directiva allow, el servicio chronyd y la regla UDP 123.
2. **No aparecen fuentes válidas:** revisar Internet, DNS y las líneas pool/server de /etc/chrony.conf; consultar chronyc sources -v y journalctl -u chronyd.
3. **El servicio está activo pero no responde en UDP 123:** confirmar que existe una directiva allow válida o un peer/broadcast configurado, y revisar Firewalld.

Comandos de diagnóstico:

```bash
sudo systemctl status chronyd
sudo journalctl -u chronyd -n 50 --no-pager
sudo firewall-cmd --list-all
chronyc sources -v
chronyc tracking
```

**Evidencia que se debe adjuntar:** archivo chrony.conf con la subred correcta (sin datos sensibles), chronyd activo, salida de sources y tracking, regla NTP del firewall y una prueba desde un cliente.
