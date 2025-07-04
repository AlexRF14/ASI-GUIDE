## SIMULACIÓN DEL ENVIRONMENT DEL EXAMEN (Rocky)

Para simular el environment del examen se ha realizado lo siguiente:

### Estructura del disco

```
sdb - 20GB
 |---sdb1 - 5GB
 |     |---hdds-hdd - 4GB - /var/datos    # hdds es el VG , hdd es el LV
 |---sdb2 - 5GB
 |---sdb3 - 5GB
 |---sdb4 - 5GB

```

En una VM se añadió un disco SATA extra de 20GB para este propósito.

### Creación de particiones en sdb

```bash
fdisk /dev/sdb
```

1. Pulsa `n` para nueva partición.
2. Elige `p` para primaria.
3. Selecciona el número de partición (ejemplo: 1).
4. Pulsa Enter para aceptar el sector inicial por defecto.
5. Para el último sector, escribe `+5G` para una partición de 5GB.
6. Repite el proceso para llenar el disco (la última solo Enter).
7. Pulsa `w` para guardar y salir.

### Crear el LV hdds-hdd en sdb1

**Inicializar sdb1 como volumen físico (PV):**

```bash
pvcreate /dev/sdb1
```

**Crear el Volume Group (VG):**

```bash
vgcreate hdds /dev/sdb1
```

**Crear el Logical Volume (LV):**

```bash
lvcreate -L 4G -n hdd hdds
```

**Formatear el LV como ext4:**

```bash
mkfs.ext4 /dev/hdds/hdd
```

**Montar el LV en `/var/datos`:**

```bash
mkdir -p /var/datos
mount /dev/hdds/hdd /var/datos
```

**Para montaje automático al reiniciar, añadir en `/etc/fstab`:**

```fstab
/dev/hdds/hdd /var/datos ext4 defaults 0 2
```

## EXPANDIR EL LV DE 4GB A 8GB

1. Inicializar otra partición (ejemplo: sdb2) como volumen físico:

    ```bash
    pvcreate /dev/sdb2
    ```

2. Extender el VG agregando sdb2:

    ```bash
    vgextend hdds /dev/sdb2
    ```

3. Extender el LV a 8GB:

    ```bash
    lvextend -L 8G /dev/hdds/hdd
    ```

4. Redimensionar el sistema de archivos (ext4):

    ```bash
    resize2fs /dev/hdds/hdd
    ```

    *Para XFS sería:*

    ```bash
    xfs_growfs /var/datos
    ```

### Estructura final con `lsblk`:

```text
sdb            8:16   0   20G  0 disk
├─sdb1         8:17   0    5G  0 part
│ └─hdds-hdd 253:2    0    8G  0 lvm  /var/datos
├─sdb2         8:18   0    5G  0 part
│ └─hdds-hdd 253:2    0    8G  0 lvm  /var/datos
├─sdb3         8:19   0    5G  0 part
└─sdb4         8:20   0    5G  0 part
```

> **Nota:** Aunque parece que hay dos LV diferentes de 8GB, en realidad solo hay uno. Verifícalo con:

```bash
lvdisplay /dev/hdds/hdd
lvs
```

## CONFIGURACIÓN DE SERVIDOR NFS PARA /var/datos

### 1. Instalación de NFS

En ambos (cliente y servidor), según la distro:

```bash
sudo dnf install nfs-utils      # Rocky
sudo apt install nfs-common     # Ubuntu Server
```

### 2. Permisos y exportación

**Dar permisos abiertos a `/var/datos`:**

```bash
sudo chmod 777 /var/datos
```

**Editar `/etc/exports`:**

```bash
sudo vim /etc/exports
```

Añadir:

```exports
/var/datos 192.168.52.0/24(rw,sync,no_subtree_check,no_root_squash,all_squash)
```

**Exportar los recursos:**

```bash
sudo exportfs -a
```

**Reiniciar y habilitar el servicio NFS:**

```bash
sudo systemctl restart nfs-server
sudo systemctl enable nfs-server
```

### 3. Cliente NFS

**Crear carpeta de montaje:**

```bash
sudo mkdir /mnt/remoto
```

**Montar el recurso compartido:**

```bash
sudo mount -t nfs :/var/datos /mnt/remoto
```

**Si da error de permisos, probar:**

```bash
sudo chown nobody /var/datos
sudo chmod 777 /var/datos
```

---
---
## Extra: Configuracion de SMB con usuario

**1. Instalacion**

Ubuntu:
```bash
sudo apt update
sudo apt install samba
```

Rocky:
```bash
sudo dnf update -y
sudo dnf install samba samba-common samba-client -y
```
---
**2. Haz un backup del archivo de config por si la lias**

Ubuntu/Rocky:
```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
```
---
**3. Edita /etc/samba/smb.conf y mete esto**

Ubuntu:
```text
[datos]
   comment = Shared Datos Folder
   path = /var/datos
   browsable = yes
   writable = yes
   guest ok = no
   valid users = @sambashare
   create mask = 0755
```

Rocky:
```text
[global]
   workgroup = WORKGROUP
   server string = Samba Server %v
   security = user
   map to guest = bad user

[datos]
   path = /var/datos
   browsable = yes
   writable = yes
   guest ok = no
   read only = no
   create mask = 0755
```
>**NOTA**: El [global] ya existe, editalo para que tenga eso tambien
---
**4. Ajusta los permisos del directorio**

Ubuntu:
```bash
sudo mkdir -p /var/datos
sudo chown -R root:sambashare /var/datos
sudo chmod -R 2770 /var/datos
```
Rocky:
```bash
sudo mkdir -p /var/datos
sudo chown -R nobody:nobody /var/datos
sudo chmod -R 755 /var/datos
sudo chcon -t samba_share_t /var/datos
```
>**NOTA**: Ajusta permisos y owners como te haga falta
---
**5. Crea un Samba user**

Ubuntu:
```bash
sudo adduser <usuario>
sudo usermod -aG sambashare <usuario>
sudo smbpasswd -a <usuario>
```
>**NOTA**: Puedes no crear un nuevo user y meterle al grupo y la smbpasswd a un user que ya exista

Rocky:
```bash
sudo useradd <usuario>
sudo smbpasswd -a <usuario>
```
>**NOTA**: Puedes no crear un nuevo user y meterle la smbpasswd a un user que ya exista
---
**6. Reinicia/start/enable smb**

Ubuntu:
```bash
sudo systemctl start smbd nmbd # por si no se ha iniciado
sudo systemctl restart smbd nmbd
sudo systemctl enable smbd nmbd
```

Rocky:
```bash
sudo systemctl start --now smb nmb
sudo systemctl restart --now smb nmb
sudo systemctl enable --now smb nmb
```
---
**7. Abrir el firewall (dudo que haga falta pero porsiaca)**

Ubuntu:
```bash
sudo ufw allow samba
```

Rocky:
```bash
sudo firewall-cmd --permanent --zone=public --add-service=samba
sudo firewall-cmd --reload
```

**¡Y finiquitado!**
