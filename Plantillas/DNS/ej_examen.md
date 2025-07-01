
## EJERCICIO EN UBUNTU SERVER

**Dominio y rangos:**

- **examen.net** → rango `60.40.0.0/16`
- **dns1.examen.net** → `60.40.5.10`
- **ns1.google.com**
- **madrid.examen.net** → `60.40.50.1`
- **sevilla.examen.net** → `60.40.50.2`
- **mail1.examen.com** (prioridad 10)
- **mail2.examen.com** (prioridad 20)
- **www.examen.net** → `60.40.200.1` y `60.40.200.2`

-----------------

### Instalación en Ubuntu Server

```bash
sudo apt update && sudo apt upgrade
sudo apt install bind9
```

**En Rocky:**

```bash
sudo dnf update
sudo dnf install bind bind-utils
```

### Configuración de zonas

**En `/etc/bind/named.conf.local` (o `/etc/named.conf` en Rocky):**

```conf
zone "examen.net" {
    type master;
    file "/etc/bind/db.examen.net";
    allow-transfer { 192.168.52.129; };
    notify yes;
};

zone "5.40.60.in-addr.arpa" {
    type master;
    file "/etc/bind/db.60.40.5";
};

zone "50.40.60.in-addr.arpa" {
    type master;
    file "/etc/bind/db.60.40.50";
};

zone "200.40.60.in-addr.arpa" {
    type master;
    file "/etc/bind/db.60.40.200";
};
```

### Archivo de zona directa

**Crear `/etc/bind/db.examen.net` (o `/var/named/examen.net.zone` en Rocky):**

```zone
$TTL 86400
@       IN      SOA     dns1.examen.net. admin.examen.net. (
                            2         ; Serial
                       604800         ; Refresh
                        86400         ; Retry
                      2419200         ; Expire
                       604800 )       ; Negative Cache TTL

; Nameservers
        IN      NS      dns1.examen.net.
        IN      NS      ns1.google.com.

; Hosts
dns1    IN      A       60.40.5.10
madrid  IN      A       60.40.50.1
sevilla IN      A       60.40.50.2
www     IN      A       60.40.200.1
        IN      A       60.40.200.2

; Mail servers
@       IN      MX 10   mail1.micorreo.com.
@       IN      MX 20   mail2.micorreo.com.
```

### Archivos de zona inversa

**Crear `/etc/bind/db.60.40.5` (o `/var/named/60.40.5.in-addr.arpa` en Rocky):**

```zone
$TTL 86400
@ IN SOA dns1.examen.net. admin.examen.net. (
    1 ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 ) ; Negative Cache TTL

@ IN NS dns1.examen.net.

10 IN PTR dns1.examen.net.
```

**Crear `/etc/bind/db.60.40.50` (o `/var/named/60.40.50.in-addr.arpa` en Rocky):**

```zone
$TTL 86400
@ IN SOA dns1.examen.net. admin.examen.net. (
    1 ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 )

@ IN NS dns1.examen.net.

1 IN PTR madrid.examen.net.
2 IN PTR sevilla.examen.net.
```

**Crear `/etc/bind/db.60.40.200` (o `/var/named/60.40.200.in-addr.arpa` en Rocky):**

```zone
$TTL 86400
@ IN SOA dns1.examen.net. admin.examen.net. (
    1 ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 )

@ IN NS dns1.examen.net.

1 IN PTR www.examen.net.
2 IN PTR www.examen.net.
```

### Comprobación de la configuración

```bash
sudo named-checkzone 50.40.60.in-addr.arpa /etc/bind/db.60.40.50
# En Rocky:
sudo named-checkzone 50.40.60.in-addr.arpa /var/named/60.40.50.in-addr.arpa
sudo named-checkconf
sudo systemctl reload bind9
```

### Problemas de permisos

```bash
sudo chown bind:bind /etc/bind/db.60.40.*
sudo chmod 644 /etc/bind/db.60.40.*
```

### Pruebas con dig

