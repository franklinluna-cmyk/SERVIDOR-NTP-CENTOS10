# Práctica 1: Las líneas de comando para levantar servicios en red

**Asignatura:** Sistemas Operativos I (Linux)  
**NRC:** 4292  
**Sistema utilizado:** CentOS Stream 10 en VirtualBox  
**Cliente:** Windows con MobaXterm y navegador web

Este repositorio organiza la documentación de las prácticas de SSH, NTP, FTP y HTTP. Los procedimientos se basan en los pasos que constan en los repositorios de trabajo originales.

## Documentación

- [01-SSH.md](wiki/01-SSH.md)
- [02-NTP.md](wiki/02-NTP.md)
- [03-FTP.md](wiki/03-FTP.md)
- [04-HTTP.md](wiki/04-HTTP.md)
- [Arquitectura y matriz de puertos](wiki/00-ARQUITECTURA-Y-MATRIZ.md)
- [Pruebas de aceptación, evidencias y troubleshooting](wiki/05-EVIDENCIAS-Y-TROUBLESHOOTING.md)

## Topología general

Windows se conecta a la máquina virtual CentOS Stream 10 mediante la red configurada en VirtualBox. MobaXterm se utiliza para SSH y FTP; un navegador permite probar el servidor HTTP. Chrony proporciona el servicio de hora NTP.

## Matriz resumida de puertos

| Servicio | Protocolo | Puerto |
|---|---|---:|
| SSH | TCP | 22 |
| NTP | UDP | 123 |
| FTP | TCP | 21 |
| HTTP | TCP | 80 |

## Evidencias registradas

- SSH: instalación, activación de sshd, regla de Firewalld y conexión mediante MobaXterm.
- NTP: instalación de Chrony, configuración del pool y de la directiva allow, activación de chronyd y comandos de verificación.
- FTP: instalación de VSFTPD, creación de ftpandres, conexión en MobaXterm y transferencia de prueba.txt.
- HTTP: instalación de Apache, creación de index.html, pruebas con apachectl y curl y acceso desde Windows.

## Datos que aparecen en los registros

- IP utilizada en la prueba FTP: 10.21.165.107.
- Usuario FTP: ftpandres.
- Carpeta de transferencia: /home/ftpandres/archivos.
- Red permitida en la configuración NTP registrada: 192.168.1.0/24.

Los datos corresponden a sus respectivas prácticas y no se asume que pertenecen a la misma red.

## Nota sobre seguridad y evidencias

La documentación incluye las reglas de Firewalld registradas para cada servicio. Los archivos originales no documentan cambios específicos de SELinux ni la configuración de FTP con shell nologin o chroot. Esos puntos se identifican en las páginas correspondientes sin afirmar que se realizaron. Las capturas de aceptación deben ser las tomadas en el laboratorio.

## Referencias

- Chrony: https://chrony-project.org/doc/latest/chrony.conf.html
- Red Hat Enterprise Linux 10 - Servidores web: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/deploying_web_servers_and_reverse_proxies/
- Red Hat Enterprise Linux 10 - SELinux: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/
- VSFTPD: https://security.appspot.com/vsftpd/vsftpd_conf.html

## Publicación de las páginas en GitHub Wiki

Los archivos Markdown se encuentran en la carpeta `wiki/` de este repositorio. El conector utilizado permite actualizar los archivos del repositorio, pero no crear directamente las páginas en el repositorio independiente de GitHub Wiki. Para tenerlas también en la pestaña **Wiki**, copia allí el contenido de cada archivo con el nombre indicado.
