Integración continua con GitLab CI/CD
=========================================

Esto es algo muy pero muy básico es solo para entender e iniciarce en Integración continua con GitLab CI/CD.
El runner que vamos a crear es con un ejecutor de Docker y el despliegue sera en un servidor en un Docker.

Paso 1: creación del repositorio de GitLab
+++++++++++++++++++++++++++++++++++++++++++

Comencemos por crear un proyecto de GitLab y agregarle un archivo HTML. Luego copiará el archivo HTML en una imagen de Nginx Docker, que a su vez implementará en el servidor.

Inicie sesión en su instancia de GitLab y haga clic en Nuevo proyecto.

.. figure:: ../images/CICD/01.png

1.- Dale un nombre de proyecto adecuado.

2.- Opcionalmente, agregue una descripción del proyecto.

3.- Asegúrese de establecer el Nivel de visibilidad en Privado o Público según sus requisitos.

4.- Finalmente haga clic en Crear proyecto

.. figure:: ../images/CICD/02.png


.. figure:: ../images/CICD/03.png


IMPORTANTE; le estamos colocando un DNS en Proyect URL, lo que significa que debes tener eso en tu DNS o en el archivo de hosts de el Docker gitlab-runner. En este ejemplo lo vamos a colocar en el archivo hosts, y la IP sera la que tenga el servidor Docker gitlab::

	docker exec -i -t gitlab-runner /bin/bash
	cat /etc/hosts
		127.0.0.1	localhost
		::1	localhost ip6-localhost ip6-loopback
		fe00::0	ip6-localnet
		ff00::0	ip6-mcastprefix
		ff02::1	ip6-allnodes
		ff02::2	ip6-allrouters
		172.17.0.2	415d1f0ca97a
		echo "172.17.0.3      gitlab.example.com gitlab" >> /etc/hosts

