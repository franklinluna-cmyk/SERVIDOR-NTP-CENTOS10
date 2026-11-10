# Evidencias, aceptación y contingencia

## 1. Lista de evidencias para la entrega

Adjuntar capturas reales del laboratorio. No sustituirlas por imágenes de ejemplo ni capturas de comandos que no se hayan ejecutado.

| N.º | Evidencia | Qué debe verse |
|---:|---|---|
| 1 | Topología y red | VirtualBox, modo de adaptador e IP de CentOS |
| 2 | Firewalld y SELinux | firewalld activo y getenforce en Enforcing |
| 3 | SSH | sshd activo y conexión MobaXterm con usuario normal |
| 4 | NTP | chronyd activo, chronyc sources y chronyc tracking |
| 5 | FTP - cuenta | usuario ftpandres, shell nologin y ausencia de sudo |
| 6 | FTP - seguridad | chroot, permisos de HOME, Firewalld y SELinux |
| 7 | FTP - transferencia | subida y descarga de prueba.txt |
| 8 | HTTP | apachectl configtest con Syntax OK y httpd activo |
| 9 | HTTP - navegador | página abierta desde Windows usando la IP de CentOS |

**Evidencia que ya consta en la documentación del trabajo:** la prueba FTP con MobaXterm utilizando ftpandres, IP 10.21.165.107 y puerto 21; se subió y descargó prueba.txt y se comprobó su contenido en CentOS. La IP debe actualizarse si ha cambiado. Las comprobaciones adicionales de seguridad deben capturarse antes de presentarlas como terminadas.

## 2. Pruebas de aceptación generales

| Servicio | Condición de aprobación |
|---|---|
| SSH | una cuenta administrativa normal puede conectarse remotamente por TCP 22 |
| NTP | Chrony está activo, tiene fuentes válidas y un cliente autorizado puede consultar la hora |
| FTP | el usuario transfiere archivos, no tiene sudo ni shell interactiva y queda limitado a su HOME |
| HTTP | Apache está activo y la página responde desde Windows por TCP 80 |
| Seguridad | SELinux permanece Enforcing y Firewalld solo permite los servicios necesarios |

## 3. Procedimiento de diagnóstico

Antes de cambiar configuraciones, revisar el servicio, sus registros, el firewall y SELinux. Guardar una copia del archivo antes de modificarlo.

### Caso 1: no hay conexión remota

Síntomas: SSH o HTTP no responde desde Windows.

Comprobaciones:

```bash
ip -br -4 addr
ip route
sudo systemctl status sshd
sudo systemctl status httpd
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
```

Contingencia: confirmar que Windows y la VM estén en redes que se puedan comunicar; actualizar la IP utilizada; activar el servicio correspondiente y corregir solo la regla de Firewalld necesaria. No desactivar el firewall.

### Caso 2: FTP permite iniciar sesión, pero no subir o descargar

Síntomas: error 550, permiso denegado o espera indefinida durante una transferencia.

Comprobaciones:

```bash
sudo systemctl status vsftpd
sudo journalctl -u vsftpd -n 50 --no-pager
sudo ls -ld /home/ftpandres /home/ftpandres/archivos
sudo firewall-cmd --list-all
getsebool ftp_home_dir
```

Contingencia: revisar write_enable, permisos de la carpeta archivos, chroot_local_user y la coincidencia del rango pasivo en VSFTPD y Firewalld. Habilitar ftp_home_dir solo si el bloqueo corresponde a ese permiso SELinux. No utilizar chmod 777 ni ftpd_full_access como atajo.

### Caso 3: servicio activo, pero no entrega el servicio esperado

Síntomas: Chrony no sincroniza, Apache muestra 403 o el usuario FTP no puede autenticarse.

Comprobaciones:

```bash
sudo journalctl -u chronyd -n 50 --no-pager
chronyc sources -v
sudo apachectl configtest
sudo journalctl -u httpd -n 50 --no-pager
sudo ausearch -m AVC -ts recent
```

Contingencia: corregir la directiva allow y la conectividad de Chrony; revisar permisos y etiquetas con restorecon para Apache; revisar PAM, el shell nologin y /etc/shells para FTP. No desactivar SELinux como solución.

## 4. Recomendaciones de operación

1. Mantener copias de seguridad antes de editar archivos de configuración.
2. Registrar la IP y la subred reales en cada captura de evidencia.
3. Comprobar los servicios después de cada cambio con systemctl y sus herramientas de diagnóstico.
4. Limitar los puertos abiertos a los servicios de la práctica.
5. No publicar contraseñas, claves privadas, direcciones sensibles ni archivos personales en el repositorio público.
6. FTP sin cifrado debe limitarse a una red de laboratorio; para producción usar FTPS o una alternativa segura.

## 5. Referencias técnicas

- Chrony Project, documentación de chrony.conf: https://chrony-project.org/doc/latest/chrony.conf.html
- Red Hat, Deploying web servers and reverse proxies (RHEL 10): https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/deploying_web_servers_and_reverse_proxies/
- Red Hat, Using SELinux (RHEL 10): https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/
- VSFTPD, documentación de configuración: https://security.appspot.com/vsftpd/vsftpd_conf.html
- Ubuntu Server, guía de VSFTPD y restricción de usuarios: https://ubuntu.com/server/docs/set-up-an-ftp-server/

Las referencias se utilizan para consultar las opciones de los servicios. Los resultados de aceptación de esta práctica deben provenir de las capturas y comandos ejecutados en la VM.
