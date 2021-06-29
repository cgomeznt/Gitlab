

CI/CD con Gitlab Gitlab-runner y con un Docker Registry
=============================================

La idea es que la empresa pueda tener **Integración Continua** 100% funcional. 

Glosario
++++++++

**Nodejs**: Es un lenguaje de programación de código abierto, no compilado es por interpretación, basado en el lenguaje de programación JavaScript.

**CI/CD**: Continuous Integration (CI), Continuous Delivery (CD), Continuous Deployment (CD)

**Continuous Integration (CI)**: funciona insertando pequeños fragmentos de código en la base de código de su aplicación alojada en un repositorio de Git, y para cada inserción, ejecute una canalización de scripts para construir, probar y validar los cambios de código antes de fusionarlos en la rama principal.

**Continuous Delivery, Continuous Deployment (CD)**: consisten en un paso más allá de la (CI), implementando su aplicación en producción en cada impulso a la rama predeterminada del repositorio.

**Docker**: Es una plataforma abierta para desarrollar, enviar y ejecutar aplicaciones. Docker le permite separar sus aplicaciones de su infraestructura para que pueda entregar software rápidamente. Con Docker, puede administrar su infraestructura de la misma manera que administra sus aplicaciones. Al aprovechar las metodologías de Docker para enviar, probar e implementar código rápidamente, puede reducir significativamente la demora entre la escritura del código y su ejecución en producción.

**Imagen**: Una imagen es una plantilla de solo lectura con instrucciones para crear un contenedor Docker según la necesidad.

**Contenedor**: Un contenedor es una instancia ejecutable de una imagen, donde ya esta una aplicación prestando servicio.

**Gitlab**: control de versiones y desarrollo, basado en Git.

**Gitlab-Runer**: funciona con GitLab CI/CD para ejecutar una canalización de scripts.

**Docker Registry**: Almacena y le permite distribuir imágenes de Docker. Altamente escalable.

Introducción
++++++++++++++

Daremos una demostración con un código muy simple realizado con **Nodejs**, este código se estará versionado en un proyecto de **Gitlab**, nuestra aplicación en **Nodejs** se ejecutara en un contenedor **Docker**. La idea es cuando el código sufra modificaciones y realicemos un push para subir las modificaciones al proyecto en **Gitlab**, se ejecuten de forma desasistida una canalización de los scripts, que se encargue de construir una nueva imagen de **Docker** con las modificaciones realizadas en el código y esta nueva imagen de **Docker** sera almacenada en el **Docker Registry**, posteriormente en un Servidor se descargara la imagen de **Docker** con el nuevo código y se instancie esta imagen en un contenedor. El contenedor corriendo en el Servidor tendrá la aplicación con las modificaciones realizadas.



**Hasta aquí**

Estaremos Dockerizando Gitlab y Gitlab-Runer en un mismo contenedor y tendremos otro contenedor con Docker Registry, es decir, tendremos dos (2) contenedores de Docker, uno con Gitlab y Gitlab-Runer llamado **gitlab**, y otro con Docker Registry llamado registry.

Gitlab-Runner estará integrado con Gitlab, para que ejecute la canalización de los scripts y el resultado sera una imagen en Docker que se almacenara en Docker Registry.

Se creara un network de Docker llamada **app** del tipo Bridge y ambos contenedores deben estar en la misma network de Docker llamada **app**

En el servidor Host debemos tener creado en el archivo HOSTS los registros de nombre de los contenedores **gitlab** y **registry**, o en su defecto en un DNS.

Empecemos con Dockerizar a Gitlab y Gitlab-Runner en un mismo contenedor, que se llamara **gitlab**:

https://github.com/cgomeznt/Docker/blob/master/guia/dockerizargitlabCentos7.rst


Crear el contenedor Docker Registry por HTTPS, que se llamara **registry**, busca esta guía **Correr un Registry accesible desde otros servidores**:

https://github.com/cgomeznt/Docker/blob/master/DeployRegistryServer.rst

Realizar la pruebas de ping entre las IP y DNS de los contenedores y también del Host.

En el contenedor gitlab descargar una imagen y subirla al registry.

Consultar desde el contenedor y desde el Host el registry las imágenes que tiene.

Crear un nuevo proyecto dentro de Gitlab llamado **my*app**.

Crear un Runner del tipo Shell.

Certificar que el runner creado este asociado al Proyecto y running.

En el nuevo proyecto crear dos (2) archivos uno llamado Dockerfile y otro .gitlab-ci.yml.

En el archivo .gitlab-ci.yml. hacemos unas simples pruebas para certificar el funcionamiento del gitlab-runner.

Clonar el repositorio del proyecto Gitlab en el host.

Crear la relación confianza desde el contenedor gitlab y el host, para poder mandar a ejecutar comandos docker a través de ssh.

Crear el Dockerfile en el nuevo proyecto::

	# FROM node:12-alpine 
	FROM registry:5000/nodejs
	MAINTAINER Carlos Gomez G cgomeznt@gmail.com
	RUN apk add --no-cache python g++ make
	WORKDIR /app
	COPY app/. .
	RUN yarn install --production
	CMD ["node", "src/index.js"]


Creamos las variables dentro del proyecto de Gitlab en la sesión de Setting -> CI/CD -> Variables
CONTAINER - nodejs
CONTAINERPORT - 3000:3000
HOSTMASTER - 172.18.0.1
REPO_DEV - registry:5000
USERMASTER - cgomeznt


Crear el .gitlab-ci-yml en el nuevo proyecto::

	stages:
	  - test
	  - deploy

	Test:
	  stage: test
	  tags:
	  - shell-01
	  script:
	    - echo "write your test here...!!!"
	    - echo $CONTAINER
	    - echo "$CI_COMMIT_SHORT_SHA - $REGISTRY_HOST/$CI_COMMIT_SHORT_SHA - $CI_COMMIT_SHORT_SHA"
	    - docker build -t $CI_COMMIT_SHORT_SHA .
	    - docker image tag $CI_COMMIT_SHORT_SHA $REGISTRY_HOST/$CI_COMMIT_SHORT_SHA
	    - docker push $REGISTRY_HOST/$CI_COMMIT_SHORT_SHA

	Deploy:
	  only:
	    refs:
	      - master
	  stage: deploy
	  tags:
	    - shell-01
	  script:
	    - touch /tmp/prueba.txt
	    - ssh $USER_MASTER@$MASTER_HOST -p 222 docker rm -f $CONTAINER
	    - ssh $USER_MASTER@$MASTER_HOST -p 222 docker run -dti --name $CONTAINER -p $CONTAINER_PORT $REGISTRY_HOST/$CI_COMMIT_SHORT_SHA
	  # except: ['master']  #Indica en las ramas en las que no se ejecutara esta actividad 
	  # except: ['develop'] #Indica en las ramas en las que no se ejecutara esta actividad 


realizar las pruebas
modificar el codigo y hacer el push
se deben activar los pipeline de script 
se debe crear la imagen
se debe subir la imagen al registry
en el servidor remoto debe matar el contenedor actualmente corriendo
descargar la imagen
instanciar el contenedor


