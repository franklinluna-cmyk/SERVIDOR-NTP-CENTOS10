# Práctica 1: Las líneas de comando para levantar servicios en red

**Asignatura:** Sistemas Operativos I (Linux)  
**NRC:** 4292  
**Sistema:** CentOS Stream 10 en VirtualBox  
**Cliente:** Windows con MobaXterm y navegador

Este repositorio organiza los procedimientos de SSH, NTP, FTP y HTTP realizados durante las prácticas. Incluye los pasos documentados, la matriz de servicios, la seguridad y las pruebas de aceptación.

## Documentación

- [01-SSH](wiki/01-SSH.md)
- [02-NTP](wiki/02-NTP.md)
- [03-FTP](wiki/03-FTP.md)
- [04-HTTP](wiki/04-HTTP.md)
- [Arquitectura y matriz de puertos](wiki/00-ARQUITECTURA-Y-MATRIZ.md)
- [Evidencias y troubleshooting](wiki/05-EVIDENCIAS-Y-TROUBLESHOOTING.md)

## Servicios y puertos

| Servicio | Protocolo | Puerto |
|---|---|---:|
| SSH | TCP | 22 |
| NTP | UDP | 123 |
| FTP control | TCP | 21 |
| HTTP | TCP | 80 |

## Seguridad

La documentación contempla Firewalld y SELinux. Para FTP se describe una cuenta sin shell interactiva, sin privilegios administrativos y confinada a su directorio mediante VSFTPD. Para HTTP se conserva el contexto SELinux estándar del contenido web.

**Importante:** los comandos de endurecimiento añadidos a la guía son el procedimiento para completar esos requisitos en el servidor. Para presentar la configuración como aplicada, deben ejecutarse y comprobarse en CentOS, y adjuntar las capturas reales. No se afirma que GitHub haya ejecutado comandos en la máquina virtual.

## Referencias

- Red Hat Enterprise Linux 10 - SELinux: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/
- Red Hat Enterprise Linux 10 - Securing networks: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/securing_networks/
- Chrony: https://chrony-project.org/doc/latest/chrony.conf.html
- VSFTPD: https://security.appspot.com/vsftpd/vsftpd_conf.html
