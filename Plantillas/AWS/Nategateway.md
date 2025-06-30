# Guía para Nat-gateway

### Comando para poder guardar *allocate-address*, reservando una IP pública:
```aws
    aws ec2 allocate-address --domain vpc --output table
```
### Comando para poder guardar *subnet-id*:
```aws
    aws ec2 describe-subnets
```
### Comando para crear *nat-gateway* para una subred en específico:
```aws
    aws ec2 create-nat-gateway --subnet-id subnet-03e867a8351fb07a8 --allocation-id eipalloc-022ee91dac0d9ce89 --output table
```
### Comando para poder guardar *vpc-id*:
```aws
    aws ec2 describe-vpcs --output table
```
### Comando para crear la *tabla de rutas* asociada al *nat-gateway*:
```aws
    aws ec2 create-route-table --vpc-id vpc-0c7d49bac9cddceb9 --output table
```
### Añadir la ruta 0.0.0.0/0 para salida a Internet vía NAT gateway:
```aws
    aws ec2 create-route \
    --route-table-id rtb-0796ad9197b11f5c2 \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id nat-0c13e4d1e74041f09 \
    --output table
```
### Comando para asociar nuestra tabla a la subred privada:
```aws
    aws ec2 associate-route-table --route-table-id rtb-0796ad9197b11f5c2 --subnet-id subnet-03e867a8351fb07a8 --output table
```

# Guía para eliminar recursos

### Eliminar la ruta hacia el NAT Gateway:
```aws
    aws ec2 delete-route --route-table-id rtb-0796ad9197b11f5c2 --destination-cidr-block 0.0.0.0/0
```
### Desasociar la tabla de rutas:
```aws
    aws ec2 disassociate-route-table --association-id rtbassoc-05eca2c1840d545fe
```
### Eliminar la tabla de rutas:
```aws
    aws ec2 delete-route-table --route-table-id rtb-0796ad9197b11f5c2
```
### Eliminar el NAT Gateway:
```aws
    aws ec2 delete-nat-gateway --nat-gateway-id nat-0c13e4d1e74041f09
```
### Eliminar la IP elástica:
```aws
    aws ec2 release-address \
  --allocation-id eipalloc-022ee91dac0d9ce89 \
  --region us-east-1 \
  --output table
```
### Eliminar la subred privada en caso de haberla creado:
```aws
    aws ec2 delete-subnet --subnet-id subnet-03e867a8351fb07a8
```