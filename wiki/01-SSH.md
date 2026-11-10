# 01-SSH

## 1. Arquitectura y Requisitos

La práctica utiliza una máquina virtual con CentOS Stream 10 en VirtualBox y un equipo Windows como cliente. La conexión remota se realiza desde MobaXterm mediante SSH.

- Servidor: CentOS Stream 10.
- Cliente: Windows con MobaXterm.
- Servicio: OpenSSH Server (sshd).
- Puerto: TCP 22.
- Herramientas: terminal de CentOS y MobaXterm.

## 2. Despliegue y Configuración

Los comandos registrados para instalar y activar SSH y el firewall son:

```bash
sudo dnf install -y openssh-server firewalld
sudo systemctl enable --now firewalld sshd
```

Se habilitó el servicio SSH en el firewall:

```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

Para consultar la dirección IP y el usuario de CentOS:

```bash
hostname -I
whoami
```

En MobaXterm se creó una sesión SSH con la IP obtenida, el usuario de CentOS y el puerto 22.

## 3. Aseguramiento Seguridad SELinux/Firewall

La regla registrada en el repositorio permite el servicio SSH a través de Firewalld. El puerto utilizado es TCP 22.

Comandos para comprobar la regla y el estado de seguridad:

```bash
sudo firewall-cmd --list-services
sudo firewall-cmd --list-all
getenforce
```

**Alcance del registro:** en los archivos originales consta la habilitación de SSH en Firewalld, pero no se documenta un cambio específico de configuración de SELinux ni el endurecimiento de sshd. Por eso no se afirma que esas medidas hayan sido aplicadas.

## 4. Pruebas de Aceptación y Troubleshooting

**Prueba registrada:** conexión desde MobaXterm utilizando la IP y el usuario consultados en CentOS, con el puerto 22.

| Comprobación | Comando o acción | Resultado esperado |
|---|---|---|
| Servicio SSH | systemctl is-active sshd | active |
| IP | hostname -I | muestra la dirección de la VM |
| Usuario | whoami | muestra el usuario conectado |
| Conexión remota | sesión SSH en MobaXterm | acceso a la terminal de CentOS |

**Tres problemas frecuentes y su revisión**

1. **Connection refused:** comprobar que sshd esté iniciado con `sudo systemctl status sshd`.
2. **Connection timed out:** revisar la IP actual, la conexión de red de VirtualBox y la regla SSH de Firewalld.
3. **Permission denied:** verificar el nombre de usuario y la contraseña utilizados en MobaXterm.

Comandos de diagnóstico:

```bash
sudo systemctl status sshd
sudo journalctl -u sshd -n 50 --no-pager
sudo firewall-cmd --list-all
```

**Captura para la entrega:** mostrar la sesión de MobaXterm conectada y, si el licenciado lo solicita, una captura de Firewalld y del estado de SELinux. No mostrar contraseñas.
