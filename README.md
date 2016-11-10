# Amazon-ECR
Esta rama se basa en el servicio de almacenamiento de imagenes Docker Amazon EC2 Container Registry, mediante el se gestionan las diferentes imagenes a modo de repositorio. Usa un modelo parecido a GitHub y se pueden definir permisos a los usuarios para que permitirles o denegarles ciertas operaciones a nivel de repositorio.

##Configuración del entorno local
Para la configuración del entorno local se va utilzar la herramienta de Amazon **AWS CLI** aunque también es posible realizarla a través de la interfaz web. A través de esta herramienta se creará una serie de usuarios e imágenes Docker, a los que se les dará un conjunto de permisos sobre las diferentes imágenes.

Una vez instalado el cliente AWS por línea de comandos de amazon, en este caso desde el gestor de paqutes de python **pip**, hay que configurar el sistem local para 
que el AWS Cli hable a la cuenta de Amazon AWS. Para ello es necesario proporcionar el **Access Key Id, Secret Key y la regisón mediante el siguiente comando:

	aws configure --profile admin
	
A través del siguiente comando podemos obtener una lista de claves válidas:

	aws --profile admin iam list-access-keys
	
Este comando puede lanzar un error al decirnos que el usuario no esta autorizado para ejecutar la operacion *list-access-keys** de iam. Para ello es necesario añadir una política de permisos sobre ese servicio en **Amazon IAM** que permite ejecutar esa operación.

##Creando políticas IAM
Como se ha comentado anteriormente, los usurarios por defecto no tienen ningún permiso en AWS, por lo que es necesario concederlos bajo alguna política. En este caso se va a crear una política para permitirá a los usuarios simplemente solicitar un token de autentificación. Para ello se creará un fichero con la siguiente estructura a partir de la siguiente orden:

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

	aws --profile admin iam create-policy --policy-name=authOnly --policy-document file://authPolicy.json
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
Se crearán tres usuarios a los que se les asignarán permisos de para autentificarse. Con el siguiente comando se crean usuarios:

	aws --profile=admin iam create-user --user-name=usr1
	
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
 
	aws --profile admin ecr set-repository-policy --repository-name repo1 --policy-text file://usr1Policy.json 
	
##Subiendo imágenes al repositorio
Para poder subir imágenes a un repositorio es necesario habilitar el demonio local docker para autentificar con el registro y poder trabajar con el. De esta forma se autentifica el cliente Docker en el Amazon ECR:

	aws --profile usr1 ecr get-login
	
Esta orden devuelve un "docker login..." con unas credenciales de poca duración. Una vez que se tienen las credenciales de Docker ya es posible subir imagenes al repositorio previamente creado:

	docker tag <local image id> <aws account id>.dkr.ecr.<aws region>.amazonaws.com/repo1:latest
	
BVr05zDj7v/JhNMtXqklY+7LslQzY6MezG+k6cop

yCYDDJSZMwAdNUOUFrW4cnqNrS38OzjPXhoft5qf