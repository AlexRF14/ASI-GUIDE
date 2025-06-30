## Preconfiguración: Instalación de AWS CLI en la máquina (si no lo tiene)

```bash
sudo systemctl status chronyd
sudo chronyc makestep

sudo yum install -y unzip curl
sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo unzip awscliv2.zip
sudo ./aws/install
aws --version
```

## 1. Crear la conexión en la consola

```bash
aws configure
```

Introduce los siguientes datos cuando se soliciten:

```
AWS Access Key ID [None]: A**************Q
AWS Secret Access Key [None]: t*************************O
Default region name [None]: eu-west2
Default output format [None]: table
```

Para añadir el session token:

```bash
sudo vim .aws/credentials
```
Y añade la línea:
```
aws_session_token = ...
```

## 2. Crear el security group 'ordinaria_sg' para SSH (22), HTTP (80) y HTTPS (443)

- **Ver la VPC:**

    ```bash
    aws ec2 describe-vpcs
    ```

- **Crear el security group:**

    ```bash
    aws ec2 create-security-group --group-name ordinaria_sg --description "Permite acceso HTTP, HTTPS y SSH" --vpc-id vpc-0cd519894aee2486e
    ```

- **Añadir reglas al grupo:**

    ```bash
    aws ec2 authorize-security-group-ingress --group-id sg-062316e8fea8e07ad --protocol tcp --port 22 --cidr 0.0.0.0/0
    aws ec2 authorize-security-group-ingress --group-id sg-062316e8fea8e07ad --protocol tcp --port 80 --cidr 0.0.0.0/0
    aws ec2 authorize-security-group-ingress --group-id sg-062316e8fea8e07ad --protocol tcp --port 443 --cidr 0.0.0.0/0
    ```

- **Comprobar configuración:**

    ```bash
    aws ec2 describe-security-groups --group-ids sg-062316e8fea8e07ad
    ```

## 3. Crear pareja de claves sin passphrase

- **Crear las claves:**

    ```bash
    aws ec2 create-key-pair --key-name Ordinaria_Keys --query 'KeyMaterial' --output text > ~/.ssh/Ordinaria_Keys.pem
    ```

- **Dar permisos al archivo:**

    ```bash
    chmod 400 ~/.ssh/Ordinaria_Keys.pem
    ```

## 4. Crear una subnet en eu-west-2b con direccionamiento 172.31.200.0/26

- **Crear la subnet:**

    ```bash
    aws ec2 create-subnet --vpc-id vpc-0cd519894aee2486e --cidr-block 172.31.200.0/26 --availability-zone eu-west-2b
    ```

## 5. Crear una instancia Amazon Linux 2003 (t2.micro) con IP pública, en la subnet creada, con las claves y security group

- **Obtener IDs de security group y subnet:**

    ```bash
    aws ec2 describe-subnets
    aws ec2 describe-security-groups
    ```

- **Crear la instancia (ejemplo con imagen proporcionada):**

    ```bash
    aws ec2 run-instances --image-id ami-0bc8d5c547360e648 --instance-type t2.micro --key-name Ordinaria_Keys --security-group-ids sg-062316e8fea8e07ad --subnet-id subnet-0bf078453c9240e6b --associate-public-ip-address
    ```

- **Si la imagen no existe, buscar una de Amazon Linux 2023:**

    ```bash
    aws ec2 describe-images --owners amazon --filters "Name=name,Values=al2023-ami-*" --query "Images[*].[ImageId,Name,CreationDate]" --output table
    ```

## 6. Conexión SSH a la instancia

- **Obtener la IP pública:**

    ```bash
    aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId, PublicIpAddress]' --output table
    ```

- **Conectarse:**

    ```bash
    ssh -i ~/.ssh/Ordinaria_Keys.pem ec2-user@
    ```

## 7. Instalar Apache y crear página predeterminada con Nombre y Apellido

- **Instalar y habilitar Apache:**

    ```bash
    sudo dnf update -y
    sudo dnf install httpd -y
    sudo systemctl start httpd
    ```

- **Crear el index.html en `/var/www/html`:**

    ```bash
    sudo vim /var/www/html/index.html
    ```

    (Escribe tu Nombre y Apellido)

- **Reiniciar Apache:**

    ```bash
    sudo systemctl restart httpd
    ```

## 8. Borrar todos los recursos creados

- **Borrar la instancia:**

    ```bash
    aws ec2 terminate-instances --instance-ids 
    ```

- **Borrar la subnet:**

    ```bash
    aws ec2 delete-subnet --subnet-id 
    ```

- **Borrar el security group:**

    ```bash
    aws ec2 delete-security-group --group-id 
    ```

- **Borrar las claves:**

    ```bash
    aws ec2 delete-key-pair --key-name Ordinaria_Keys
    ```

- **Liberar Elastic IP (si se asignó):**

    ```bash
    # Ver allocation ID
    aws ec2 describe-addresses

    # Disassociate (si está asociada)
    aws ec2 disassociate-address --association-id 

    # Liberar la Elastic IP
    aws ec2 release-address --allocation-id 
    ```

**FINAL**