```bash
dig @localhost www.examen.com
dig @localhost -x 60.40.5.10
# ...
```

---
---

## EXTRA: Configuracion de zona slave

Por ejemplo, voy a configurar una zona slave en una maquina con la IP 192.168.52.129 (la master es la 192.168.52.128)

Primero en la master ponemos el 'allow-trasfer' y el 'allow-notify' para cada zona en el archivo de ***/etc/bind/named.conf.local*** (Ubuntu) || ***/etc/named.conf*** (Rocky):
>**RECORDAR**: Instalar bind en ambas maquinas

```zone
zone "examen.net" {
        type master;
        file "/etc/bind/db.examen.net";
        allow-transfer { 192.168.52.129; };
        also-notify { 192.168.52.129; };
};

zone "5.40.60.in-addr.arpa" {
    type master;
    file "/etc/bind/db.60.40.5";
    allow-transfer { 192.168.52.129; };
    also-notify { 192.168.52.129; };
};

zone "50.40.60.in-addr.arpa" {
    type master;
    file "/etc/bind/db.60.40.50";
    allow-transfer { 192.168.52.129; };
    also-notify { 192.168.52.129; };
};

zone "200.40.60.in-addr.arpa" {
    type master;
    file "/etc/bind/db.60.40.200";
    allow-transfer { 192.168.52.129; };
    also-notify { 192.168.52.129; };
};
```
---
Hacemos un reload del servicio

```bash
sudo systemctl reload bind9 # Ubuntu
sudo systemctl reload named # Rocky
```
>**NOTA**: 'sudo systemctl restart <servicio>' hace basicamente lo mismo

---
Configuramos la slave con las zonas (***/etc/bind/named.conf.local*** (Ubuntu)  ||  ***/etc/named.conf*** (Rocky))

```zone
zone "examen.net" {
        type slave;
        file "slaves/db.examen.net";
        masters { 192.168.52.128; };
};

zone "5.40.60.in-addr.arpa" {
    type slave;
    file "slaves/db.60.40.5";
    masters { 192.168.52.128; };
};

zone "50.40.60.in-addr.arpa" {
    type slave;
    file "slaves/db.60.40.50";
    masters { 192.168.52.128; };
};

zone "200.40.60.in-addr.arpa" {
    type slave;
    file "slaves/db.60.40.200";
    masters { 192.168.52.128; };
};
```
>**NOTA 1**: El file path es relativo a /var/named por defecto enn Rocky(se puede poner completo si se quiere)

>**NOTA 2**: el file path si la slave es la Ubuntu y la master Rocky seria del rollo '/var/cache/bind/examen.net.zone' (Fijarse bien en como se llama los archivos de zonas el la master originalmente)

---

Asegurarse que la slave tiene las siguientes opciones puestas en ***/etc/bind/named.conf.options*** (Ubuntu)  ||  ***/etc/named.conf*** (Rocky)

```zone
options {
    listen-on port 53 { 127.0.0.1; 192.168.52.129; };
    listen-on-v6 { none; };
    allow-query { any; }; # o restringirlo a la subnet 'allow-query { 127.0.0.1; 192.168.52.0/24; };'
    ...
};
```
>**NOTA**: Para mas info mirar el archivo '/Plantillas/DNS/Permitir acceso a otras IPs.txt' del GitHub

---
Crear la carpeta '/var/named/slaves' y dar permisos (Rocky):

```bash
sudo mkdir -p /var/named/slaves
sudo chown named:named /var/named/slaves
sudo restorecon -Rv /var/named/slaves
```
>**NOTA**: En Ubuntu, que usa '/var/cache/bind' para guardar los slaves, ya existe, solo hay que darle permisos

Permisos para '/var/cache/bind' (Ubuntu):

```bash
sudo chown bind:bind /var/cache/bind
sudo chmod 755 /var/cache/bind
```

---


Y reiniciar el servicio DNS:
```bash
sudo systemctl restart bind9 # Ubuntu
sudo systemctl restart named # Rocky
```
---

## FIN
