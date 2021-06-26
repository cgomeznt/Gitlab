

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


Estaremos Dockerizando Gitlab y Gitlab-Runer en un mismo contenedor y tendremos otro contenedor con Docker Registry.

Gitlab-Runner estara integrado con Gitlab, para que ejecute la canalización de los scripts y el resultado sera una imagen en Docker que se almacenara en Docker Registry.



Daremos una demostración con un código muy simple realizado con **Nodejs**, este código se estará versionado en un proyecto de **Gitlab**, nuestra aplicación en **Nodejs** se ejecutara en un contenedor **Docker**. La idea es cuando el código sufra modificaciones y realicemos un push para subir las modificaciones al proyecto en **Gitlab**, se ejecuten de forma desasistida una canalización de los scripts, que se encargue de construir una nueva imagen de **Docker** con las modificaciones realizadas en el código y esta nueva imagen de **Docker** sera almacenada en el **Docker Registry**, posteriormente en un Servidor se descargara la imagen de **Docker** con el nuevo código y se instancie esta imagen en un contenedor. El contenedor corriendo en el Servidor tendrá la aplicación con las modificaciones realizadas.



**Hasta aqui**

Entonces tendremos dos (2) contenedores de Docker, uno con Gitlab y Gitlab-Runer llamado **gitlab**, y otro con Docker Registry llamado registry.

Debemos crear un network de Docker llamada **app** del tipo Bridge y ambos contenedores deben estar en la misma network de Docker llamada **app**

En el servidor Host debemos tener creardo en el archivo HOSTS los registros de nombre de los contenedores **gitlab** y **registry**

Empecemos con Dockerizar a Gitlab y Gitlab-Runner en un mismo contenedor, que se llamara **gitlab**:

https://github.com/cgomeznt/Docker/blob/master/guia/dockerizargitlabCentos7.rst


Crear el contenedor Docker Registry por HTTPS, que se llamara **registry**:

https://github.com/cgomeznt/Docker/blob/master/DeployRegistryServer.rst

Crear un Runner del tipo Shell.

Crear un nuevo proyecto dentro de Gitlab.

Crear el Dockerfile en el nuevo proyecto::

	# FROM registry:5000/nodejs
	# MAINTAINER Carlos Gomez G cgomeznt@gmail.com

	FROM node:12-alpine
	RUN apk add --no-cache python g++ make
	WORKDIR /app
	COPY . .
	RUN yarn install --production
	CMD ["node", "src/index.js"]



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
	    - echo $REGISTRY
	    - echo "$CI_COMMIT_SHORT_SHA - $REPO_DEV:5000/$CI_COMMIT_SHORT_SHA - $CI_COMMIT_SHORT_SHA"
	    - docker build -t $CI_COMMIT_SHORT_SHA .
	    - docker image tag $CI_COMMIT_SHORT_SHA $REPO_DEV:5000/$CI_COMMIT_SHORT_SHA
	    - docker push $REPO_DEV:5000/$CI_COMMIT_SHORT_SHA

	Deploy:
	  only:
	    refs:
	      - master
	  stage: deploy
	  tags:
	    - shell-01
	  script:
	    - touch /tmp/prueba.txt
	  except: ['master']  #Indica en las ramas en las que no se ejecutara esta actividad 
	  except: ['develop'] #Indica en las ramas en las que no se ejecutara esta actividad 

Crear la relación confianza desde el contenedor gitlab y el host, para poder mandar a ejecutar comandos docker a través de ssh

Clonar el repositorio en el host

docker run -dti --name nodejs -p 3000:3000 registry:5000/e4c400e1
docker rm -f nodejs

realizar las pruebas


