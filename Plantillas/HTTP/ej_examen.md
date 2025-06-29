## Para la configuración en Rocky (/etc/httpd/)

**1. Instalación**

```bash
dnf update -y
dnf install httpd -y
```

**2. No hace falta tocar el httpd.conf**  
Crear un archivo de VirtualHost en `/etc/httpd/conf.d/.conf`  
Para que cumpla lo que nos pidieron en el examen (de lo que recuerdo al menos):

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

**3. Crear el archivo .htpasswd en `/web/privado`**

```bash
htpasswd -c /web/privado/.htpasswd 
```
> **IMPORTANTE:** No poner la opción `-c` si el archivo ya existe porque lo reescribe.

**4. Crear el archivo .htaccess en `/web/privado` con lo siguiente:**

```apache
AuthUserFile /web/privado/.htpasswd
AuthType Basic
AuthName "Private Directory"
Require valid-user
```

And that's all folks
