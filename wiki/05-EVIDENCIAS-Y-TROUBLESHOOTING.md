# Pruebas de aceptación, evidencias y troubleshooting

## 1. Evidencias que constan en los repositorios

| Servicio | Actividad documentada |
|---|---|
| SSH | instalación y activación de sshd; conexión desde MobaXterm con IP, usuario y puerto 22 |
| NTP | instalación de Chrony, edición de chrony.conf, activación de chronyd y comandos chronyc sources y tracking |
| FTP | conexión con MobaXterm, subida de prueba.txt, comprobación del contenido en CentOS y descarga a Windows |
| HTTP | instalación y activación de Apache, creación de index.html, prueba con curl y acceso desde el navegador |

La práctica FTP incluye el texto comprobado: `Prueba FTP de Andres Luna.`

## 2. Lista de capturas para organizar la entrega

Utilizar las capturas originales que se hayan tomado durante las prácticas. No se deben crear capturas simuladas.

| N.º | Captura | Contenido |
|---:|---|---|
| 1 | SSH | sesión de MobaXterm conectada a CentOS |
| 2 | NTP | chrony.conf y salida de chronyc sources / tracking |
| 3 | FTP - conexión | sesión MobaXterm con usuario ftpandres y puerto 21 |
| 4 | FTP - transferencia | archivo prueba.txt subido y descargado |
| 5 | HTTP - configuración | salida de apachectl configtest |
| 6 | HTTP - página | contenido de la página en el navegador |
| 7 | Firewall | reglas de los servicios trabajados |
| 8 | SELinux | estado consultado durante la práctica, si se tomó esa evidencia |

## 3. Pruebas de aceptación documentadas

| Servicio | Prueba | Criterio |
|---|---|---|
| SSH | conexión en MobaXterm | acceso a la terminal de CentOS |
| NTP | chronyc sources y chronyc tracking | consultar fuentes y estado de sincronización |
| FTP | subir, leer y descargar prueba.txt | conservar el contenido del archivo |
| HTTP | apachectl configtest, curl y navegador | configuración correcta y página accesible |

## 4. Troubleshooting: tres errores frecuentes

### Caso 1. No hay conexión con el servidor

Puede afectar a SSH, FTP o HTTP.

Comprobar la IP:

```bash
hostname -I
```

Revisar el estado del servicio correspondiente:

```bash
sudo systemctl status sshd
sudo systemctl status vsftpd
sudo systemctl status httpd
```

Revisar Firewalld:

```bash
sudo firewall-cmd --list-all
```

La IP debe coincidir con la utilizada en el programa cliente y la VM debe tener conectividad con Windows.

### Caso 2. El cliente FTP no permite iniciar sesión o transferir archivos

Comprobar el servicio y la configuración básica:

```bash
sudo systemctl status vsftpd
sudo grep -E '^(anonymous_enable|local_enable|write_enable)=' /etc/vsftpd/vsftpd.conf
sudo firewall-cmd --list-all
```

También se debe revisar que MobaXterm tenga el usuario, la contraseña, el puerto 21 y el modo pasivo correctos.

### Caso 3. El servicio está activo, pero no entrega el resultado esperado

Para NTP:

```bash
chronyc sources
chronyc tracking
sudo systemctl status chronyd
```

Para HTTP:

```bash
sudo apachectl configtest
systemctl is-active httpd
curl http://localhost
```

Revisar la configuración del servicio y sus registros:

```bash
sudo journalctl -u chronyd -n 50 --no-pager
sudo journalctl -u httpd -n 50 --no-pager
```

## 5. Observación sobre los requisitos de seguridad

Los README originales registran reglas de Firewalld para los servicios y la configuración de acceso de Chrony. No muestran comandos de modificación de SELinux ni configuraciones de usuario FTP sin shell o chroot. Estos puntos se dejan identificados para no afirmar que fueron aplicados sin evidencia.

## 6. Referencias técnicas

- Chrony Project: https://chrony-project.org/doc/latest/chrony.conf.html
- Red Hat Enterprise Linux 10, Deploying web servers and reverse proxies: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/deploying_web_servers_and_reverse_proxies/
- Red Hat Enterprise Linux 10, Using SELinux: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/
- VSFTPD configuration reference: https://security.appspot.com/vsftpd/vsftpd_conf.html
