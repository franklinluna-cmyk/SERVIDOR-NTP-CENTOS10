# Práctica 3: La línea de comando para levantar servicios en red

**Asignatura:** Sistemas Operativos I (Linux)  
**NRC:** 4292  
**Código:** TICS-00087  
**Nivel:** Primero  
**Periodo académico:** Septiembre 2026 - Febrero 2027  
**Sistema:** CentOS Stream 10 en VirtualBox  
**Cliente:** Windows con MobaXterm y navegador

Este repositorio reúne la documentación de los servicios SSH, NTP, FTP y HTTP implementados en CentOS Stream 10. La documentación principal de la práctica se encuentra en la **Wiki nativa de GitHub**.

## Wiki de la práctica

- [Portada de la Wiki](https://github.com/franklinluna-cmyk/SERVIDOR-NTP-CENTOS10/wiki)
- [Arquitectura y matriz de servicios](https://github.com/franklinluna-cmyk/SERVIDOR-NTP-CENTOS10/wiki/00%E2%80%90ARQUITECTURA%E2%80%90Y%E2%80%90MATRIZ)
- [01 - SSH](https://github.com/franklinluna-cmyk/SERVIDOR-NTP-CENTOS10/wiki/01%E2%80%90SSH)
- [02 - NTP](https://github.com/franklinluna-cmyk/SERVIDOR-NTP-CENTOS10/wiki/02%E2%80%90NTP)
- [03 - FTP](https://github.com/franklinluna-cmyk/SERVIDOR-NTP-CENTOS10/wiki/03%E2%80%90FTP)
- [04 - HTTP](https://github.com/franklinluna-cmyk/SERVIDOR-NTP-CENTOS10/wiki/04%E2%80%90HTTP)
- [05 - Pruebas de aceptación y troubleshooting](https://github.com/franklinluna-cmyk/SERVIDOR-NTP-CENTOS10/wiki/05%E2%80%90EVIDENCIAS%E2%80%90Y%E2%80%90TROUBLESHOOTING)

## Servicios y puertos de referencia

| Servicio | Protocolo | Puerto o rango | Función |
|---|---|---:|---|
| SSH | TCP | 22 | Administración remota |
| NTP | UDP | 123 | Sincronización horaria |
| FTP - control | TCP | 21 | Control de la sesión FTP |
| FTP - datos activo | TCP | 20 | Canal de datos en modo activo |
| FTP - datos pasivo | TCP | Rango definido en VSFTPD | Transferencia de datos en modo pasivo |
| HTTP | TCP | 80 | Acceso al servidor web |
| HTTPS | TCP | 443, si se configura TLS | Acceso web cifrado |

El rango pasivo de FTP debe coincidir en la configuración de VSFTPD y en las reglas de Firewalld. El puerto 443 solo se considera habilitado cuando se implementa HTTPS.

## Seguridad

La documentación contempla Firewalld y SELinux. Para FTP se describe una cuenta dedicada sin shell interactiva ni privilegios administrativos, restringida al directorio previsto. Para HTTP se conserva el contexto SELinux de solo lectura del contenido web.

**Nota de validación:** los procedimientos documentados deben ejecutarse y comprobarse en la máquina virtual. Este repositorio no ejecuta comandos en CentOS ni demuestra por sí solo el estado operativo de los servicios. Los resultados deben corresponder a las comprobaciones realizadas en el laboratorio.

## Referencias técnicas

- Red Hat Enterprise Linux 10 - [Using SELinux](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/)
- Red Hat Enterprise Linux 10 - [Securing networks](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/securing_networks/)
- Red Hat Enterprise Linux 10 - [Configuring time synchronization](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_time_synchronization/)
- Chrony - [chrony.conf documentation](https://chrony-project.org/doc/latest/chrony.conf.html)
- VSFTPD - [configuration reference](https://security.appspot.com/vsftpd/vsftpd_conf.html)
