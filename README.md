# Práctica 1: Servicios de red en CentOS Stream 10

**Asignatura:** Sistemas Operativos I (Linux)  
**NRC:** 4292  
**Tema:** Líneas de comando para levantar servicios en red  
**Sistema:** CentOS Stream 10 en VirtualBox

Este repositorio reúne la documentación de la práctica de servicios SSH, NTP, FTP y HTTP. Las guías están organizadas para trasladarse a las páginas de GitHub Wiki con los nombres solicitados por la práctica.

## Contenido

| Página solicitada | Documento |
|---|---|
| 01-SSH.md | [Abrir documentación SSH](wiki/01-SSH.md) |
| 02-NTP.md | [Abrir documentación NTP](wiki/02-NTP.md) |
| 03-FTP.md | [Abrir documentación FTP](wiki/03-FTP.md) |
| 04-HTTP.md | [Abrir documentación HTTP](wiki/04-HTTP.md) |

Documentación complementaria:

- [Arquitectura, topología y matriz de puertos/seguridad](wiki/00-ARQUITECTURA-Y-MATRIZ.md)
- [Evidencias, pruebas de aceptación y troubleshooting](wiki/05-EVIDENCIAS-Y-TROUBLESHOOTING.md)

## Arquitectura general

La VM CentOS Stream 10 se conecta a la red mediante VirtualBox. Desde Windows se administra SSH con MobaXterm, se prueba FTP con MobaXterm y se consulta HTTP con un navegador. Chrony ofrece la hora a los clientes autorizados.

## Matriz resumida de puertos

| Servicio | Protocolo/puerto |
|---|---|
| SSH | TCP 22 |
| NTP | UDP 123 |
| FTP control | TCP 21 |
| FTP pasivo | TCP 30000-30100 |
| HTTP | TCP 80 |
| HTTPS | TCP 443, solo si se configura TLS |

## Datos que deben confirmarse antes de entregar

- Consultar la IP y la máscara reales con `ip -br -4 addr` e `ip route`.
- La IP 10.21.165.107 fue usada en una prueba FTP anterior; verificar si sigue asignada.
- En Chrony, reemplazar `<RED_LOCAL_CIDR>` por la subred real. No utilizar una red de ejemplo.
- Adjuntar capturas propias de SSH, NTP, FTP, HTTP, Firewalld y SELinux.
- Confirmar la configuración de seguridad FTP: usuario sin shell interactiva, sin privilegios sudo, encerrado en HOME y con escritura solo en la carpeta `archivos`.

## Alcance de las evidencias actuales

La documentación existente registra una prueba FTP de subida y descarga del archivo `prueba.txt` con el usuario `ftpandres`. Las demás condiciones deben acreditarse con las capturas indicadas en la página de evidencias. No se declaran como aprobadas pruebas que todavía no tienen evidencia.

## Importante: publicar en GitHub Wiki

Los documentos de las cuatro páginas están guardados dentro de `wiki/` en este repositorio. El conector de GitHub disponible permite editar archivos del repositorio, pero no escribir directamente en el repositorio interno de GitHub Wiki. Para que aparezcan como páginas de la pestaña **Wiki**, crea cada página allí copiando el contenido del archivo homónimo de `wiki/`. Los documentos ya están preparados con los nombres y secciones requeridos.

## Referencias

- [Chrony - chrony.conf](https://chrony-project.org/doc/latest/chrony.conf.html)
- [Red Hat Enterprise Linux 10 - Web servers](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/deploying_web_servers_and_reverse_proxies/)
- [Red Hat Enterprise Linux 10 - SELinux](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/)
- [VSFTPD configuration reference](https://security.appspot.com/vsftpd/vsftpd_conf.html)
