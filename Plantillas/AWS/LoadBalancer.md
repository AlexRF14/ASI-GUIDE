# Guía para Load Balancer (ELB)

### Comando para obtener el *vpc-id* donde crear el Load Balancer:
```aws
    aws ec2 describe-vpcs --output table
```
### Comando para obtener el *segurity-group*:
```aws
    aws ec2 describe-security-groups
```
### Comando para obtener subnet-ids disponibles en la VPC:
```aws
    aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-xxxxxxxx" --output table
```
### Comando para crear un Load Balancer tipo Application (ALB):
```aws
    aws elbv2 create-load-balancer \
    --name mi-alb \
    --subnets id-subnet-aaa id-subnet-bbb \
    --security-groups sg-xxxxxxx \
    --scheme internet-facing \
    --type application \
    --output table
```
### Comando para crear un Target Group para instancias EC2:
```aws
    aws elbv2 create-target-group \
    --name mi-target-group \
    --protocol HTTP \
    --port 80 \
    --vpc-id vpc-xxxxxxxx \
    --health-check-protocol HTTP \
    --health-check-path /health \
    --output table
```
### Comando para registrar instancias EC2 al Target Group, donde pone la Id=i- van las instancias:
```aws
    aws elbv2 register-targets \
    --target-group-arn arn:aws:elasticloadbalancing:region:account-id:targetgroup/mi-target-group/xxxxxx \
    --targets Id=i-xxxxxxxx Id=i-yyyyyyyy 
```
### Comando para crear un Listener en el Load Balancer para tráfico HTTP:
```aws
    aws elbv2 create-listener \
    --load-balancer-arn arn:aws:elasticloadbalancing:region:account-id:loadbalancer/app/mi-alb/xxxxxx \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:region:account-id:targetgroup/mi-target-group/xxxxxx \
    --output table
```
# Por si hace falta visualizar algo:

### Listar Load Balancers para obtener ARN
```aws
    aws elbv2 describe-load-balancers --output table
```
### Listar Target Groups para obtener ARN
```aws
    aws elbv2 describe-target-groups --output table
```
### Listar Listeners de un Load Balancer
```aws
    aws elbv2 describe-listeners --load-balancer-arn <load-balancer-arn> --output table
```

# Guía para eliminar recursos

### Eliminar Listener:
```aws
    aws elbv2 delete-listener --listener-arn arn:aws:elasticloadbalancing:region:account-id:listener/app/mi-alb/xxxxxx
```
### Eliminar Target Group:
```aws
    aws elbv2 delete-target-group --target-group-arn arn:aws:elasticloadbalancing:region:account-id:targetgroup/mi-target-group/xxxxxx
```
### Eliminar Load Balancer:
```aws
    aws elbv2 delete-load-balancer --load-balancer-arn arn:aws:elasticloadbalancing:region:account-id:loadbalancer/app/mi-alb/xxxxxx
```
### Esperar a que el Load Balancer sea eliminado:
```aws
    aws elbv2 wait load-balancers-deleted --load-balancer-arns arn:aws:elasticloadbalancing:region:account-id:loadbalancer/app/mi-alb/xxxxxx
```