## Intercambio de archivos
### SCP
Permite copiar archivos entre cliente y servidor, en cualquier sentido. Su sintaxis es la misma que el comando
cp (archivo origen, directorio destino) pero precediendo el extremo remoto de usuario@servidor:

```bash
scp /tmp/prueba.txt pepe@192.168.1.100:/var/users
```
---
### SFTP
Interface en modo ftp a través de una conexión segura. La llamada a sftp se realiza indicando la cuenta de usuario y el
servidor remoto.

```bash
sftp pepe@192.168.1.100
```
