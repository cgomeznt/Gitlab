Building a Docker in Docker image with Gitlab
=======================================================

Vamos a utilizar la tecnica del Multi-Stage, ver link oficial::

	https://docs.docker.com/build/building/multi-stage/

Esto se debe garantizar que el servicio de Dockerd lo inicie, para eso nos vamos al archivo de configuracion y sino existe lo creamos::

	/etc/docker/daemon.json
	
Agregamos la siguiente linea y reiniciamos el servicio de docker::

	{"hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]}

En este link fue que encontre el trabajo de gitlab-ci.yml de Docker in Docker::

	https://faun.pub/building-a-docker-image-with-gitlab-ci-and-net-core-8f59681a86c4
	
Este es el link en donde se compila un Angular CLI en un Docker y de aquí se utilizo el codigo para compilar::

	https://github.com/cgomeznt/nodejs/blob/main/1.Creando_aplicaciones_Angular_con_Angular_CLI.rst

Este es el Dockerfile::

	### STAGE 1: Build ###
	FROM node AS build
	WORKDIR  /usr/src/app
	COPY package*.json ./

	RUN npm install -g @angular/cli
	RUN npm install
	COPY . .
	RUN ng build

	### STAGE 2: Run ###
	FROM httpd
	COPY --from=build /usr/src/app/myfile /usr/local/apache2/htdocs/
	COPY --from=build /usr/src/app/dist/* /usr/local/apache2/htdocs/

	
Este es el YAML::

	image: docker:stable

	services:
	  - name: docker:dind
		alias: 10.134.0.89
		entrypoint: ["dockerd-entrypoint.sh", "--tls=false"]

	variables:
		DOCKER_HOST: tcp://10.134.0.89:2375/
		DOCKER_DRIVER: overlay2
		DOCKER_TLS_CERTDIR: ""

	before_script:
	  - docker info

	build:
	  only:
		changes:
		  - deploy.txt

	  script:
		- docker build -t ${CI_PROJECT_NAME}/${CI_PIPELINE_ID}:lates .
	  stage: build
	  tags: 
		- docker

NOTA: presentamos problemas y al agregar el **alias** y **DOCKER_HOST** solventamos, pero si utilizamod nombres, entonces debemos tener configurado el 
DNS o el archivo hosts
