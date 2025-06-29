
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

###########
### FIN ###
###########


