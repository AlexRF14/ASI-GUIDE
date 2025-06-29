# Configuración en Rocky (/etc/httpd/)

### 1. Instalación

```bash
sudo dnf update -y
sudo dnf install httpd -y
```

### 2. VirtualHost

No es necesario modificar `httpd.conf`.  
Crea un archivo de VirtualHost en `/etc/httpd/conf.d/<nombre>.conf` con el siguiente contenido:

```apache

<VirtualHost *:80>
        ServerName www.prueba.es
        ServerAlias prueba.es www.prueba.com prueba.com # Todos los alias que podia tener
        DocumentRoot "/web/prueba"

        <Directory "/web/prueba">
                DirectoryIndex default.html # Para que encuentre nuestro index file
                Require all granted
                AllowOverride All
        </Directory>

        Alias /privado /web/privado # Como esta fuera del DocumentRoot sin esto no lo identifica

        <Directory "/web/privado">
                Require all granted
                AllowOverride AuthConfig # Mira en el archivo de .htaccess en el directorio /web/privado
        </Directory>
</VirtualHost>
    

```

### 3. Crear el archivo .htpasswd en /web/privado

```bash
sudo htpasswd -c /web/privado/.htpasswd 
```
> **IMPORTANTE:** No uses la opción `-c` si el archivo ya existe, ya que lo sobrescribe.

### 4. Crear el archivo .htaccess en /web/privado

```apache
AuthUserFile /web/privado/.htpasswd
AuthType Basic
AuthName "Private Directory"
Require valid-user
```

# Versión Ubuntu (/etc/apache2/)

### 1. Instalación

```bash
sudo apt update && sudo apt upgrade
sudo apt install apache2 -y
```

### 2. VirtualHost

No es necesario modificar `apache2.conf`.  
Edita el archivo de VirtualHost en `/etc/apache2/sites-available/000-default.conf` con el siguiente contenido:

```apache

<VirtualHost *:80>
    ServerName www.prueba.es
    ServerAlias prueba.es www.prueba.com prueba.com  # Todos los alias que podía tener
    DocumentRoot "/web/prueba"

    <Directory "/web/prueba">
        DirectoryIndex default.html  # Para que encuentre nuestro index file
        Require all granted
        AllowOverride All
     </Directory>

    Alias /privado /web/privado  # Como está fuera del DocumentRoot sin esto no lo identifica

     <Directory "/web/privado">
        Require all granted
        AllowOverride AuthConfig  # Mira en el archivo de .htaccess en el directorio /web/privado
    </Directory>
</VirtualHost>

```

### 3. Crear el archivo .htpasswd en /web/privado

```bash
sudo htpasswd -c /web/privado/.htpasswd 
```
> **IMPORTANTE:** No uses la opción `-c` si el archivo ya existe, ya que lo sobrescribe.

### 4. Crear el archivo .htaccess en /web/privado

```apache
AuthUserFile /web/privado/.htpasswd
AuthType Basic
AuthName "Private Directory"
Require valid-user
```

----
----
----

## Detalle extra, si se quiere meter archivo de log, se puede hacer asi:

Normalmente los archivos de log de apache estan en (ubuntu-> /var/log/apache2/ | rocky-> /var/log/httpd), pero si queremos podemos crear nuevos archivo en nuestro directorio de la webpage.

```bash
sudo mkdir /we/prueba/log
sudo touch /web/prueba/log/access.log
sudo touch /web/prueba/log/error.log
```

Meter en el archivo de VirtualHost (ubuntu-> /etc/apache2/sites-availible/000-default.conf | rocky-> /etc/httpd/conf.d/<nombre>.conf):

```apache
<VirtualHost *:80>
    ServerName www.prueba.es
    ServerAlias prueba.es www.prueba.com prueba.com  # Todos los alias que podía tener
    DocumentRoot "/web/prueba"
    CustomLog /web/prueba/log/access.log combined
    ErrorLog /web/prueba/log/error.log
    LogLevel warn
    ...

```
Y ya deberian de salir los logs en el archivo que hemos creado en vez de donde suelen salir.

El LogFormat, ubicado en el archivo de /etc/apache2/apache2.conf (ubuntu), o en /etc/httpd/conf/httpd.conf (rocky), tiene varios parámetros para ajustar lo que describen los logs. Las opciones son las siguientes: 

    %h: Nombre del host remoto o dirección IP. 

    %l: Nombre de log remoto (proporcionado por identd, si está disponible). 

    %u: Usuario remoto si la solicitud fue autenticada. 

    %t: Hora en que se recibió la solicitud. 

    %r: Primera línea de la solicitud (método, recurso y protocolo). 

    %>s: Código de estado final. 

    %b: Tamaño de la respuesta en bytes, excluyendo los encabezados HTTP. 

    %O: Bytes enviados, incluidos los encabezados. 

    %{Referer}i: Encabezado Referer (referencia de origen). 

    %{User-Agent}i: Encabezado User-Agent (información del cliente/navegador). 

    %v: Nombre canónico del servidor que atiende la solicitud. 

    %p: Puerto canónico del servidor que atiende la solicitud. 

---

## **And that's all folks!!!**
