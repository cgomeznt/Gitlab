

CI/CD con Gitlab Gitlab-runner y con un Docker Registry
=============================================

En este laboratorio se va Dockerizar Gitlab y Gitlab-Runer en un mismo contenedor y Gitlab-Runner se va integrar al Gitlab, por otro lado tendremos un contenedor Docker Registry.

La idea del laboratorio es crear un proyecto en Gitlab y que este proyecto tenga asociado un Gitlab-Runner del tipo shell que nos va permitir ejecutar unos comandos de prueba, descargar una imagen desde el Docker Registry, compilar una nueva imagen con la ayuda de Dockerfile, luego etiquetar esta nueva imagen y subirla al Docker Registry, por ultimo instanciar esta nueva imagen que esta en el Docker Registry en el servidor Hosts.

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

	FROM registry:5000/nodejs

	MAINTAINER Carlos Gomez G cgomeznt@gmail.com



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

Crear la relación confianza desde el contenedor gitlab y el host, para poder mandar a ejecutar comandos docker a través de ssh

docker run -dti --name nodejs -p 3000:3000 registry:5000/e4c400e1
docker rm -f nodejs

realizar las pruebas


