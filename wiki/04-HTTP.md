# 04-HTTP

## 1. Arquitectura y Requisitos

La práctica configura un servidor web en CentOS Stream 10 usando Apache HTTPD. La página se consulta desde Windows mediante un navegador.

- Servidor: CentOS Stream 10.
- Servicio: Apache HTTPD (httpd).
- Cliente: navegador web de Windows.
- Herramienta de prueba local: curl.
- Directorio de publicación: /var/www/html.
- Puerto: TCP 80.

## 2. Despliegue y Configuración

Instalar Apache y curl:

```bash
sudo dnf install -y httpd curl
```

Iniciar Apache y habilitar su arranque automático:

```bash
sudo systemctl enable --now httpd
```

Permitir conexiones HTTP:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

Crear la página principal:

```bash
echo '<h1>Servidor web de Franklin</h1><p>CentOS con Apache HTTPD funcionando.</p>' | sudo tee /var/www/html/index.html
```

Comprobar la configuración y el funcionamiento:

```bash
sudo apachectl configtest
systemctl is-active httpd
curl http://localhost
```

Desde el navegador de Windows se ingresa `http://` seguido de la IP del servidor.

## 3. Aseguramiento Seguridad SELinux/Firewall

### Firewalld

Permitir HTTP en el firewall:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

El servicio HTTP utiliza TCP 80. El puerto HTTPS 443 solo debe abrirse si se configura TLS.

### SELinux

Mantener SELinux en modo Enforcing. El contenido estático publicado en `/var/www/html` debe tener el contexto `httpd_sys_content_t`, que permite a Apache leer los archivos sin concederle permisos de escritura sobre el contenido:

```bash
getenforce
ls -Zd /var/www/html
ls -Z /var/www/html
sudo restorecon -Rv /var/www/html
```

No es necesario activar permisos de escritura para Apache para servir una página HTML estática. Tampoco se debe desactivar SELinux ni usar permisos generales como `chmod 777`.

Si Apache no puede leer un archivo, revisar los eventos de SELinux:

```bash
sudo ausearch -m AVC -ts recent
```

## 4. Pruebas de Aceptación y Troubleshooting

| Comprobación | Comando o acción | Resultado esperado |
|---|---|---|
| Configuración Apache | apachectl configtest | Syntax OK |
| Estado del servicio | systemctl is-active httpd | active |
| Consulta local | curl http://localhost | muestra el HTML |
| Contexto SELinux | ls -Z /var/www/html | archivos con httpd_sys_content_t |
| Consulta desde Windows | navegador con http://IP | aparece la página |

La página creada contiene:

**Servidor web de Franklin**  
CentOS con Apache HTTPD funcionando.

### Tres problemas frecuentes y su revisión

1. **La página no abre desde Windows:** revisar IP, red de VirtualBox y regla HTTP de Firewalld.
2. **Apache no inicia:** revisar `sudo systemctl status httpd` y `sudo apachectl configtest`.
3. **Apache no puede leer el contenido:** revisar los permisos y el contexto SELinux; aplicar `sudo restorecon -Rv /var/www/html` y consultar los eventos AVC si continúa el error.

Comandos de diagnóstico:

```bash
sudo systemctl status httpd
sudo apachectl configtest
sudo journalctl -u httpd -n 50 --no-pager
sudo firewall-cmd --list-all
ls -Z /var/www/html
getenforce
sudo ausearch -m AVC -ts recent
```

**Evidencia para adjuntar:** captura de `apachectl configtest`, estado activo de httpd, página abierta desde Windows y contexto SELinux de /var/www/html.
