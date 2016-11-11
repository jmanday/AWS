# AWS-ECS
Esta rama está basada en el servicio para defirnir tareas de **Amazon ECS**, a través de las cuales pueden lanzar contenedores en base a imágenes Docker.


## Registro de definición de tareas
Una vez que la imagen de Docker ha sido creada y subida al repositorio de Amazon ECR, lo siguiente es usar esa imagen en las definiciones de tareas de **Amazon ECS** para ejecutarlas.

	{
    	"family": "console-sample-app",
    	"volumes": [
        	{
            	"name": "my-vol",
            	"host": {}
        	}
    	],
    	"containerDefinitions": [
        	{
            	"environment": [],
            	"name": "simple-app",
            	"image": "amazon/amazon-ecs-sample",
            	"cpu": 10,
            	"memory": 500,
            	"portMappings": [
                	{
                    	"containerPort": 80,
                    	"hostPort": 80
                	}
            	],
            	"mountPoints": [
                	{
                    	"sourceVolume": "my-vol",
                    	"containerPath": "/var/www/my-vol"
                	}
            	],
            	"entryPoint": [
                	"/usr/sbin/apache2",
                	"-D",
                	"FOREGROUND"
            	],
            	"essential": true
        	},
        	{
            	"name": "busybox",
            	"image": "busybox",
            	"cpu": 10,
            	"memory": 500,
            	"volumesFrom": [
            	{
              	"sourceContainer": "simple-app"
            	}
            	],
            	"entryPoint": [
                	"sh",
                	"-c"
            	],
            	"command": [
                	"/bin/sh -c \"while true; do /bin/date > /var/www/my-vol/date; sleep 1; done\""
            	],
            	"essential": false
        	}
    	]
	}
	
La tarea definida en el anterior fichero JSON especifica la creación de dos contenedores en base a una imagen que por defecto es descargada del repositorio **Amazon Docker Hub**, aunque es posible modificarlo para que tome una imagen de un repositorio propio de **ECR**.

Para registar una definición de tarea mediante el fichero JSON se utiliza el siguiente comando:

	aws ecs register-task-definition --cli-input-json ./NAME_FILE.json
	
Esta definición de tarea es registrada en la familia que se indicó en el campo *family* del fichero JSON. Es importante conocer este parámetro porque cuando se ejecuta una definición de tareas, se hacen a través de una *family*, es decir, lo que se ejecuta es una familia de tareas:

	aws ecs run-task --task-definition NAME_FAMILY