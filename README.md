# Amazon-ECR
Esta rama se basa en el servicio de almacenamiento de imagenes Docker Amazon EC2 Container Registry, mediante el se gestionan las diferentes imagenes a modo de repositorio. Usa un modelo parecido a GitHub y se pueden definir permisos a los usuarios para que permitirles o denegarles ciertas operaciones a nivel de repositorio. A través del **Docker CLI** es posible poder realizar operaciones para subir o descargar imágenes Docker desde el repositorio creado en **ECS**.

##Configuración del entorno local
Para la configuración del entorno local se va utilzar la herramienta de Amazon **AWS CLI** aunque también es posible realizarla a través de la interfaz web. A través de esta herramienta se realizará todo el proceso de configuración y todas los pasos necesarios para almacenar una imagen Docker en un repositorio ECR.

Una vez instalado el cliente AWS por línea de comandos de amazon, en este caso desde el gestor de paqutes de python **pip**, lo primero que se necesita es configurar el perfil del usuario AWS en la máquina desde donde se quiere realizar las operaciones con ECR. Para ello es necesario proporcionar los datos del usuario de **Access Key Id**, **Secret Key** y la región mediante el siguiente comando:

	aws configure --profile NAME_USER


##Creando políticas IAM
Como se ha comentado anteriormente, los usurarios por defecto no tienen ningún permiso en AWS, por lo que es necesario concederlos bajo alguna política. En este caso se va a crear una política para permitir a los usuarios simplemente solicitar un token de autentificación necesario para la comunicación con ecr. Para ello se creará un fichero json con la siguiente estructura a partir de la siguiente orden:

	cat << EOM > authPolicy.json
	{
		"Version": "2012-10-17",
  		"Statement": [
    		{
      			"Effect": "Allow",
      			"Action": [
        			"ecr:GetAuthorizationToken"
      			],
      			"Resource": "*"
    		}
  		]
	}

A continuación se ejecutará  el siguiente comando:

	aws --profile NAME_USER iam create-policy --policy-name=authOnly --policy-document ./authPolicy.json
	{
  		"Policy": {
    		"PolicyName": "authOnly",
    		"CreateDate": "<date>",
    		"AttachmentCount": 0,
    		"IsAttachable": true,
    		"PolicyId": "<policy id>",
    		"DefaultVersionId": "v1",
    		"Path": "/",
    		"Arn": "arn:aws:iam::<account number>:policy/authOnly",
    		"UpdateDate": "<date>"
  		}
	}
	
Una vez creadas las políticas ya se pueden crear usuarios y asignárselas.

##Crear usuarios
A través del perfil de un usuario de AWS se pueden crear otros usuarios con el siguiente comando:

	aws --profile=NAME_USER iam create-user --user-name=usr1
	
A través de este otro comando le asignamos una política:

	aws --profile=admin iam attach-user-policy --user-name usr1 --policy-arn arn:aws:iam::<account number>:policy/authOnly
	
Con los usuarios ya creados solo queda configurar el entorno local con sus credenciales:

	aws configure --profile usr1
	
##Crear repositorios
Una imagen debería estar almacenada en un diferente repositorio de Amazon ECR, aunque es posible tener múltiples imágenes en el mismo repositorio con diferentes tags, esto en ocasiones puede llegar a dificultar su manejo.

Con la siguiente orden se cre un repositorio en Amazon:

	aws ecr create-repository --repository-name=repo1
	
##Permisos en repositorios
Cada repositorio tiene sus propios permisos en relación a cada usuario, es decir, un repositorio tiene un documento de política por cada usuario.

Es necesario crear el documento con la siguiente estructura:
	
	cat << EOM > usr1Policy.json
	{
  		"Version": "2008-10-17",
  		"Statement": [
    		{
      			"Sid": "new statement",
      			"Effect": "Allow",
      			"Principal": {
        		"AWS": "arn:aws:iam::<account id>:user/<usr1>"
      			},
      			"Action": [
        			"ecr:GetDownloadUrlForLayer",
        			"ecr:PutImage",
        			"ecr:InitiateLayerUpload",
        			"ecr:UploadLayerPart",
        			"ecr:CompleteLayerUpload",
       			"ecr:DescribeRepositories",
       		 	"ecr:GetRepositoryPolicy",
        			"ecr:ListImages",
        			"ecr:DeleteRepository",
        			"ecr:BatchDeleteImage",
        			"ecr:SetRepositoryPolicy",
        			"ecr:DeleteRepositoryPolicy",
        			"ecr:GetAuthorizationToken",
        			"ecr:BatchCheckLayerAvailability",
        			"ecr:BatchGetImage"
      			]
    		}
  		]
	}

Lo siguiente es asignarle el documento de política al repositorio:
 
	aws ecr set-repository-policy --repository-name repo1 --policy-text ./usr1Policy.json --profile usr1
	
##Trabajando con imágenes en el repositorio
Es necesario obtener las credenciales de Amazon ECR para poder trabajar con el cliene de Docker. A través de esta comando se obtienen para un perfil de usuario previamente configurado:

	aws ecr get-login --profile usr1
	
Esta orden devuelve un "**docker login -u AWS -p AQ.....**" con unas credenciales de poca duración. Una vez que se tienen las credenciales de Docker hay que ejecutar el comando que nos devuelve para autentificarse como usuario de Docker:

	docker login -u AWS -p AQ.....

Una vez registrado le decimos a docker el tag de la imagen y el repositoro al que la queremos subir:

	docker tag REPOSITORY:TAG MY_IMAGE_TAG
	
Con la siguiente orden se sube la imagen al respositorio indicado:

	docker push aws_account_id.dkr.ecr.name_region.amazonaws.com/name_repository
	
De la misma forma es posible traerse imágenes del respotirio con la siguiente orden:

	docker pull aws_account_id.dkr.ecr.name_region.amazonaws.com/name_repository