Importante como las imagenes que se van a generar no tinen el DNS configurado, agregamos unos DNS y le damos unos extra host que vendrian siendo como el archivo de HOSTS, se debe agregar esta linea, en el segmento de [runners.docker], en el archivo /home/srv/gitlab-runner/config/config.tom del gitlab-runner::

	dns = ["192.168.1.1","8.8.8.8"
	extra_hosts = ["gitlab.example.com:172.17.0.3"]

Asi quedaria, claro debe buscar cual es el Runner que va llamar, el cat se ejecuta en este caso desde el HOST Debian::

	cat /home/srv/gitlab-runner/config/config.tom
		[[runners]]
		  name = "My runner Docker"
		  url = "http://192.168.1.3"
		  token = "22dyykXPFzQ7qFL2z8kh"
		  executor = "docker"
		  [runners.custom_build_dir]
		  [runners.cache]
		    [runners.cache.s3]
		    [runners.cache.gcs]
		    [runners.cache.azure]
		  [runners.docker]
		    tls_verify = false
		    image = "alpine:latest"
		    privileged = false
		    disable_entrypoint_overwrite = false
		    oom_kill_disable = false
		    disable_cache = false
		    volumes = ["/cache"]
		    shm_size = 0
		    dns = ["192.168.1.1","8.8.8.8"]
		    extra_hosts = ["gitlab.example.com:172.17.0.3"]


Creemos el archivo HTML. En la página de descripción general de su proyecto, haga clic en Archivo nuevo.


.. figure:: ../images/CICD/04.png



Establezca el nombre del archivo en index.html y agregue el siguiente HTML al cuerpo del archivo:



.. figure:: ../images/CICD/05.png


.. figure:: ../images/CICD/06.png

Haga clic en Commit para aplicar los cambios y crear el archivo.

Este HTML producirá una página en blanco con un título que muestra Mi sitio web personal cuando se abre en un navegador.

Los Dockerfiles son recetas utilizadas por Docker para crear imágenes de Docker. Creemos un Dockerfile para copiar el archivo HTML en una imagen Nginx.

Vuelva a la página de descripción general del proyecto, haga clic en el botón + y seleccione la opción Nuevo archivo.

.. figure:: ../images/CICD/07.png

Establezca el nombre del archivo en Dockerfile y agregue estas instrucciones al cuerpo del archivo::

	FROM nginx:1.18
	COPY index.html /usr/share/nginx/html

Aplicamos el Commit.

.. figure:: ../images/CICD/08.png

La instrucción FROM especifica la imagen base de Docker, en este caso la imagen nginx: 1.18. 1.18 es la etiqueta de imagen que representa la versión de Nginx. La etiqueta nginx: latest hace referencia a la última versión de Nginx, pero eso podría interrumpir su aplicación en el futuro, por lo que se recomiendan las versiones fijas.

La instrucción COPY copia el archivo index.html en /usr/share/nginx/html en la imagen de Docker. Este es el directorio donde Nginx almacena contenido HTML estático.

No olvidemos hacer clic en Commit para guardar los cambios.

Paso 2 - Creación de un usuario de implementación
++++++++++++++++++++++++++++++++++++++++++++++

Va a crear un usuario dedicado a la tarea de implementación. Posteriormente, configurará la pipeline de CI/CD para iniciar sesión en el servidor con ese usuario.

En su servidor, cree un nuevo usuario::

	sudo adduser deployer

Se le guiará a través del proceso de creación de usuarios. Introduzca una contraseña segura y, opcionalmente, cualquier otra información de usuario que desee especificar. Finalmente confirme la creación del usuario con Y.

Agregue el usuario al grupo de Docker::

	sudo usermod -aG docker deployer

Esto permite que el implementador ejecute el comando de la ventana acoplable, que es necesario para realizar la implementación.


Paso 3: registrar un Runner de GitLab
++++++++++++++++++++

Para realizar un seguimiento de los entornos que tendrán contacto con la clave privada SSH, registrará su servidor como un runner de GitLab.

En su canal de implementación, desea iniciar sesión en su servidor mediante SSH. Para lograr esto, almacenará la clave privada SSH en una variable GitLab CI/CD (paso 5). La clave privada SSH es un dato muy sensible, porque es el token de entrada a su servidor. Por lo general, la clave privada nunca abandona el sistema en el que se generó. En el caso habitual, generaría una clave SSH en su máquina host y luego la autorizaría en el servidor (es decir, copiaría la clave pública en el servidor) para iniciar sesión manualmente y realizar la rutina de implementación.

Aquí la situación cambia ligeramente: desea otorgar acceso a una autoridad autónoma (GitLab CI/CD) a su servidor para automatizar la rutina de implementación. Por lo tanto, la clave privada debe salir del sistema en el que se generó y entregarse de forma fiduciaria a GitLab y otras partes involucradas. Nunca querrá que su clave privada entre en un entorno que no esté controlado o en el que no confíe.

Además de GitLab, el runner de GitLab es otro sistema en el que ingresará su clave privada. Para cada pipeline, GitLab utiliza runneres para realizar el trabajo pesado, es decir, ejecutar los trabajos que ha especificado en la configuración de CI / CD. Eso significa que el trabajo de implementación finalmente se ejecutará en un runner de GitLab, por lo tanto, la clave privada se copiará al runner para que pueda iniciar sesión en el servidor usando SSH.

Si usa GitLab Runners desconocidos (por ejemplo, runneres compartidos) para ejecutar el trabajo de implementación, entonces no se dará cuenta de que los sistemas se ponen en contacto con la clave privada. Aunque los runneres de GitLab limpian todos los datos después de la ejecución del trabajo, puede evitar enviar la clave privada a sistemas desconocidos si registra su propio servidor como un runner de GitLab. Luego, la clave privada se copiará en el servidor controlado por usted.

Empiece por ver este link que le enseñara como registrar el runner contra el Gitlab:

https://github.com/cgomeznt/Gitlab/blob/master/guia/registrargitlabrunner.rst

Con el usuario de implementación vamos a crear un runner con **Executor del tipo SHELL** y utilizaremos los datos que se obtuvierón en los **Requerimientos**::

Si tiene el Gitlab-runner en ejecucion detengalo::

	docker ps
	docker stop gitlab-runner

Coloque como respuesta a las preguntas::

		Enter the GitLab instance URL (for example, https://gitlab.com/):
			http://192.168.1.3
		Enter the registration token:
			diwM-bTpiJxqndAtjacd -> este es el Token y lo obtiene de los Requerimientos.
		Enter a description for the runner:
			Runner para ejecutar un Docker
		Enter tags for the runner (comma-separated):
			docker-demo
		Enter an executor:
			docker
		Enter the default Docker image
			alpine:latest

Ejecute el siguiente comando y complete las preguntas con este me generaba error::

	docker run --rm -it -v /home/srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register

Ejecute este otro y paso pero borre el contenedor existente y genere uno nuevo::

	docker run -d --name gitlab-runner --restart always \
	  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
	  -v /var/run/docker.sock:/var/run/docker.sock \
	  gitlab/gitlab-runner:latest

Cuidado tambien edite el archvio en el /home/srv/gitlab-runner/config/config.toml

	I worked around this issue with volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]

Esta seria la ejecución del comando las salidas y los valores que se escribieron::

	docker run --rm -it -v /home/srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register
		Runtime platform                                    arch=amd64 os=linux pid=7 revision=2ebc4dc4 version=13.9.0
		Running in system-mode.                            
				                                   
		Enter the GitLab instance URL (for example, https://gitlab.com/):
		http://192.168.1.3
		Enter the registration token:
		diwM-bTpiJxqndAtjacd
		Enter a description for the runner:
		[6bc13b2a8941]: Runner para ejecutar un Docker
		Enter tags for the runner (comma-separated):
		docker-demo
		Registering runner... succeeded                     runner=diwM-bTp
		Enter an executor: custom, docker, virtualbox, docker+machine, docker-ssh, parallels, shell, ssh, docker-ssh+machine, kubernetes:
		docker
		Enter the default Docker image (for example, ruby:2.6):
		alpine:latest
		Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 


Importante recordar estos valores, porque deben estar asociados al proyecto al archivo gitlab-ci.yml y el tipo de ejecución::

	Enter tags for the runner
	Enter an executor

Verificar que Gitlab-runner este en ejecución::

	docker ps
	docker start gitlab-runner

Paso 4: Verificar que el Runner este activo.
++++++++++++++++++++++++++++++++++++++++

Debemos antes de continuar verificar que el Runner se encuentre activo y asociado al proyecto.

Si todo fue bien, ya podremos ver en el servidor de Gitlab nuestro registro del Gitlab-runner.

.. figure:: ../images/CICD/15.png

Pero es importante revisar que el runner este conectado al Gitlab (Pendiente con el archivo hosts en los servidores de Docker), en este caso recuerda que tuve que conectar al gitlab-runner y agregar en el archivo  hosts la ip y DNS del Gitlab, pero la IP que suministra el Docker::

Luego debo verificar en el proyecto que este asociado y conectado el runner, en el menú Settting > CI/CD del proyecto y Runners

.. figure:: ../images/CICD/16.png


Paso 5: configuración de una clave SSH
++++++++++++++++++++++++++++++++++

Va a crear una clave SSH para el usuario de implementación. GitLab CI/CD luego usará la clave para iniciar sesión en el servidor y realizar la rutina de implementación.

Comencemos por cambiar al usuario de implementación recién creado para quien generará la clave SSH::

	su - deployer

Se le pedirá la contraseña del deployer para completar el cambio de usuario.

A continuación, genere una clave SSH de 4096 bits. Es importante responder correctamente a las preguntas del comando ssh-keygen:

1.- Primera pregunta: respóndela con ENTER, que almacena la clave en la ubicación predeterminada (el resto de este tutorial asume que la clave está almacenada en la ubicación predeterminada).

2.- Segunda pregunta: configura una contraseña para proteger la clave privada SSH (la clave utilizada para la autenticación). Si especifica una frase de contraseña, deberá ingresarla cada vez que se use la clave privada. En general, una frase de contraseña agrega otra capa de seguridad a las claves SSH, lo cual es una buena práctica. Alguien en posesión de la clave privada también necesitaría la frase de contraseña para usar la clave. Para los propósitos de este tutorial, es importante que tenga una frase de contraseña vacía, porque la pipeline de CI/CD se ejecutará de forma no interactiva y, por lo tanto, no permite ingresar una frase de contraseña.

Para resumir, ejecute el siguiente comando y confirme ambas preguntas con ENTER para crear una clave SSH de 4096 bits y almacenarla en la ubicación predeterminada con una frase de contraseña vacía::

	ssh-keygen -b 4096


Para autorizar la clave SSH para el usuario deployer, debe agregar la clave pública al archivo Authorized_keys::

	cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

~ es la abreviatura del home Directory del usuario en Linux. El programa cat imprimirá el contenido de un archivo; aquí se usa el operador >> para redirigir la salida de cat y agregarla al archivo allowed_keys.

En este paso, ha creado un par de claves SSH para que la pipeline de CI/CD inicie sesión e implemente la aplicación. A continuación, almacenará la clave privada en GitLab para que sea accesible durante el proceso de pipeline.

Paso 6: almacenar la clave privada en una variable GitLab CI/CD
+++++++++++++++++++++++++++++++++++++++++++++++++++++++

Va a almacenar la clave privada SSH en una variable de archivo GitLab CI/CD, de modo que la pipeline pueda hacer uso de la clave para iniciar sesión en el servidor.

Cuando GitLab crea una pipeline de CI/CD, enviará todas las variables al ejecutor correspondiente y las variables se establecerán como variables de entorno durante la duración del trabajo. En particular, los valores de las variables de archivo se almacenan en un archivo y la variable de entorno contendrá la ruta a este archivo.

Mientras esté en la sección de variables, también agregará una variable para la IP del servidor y el usuario del servidor, que informará a la pipeline sobre el servidor de destino y el usuario para iniciar sesión.

Comience mostrando la clave privada SSH::

	$ cat ~/.ssh/id_rsa

Copie la salida a su portapapeles. Asegúrese de agregar un salto de línea después

	-----END RSA PRIVATE KEY-----:

~/.ssh/id_rsa, se copia esta salida::

	-----BEGIN RSA PRIVATE KEY-----
	...
	-----END RSA PRIVATE KEY-----

Ahora navegue a Settings > CI/CD > Variables en su proyecto de GitLab y haga clic en Add Variable. Complete el formulario de la siguiente manera:

Key: ID_RSA

Value: pegue su clave privada SSH desde su portapapeles (incluido un salto de línea al final).

Type: File

Environment Scope: All (default)

Protect variable: Checked

Mask variable: Unchecked


Se creará un archivo que contiene la clave privada en el runner para cada trabajo de CI/CD y su ruta se almacenará en la variable de entorno $ID_RSA.

Crea otra variable con la IP de tu servidor. Haga clic en Add Variable y complete el formulario de la siguiente manera:

Key: SERVER_IP

Value: your_server_IP

Type: Variable

Environment scope: All (default)

Protect variable: Checked

Mask variable: Checked


Finalmente, cree una variable con el usuario de inicio de sesión. Haga clic en Add Variable y complete el formulario de la siguiente manera:

Key: SERVER_USER

Value: deployer

Type: Variable

Environment scope: All (default)

Protect variable: Checked

Mask variable: Checked


.. figure:: ../images/CICD/10.png

Ahora ha almacenado la clave privada en una variable GitLab CI/CD, que hace que la clave esté disponible durante la ejecución de la pipeline. En el siguiente paso, pasará a configurar la pipeline de CI/CD.

Paso 6: configuración del archivo .gitlab-ci.yml
++++++++++++++++++++++++++++++++++++++++

Vas a configurar la pipeline GitLab CI/CD. La pipeline creará una imagen de Docker y la enviará al registro del contenedor. GitLab proporciona un registro de contenedores para cada proyecto. Puede explorar el registro de contenedores yendo a Packages & Registries > Container Registry en su proyecto de GitLab (lea más en la documentación del registro de contenedores de GitLab). El último paso en su pipeline es iniciar sesión en su servidor, extraer la última imagen de Docker, eliminar el contenedor viejo y comience un nuevo contenedor.

Ahora va a crear el archivo .gitlab-ci.yml que contiene la configuración de la pipeline. En GitLab, vaya a la página de descripción general del proyecto, haga clic en el botón + y seleccione New File. Luego, establezca el nombre del archivo en .gitlab-ci.yml.

(Alternativamente, puede clonar el repositorio y realizar todos los cambios siguientes en .gitlab-ci.yml en su máquina local, luego confirmar y enviar al repositorio remoto).

Para comenzar, agregue lo siguiente::

	stages:
	  - publish
	  - deploy

Cada trabajo está asignado a una etapa. Los trabajos asignados a la misma etapa se ejecutan en paralelo (si hay suficientes Runners disponibles). Las etapas se ejecutarán en el orden en que se especificaron. Aquí, la etapa de publicación irá en primer lugar y la etapa de implementación en segundo lugar. Las etapas sucesivas solo comienzan cuando la etapa anterior terminó con éxito (es decir, todos los trabajos han pasado). Los nombres artísticos se pueden elegir arbitrariamente.

Cuando desee combinar esta configuración de CD con su pipeline de CI existente, que prueba y compila la aplicación, es posible que desee agregar las etapas de publicación e implementación después de las etapas existentes, de modo que la implementación solo se lleve a cabo si se aprobaron las pruebas.

Después de esto, agregue esto a su archivo .gitlab-ci.yml::

	. . .
	variables:
	  TAG_LATEST: $CI_REGISTRY_IMAGE/$CI_COMMIT_REF_NAME:latest
	  TAG_COMMIT: $CI_REGISTRY_IMAGE/$CI_COMMIT_REF_NAME:$CI_COMMIT_SHORT_SHA

La sección de variables define las variables de entorno que estarán disponibles en el contexto de la sección de secuencia de comandos de un trabajo. Estas variables estarán disponibles como variables de entorno Linux habituales; es decir, puede hacer referencia a ellos en el script colocando un prefijo con un signo de dólar como $TAG_LATEST. GitLab crea algunas variables predefinidas para cada trabajo que brindan información específica del contexto, como el nombre de la rama o el hash de confirmación en el que está trabajando el trabajo (lea más sobre la variable predefinida). Aquí compones dos variables de entorno a partir de variables predefinidas. Ellos representan:

* CI_REGISTRY_IMAGE: representa la URL del registro del contenedor vinculado al proyecto específico. Esta URL depende de la instancia de GitLab. Por ejemplo, las URL de registro para proyectos de gitlab.com siguen el patrón: registry.gitlab.com/your_user/your_project. Pero dado que GitLab proporcionará esta variable, no es necesario que conozca la URL exacta.

* CI_COMMIT_REF_NAME: el nombre de la rama o etiqueta para el que se construyó el proyecto.

* CI_COMMIT_SHORT_SHA: Los primeros ocho caracteres de la revisión de confirmación para la que se construyó el proyecto.

Ambas variables están compuestas por variables predefinidas y se utilizarán para etiquetar la imagen de Docker.

TAG_LATEST agregará la última etiqueta a la imagen. Esta es una estrategia común para proporcionar una etiqueta que siempre represente la última versión. Para cada implementación, la imagen más reciente se anulará en el registro del contenedor con la imagen de Docker recién creada.

TAG_COMMIT, por otro lado, usa los primeros ocho caracteres del SHA de confirmación que se implementa como etiqueta de imagen, creando así una imagen de Docker única para cada confirmación. Podrá rastrear el historial de las imágenes de Docker hasta la granularidad de las confirmaciones de Git. Esta es una técnica común cuando se realizan implementaciones continuas, porque le permite implementar rápidamente una versión anterior del código en caso de una implementación defectuosa.

Como explorará en los próximos pasos, el proceso de revertir una implementación a una revisión de Git anterior se puede realizar directamente en GitLab.

$CI_REGISTRY_IMAGE/$CI_COMMIT_REF_NAME especifica el nombre base de la imagen de Docker. Según la documentación de GitLab, el nombre de una imagen de Docker debe seguir este esquema::

	<registry URL>/<namespace>/<project>/<image>

$CI_REGISTRY_IMAGE representa la parte <URL del registro>/<espacio de nombres>/<proyecto> y es obligatorio porque es la raíz del registro del proyecto. $CI_COMMIT_REF_NAME es opcional pero útil para alojar imágenes de Docker para diferentes ramas. En este tutorial solo trabajará con una rama, pero es bueno construir una estructura extensible. En general, hay tres niveles de nombres de repositorios de imágenes compatibles con GitLab::

	registry.example.com/group/project:some-tag
	registry.example.com/group/project/image:latest
	registry.example.com/group/project/my/image:rc1

Para su variable TAG_COMMIT, usó la segunda opción, donde la imagen se reemplazará con el nombre de la rama.

A continuación, agregue lo siguiente a su archivo .gitlab-ci.yml::

	. . .
	publish:
	  image: docker:latest
	  stage: publish
	  services:
	    - docker:dind
	  script:
	    - docker build -t $TAG_COMMIT -t $TAG_LATEST .
	    - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN $CI_REGISTRY
	    - docker push $TAG_COMMIT
	    - docker push $TAG_LATEST


La sección de publicación es el primer trabajo en su configuración de CI/CD. Vamos a desglosarlo:

* image es la imagen de Docker que se utilizará para este trabajo. El runner de GitLab creará un contenedor Docker para cada trabajo y ejecutará el script dentro de este contenedor. docker: la última imagen garantiza que el comando docker estará disponible.

* etapa asigna el trabajo a la etapa de publicación.

* services especifica Docker-in-Docker, el servicio dind. Esta es la razón por la que registró el runner de GitLab en modo privilegiado.


La sección de script de comandos del trabajo de publicación especifica los comandos de shell que se ejecutarán para este trabajo. El directorio de trabajo se establecerá en la raíz del repositorio cuando se ejecuten estos comandos.

* docker build ...: crea la imagen de Docker basada en el Dockerfile y la etiqueta con la última etiqueta de confirmación definida en la sección de variables.

* Docker login ...: inicia sesión en Docker en el registro de contenedores del proyecto. Utiliza la variable predefinida $ CI_BUILD_TOKEN como un token de autenticación. GitLab generará el token y seguirá siendo válido durante la vida útil del trabajo.

* docker push ...: inserta ambas etiquetas de imagen en el registro del contenedor.


Después de esto, agregue el trabajo de implementación a su .gitlab-ci.yml::

	. . .
	deploy:
	  image: alpine:latest
	  stage: deploy
	  tags:
	    - docker-demo
	  script:
	    - chmod og= $ID_RSA
	    - apk update && apk add openssh-client
	    - ssh -i $ID_RSA -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN $CI_REGISTRY"
	    - ssh -i $ID_RSA -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "docker pull $TAG_COMMIT"
	    - ssh -i $ID_RSA -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "docker container rm -f my-app || true"
	    - ssh -i $ID_RSA -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "docker run -d -p 80:80 --name my-app $TAG_COMMIT"


Alpine es una distribución de Linux ligera y es suficiente como imagen de Docker aquí. Usted debe asignar el job al deploy stage. El deployment tag debe asegurar que el Job se ejecutará en los runners que están en el Tag como deployment, como el runner que configuró en el Paso 2.

job to the deploy stage

La sección de secuencia de comandos del trabajo de implementación comienza con dos comandos configurativos:

* chmod og = $ID_RSA: Revoca todos los permisos para el grupo y otros de la clave privada, de modo que solo el propietario pueda usarla. Este es un requisito, de lo contrario, SSH se niega a trabajar con la clave privada.

* apk update && apk add openssh-client: actualiza el administrador de paquetes (apk) de Alpine e instala el openssh-client, que proporciona el comando ssh.


Siguen cuatro comandos ssh consecutivos. El patrón para cada uno es::

	ssh -i $ID_RSA -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "command"


En cada declaración ssh, está ejecutando un comando en el servidor remoto. Para hacerlo, se autentica con su clave privada.

Las opciones son las siguientes:

* -i significa archivo de identidad y $ ID_RSA es la variable de GitLab que contiene la ruta al archivo de clave privada.

* -o StrictHostKeyChecking = no se asegura de omitir la pregunta, ya sea que confíe o no en el host remoto. Esta pregunta no se puede responder en un contexto no interactivo como el pipeline.

* $SERVER_USER y $SERVER_IP son las variables de GitLab que creó en el Paso 5. Especifican el host remoto y el usuario de inicio de sesión para la conexión SSH.

* El comando se ejecutará en el host remoto.

La implementación finalmente se lleva a cabo ejecutando estos cuatro comandos en su servidor:

1.- inicio de sesión de docker ...: inicia sesión en Docker en el registro del contenedor.

2.- docker pull ...: extrae la última imagen del registro del contenedor.

3.- docker container rm ...: Elimina el contenedor existente si existe. || true asegura que el código de salida siempre sea exitoso, incluso si no había ningún contenedor ejecutándose con el nombre my-app. Esto garantiza una rutina de eliminación si existe sin romper la pipeline cuando el contenedor no existe (por ejemplo, para la primera implementación).

4.- docker run ...: inicia un nuevo contenedor con la última imagen del registro. El contenedor se llamará my-app. El puerto 80 del host estará vinculado al puerto 80 del contenedor (el orden es -p host: container). -d inicia el contenedor en modo separado; de lo contrario, la pipeline se atascará esperando que termine el comando.

Continuemos agregando esto al trabajo de implementación en su .gitlab-ci.yml::

	. . .
	deploy:
	. . .
	  environment:
	    name: production
	    url: http://your_server_IP
	  only:

Los entornos de GitLab le permiten controlar las implementaciones dentro de GitLab. Puede examinar los entornos en su proyecto de GitLab yendo a Operations > Environments. Si la pipeline aún no ha finalizado, no habrá ningún entorno disponible, ya que hasta el momento no se ha realizado ninguna implementación.

Cuando un trabajo de pipeline define una sección de entorno, GitLab creará una implementación para el entorno dado (aquí producción) cada vez que el trabajo finalice con éxito. Esto le permite rastrear todas las implementaciones creadas por GitLab CI / CD. Para cada implementación, puede ver la confirmación relacionada y la rama para la que se creó.

También hay un botón disponible para volver a implementar que le permite retroceder a una versión anterior del software. La URL que se especificó en la sección de entorno se abrirá al hacer clic en el botón View deployment.

La única sección define los nombres de las ramas y las etiquetas para las que se ejecutará el trabajo. De forma predeterminada, GitLab iniciará una pipeline para cada envío al repositorio y ejecutará todos los trabajos (siempre que exista el archivo .gitlab-ci.yml). La única sección es una opción para restringir la ejecución del trabajo a ciertas ramas/etiquetas. Aquí desea ejecutar el trabajo de implementación solo para la rama maestra. Para definir reglas más complejas sobre si un trabajo debe ejecutarse o no, eche un vistazo a la sintaxis de las reglas.

Su archivo .gitlab-ci.yml completo se verá así::

	stages:
	  - publish
	  - deploy

	variables:
	  TAG_LATEST: $CI_REGISTRY_IMAGE/$CI_COMMIT_REF_NAME:latest
	  TAG_COMMIT: $CI_REGISTRY_IMAGE/$CI_COMMIT_REF_NAME:$CI_COMMIT_SHORT_SHA

	publish:
	  image: docker:latest
	  stage: publish
	  services:
	    - docker:dind
	  tags:
	    - docker-demo
	  script:
	    - docker build -t $TAG_COMMIT -t $TAG_LATEST .
	    - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN $CI_REGISTRY
	    - docker push $TAG_COMMIT
	    - docker push $TAG_LATEST

	deploy:
	  image: alpine:latest
	  stage: deploy
	  tags:
	    - docker-demo
	  script:
	    - chmod og= $ID_RSA
	    - apk update && apk add openssh-client
	    - ssh -i $ID_RSA -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN $CI_REGISTRY"
	    - ssh -i $ID_RSA -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "docker pull $TAG_COMMIT"
	    - ssh -i $ID_RSA -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "docker container rm -f my-app || true"
	    - ssh -i $ID_RSA -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "docker run -d -p 80:80 --name my-app $TAG_COMMIT"
	  environment:
	    name: production
	    url: http://192.168.1.3
	  only:
	    - master


Finalmente, haga clic en Commit para guardar cambios del archivo .gitlab-ci.yml. Alternativamente, cuando haya clonado el repositorio de Git localmente, confirme y envíe el archivo al control remoto.

Ha creado una configuración de GitLab CI/CD para crear una imagen de Docker e implementarla en su servidor. En el siguiente paso, valida la implementación.


Paso 7: Validación de la implementación
+++++++++++++++++++++++++++++++++++


Ahora validará la implementación en varios lugares de GitLab, así como en su servidor y en un navegador.

Cuando se envía un archivo .gitlab-ci.yml al repositorio, GitLab lo detectará automáticamente e iniciará una pipeline de CI / CD. En el momento en que creó el archivo .gitlab-ci.yml, GitLab inició la primera pipeline.

Vaya a CI/CD pipeline en su proyecto de GitLab para ver el estado de la pipeline. Si los trabajos aún están en ejecución/pendientes, espere hasta que se completen. Verá una pipeline aprobada con dos marcas de verificación verdes, lo que indica que el trabajo de publicación e implementación se ejecutó correctamente.


.. figure:: ../images/CICD/11.png

Luego que todos los procesos culminen

.. figure:: ../images/CICD/12.png

Examinemos la pipeline. Haga clic en el botón aprobado en la columna Estado para abrir la página de descripción general de la pipeline. Obtendrá una descripción general de información general como:

* Duración de la ejecución de todo el pipeline.

* Para qué confirmación y bifurcación se ejecutó la pipeline.

* Solicitudes de fusión relacionadas. Si hay una solicitud de fusión abierta para la sucursal a cargo, aparecerá aquí.

* Todos los trabajos ejecutados en esta pipeline, así como su estado.

A continuación, haga clic en el botón de deploy para abrir la página de resultados del trabajo de implementación.


