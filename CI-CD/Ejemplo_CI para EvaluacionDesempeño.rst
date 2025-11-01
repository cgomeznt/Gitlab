
El servidor de Gitlab 10.10.10.21 - servergitlab debe poder llegar a los servidores de ServerTest por SSH puerto 22.

En los servidores de ServerTest debe existir un usuario llamado::
	
	gitlab-runner

Se debe crear una relacion de confianza entre el servidor Gitlab y los servidores de ServerTest por medio del usuario gitlab-runner
y debe ser sudoers::

	useradd -c "gitlab-runner - Usuario utilizado en Gitlab para el CI-CD" -d /home/gitlab-runner gitlab-runner -s /bin/bash -G soinfra
	passwd gitlab-runner
	r00tme


Agregar o descomentar la siguientes línea en el archivo /etc/ssh/sshd_config::

	PubkeyAuthentication yes
	AuthorizedKeysFile      .ssh/authorized_keys
	PubkeyAcceptedKeyTypes +ssh-dss #this is the correct entry

Preparar el directorio .ssh::

	mkdir /home/gitlab-runner/.ssh
	chown gitlab-runner. /home/gitlab-runner/.ssh/

Reiniciar el servicio de sshd::

	[root@TuServerTest .ssh]# systemctl restart sshd


En el servidor de Gitlab, se debe realizar la copia del id_rsa.pub a los servidores de ServerTest:

Entramos al servidor de Gitlab y nos vamos al directorio .ssh del usuario gitlab-runner::

	[root@servergitlab]# su - gitlab-runner
	Last login: Mon May 23 12:29:03 -04 2022
	[gitlab-runner@servergitlab ~]$ cd .ssh/
	
	[gitlab-runner@servergitlab .ssh]$ ls -l
	total 12
	-rw------- 1 gitlab-runner gitlab-runner 3247 Jan 28  2020 id_rsa
	-rw-r--r-- 1 gitlab-runner gitlab-runner  757 Jan 28  2020 id_rsa.pub
	-rw-r--r-- 1 gitlab-runner gitlab-runner 3645 May  4 14:34 known_hosts
	
Copiamos el contenido del id_rsa.pub a cada uno de los servidores de ServerTest::

	[root@servergitlab]# cat id_rsa.pub |  ssh gitlab-runner@10.10.10.222 'cat > .ssh/authorized_keys'

Nos vamos a los servidores de ServerTest y en cada uno de ellos no vamos a la ruta /home/gitlab-runner/.ssh/ y cambiamos los permisos a 400 del archivo authorized_keys::

	[root@TuServerTest gitlab-runner]# cd /home/gitlab-runner/.ssh/
	[root@TuServerTest .ssh]# chmod 400 authorized_keys
	
	[root@TuServerTest .ssh]# ls -l
	total 4
	-r-------- 1 gitlab-runner gitlab-runner 757 May 23 15:32 authorized_keys
	[root@TuServerTest .ssh]#

Hacemos inicio de sesión con gitlab-runner::

	# su - gitlab-runner
	
Luego certificamos desde el Gitlab que se pueda hacer ssh gitlab-runner@10.10.10.222 a cada uno de los servidores de ServerTest e ingrese sin pedir clave::

	[gitlab-runner@servergitlab .ssh]$ ssh gitlab-runner@10.10.10.222
	Last login: Mon May 23 15:38:45 2022 from gitlab.local.com.ve



	[gitlab-runner@TuServerTest ~]$



Este es el  contenido de gitlab-ci.yml::

stages:
  - deployDev
  - deployQa
  - deployPro

deployDev:
  stage: deployDev
  script: 
        - ssh -t -t gitlab-runner@10.10.10.555 "sudo df -h > espacioDEV.txt"
  tags:
        - shell
  only:
        - dev

deployQa:
  stage: deployQa
  script:
        - ssh -t -t gitlab-runner@10.132.7.666 "sudo df -h > espacioQA.txt"
  tags:
        - shell
  only:
        - qa

deployPro:
  stage: deployPro
  script:
        - scp -r ./Autogestion.war  gitlab-runner@10.10.10.444:/home/gitlab-runner
        - ssh -t -t gitlab-runner@10.10.10.444 "sudo chown tomcat.tomcat /home/gitlab-runner/Autogestion.war"
        - ssh -t -t gitlab-runner@10.10.10.444 "sudo systemctl stop tomcat.service"
        - ssh -t -t gitlab-runner@10.10.10.444 "sudo cp -dp /home/gitlab-runner/Autogestion.war /opt/tomcat/webapps/"
        - ssh -t -t gitlab-runner@10.10.10.444 "sudo rm -rf /home/gitlab-runner/Autogestion.war"
        - ssh -t -t gitlab-runner@10.10.10.444 "sudo rm -rf /opt/tomcat/work"
        - ssh -t -t gitlab-runner@10.10.10.444 "sudo systemctl start tomcat.service"
  tags:
        - shell
  only:
      changes:
        - deploy.txt
  except:
    - qa
    - dev
    - repowar



	
	
- ssh -t -t gitlab-runner@10.134.0.209 "sudo rm -Rf /opt/SMIWEB/smiweb/.git"

	  
Crear Clone Strategy esto se hace en las variables CI/CD del proyecto. hay dos tecnicas de hacerlo, solo seleccione una::
Tecnica 1::


	variables:
	  GIT_STRATEGY: clone  
	  
Tecnica 2::

	Ir al project,

	Luego en  "Settings -> CI / CD -> General pipelines".

	Colocar  la opcion"git clone"


	Git shallow clone colocarlo en 0
