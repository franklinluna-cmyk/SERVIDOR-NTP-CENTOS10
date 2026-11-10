# 03-FTP

## 1. Arquitectura y Requisitos

FTP permite transferir archivos entre Windows y CentOS Stream 10 mediante VSFTPD.

- **Servidor:** máquina virtual CentOS Stream 10.
- **Cliente:** MobaXterm en Windows.
- **Servicio:** vsftpd.
- **Usuario FTP:** ftpandres (cuenta normal, sin privilegios administrativos).
- **Directorio permitido:** /home/ftpandres, con una subcarpeta escribible llamada archivos.
- **Puerto de control:** TCP 21.
- **Modo de datos:** pasivo, TCP 30000-30100 en esta guía.

FTP tradicional no cifra usuario, contraseña ni datos. Utilizarlo solo en la red controlada del laboratorio. Para un entorno real expuesto a otras redes, configurar FTPS o preferir un protocolo seguro.

## 2. Despliegue y Configuración

Instalar VSFTPD e iniciar el servicio:

```bash
sudo dnf install -y vsftpd
sudo systemctl enable --now vsftpd
systemctl is-active vsftpd
```

Crear el usuario si todavía no existe. Si ya existe, conservarlo y ajustar su shell:

```bash
getent passwd ftpandres
```

Si el comando anterior no devuelve una cuenta, crearla:

```bash
sudo useradd -m -s /usr/sbin/nologin ftpandres
```

Si ftpandres ya existe, asignarle el shell sin inicio de sesión:

```bash
sudo usermod -s /usr/sbin/nologin ftpandres
```

Asignar o actualizar la contraseña FTP:

```bash
sudo passwd ftpandres
```

VSFTPD utiliza autenticación PAM. En sistemas donde pam_shells.so está activo, el shell nologin debe estar incluido en /etc/shells para permitir el acceso FTP. Comprobar la ruta y agregarla si falta:

```bash
command -v nologin
grep -E 'nologin' /etc/shells
```

Si la ruta que devuelve command -v nologin no aparece en /etc/shells, agregar exactamente esa ruta. Ejemplo habitual:

```bash
echo /usr/sbin/nologin | sudo tee -a /etc/shells
```

Preparar el directorio para que el usuario quede encerrado en su HOME, pero pueda escribir únicamente en archivos:

```bash
sudo chown root:root /home/ftpandres
sudo chmod 755 /home/ftpandres
sudo install -d -o ftpandres -g ftpandres -m 750 /home/ftpandres/archivos
```

Editar la configuración:

```bash
sudo cp -a /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
sudo nano /etc/vsftpd/vsftpd.conf
```

Asegurarse de que las siguientes opciones existan una sola vez y tengan estos valores:

```text
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
allow_writeable_chroot=NO
pasv_enable=YES
pasv_min_port=30000
pasv_max_port=30100
```

No dejar el HOME directamente escribible por ftpandres; la carpeta archivos es la zona destinada a las transferencias. Reiniciar y verificar:

```bash
sudo systemctl restart vsftpd
systemctl is-active vsftpd
sudo ss -tlnp | grep ':21'
```

En MobaXterm crear una sesión FTP con la IP actual de CentOS, usuario ftpandres, puerto 21 y modo pasivo habilitado. Subir un archivo a /archivos y descargarlo nuevamente.

## 3. Aseguramiento Seguridad SELinux/Firewall

**Cuenta limitada**

Comprobar que el usuario no pertenece al grupo wheel y que no tiene permisos sudo:

```bash
id ftpandres
sudo -l -U ftpandres
getent passwd ftpandres
```

El shell debe terminar en nologin. El usuario FTP no debe utilizarse para administración del sistema.

**Encierro en HOME**

Con chroot_local_user=YES, los usuarios locales quedan encerrados en su directorio personal. El HOME debe pertenecer a root y no ser escribible por ftpandres; la subcarpeta archivos pertenece al usuario.

