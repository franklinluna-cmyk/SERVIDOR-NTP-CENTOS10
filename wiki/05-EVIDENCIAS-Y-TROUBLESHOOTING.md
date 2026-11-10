# Pruebas de aceptación, evidencias y troubleshooting

## 1. Evidencias de las prácticas

| Servicio | Actividad documentada |
|---|---|
| SSH | instalación y activación de sshd, regla Firewalld y conexión mediante MobaXterm |
| NTP | instalación de Chrony, edición de chrony.conf, activación de chronyd y comandos chronyc |
| FTP | instalación de VSFTPD, usuario ftpandres, conexión en MobaXterm y transferencia de prueba.txt |
| HTTP | instalación de Apache, creación de index.html, pruebas con apachectl y curl y acceso desde Windows |

La práctica FTP incluye el texto comprobado: `Prueba FTP de Andres Luna.`

## 2. Capturas para la entrega

Adjuntar capturas reales tomadas en el laboratorio. No utilizar imágenes simuladas.

| N.º | Captura | Contenido |
|---:|---|---|
| 1 | SSH | sesión de MobaXterm conectada a CentOS |
| 2 | NTP | chrony.conf y salida de chronyc sources / tracking |
| 3 | FTP - usuario | cuenta ftpandres con shell /sbin/nologin y sin permisos sudo |
| 4 | FTP - restricción | directorio HOME y carpeta archivos con sus permisos |
| 5 | FTP - SELinux | contexto de la carpeta de transferencia y booleano aplicado |
| 6 | FTP - transferencia | archivo prueba.txt subido y descargado |
| 7 | HTTP | apachectl configtest y página en el navegador |
| 8 | HTTP - SELinux | contexto httpd_sys_content_t de /var/www/html |
| 9 | Firewall | reglas de los servicios trabajados |

## 3. Pruebas de aceptación

| Servicio | Prueba | Criterio |
|---|---|---|
| SSH | conexión en MobaXterm | acceso a la terminal de CentOS |
| NTP | chronyc sources y tracking | consultar fuentes y sincronización |
| FTP | subir, leer y descargar prueba.txt | conservar el contenido del archivo |
| FTP - seguridad | shell, grupos y chroot | sin terminal interactiva, sin sudo y limitado al HOME FTP |
| HTTP | apachectl configtest, curl y navegador | configuración correcta y página accesible |
| SELinux | getenforce y ls -Z | SELinux activo y contextos correctos |

## 4. Troubleshooting: tres errores frecuentes

### Caso 1. No hay conexión con el servidor

Puede afectar a SSH, FTP o HTTP.

```bash
hostname -I
sudo systemctl status sshd
sudo systemctl status vsftpd
sudo systemctl status httpd
sudo firewall-cmd --list-all
```

Verificar que la IP coincida con la utilizada en el cliente y que la máquina virtual tenga conectividad con Windows.

### Caso 2. El cliente FTP no permite iniciar sesión o transferir archivos

```bash
sudo systemctl status vsftpd
sudo grep -E '^(anonymous_enable|local_enable|write_enable|chroot_local_user)=' /etc/vsftpd/vsftpd.conf
getent passwd ftpandres
id ftpandres
sudo -l -U ftpandres
sudo firewall-cmd --list-all
```

Revisar usuario, contraseña, shell nologin, permisos de la carpeta, encierro chroot y rango pasivo.

### Caso 3. SELinux bloquea el acceso a archivos

Comprobar el modo y buscar denegaciones:

```bash
getenforce
sudo ausearch -m AVC -ts recent
ls -Z /var/www/html
ls -Zd /home/ftpandres/archivos
```

Para restaurar el contexto web estándar:

```bash
sudo restorecon -Rv /var/www/html
```

Para FTP, comprobar que la carpeta de transferencia tenga el tipo asignado y que el booleano habilitado corresponda a la política instalada. No desactivar SELinux para resolver el problema.

## 5. Referencias técnicas

- Red Hat Enterprise Linux 10 - Using SELinux: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/
- Red Hat Enterprise Linux 10 - Securing networks: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/securing_networks/
- Chrony Project: https://chrony-project.org/doc/latest/chrony.conf.html
- VSFTPD: https://security.appspot.com/vsftpd/vsftpd_conf.html
