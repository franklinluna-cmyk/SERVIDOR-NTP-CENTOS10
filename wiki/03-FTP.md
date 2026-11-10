# 03-FTP

## 1. Arquitectura y Requisitos

La práctica configura un servidor FTP en CentOS Stream 10 con VSFTPD. La transferencia se realiza desde Windows utilizando MobaXterm.

- Servidor: CentOS Stream 10.
- Servicio: VSFTPD.
- Cliente: MobaXterm.
- Usuario utilizado: ftpandres.
- Puerto de control: TCP 21.
- Modo de transferencia utilizado: pasivo.
- Directorio usado en la prueba: /home/ftpandres/archivos.

## 2. Despliegue y Configuración

Comprobar si VSFTPD está instalado:

```bash
rpm -q vsftpd
```

Instalar el servicio:

```bash
sudo dnf install vsftpd -y
```

Iniciar el servicio y habilitar el arranque automático:

```bash
sudo systemctl enable --now vsftpd
systemctl status vsftpd
```

Crear el usuario y asignarle una contraseña:

```bash
sudo useradd -m ftpandres
sudo passwd ftpandres
```

Crear la carpeta para los archivos:

```bash
sudo -u ftpandres mkdir -p /home/ftpandres/archivos
```

Consultar la IP de CentOS y las opciones principales de VSFTPD:

```bash
hostname -I
sudo grep -E '^(anonymous_enable|local_enable|write_enable)=' /etc/vsftpd/vsftpd.conf
```

Permitir FTP en Firewalld:

```bash
sudo firewall-cmd --add-service=ftp
sudo firewall-cmd --permanent --add-service=ftp
```

**Datos utilizados en MobaXterm**

| Campo | Valor registrado |
|---|---|
| IP | 10.21.165.107 |
| Usuario | ftpandres |
| Puerto | 21 |
| Modo | Pasivo |

## 3. Aseguramiento Seguridad SELinux/Firewall

En la práctica registrada se habilitó el servicio FTP en Firewalld. El puerto de control es TCP 21.

Comandos de consulta:

```bash
sudo firewall-cmd --list-services
sudo firewall-cmd --list-all
getenforce
```

**Alcance del registro:** la documentación original muestra la creación del usuario local `ftpandres` y la apertura del servicio FTP. No registra que se haya cambiado su shell a `nologin`, configurado el encierro `chroot`, abierto un rango pasivo específico o modificado una política SELinux. Por eso esos ajustes no se presentan como realizados.

El enunciado de la práctica menciona el aislamiento del usuario y SELinux. Si el licenciado exige evidencia de esas configuraciones, deben aparecer en capturas o comandos ejecutados durante el laboratorio; no es correcto afirmar que se aplicaron basándose únicamente en la creación del usuario.

## 4. Pruebas de Aceptación y Troubleshooting

**Prueba de transferencia registrada**

1. Se conectó MobaXterm a la IP `10.21.165.107`, con el usuario `ftpandres`, puerto 21 y modo pasivo.
2. Se subió el archivo `prueba.txt` a `/home/ftpandres/archivos`.
3. Se comprobó el archivo desde CentOS:

```bash
sudo cat /home/ftpandres/archivos/prueba.txt
```

El contenido mostrado fue:

```text
Prueba FTP de Andres Luna.
```

4. Se descargó el archivo a Windows mediante MobaXterm y se verificó que conservaba el mismo contenido.

**Criterios de comprobación**

| Comprobación | Acción o comando | Resultado registrado/esperado |
|---|---|---|
| Servicio | systemctl status vsftpd | revisar que esté activo |
| IP y usuario | hostname -I; id ftpandres | consultar datos de conexión |
| Firewall | firewall-cmd --list-services | aparece ftp |
| Subida | cargar prueba.txt con MobaXterm | archivo recibido |
| Contenido | cat /home/ftpandres/archivos/prueba.txt | texto correcto |
| Descarga | descargar desde MobaXterm | archivo conservado |

**Tres problemas frecuentes y su revisión**

1. **No conecta al servidor:** revisar la IP, el servicio vsftpd y la regla FTP del firewall.
2. **No permite iniciar sesión:** comprobar el usuario, la contraseña y las opciones local_enable y anonymous_enable.
3. **No permite transferir archivos:** comprobar write_enable, la carpeta de destino y el modo pasivo en MobaXterm.

Comandos de diagnóstico:

```bash
sudo systemctl status vsftpd
sudo journalctl -u vsftpd -n 50 --no-pager
sudo grep -E '^(anonymous_enable|local_enable|write_enable)=' /etc/vsftpd/vsftpd.conf
sudo firewall-cmd --list-all
```

**Evidencia registrada:** la subida, lectura y descarga de `prueba.txt` se encuentran descritas en el README del repositorio servidor-web.
