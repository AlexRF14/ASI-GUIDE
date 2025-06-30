## RAID

Selecciona primero el **RAID virtual** (el que hace que se comporte como un único disco físico, aunque esté compuesto por varios discos físicos o particiones) y los volúmenes a integrar.

### Verificar progreso del RAID

```bash
cat /proc/mdstat
```
> **IMPORTANTE:** Debe estar al 100%.

### Aclaraciones

- Cada RAID creado con `mdadm` debe tener un único RAID virtual (`/dev/mdX`, donde X es secuencial).
- Para montarlos, se recomienda crear un punto de montaje exclusivo por cada RAID (`/mnt/raidX`).
- Si montas varios RAIDs en el mismo punto, se sobreescriben temporalmente.

### Comandos para crear RAIDs

#### RAID 0

```bash
sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdX1 /dev/sdY1
```

#### RAID 1

```bash
sudo mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdX1 /dev/sdY1
```

#### RAID 5 (Minimo 3 discos)

```bash
sudo mdadm --create --verbose /dev/md2 --level=5 --raid-devices=3 /dev/sdX1 /dev/sdY1 /dev/sdZ1
```

#### RAID 10 (Mínimo 4 discos)

```bash
sudo mdadm --create --verbose /dev/md3 --level=10 --raid-devices=4 /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
```

### Crear sistema de archivos

```bash
sudo mkfs.ext4 /dev/md0
```

### Crear punto de montaje

```bash
sudo mkdir -p /mnt/raid0
```

### Montar RAID

```bash
sudo mount /dev/md0 /mnt/raid0
```

### Persistencia en /etc/fstab

Copia los UUIDs usando `lsblk -f` y añádelos en `/etc/fstab` para hacerlos persistentes.  
Ejemplo de estructura:

```
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt/raid ext4 defaults 0 0
```

### Guardar la configuración RAID para el arranque

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

## Desmontar el RAID 0 del Ejemplo 1

- **Desmontar el punto de montaje:**

    ```bash
    sudo umount /mnt/raid0
    ```

- **Detener el dispositivo RAID:**

    ```bash
    sudo mdadm --stop /dev/md0
    ```

- **Eliminar la configuración RAID:**

    ```bash
    sudo mdadm --zero-superblock /dev/sdX1
    sudo mdadm --zero-superblock /dev/sdY1
    ```

- **Eliminar el dispositivo (`/dev/md0`):**

    ```bash
    sudo rm -f /dev/md0
    ```

- **Revisar el estado:**

    ```bash
    lsblk -f
    ```
