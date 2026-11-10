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

Instalar SSH, Firewalld, Apache y curl según los pasos registrados:

```bash
sudo dnf install -y openssh-server firewalld
sudo systemctl enable --now firewalld sshd
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

Consultar la IP y el usuario de CentOS:

```bash
hostname -I
whoami
```

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

Desde el navegador de Windows se ingresó `http://` seguido de la IP del servidor.

## 3. Aseguramiento Seguridad SELinux/Firewall

La práctica registra la apertura del servicio HTTP en Firewalld. Apache utiliza TCP 80 para entregar la página.

Comandos de consulta:

```bash
sudo firewall-cmd --list-services
sudo firewall-cmd --list-all
getenforce
ls -Z /var/www/html
```

**Alcance del registro:** el README original documenta la regla HTTP del firewall y la página web en /var/www/html. No registra cambios específicos de SELinux; por lo tanto, no se afirma que se haya modificado esa política.

## 4. Pruebas de Aceptación y Troubleshooting

Las comprobaciones registradas son:

| Comprobación | Comando o acción | Resultado esperado |
|---|---|---|
| Configuración Apache | apachectl configtest | Syntax OK |
| Estado del servicio | systemctl is-active httpd | active |
| Consulta local | curl http://localhost | muestra el HTML |
| Consulta desde Windows | navegador con http://IP | aparece la página |

El contenido de la página creada fue:

**Servidor web de Franklin**  
CentOS con Apache HTTPD funcionando.

**Tres problemas frecuentes y su revisión**

1. **La página no abre desde Windows:** revisar la IP actual, la red de VirtualBox y la regla HTTP del firewall.
2. **Apache no inicia:** revisar `sudo systemctl status httpd` y `sudo apachectl configtest`.
3. **Aparece un error al mostrar el contenido:** comprobar que el archivo exista en /var/www/html/index.html y revisar los registros con `sudo journalctl -u httpd -n 50`.

**Evidencia registrada:** el README del repositorio servidor-web contiene los pasos para crear la página y probarla con curl y desde el navegador. Para la entrega, adjuntar las capturas originales de las pruebas.
