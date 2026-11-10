# 04-HTTP

## 1. Arquitectura y Requisitos

HTTP permite publicar una página web alojada en CentOS Stream 10 para consultarla desde un navegador de la red.

- **Servidor:** máquina virtual CentOS Stream 10.
- **Servidor web:** Apache HTTP Server (paquete httpd).
- **Cliente:** navegador de Windows o curl desde CentOS.
- **Directorio web:** /var/www/html.
- **Puerto:** TCP 80.
- **Requisitos:** IP accesible y Firewalld activo.

Identificar la IP actual:

```bash
hostname -I
ip -br -4 addr
```

## 2. Despliegue y Configuración

Instalar Apache y curl:

```bash
sudo dnf install -y httpd curl
```

Iniciar Apache y habilitar el arranque automático:

```bash
sudo systemctl enable --now httpd
systemctl is-active httpd
```

Crear una página HTML sencilla:

```bash
echo '<h1>Servidor web de Franklin</h1><p>CentOS con Apache HTTPD funcionando.</p>' | sudo tee /var/www/html/index.html
```

Revisar la sintaxis de la configuración:

```bash
sudo apachectl configtest
```

El resultado esperado es Syntax OK.

Permitir HTTP en Firewalld:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
sudo firewall-cmd --list-services
```

Probar desde el propio servidor:

```bash
curl -I http://localhost
curl http://localhost
```

Desde Windows abrir un navegador e ingresar:

```text
http://<IP_CENTOS>
```

Debe mostrarse la página creada.

## 3. Aseguramiento Seguridad SELinux/Firewall

Mantener SELinux en Enforcing:

```bash
getenforce
sestatus
```

El contenido web estándar debe tener el contexto SELinux httpd_sys_content_t. Comprobarlo y restaurar las etiquetas predeterminadas si fuera necesario:

```bash
ls -Z /var/www/html
sudo restorecon -Rv /var/www/html
```

No desactivar SELinux ni asignar permisos 777. Para una página estática, Apache solo necesita leer el contenido; no necesita permiso general de escritura sobre el directorio.

Permitir HTTP, que utiliza TCP 80:

```bash
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
```

No abrir TCP 443 como si HTTPS ya estuviera configurado. El puerto 443 se utiliza solamente después de configurar TLS y un certificado válido.

Si se utiliza un directorio distinto de /var/www, se debe asignar un contexto SELinux adecuado en vez de desactivar la protección.

## 4. Pruebas de Aceptación y Troubleshooting

**Criterios de aceptación**

| Prueba | Comando o acción | Resultado esperado |
|---|---|---|
| Servicio | systemctl is-active httpd | active |
| Configuración | apachectl configtest | Syntax OK |
| Puerto | ss -tlnp | grep :80 | httpd escucha en TCP 80 |
| Firewall | firewall-cmd --list-services | aparece http |
| Página local | curl -I http://localhost | HTTP/1.1 200 OK |
| Página remota | navegador de Windows | aparece el contenido HTML |
| SELinux | ls -Z /var/www/html | etiqueta httpd_sys_content_t |

**Errores frecuentes**

1. **Connection timed out o no abre desde Windows:** revisar IP, red de VirtualBox, conectividad y servicio http en Firewalld.
2. **403 Forbidden:** revisar permisos de lectura y etiquetas SELinux con ls -Z; aplicar restorecon -Rv /var/www/html.
3. **Apache no inicia:** ejecutar apachectl configtest, revisar journalctl -u httpd y comprobar que otro proceso no esté ocupando el puerto 80.

Comandos de diagnóstico:

```bash
sudo systemctl status httpd
sudo journalctl -u httpd -n 50 --no-pager
sudo apachectl configtest
sudo ss -tlnp | grep ':80'
sudo firewall-cmd --list-all
```

**Evidencia que se debe adjuntar:** servicio httpd activo, resultado Syntax OK, puerto 80 y regla HTTP, etiqueta SELinux del archivo index.html, respuesta curl y página abierta desde Windows.
