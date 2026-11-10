# 02-NTP

## 1. Arquitectura y Requisitos

La práctica configura un servidor local de hora en CentOS Stream 10 utilizando Chrony. Los equipos de la red pueden consultar la hora del servidor.

- Servidor: CentOS Stream 10.
- Servicio: Chrony (chronyd).
- Protocolo y puerto: UDP 123.
- Archivo de configuración: /etc/chrony.conf.
- Herramienta de comprobación: chronyc.

## 2. Despliegue y Configuración

Instalación de Chrony:

```bash
sudo dnf install -y chrony
```

Edición de la configuración:

```bash
sudo nano /etc/chrony.conf
```

En el archivo se mantuvo el pool público y se registró la red autorizada:

```text
pool 2.centos.pool.ntp.org iburst
allow 192.168.1.0/24
```

La red `192.168.1.0/24` corresponde al valor que aparece en el registro original de la práctica.

Activación del servicio:

```bash
sudo systemctl enable --now chronyd
```

Apertura del servicio NTP en Firewalld:

```bash
sudo firewall-cmd --add-service=ntp --permanent
sudo firewall-cmd --reload
```

## 3. Aseguramiento Seguridad SELinux/Firewall

La configuración registrada limita las consultas NTP a la red indicada mediante la directiva `allow`. En Firewalld se permitió el servicio NTP, que utiliza UDP 123.

Comandos para revisar la configuración y el servicio:

```bash
sudo firewall-cmd --list-services
sudo firewall-cmd --list-all
getenforce
```

**Alcance del registro:** el README original documenta la regla NTP de Firewalld y la directiva `allow` de Chrony. No registra cambios específicos de SELinux; por tanto, no se afirma que se haya modificado esa política.

## 4. Pruebas de Aceptación y Troubleshooting

Los comandos documentados para comprobar la sincronización son:

```bash
chronyc sources
chronyc tracking
```

| Comprobación | Comando | Qué permite revisar |
|---|---|---|
| Fuentes de hora | chronyc sources | fuentes configuradas y su estado |
| Seguimiento | chronyc tracking | información de sincronización |
| Servicio | systemctl status chronyd | estado de Chrony |
| Firewall | firewall-cmd --list-services | servicio NTP permitido |

**Tres problemas frecuentes y su revisión**

1. **No aparecen fuentes de hora:** revisar las líneas pool o server de /etc/chrony.conf y la conexión a Internet.
2. **El servicio no inicia:** comprobar `sudo systemctl status chronyd` y `sudo journalctl -u chronyd -n 50`.
3. **Los clientes no reciben la hora:** revisar la directiva allow, la red del cliente y la regla NTP de Firewalld.

**Capturas para la entrega:** mostrar /etc/chrony.conf, el servicio chronyd activo, la salida de chronyc sources y chronyc tracking, y la regla NTP del firewall. El repositorio contiene los comandos de comprobación, pero no una captura de sus resultados.
