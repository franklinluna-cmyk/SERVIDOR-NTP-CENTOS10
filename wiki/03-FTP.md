# 03-FTP

## 1. Arquitectura y Requisitos

La práctica configura un servidor FTP en CentOS Stream 10 con VSFTPD. La transferencia se realiza desde Windows utilizando MobaXterm.

- Servidor: CentOS Stream 10.
- Servicio: VSFTPD.
- Cliente: MobaXterm.
- Usuario FTP: ftpandres.
- Puerto de control: TCP 21.
- Modo de transferencia utilizado: pasivo.
- Directorio de trabajo: /home/ftpandres/archivos.

El usuario FTP es una cuenta normal del sistema y no debe pertenecer al grupo administrativo `wheel` ni tener permisos sudo.

## 2. Despliegue y Configuración

Comprobar e instalar VSFTPD:

```bash
rpm -q vsftpd
sudo dnf install vsftpd -y
```

Activar el servicio:

```bash
sudo systemctl enable --now vsftpd
systemctl status vsftpd
```

Crear el usuario local, si todavía no existe:

```bash
sudo useradd -m ftpandres
sudo passwd ftpandres
```

Crear el directorio para transferencias:

```bash
sudo -u ftpandres mkdir -p /home/ftpandres/archivos
```

### Restringir la cuenta FTP

Asignar una shell que impida el acceso a una terminal interactiva:

```bash
sudo usermod -s /sbin/nologin ftpandres
```

Para que VSFTPD/PAM pueda reconocer esa shell en sistemas que consultan `/etc/shells`, agregarla solo si todavía no aparece:

```bash
grep -qxF /sbin/nologin /etc/shells || echo /sbin/nologin | sudo tee -a /etc/shells
```

Comprobar que no tenga privilegios administrativos:

```bash
id ftpandres
sudo -l -U ftpandres
```

No debe aparecer en el grupo `wheel` ni tener reglas sudo asignadas.

### Limitar el acceso al HOME

Editar la configuración:

```bash
sudo nano /etc/vsftpd/vsftpd.conf
```

Asegurar estas opciones (si ya existen, editar su valor en vez de duplicarlas):

```ini
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
```

El directorio raíz del usuario debe pertenecer a root y no ser modificable por la cuenta FTP. La carpeta de transferencia sí pertenece al usuario:

```bash
sudo chown root:root /home/ftpandres
sudo chmod 755 /home/ftpandres
sudo chown -R ftpandres:ftpandres /home/ftpandres/archivos
sudo chmod 750 /home/ftpandres/archivos
```

Reiniciar el servicio para aplicar los cambios:

```bash
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
```

### Datos utilizados en MobaXterm

| Campo | Valor registrado |
|---|---|
| IP | 10.21.165.107 |
| Usuario | ftpandres |
| Puerto | 21 |
| Modo | Pasivo |

## 3. Aseguramiento Seguridad SELinux/Firewall

### Firewalld

Permitir el servicio FTP:

```bash
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

El puerto de control FTP es TCP 21. Como la práctica utiliza modo pasivo, el servidor también necesita un rango de puertos pasivos definido en `vsftpd.conf` y permitido en Firewalld. El rango debe coincidir en ambos lugares; no se registra aquí un rango como realizado porque no aparece en la evidencia original.

### SELinux

Mantener SELinux en modo Enforcing y consultar las reglas disponibles:

```bash
getenforce
getsebool -a | grep ftpd
ls -Zd /home/ftpandres /home/ftpandres/archivos
```

Para permitir que VSFTPD escriba únicamente en la carpeta de transferencia, asignar a esa carpeta el tipo de contenido FTP de lectura y escritura y restaurar el contexto:

```bash
sudo semanage fcontext -a -t public_content_rw_t '/home/ftpandres/archivos(/.*)?'
sudo restorecon -Rv /home/ftpandres/archivos
```

En las políticas que ofrecen el booleano `ftpd_anon_write`, este permite al servicio FTP escribir en contenido etiquetado como `public_content_rw_t`. Comprobar primero que exista:

```bash
getsebool ftpd_anon_write
```

Si el booleano existe y la prueba de escritura muestra una denegación SELinux, habilitarlo de forma persistente:

```bash
sudo setsebool -P ftpd_anon_write on
```

No activar `ftpd_full_access` como primera solución: concede permisos mucho más amplios al servicio FTP. Revisar los eventos de SELinux si la transferencia falla:

```bash
sudo ausearch -m AVC -ts recent
```

## 4. Pruebas de Aceptación y Troubleshooting

### Pruebas de aceptación

Comprobar cuenta, shell y grupos:

```bash
getent passwd ftpandres
id ftpandres
sudo -l -U ftpandres
```

Comprobar que VSFTPD está activo:

```bash
sudo systemctl status vsftpd
```

Desde MobaXterm conectarse a la IP del servidor con el usuario `ftpandres`, puerto 21 y modo pasivo. La cuenta debe poder transferir archivos, pero no obtener una terminal SSH interactiva ni navegar fuera de su HOME FTP.

La prueba de transferencia documentada consistió en subir `prueba.txt` a `/home/ftpandres/archivos`, comprobarlo desde CentOS y descargarlo a Windows:

```bash
sudo cat /home/ftpandres/archivos/prueba.txt
```

Contenido comprobado:

```text
Prueba FTP de Andres Luna.
```

### Tres problemas frecuentes y su revisión

1. **No conecta al servidor:** revisar IP, servicio VSFTPD y regla FTP de Firewalld.
2. **No permite iniciar sesión con nologin:** comprobar la shell asignada, el archivo `/etc/shells` y los registros de VSFTPD/PAM.
3. **Permite iniciar sesión, pero no transferir:** revisar permisos de `archivos`, el contexto SELinux, el booleano FTP disponible y el rango pasivo configurado.

Comandos de diagnóstico:

```bash
sudo systemctl status vsftpd
sudo journalctl -u vsftpd -n 50 --no-pager
sudo firewall-cmd --list-all
getenforce
getsebool -a | grep ftpd
sudo ausearch -m AVC -ts recent
```

**Evidencia para adjuntar:** captura de la conexión FTP, usuario sin privilegios administrativos, shell `/sbin/nologin`, directorio HOME restringido, contexto SELinux y transferencia correcta del archivo. Las capturas deben tomarse después de aplicar y comprobar la configuración.
