## Túneles SSH

### Túnel Local

Se utiliza para que las peticiones que llegan a un **cliente** sean gestionadas por el puerto de un **servidor**.  
Permite acceder a servicios que están en el servidor o detrás de él (en su red local), desde tu propia máquina.


```bash
ssh -L <puerto_local>:<IP_remoto>:<puerto_remoto> user@<IP_remoto>
```

- La opción `-L` crea el túnel local.
- Con la opción `-g`, el puerto queda visible para otras máquinas en el lado del cliente.

---

### Túnel Reverse

Se utiliza para exponer un puerto de una máquina (normalmente en una red privada) hacia el exterior, permitiendo que otros accedan a ese recurso.  
En vez de conectarte directamente a la máquina remota, puedes usar tu propio puerto local para acceder a recursos al otro lado del túnel.


```bash
ssh -R <puerto_local>:<IP_remoto>:<puerto_remoto> user@<IP_remoto>
```

- La opción `-R` crea el tunel reverse.
- Permite que, desde el lado del servidor, se llegue a recursos del cliente o incluso más allá (esto puede ser peligroso, ya que abre acceso a una red privada).

> **NOTA:** Es necesario tener `GatewayPorts yes` en `/etc/ssh/sshd.conf` (equivalente a usar la opción `-g`).

----------------------
### Archivo /etc/ssh/sshd_config. Algunas directivas: 
+ **Port**. Puerto de escucha [22] 
+ **Protocol**. Versiones de SSH soportadas [2]
+ **PermitRootLogin**. Permite el acceso de root [no]
+ **AllowGroups**. Especifica qué grupos tienen acceso por SSH [todos los grupos]
+ **PasswordAuthentication**. [yes]
+ **PubkeyAuthentication**. [yes]
+ **HostbasedAuthentication**. [no] 
+ **PermitEmptyPasswords**. Se permite el acceso a cuentas que no posean clave. [no]
+ **StrictModes**. Especifica si sshd debe comprobar los permisos de los ficheros (600) y del directorio  ~/.ssh (700) antes de aceptar una conexión [yes]
