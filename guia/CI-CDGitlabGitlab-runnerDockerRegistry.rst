

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

Daremos una demostración con un código muy simple realizado con **Nodejs**, este código se estará versionado en un proyecto de **Gitlab**, nuestra aplicación en **Nodejs** se ejecutara en un contenedor **Docker**. La idea es cuando el código sufra modificaciones y realicemos un push para subir las modificaciones al proyecto en **Gitlab**, se ejecuten de forma desasistida una canalización de scripts, que se encargue de construir una nueva imagen de **Docker** con las modificaciones realizadas en el código y esta nueva imagen de **Docker** sera almacenada en el **Docker Registry**, posteriormente en un Servidor que tenga acceso al **Docker Registry** se descargara la nueva imagen de **Docker** y se instanciara esta imagen en un contenedor. El contenedor corriendo en el Servidor tendrá la aplicación con las modificaciones realizadas.



Estaremos Dockerizando Gitlab y Gitlab-Runer en un mismo contenedor y tendremos otro contenedor con Docker Registry, es decir, tendremos dos (2) contenedores de Docker, uno con Gitlab y Gitlab-Runer llamado **gitlab.dominio.local**, y otro con Docker Registry llamado **registry.dominio.local**.

En resumen Gitlab-Runner estará integrado con Gitlab, para que ejecute la canalización de los scripts y el resultado sera una imagen en Docker que se almacenara en Docker Registry y esta nueva imagen sera el contenedor que corra en un servidor.

Se creara un network de Docker llamada **app** del tipo Bridge y ambos contenedores deben estar en la misma network de Docker llamada **app**

En el servidor Host debemos tener creado en el archivo HOSTS los registros de nombre de los contenedores **gitlab.dominio.local** y **registry.dominio.local**, o en su defecto en un DNS.

Empecemos
+++++++++++++

Dockerizar a Gitlab y Gitlab-Runner en un mismo contenedor, que se llamara **gitlab.dominio.local**:

https://github.com/cgomeznt/Docker/blob/master/guia/dockerizargitlabCentos7.rst


Crear el contenedor Docker Registry por HTTPS, que se llamara **registry.dominio.local**, busca en esta guía **Correr un Registry accesible desde otros servidores**:

https://github.com/cgomeznt/Docker/blob/master/DeployRegistryServer.rst

Realizar la pruebas de ping entre las IP y DNS de los contenedores y también del Host.

Desde el Host hacia los contenedores::

	ping -c2 gitlab.dominio.local
	ping -c2 registry.dominio.local

Entre los contenedores::

	docker exec -ti gitlab.dominio.local ping -c2 registry.dominio.local
	docker exec -ti registry.dominio.local ping -c2 gitlab.dominio.local

Ubicar los certificados creados para el **registry.dominio.local**, debemos crear una carpeta en los servidores que vayan a utilizar el Docker Registry, es decir, en el Host y en el contenedor **gitlab.dominio.local**::

	docker exec -ti gitlab.dominio.local bash
	mkdir -p /etc/docker/certs.d/registry.dominio.local\:4443/

Y dentro de esa carpeta debemos copiar el certificado del servidor y la CA que lo firmo.::

	docker cp certs/registry.crt gitlab.dominio.local:/etc/docker/certs.d/registry.dominio.local\:4443/
	docker cp certs/rootCA.crt gitlab.dominio.local:/etc/docker/certs.d/registry.dominio.local\:4443/

Consultar desde el contenedor el registry las imágenes que tiene.::

	docker exec -ti gitlab.dominio.local curl -k https://registry.dominio.local:4443/v2/_catalog
	{"repositories":[""]}

Consultar desde el Host el registry las imágenes que tiene.::

	curl -k https://registry.dominio.local:4443/v2/_catalog
	{"repositories":[""]}

