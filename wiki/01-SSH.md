# 01-SSH

## 1. Arquitectura y Requisitos

SSH permite administrar CentOS Stream 10 desde otro equipo de la red mediante una conexión cifrada.

- **Servidor:** máquina virtual con CentOS Stream 10.
- **Cliente:** Windows con MobaXterm o PowerShell.
- **Servicio:** OpenSSH Server (sshd).
- **Puerto:** TCP 22.
- **Cuenta:** usuario administrador normal, distinto de root, con permisos sudo cuando sea necesario.
- **Red:** la máquina virtual debe ser alcanzable desde el equipo cliente. En VirtualBox se puede utilizar Adaptador puente si la red del laboratorio lo permite.

Antes de configurar, identificar la interfaz y la IP actuales:

```bash
ip -br -4 addr
ip route
hostname -I
```

La IP puede cambiar si la interfaz utiliza DHCP. En ese caso, volver a consultarla antes de conectarse.

## 2. Despliegue y Configuración

Instalar OpenSSH Server y Firewalld:

```bash
sudo dnf install -y openssh-server firewalld
```

Iniciar ambos servicios y habilitar su arranque automático:

```bash
sudo systemctl enable --now firewalld
sudo systemctl enable --now sshd
systemctl is-active firewalld
systemctl is-active sshd
```

Permitir SSH en Firewalld:

```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
sudo firewall-cmd --list-services
```

La lista debe incluir ssh. Comprobar que el puerto está escuchando:

```bash
sudo ss -tlnp | grep ':22'
```

Desde Windows, abrir MobaXterm y crear una sesión SSH con estos datos:

- Remote host: IP actual de CentOS.
- Port: 22.
- Username: cuenta administrativa creada para el laboratorio.

También se puede probar desde PowerShell:

```powershell
Test-NetConnection <IP_CENTOS> -Port 22
```

## 3. Aseguramiento Seguridad SELinux/Firewall

Comprobar que SELinux continúa activo:

```bash
getenforce
sestatus
```

El resultado recomendado para la práctica es Enforcing. No se debe desactivar SELinux para resolver errores de conexión.

La administración diaria debe realizarse con una cuenta personal normal que tenga sudo, no iniciando sesión como root. Revisar la política efectiva de SSH:

```bash
sudo sshd -T | grep -i permitrootlogin
```

Como endurecimiento recomendado, establecer PermitRootLogin no en la configuración de SSH, después de confirmar que la cuenta administrativa normal funciona:

```bash
sudo nano /etc/ssh/sshd_config
```

Buscar o agregar la directiva:

```text
PermitRootLogin no
```

Validar antes de aplicar cualquier cambio y después recargar el servicio:

```bash
sudo sshd -t
sudo systemctl reload sshd
```

No cerrar la sesión administrativa actual hasta probar una segunda conexión. Si el laboratorio utiliza otro puerto, también debe coincidir la regla de Firewalld y la política de SELinux.

## 4. Pruebas de Aceptación y Troubleshooting

**Criterios de aceptación**

| Prueba | Comando o acción | Resultado esperado |
|---|---|---|
| Servicio activo | systemctl is-active sshd | active |
| Puerto abierto | ss -tlnp | grep :22 | sshd escucha en TCP 22 |
| Firewall | firewall-cmd --list-services | aparece ssh |
| Acceso remoto | MobaXterm o Test-NetConnection | conexión establecida |
| Usuario | whoami; id | muestra la cuenta normal, no root |
| SELinux | getenforce | Enforcing |

**Errores frecuentes**

1. **Connection refused:** comprobar que sshd esté activo, que escuche en TCP 22 y que no exista un error de configuración. Usar systemctl status sshd y journalctl -u sshd -n 50.
2. **Connection timed out:** revisar la IP actual, el modo de red de VirtualBox, la zona activa de Firewalld y la conectividad entre Windows y la VM.
3. **Permission denied:** verificar el nombre de usuario y la contraseña o clave autorizada. No habilitar acceso root como solución rápida.

Comandos de diagnóstico:

```bash
sudo systemctl status sshd
sudo journalctl -u sshd -n 50 --no-pager
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
```

**Evidencia que se debe adjuntar:** captura del servicio sshd activo, regla ssh en Firewalld y sesión de MobaXterm conectada con el usuario normal. No publicar contraseñas ni claves privadas.