```bash
ls -ld /home/ftpandres /home/ftpandres/archivos
```

**Firewall**

Abrir el puerto de control y el rango pasivo configurado:

```bash
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --permanent --add-port=30000-30100/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

El rango del firewall debe coincidir con pasv_min_port y pasv_max_port. Si se cambia el rango en VSFTPD, actualizar también Firewalld. Para una VM detrás de NAT puede ser necesario configurar la dirección pasiva y redirección de puertos; con adaptador puente normalmente se accede directamente a la IP de la VM.

**SELinux**

Mantener SELinux activo y revisar el booleano específico para directorios HOME:

```bash
getenforce
getsebool ftp_home_dir
```

Si el acceso a los directorios HOME es bloqueado por SELinux, habilitar únicamente el permiso específico requerido:

```bash
sudo setsebool -P ftp_home_dir on
```

No habilitar ftpd_full_access como solución general: concede permisos mucho más amplios. Si continúa el bloqueo, revisar las etiquetas y los eventos AVC:

```bash
ls -Zd /home/ftpandres /home/ftpandres/archivos
sudo ausearch -m AVC -ts recent
```

## 4. Pruebas de Aceptación y Troubleshooting

**Criterios de aceptación**

| Prueba | Comando o acción | Resultado esperado |
|---|---|---|
| Servicio | systemctl is-active vsftpd | active |
| Puerto | ss -tlnp | grep :21 | vsftpd escucha en TCP 21 |
| Usuario | id ftpandres; sudo -l -U ftpandres | cuenta normal, sin sudo ni grupo wheel |
| Shell | getent passwd ftpandres | muestra nologin |
| Encierro | iniciar FTP y navegar directorios | no puede salir de HOME |
| Subida | enviar prueba.txt a /archivos | archivo recibido |
| Lectura | cat /home/ftpandres/archivos/prueba.txt | contenido correcto |
| Descarga | descargar desde MobaXterm | archivo coincide con el original |
| Firewall | firewall-cmd --list-all | FTP y rango pasivo permitidos |
| SELinux | getenforce | Enforcing |

**Resultado documentado previamente:** se realizó una transferencia con MobaXterm usando la cuenta ftpandres, la IP 10.21.165.107 y el puerto 21. Se subió prueba.txt a /home/ftpandres/archivos; el contenido comprobado en CentOS fue “Prueba FTP de Andres Luna.” y luego se descargó a Windows. Esta evidencia demuestra la transferencia, pero no sustituye las pruebas adicionales de usuario nologin, chroot y SELinux.

**Errores frecuentes**

1. **530 Login incorrect:** revisar contraseña, local_enable=YES, anonymous_enable=NO, la existencia del usuario y la autenticación PAM. Comprobar que la ruta nologin esté incluida en /etc/shells cuando pam_shells.so la requiera.
2. **550 Permission denied al subir:** revisar write_enable=YES, permisos de /home/ftpandres/archivos, propiedad del directorio y el booleano ftp_home_dir. No solucionar con chmod 777 ni con ftpd_full_access.
3. **La sesión conecta, pero se queda esperando al transferir:** comprobar modo pasivo en MobaXterm, rango 30000-30100 en vsftpd.conf y Firewalld, y configuración NAT si corresponde.

Comandos de diagnóstico:

```bash
sudo systemctl status vsftpd
sudo journalctl -u vsftpd -n 50 --no-pager
sudo grep -E '^(anonymous_enable|local_enable|write_enable|chroot_local_user|pasv_enable|pasv_min_port|pasv_max_port)=' /etc/vsftpd/vsftpd.conf
sudo firewall-cmd --list-all
getsebool ftp_home_dir
```

**Evidencia que se debe adjuntar:** usuario y shell (sin mostrar contraseña), resultado de sudo -l -U ftpandres, permisos del HOME y archivos, configuración VSFTPD, Firewalld, SELinux y captura de subida/descarga en MobaXterm.
