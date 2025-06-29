# Configuración en Rocky (/etc/httpd/)

### 1. Instalación

```bash
dnf update -y
dnf install httpd -y
```

### 2. VirtualHost

No es necesario modificar `httpd.conf`.  
Crea un archivo de VirtualHost en `/etc/httpd/conf.d/.conf` con el siguiente contenido:

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
htpasswd -c /web/privado/.htpasswd 
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

    ServerName www.prueba.es
    ServerAlias prueba.es www.prueba.com prueba.com  # Todos los alias que podía tener
    DocumentRoot "/web/prueba"

    
        DirectoryIndex default.html  # Para que encuentre nuestro index file
        Require all granted
        AllowOverride All
    

    Alias /privado /web/privado  # Como está fuera del DocumentRoot sin esto no lo identifica
    
        Require all granted
        AllowOverride AuthConfig  # Mira en el archivo de .htaccess en el directorio /web/privado
    

```

### 3. Crear el archivo .htpasswd en /web/privado

```bash
htpasswd -c /web/privado/.htpasswd 
```
> **IMPORTANTE:** No uses la opción `-c` si el archivo ya existe, ya que lo sobrescribe.

### 4. Crear el archivo .htaccess en /web/privado

```apache
AuthUserFile /web/privado/.htpasswd
AuthType Basic
AuthName "Private Directory"
Require valid-user
```

**And that's all folks**
