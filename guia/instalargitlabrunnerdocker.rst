Instalar GitLab-Runner Docker images
===================================

Todo siempre desde el link oficial de Gitlab:

https://docs.gitlab.com/omnibus/docker/


Así es como puede ejecutar GitLab Runner dentro de un contenedor Docker.

Uso general de imágenes de GitLab Runner Docker
+++++++++++++++++++++++++++++++++++++++++++++++++

Las imágenes de GitLab Runner Docker (basadas en Ubuntu o Alpine Linux) están diseñadas como envolturas alrededor del comando gitlab-runner estándar, como si GitLab Runner se instalara directamente en el host.


La regla general es que todos los comandos de GitLab Runner que normalmente se ejecutarían como::

	gitlab-runner <runner command and options...>


se puede ejecutar con::

	docker run <chosen docker options...> gitlab/gitlab-runner <runner command and options...>

Por ejemplo, obtener la información de ayuda de nivel superior para el comando GitLab Runner podría ejecutarse como::

	docker run --rm -t -i gitlab/gitlab-runner --help

	NAME:
	   gitlab-runner -a GitLab Runner

	USAGE:
	   gitlab-runner <global options> command <command options> <arguments...>

	VERSION:
	   10.7.0 (7c273476)

	(...)

En resumen, la parte gitlab-runner del comando se reemplaza con docker run [opciones de docker] gitlab/gitlab-runner, mientras que el resto del comando permanece como se describe en la documentación del register. La única diferencia es que el comando gitlab-runner se ejecuta dentro de un contenedor Docker.

Instale la imagen de Docker e inicie el contenedor
+++++++++++++++++++++++++++++++++++++++++++++++++

Se requiere Docker. Ver la documentación oficial de instalación o en este GitHub la sección de Docker.

Para ejecutar gitlab-runner dentro de un contenedor Docker, debe asegurarse de que la configuración no se pierda cuando se reinicia el contenedor. Para hacer esto, hay dos opciones, que se describen a continuación.


Configurar la ubicación de los volúmenes
++++++++++++++++++++++++++++++++++++++++++

Antes de configurar todo lo demás, configure una nueva variable de entorno $GITLAB_HOME que apunte al directorio donde residirán la configuración, los registros y los archivos de datos. Asegúrese de que el directorio exista y de que se haya concedido el permiso adecuado y tenga suficiente espacio.

Para usuarios Linux, configurar la ruta /srv/gitlab::

	export GITLAB_HOME=/srv/gitlab-runner

Opción 1: usar montajes de volumen del sistema local para iniciar el contenedor Runner
++++++++++++++++

Este ejemplo usa el sistema local para el volumen de configuración que está montado en el contenedor gitlab-runner. Este volumen se usa para configuraciones y otros recursos.::

	   docker run -d --name gitlab-runner --restart always \
	     -v $GITLAB_HOME/config:/etc/gitlab-runner \
	     -v /var/run/docker.sock:/var/run/docker.sock \
	     gitlab/gitlab-runner:latest

En esta documentación vamos a utilizar este caso, cuando lo ejecutemos veremos que tendremos errores como el siguiente::

	docker logs -f gitlab-runner
	Runtime platform                                    arch=amd64 os=linux pid=7 revision=2ebc4dc4 version=13.9.0
	Starting multi-runner from /etc/gitlab-runner/config.toml...  builds=0
	Running in system-mode.                            
		                                           
	Configuration loaded                                builds=0
	listen_address not defined, metrics & debug endpoints disabled  builds=0
	[session_server].listen_address not defined, session endpoints disabled  builds=0
	ERROR: Failed to load config stat /etc/gitlab-runner/config.toml: no such file or directory  builds=0
	ERROR: Failed to load config stat /etc/gitlab-runner/config.toml: no such file or directory  builds=0
	ERROR: Failed to load config stat /etc/gitlab-runner/config.toml: no such file or directory  builds=0
	ERROR: Failed to load config stat /etc/gitlab-runner/config.toml: no such file or directory  builds=0
	ERROR: Failed to load config stat /etc/gitlab-runner/config.toml: no such file or directory  builds=0

Esto es motivado que debemos registrar primero el Gitlab-runner en nuestro Gitlab. Para esta documentación busca en este GitHub, Registrando Gitlab-Runners

Opción 2: usar volúmenes de Docker para iniciar el contenedor Runner
++++++++++++++++++++

En este ejemplo, puede utilizar un contenedor de configuración para montar su volumen de datos personalizado.:

1.- Crear el volumen de Docker::

	docker volume create gitlab-runner-config

2.- Iniciar el contenedor de GitLab-Runner usando el volumen que fue creado::

	docker run -d --name gitlab-runner --restart always \
	    -v /var/run/docker.sock:/var/run/docker.sock \
	    -v gitlab-runner-config:/etc/gitlab-runner \
	    gitlab/gitlab-runner:latest

Registrar al Runner
+++++++++++++++++

El último paso es registrar un nuevo Runner. El GitLab-Runner Container no recogerá ningún trabajo hasta que esté registrado. ver más documentación en. https://docs.gitlab.com/runner/register/index.html#docker.
Para esta documentación busca en este GitHub, Registrando Gitlab-unners

Actualizar configuración
+++++++++++++++++

Si cambia la configuración en config.toml, es posible que deba reiniciar el runner para aplicar el cambio. Asegúrese de reiniciar todo el contenedor en lugar de usar gitlab-runner restart::

	docker restart gitlab-runner

Versión de actualización
++++++++++++++++++++++

Extraiga la última versión (o una etiqueta específica)::

	docker pull gitlab/gitlab-runner:latest

Detenga y retire el contenedor existente::

	docker stop gitlab-runner && docker rm gitlab-runner

Inicie el contenedor como lo hizo originalmente::

	docker run -d --name gitlab-runner --restart always \
	  -v /var/run/docker.sock:/var/run/docker.sock \
	  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
	  gitlab/gitlab-runner:latest

Leer LOGs de GitLab Runner
++++++++++++++++++++++++

Cuando se inicia GitLab Runner como una tarea en primer plano (ya sea un binario instalado localmente o dentro de un contenedor Docker), los registros se imprimen en la salida estándar. Cuando GitLab Runner se inicia como un servicio del sistema (por ejemplo, con Systemd), los registros se registran en la mayoría de los casos a través de Syslog u otro mecanismo de registro del sistema.

Con GitLab Runner iniciado como un servicio basado en Docker, dado que el comando gitlab-runner ... es el proceso principal del contenedor, los registros se pueden leer usando el comando docker logs.

Por ejemplo, si GitLab Runner se inició con el siguiente comando::

	docker run -d --name gitlab-runner --restart always \
	  -v /var/run/docker.sock:/var/run/docker.sock \
	  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
	  gitlab/gitlab-runner:latest


puede obtener los registros con::

	docker logs gitlab-runner


donde gitlab-runner es el nombre del contenedor, establecido con --name gitlab-runner por el primer comando.

Puede encontrar más información sobre el manejo de registros de contenedores en la página de documentación de Docker. https://docs.docker.com/engine/reference/commandline/logs/



