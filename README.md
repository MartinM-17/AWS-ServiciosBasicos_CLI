### Preparativos

1. **Instalar AWS CLI**:
   Si no tienes instalada la AWS CLI, puedes instalarla en tu máquina local siguiendo la [documentación oficial](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

2. **Configurar AWS CLI**:
   Después de instalar la CLI, configura tus credenciales ejecutando:

   aws configure
   Ingresa las claves de acceso (`Access Key`, `Secret Key`), la región (`us-east-1`) y el formato de salida (`json` por defecto).

---

### Implementación de Servicios Básicos en AWS usando AWS CLI

#### 1. Crear un Usuario con Permisos de Administrador
   Para crear un usuario con permisos de administrador (en lugar de usar el usuario root):

   aws iam create-user --user-name AdminUser
   aws iam attach-user-policy --user-name AdminUser --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

   Esto crea un usuario llamado `AdminUser` y le otorga la política de `AdministratorAccess`.

#### 2. Configurar Credenciales del Usuario
   Configura las credenciales de `AdminUser` en tu entorno local usando:

   aws configure

#### 3. Definir Etiquetas (Tags)
   Define un conjunto de etiquetas que usarás para todos los recursos. Estas etiquetas ayudan a organizar y gestionar tus recursos en AWS. Los valores de etiquetas serán los siguientes:

   TAGS="Key=Owner,Value=Martin Key=Email,Value=martin@alumno.com Key=Team,Value=Grupo-1 Key=Proyectogrupo-1,Value=Actividad-AWS"

#### 4. Crear una Instancia EC2
   Crea una instancia EC2 dentro del Free Tier usando un script que instala Apache automáticamente al iniciarse:

   aws ec2 run-instances \
       --image-id ami-0c55b159cbfafe1f0 \ # AMI de Amazon Linux 2 (us-east-1)
       --instance-type t2.micro \
       --key-name MiClave \
       --security-group-ids sg-xxxxxxxx \
       --subnet-id subnet-xxxxxxxx \
       --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=DesafioEC2},'$TAGS']' \
       --user-data '#!/bin/bash
                    yum update -y
                    yum install -y httpd
                    systemctl start httpd
                    systemctl enable httpd' \
       --region us-east-1

   Esto crea una instancia EC2 usando la imagen Amazon Linux 2 y un tipo de instancia `t2.micro`, que entra en el Free Tier de AWS. El script de `User Data` instala y configura el servicio Apache automáticamente.

#### 5. Crear un Security Group
   Un **Security Group** es esencial para controlar el tráfico de red hacia la instancia. Crea y configura un Security Group:

   aws ec2 create-security-group --group-name MiGrupoSeguridad --description "Grupo de seguridad para EC2"
   
   # Permitir tráfico HTTP (80), HTTPS (443) y SSH (22)
   aws ec2 authorize-security-group-ingress --group-name MiGrupoSeguridad --protocol tcp --port 22 --cidr 0.0.0.0/0
   aws ec2 authorize-security-group-ingress --group-name MiGrupoSeguridad --protocol tcp --port 80 --cidr 0.0.0.0/0
   aws ec2 authorize-security-group-ingress --group-name MiGrupoSeguridad --protocol tcp --port 443 --cidr 0.0.0.0/0

#### 6. Conectar Remotamente a la Instancia EC2
   Conéctate a la instancia EC2 usando tu clave SSH o mediante AWS Systems Manager (SSM). Si usas SSH:

   ssh -i MiClave.pem ec2-user@<IP-de-la-instancia>

---

### S3: Simple Storage Service

#### 1. Crear un Bucket S3
   Crea un bucket en S3 con un nombre único:

   aws s3 mb s3://mi-bucket-desafio

#### 2. Subir un Archivo de Prueba
   Sube un archivo (por ejemplo, el PDF del desafío) al bucket:

   aws s3 cp ./archivo.pdf s3://mi-bucket-desafio/

---

### EBS: Elastic Block Store

#### 1. Crear un Volumen EBS
   Crea un volumen EBS de 2 GB en la misma zona de disponibilidad que tu instancia EC2:

   aws ec2 create-volume --size 2 --availability-zone us-east-1a --volume-type gp2

#### 2. Adjuntar el Volumen EBS a la Instancia
   Después de crear el volumen, adjúntalo a tu instancia EC2:

   aws ec2 attach-volume --volume-id vol-xxxxxxxx --instance-id i-xxxxxxxx --device /dev/sdf

#### 3. Formatear y Montar el Volumen
   Accede a tu instancia EC2 y formatea el volumen:

   ssh -i MiClave.pem ec2-user@<IP-de-la-instancia>
   sudo mkfs -t ext4 /dev/xvdf
   sudo mkdir /desafios
   sudo mount /dev/xvdf /desafios

   Agrega el volumen a `/etc/fstab` para que se monte automáticamente en cada reinicio.

#### 4. Descargar y Mover Archivos desde S3
   Descarga el archivo PDF desde el bucket S3 y muévelo al directorio `/desafios`:

   aws s3 cp s3://mi-bucket-desafio/archivo.pdf /desafios/

---

### Limpieza de Recursos 
Como esto es parte de una practica para evitar costos adicionale vamos a borrar lo que creamos
   
   aws ec2 terminate-instances --instance-ids i-xxxxxxxx
   aws ec2 delete-volume --volume-id vol-xxxxxxxx
   aws s3 rb s3://mi-bucket-desafio --force
   aws ec2 delete-security-group --group-id sg-xxxxxxxx