En el contenedor gitlab descargar una imagen y subirla al registry.::

	docker exec -ti gitlab.dominio.local bash

	docker pull node:12-alpine 
		12-alpine: Pulling from library/node
		ddad3d7c1e96: Pull complete 
		3a8370f05d5d: Pull complete 
		71a8563b7fea: Pull complete 
		119c7e14957d: Pull complete 
		Digest: sha256:be24b4fe27c92231c051a06e717b67e2a4dfc70d8edb0281285762292b854c03
		Status: Downloaded newer image for node:12-alpine
		docker.io/library/node:12-alpine

	docker images
		REPOSITORY                                  TAG         IMAGE ID       CREATED        SIZE
		node                                        12-alpine   deeae3752431   2 months ago   88.9MB

	docker tag node:12-alpine registry.dominio.local:4443/nodejs

	# docker images
	REPOSITORY                                  TAG         IMAGE ID       CREATED        SIZE
	registry.dominio.local:4443/nodejs          latest      deeae3752431   2 months ago   88.9MB
	node                                        12-alpine   deeae3752431   2 months ago   88.9MB

	docker push registry.dominio.local:4443/nodejs
		Using default tag: latest
		The push refers to repository [registry.dominio.local:4443/nodejs]
		c29549fbad68: Pushed 
		efc48a5f6f42: Pushed 
		33816ea3af7a: Pushed 
		9a5d14f9f550: Pushed 
		latest: digest: sha256:d714e8b527d784cd12b3dfc022f771c7f3531acb57f483be2c8f0997924a37df size: 1158

Volvemos a consultar las imágenes que tenga registry.dominio.local, lo podemos hacer desde el host o desde el contenedor::

	curl -k https://registry.dominio.local:4443/v2/_catalog
	{"repositories":["nodejs"]}


Crear un nuevo proyecto dentro de Gitlab llamado **my_app**.

.. figure:: ../images/cicd/01.png

Vamos a crear ahora un Runner del tipo Shell, necesitamos entrar en en Gitlab -> Admin area -> Overview -> Runner y copiar el token.

.. figure:: ../images/cicd/02.png

