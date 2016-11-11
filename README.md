# AWS-EC2
Esta rama esta orientada al uso del servicio de Amazon EC2, mediante el cual hacemos uso de instancias remotas con las que poder trabajar.


##Conectar a instancia Linux AWS usando SSH

- **Preview**

En esta parte se especificaran los pasos para conectarse a una instancia remota Linux EC2 de AWS desde una máquina local.

Lo primero que debemos hacer es crearnos nuestra instancia de AWS EC2 seleccionando el tipo de AMI (Amazon Machine Image) que vamos a querer. En este caso será una Amazon Linux AMI, que soporta EBS e incluye AWS Command Line Tools, Python, Ruby, Perl y Java. Además los repositorios de esta imagen incluyen los paquetes de Docker, PHP,  MySQL, PostgreSQL entre otros. 

Lo siguiente será  elegir el tipo de instancia en cuanto a prestaciones, CPU, memoria, almacenamiento, etc. Una vez escogido el tipo de instancia lo siguiente será configurarla en cuanto a los parámetros de crear una nueva subred o un nuevo rol IAM. Después se le añade el almacenamiento, por defecto es un volumen SSD de 8 GB de capacidad. Es posible también asignarle un nombre (tag) para referirse a ella. Ya por último configuramos los grupos de seguridad para permitir el acceso de tráfico entrante a nuestra máquina remota añadiendo reglas en las que se deben indicar tanto el protocolo soportado, el puerto por el que escuchará, y las direcciones ip que podrán enviar tráfico a la instancia EC2.

Una vez que ya se ha configurado la instancia EC2 y se ha lanzado, lo siguiente será cumplir con una serie de requisitos necesarios para poder realizar la conexión a través de SSH.

-**Requisitos**

1. Instalar un cliente SSH en el host local (por defecto viene instalado, en caso contrario instalarlo a través de http://www.openssh.com).

2. Instalar el AWS CLI Tools (es opcional, para casos en los que se va a usar una AMI pública de un tercero).

3. Obtener el ID de la instancia (desde la consola de Amazon EC2 - Instance ID).

4. Obtener el nombre DNS público de la instancia (desde la consola de  Amazon EC2 - Public DNS).

5. Localizar la clave privada (la ruta completa del fichero .pem previamente creado - Key pair name y que se encuentra descargado en el host local).

6. Habilitar el trafico SSH entrante en la instancia remota desde la IP del host local (asegurar en este caso que el grupo de seguridad asociado con la instancia lo permite, por defecto no viene habilitado).

-**Conectar a instancia EC2**

1. (Opcional) Verificar la clave RSA en la instancia remota ejecutando el siguiente comando. Útil para casos en los que la instancia se ha lanzado desde una AMI pública de terceros (hay que tener instalado previamente el awscli	) :
		
		aws ec2 get-console-output --instance-id instance_id


2. Cambiar a la localización donde se encuentra el fichero de la clave privada creado cuando la instancia se lanzó.

3. Modificar los permisos al fichero de clave privada para evitar que sea visible públicamente:
		
		chmod 400 /home/my-key-par.perm

4. Usar el comando ssh para conectarse a la instancia. Para ello hay que 	especificar la clave privada (.perm), así como el user_name@public_dns_name. Para las distribuciones de Amazon Linux el user_name por defecto es ec2-user, por lo que el comando a ejecutar sería el siguiente:

		ssh -i /home/my-key-par.perm ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
	
		
A continuación se mostrará un mensaje mostrando la RSA key fingerprint y preguntará si se quiere continuar con la conexión. Al contestar si aparecerá otro mensaje indicando que se ha añadido a la lista de hosts conocidos.

Para salir de la conexión remota basta con ejecutar la orden exit en la instancia para volver al entorno del host local.


##Transferir archivos a una instancia Linux AWS usando SCP

-**Preview**

Una de las maneras que existen para transferir archivos entre la instancia remota y  el host local, es a través de SCP (Secure Copy). El procedimiento para realizar la transferencia de datos es muy similar a los pasos seguidos para establecer la conexión con la instancia.

-**Requisitos**

1. Instalar un cliente SSH en el host local (por defecto viene instalado, en caso contrario instalarlo a través de http://www.openssh.com).

2. Obtener el ID de la instancia (desde la consola de Amazon EC2 - Instance ID).

3. Obtener el nombre DNS público de la instancia (desde la consola de  Amazon EC2 - Public DNS).

4. Localizar la clave privada (la ruta completa del fichero .pem previamente creado - Key pair name y que se encuentra descargado en el host local).

5. Habilitar el trafico SSH entrante desde la IP del host local a la instancia (asegurar en este caso que el grupo de seguridad asociado con la instancia lo permite, por defecto no viene habilitado).

-**Usar scp para transferir un fichero**

1. (Opcional) Verificar la clave RSA en la instancia remota ejecutando el siguiente comando. Útil para casos en los que la instancia se ha lanzado desde una AMI pública de terceros (hay que tener instalado previamente el awscli) :
		
		aws ec2 get-console-output --instance-id instance_id

2. Cambiar a la localización donde se encuentra el fichero de la clave privada creado cuando la instancia se lanzó.

3. Modificar los permisos al fichero de clave privada para evitar que sea visible públicamente:
		
		chmod 400 /home/my-key-par.perm

4. Se usa el DNS público de la instancia para transferir un fichero y el comando scp para conectarse a la instancia. Para ello hay que especificar la clave privada (.perm), así como el **user_name@public_dns_name** como ya se hizo antes, además de indicar el path del fichero a transferir. Para las distribuciones de Amazon Linux el user_name por defecto es ec2-user, por lo que el comando a ejecutar sería el siguiente:
		
		scp -i /home/my-key-par.perm /home/Descargas/file.xml ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com:~

A continuación se mostrará un mensaje mostrando la RSA key fingerprint y preguntará si se quiere continuar con la conexión. Al contestar si aparecerá otro mensaje indicando que se ha añadido a la lista de hosts conocidos.

Para salir de la conexión remota basta con ejecutar la orden exit en la instancia para volver al entorno del host local.

Es posible que no esté instalado el paquete scp en la instancia remota, por lo que en ese caso será necesario instalarlo. En instancias AMI se realiza ejecutando el siguiente comando:
		
		sudo yum install -y opens-clients

También se puede dar el caso en el que se quiera transferir instancias en la dirección contraria, es decir, de la instancia EC2 al host local, en este tipo de situaciones se usa la misma orden usada anteriormente, con la salvedad de que los parámetros deben ser intercambiados como se muestra a continuación:
	
	scp -i /home/my-key-par.perm ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com:~/home/Descargas/file.xml ~/home/Descargas/file2.xml


##Instalar Docker en una instancia Linux AWS

Para realizar la instalación de Docker en una instancia Linux EC2 de AWS hay que seguir los siguientes pasos:
Conectarse a la instancia por SSH.

1. Actualizar los paquetes instalados y la caché de paquetes en la instancia:
		
		sudo yum update -y

2. Instalar Docker:
		
		sudo yum install -y  docker


3. Iniciar el servicio Docker:
		
		sudo service docker start

4. Añadir el usuario ec2-user al grupo docker para poder ejecutar comandos Docker sin ser usuario root (es necesario salir y lograrse de nuevo para que tenga efecto):
		
		sudo usermod -a -G docker ec2-user

5. Verificar que la cuenta de usuario ec2-user puede ejecutar comandos Docker sin sudo:
	
		docker info