Dentro del contenedor **gitlab.dominio.local**, crear un Runner del tipo Shell.::

	docker exec -ti gitlab.dominio.local gitlab-runner register

	Runtime platform                                    arch=amd64 os=linux pid=5550 revision=c1edb478 version=14.0.1
	Running in system-mode.                            
		                                           
	Enter the GitLab instance URL (for example, https://gitlab.com/):
	http://gitlab.dominio.local
	Enter the registration token:
	uPKaQBaMJy2hN5Po25Fg
	Enter a description for the runner:
	[gitlab.dominio.local]: My First Runner Shell
	Enter tags for the runner (comma-separated):
	shell-01
	Registering runner... succeeded                     runner=uPKaQBaM
	Enter an executor: docker, docker-ssh, parallels, shell, docker-ssh+machine, kubernetes, custom, ssh, virtualbox, docker+machine:
	shell
	Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 


Certificar que el runner este creado.

.. figure:: ../images/cicd/03.png

Ir hasta el Proyecto, Setting -> CI/CD -> Runner y verificar que tenga el runner asociado y que este operativo.

.. figure:: ../images/cicd/04.png

Clonar el repositorio del proyecto Gitlab en el host, para trabajar de forma simple.

.. figure:: ../images/cicd/05.png

En el host nos vamos a una carpeta de trabajos y clonamos el repositorio::

	cd laboratorio

	git clone http://gitlab.dominio.local/root/my-app.git
	Clonando en 'my-app'...
	warning: Pareces haber clonado un repositorio sin contenido.

Dentro del proyecto clonado copiamos nuestro codigo en **Nodejs**, este proyecto esta aquí en la carpeta `codigo <https://github.com/cgomeznt/Gitlab/blob/master/codigo/app.zip>`_. ::

	unzip app.zip
	mv app my-app/

En el nuevo proyecto crear dos (2) archivos uno llamado Dockerfile y otro .gitlab-ci.yml.

**NOTA** Se da por entendido que ya se realizaron pruebas para certificar el funcionamiento del gitlab-runner.

Crear la relación confianza desde el contenedor **gitlab.domio.local**, con el usuario **gitlab-runner** hacia el host, para poder mandar a ejecutar comandos docker a través de ssh. ver este link.

https://github.com/cgomeznt/SSH/blob/master/guia/ssh_sin_password.rst

Creamos las variables dentro del proyecto de Gitlab en la sesión de Setting -> CI/CD -> Variables.

* CONTAINER - nodejs	# Este sera el nombre que le daremos al contenedor.

* CONTAINER_PORT - 3000:3000	# La aplicación dentro del contenedor estará en escucha por este puerto.

* HOST_MASTER - 172.18.0.1	# Esta es la dirección IP del adaptador Docker del Host.

* REGISTRY_HOST - registry.dominio.local:4443	# Este es el nombre DNS y puerto del registry.

* USER_MASTER - cgomeznt	# Este es el usuario con privilegios desde el contenedor gitlab.dominio.local hacer ssh en el Host.


.. figure:: ../images/cicd/06.png

Crear el Dockerfile en la raíz del proyecto::

	vi Dockerfile

	# FROM node:12-alpine 
	FROM registry.dominio.local:4443/nodejs
	MAINTAINER Carlos Gomez G cgomeznt@gmail.com
	RUN apk add --no-cache python g++ make
	WORKDIR /app
	COPY app/. . 
	RUN yarn install --production
	CMD ["node", "src/index.js"]

Crear el .gitlab-ci-yml en la raíz del proyecto::

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


Agregamos todos los cambios, hacemos el commit y el push::

	git add * && git commit -m "My Commit" && git push
	git add .gitlab-ci.yml && git commit -m "My Commit" && git push

Nos vamos al proyecto y en CI/CD veremos que se activo un pipeline gracias al Runner Shell.

Vemos como primero esta en **pending**

.. figure:: ../images/cicd/07.png

Vemos como pasa a **running**

.. figure:: ../images/cicd/08.png

Vemos dentro de **running**  todos los pipeline

.. figure:: ../images/cicd/09.png

Y este es el detalle que podemos ir viendo mientras se ejecuta el pipeline.

.. figure:: ../images/cicd/10.png

Vemos como se ejecuta el otro pipeline

.. figure:: ../images/cicd/11.png

Observamos el detalle

.. figure:: ../images/cicd/12.png

Listo ahora nos vamos al Host y verificamos que este el contenedor llamado **nodejs** este corriendo ::

	docker ps

	CONTAINER ID   IMAGE                                  COMMAND                  CREATED         STATUS         PORTS                                                                                                         NAMES
	a35c397af34f   registry.dominio.local:4443/48e66f49   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp                                                                     nodejs
	3ee06f605183   registry:2                             "/entrypoint.sh /etc…"   21 hours ago    Up 10 hours    0.0.0.0:4443->4443/tcp, :::4443->4443/tcp, 5000/tcp                                                           registry.dominio.local
	1477efa8cd73   centos:7                               "/usr/sbin/init"         24 hours ago    Up 10 hours    0.0.0.0:22->22/tcp, :::22->22/tcp, 0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   gitlab.dominio.local

Abrimos un navegado y colocamos la dirección IP del Host con el puerto 3000:


.. figure:: ../images/cicd/13.png

Vemos como el aplicativo esta operativo

.. figure:: ../images/cicd/14.png

Ahora vamos a modificar nuestro codigo de **nodejs** hacemos el push y veremos que simple es la CI/CD::

	vi app/src/static/js/app.js
	# Editamos esta linea
	#<p className="text-center">No items yet! Add one above!</p>
	<p className="text-center">Aun no tienes componentes..!!! Agregue uno arriba...!</p>


Hacemos el push.::

	git add * && git commit -m "My Commit" && git push

Y listo, verifica el CI/CD y luego verifica la URL.

.. figure:: ../images/cicd/15.png

Vemos el contenedor cual es su imagen de origen y vemos como coincide con el commit::

	docker ps
	CONTAINER ID   IMAGE                                  COMMAND                  CREATED         STATUS         PORTS                                                                                                         NAMES
	6f2c041c122f   registry.dominio.local:4443/2d366d94   "docker-entrypoint.s…"   3 minutes ago   Up 2 minutes   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp                                                                     nodejs

Vemos nuestras modificaciones 

.. figure:: ../images/cicd/16.png




Tips


hacer el curl -k https://registry.dominio.local:4443/v2/_catalog

docker images dentro de gitlab.dominio.local
