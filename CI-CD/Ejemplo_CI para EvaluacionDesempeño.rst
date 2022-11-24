
El servidor de Gitlab 10.134.0.29 - lcsdevapptdgitlab debe poder llegar a los servidores de EvaluacionDesempeño por SSH puerto 22.

En los servidores de EvaluacionDesempeño debe existir un usuario llamado::
	
	gitlab-runner

Se debe crear una relacion de confianza entre el servidor Gitlab y los servidores de EvaluacionDesempeño por medio del usuario gitlab-runner
y debe ser sudoers::

	useradd -c "gitlab-runner - Usuario utilizado en Gitlab para el CI-CD" -d /home/gitlab-runner gitlab-runner -s /bin/bash -G soinfra
	passwd gitlab-runner
	Cr3d1c4rd


Agregar o descomentar la siguientes línea en el archivo /etc/ssh/sshd_config::

	PubkeyAuthentication yes
	AuthorizedKeysFile      .ssh/authorized_keys
	PubkeyAcceptedKeyTypes +ssh-dss #this is the correct entry

Preparar el directorio .ssh::

	mkdir /home/gitlab-runner/.ssh
	chown gitlab-runner. /home/gitlab-runner/.ssh/

Reiniciar el servicio de sshd::

	[root@lcsqaappevaluaciond .ssh]# systemctl restart sshd


En el servidor de Gitlab, se debe realizar la copia del id_rsa.pub a los servidores de EvaluacionDesempeño:

Entramos al servidor de Gitlab y nos vamos al directorio .ssh del usuario gitlab-runner::

	[root@lcsdevapptdgitlab]# su - gitlab-runner
	Last login: Mon May 23 12:29:03 -04 2022
	[gitlab-runner@lcsdevapptdgitlab ~]$ cd .ssh/
	
	[gitlab-runner@lcsdevapptdgitlab .ssh]$ ls -l
	total 12
	-rw------- 1 gitlab-runner gitlab-runner 3247 Jan 28  2020 id_rsa
	-rw-r--r-- 1 gitlab-runner gitlab-runner  757 Jan 28  2020 id_rsa.pub
	-rw-r--r-- 1 gitlab-runner gitlab-runner 3645 May  4 14:34 known_hosts
	
Copiamos el contenido del id_rsa.pub a cada uno de los servidores de EvaluacionDesempeño::

	[root@lcsdevapptdgitlab]# cat id_rsa.pub |  ssh gitlab-runner@10.134.4.108 'cat > .ssh/authorized_keys'

Nos vamos a los servidores de EvaluacionDesempeño y en cada uno de ellos no vamos a la ruta /home/gitlab-runner/.ssh/ y cambiamos los permisos a 400 del archivo authorized_keys::

	[root@lcsqaappevaluaciond gitlab-runner]# cd /home/gitlab-runner/.ssh/
	[root@lcsqaappevaluaciond .ssh]# chmod 400 authorized_keys
	
	[root@lcsqaappevaluaciond .ssh]# ls -l
	total 4
	-r-------- 1 gitlab-runner gitlab-runner 757 May 23 15:32 authorized_keys
	[root@lcsqaappevaluaciond .ssh]#

Hacemos inicio de sesión con gitlab-runner::

	# su - gitlab-runner
	
Luego certificamos desde el Gitlab que se pueda hacer ssh gitlab-runner@10.134.4.108 a cada uno de los servidores de EvaluacionDesempeño e ingrese sin pedir clave::

	[gitlab-runner@lcsdevapptdgitlab .ssh]$ ssh gitlab-runner@10.134.4.108
	Last login: Mon May 23 15:38:45 2022 from gitlab.credicard.com.ve

	######################################################
	#     _____              _ _  _____              _   #
	#    / ____|            | (_)/ ____|            | |  #
	#   | |     _ __ ___  __| |_| |     __ _ _ __ __| |  #
	#   | |    | '__/ _ \/ _` | | |    / _` | '__/ _` |  #
	#   | |____| | |  __/ (_| | | |___| (_| | | | (_| |  #
	#    \_____|_|  \___|\__,_|_|\_____\__,_|_|  \__,_|  #
	#                                                    #
	######################################################

	[gitlab-runner@lcsqaappevaluaciond ~]$



Este es el  contenido de gitlab-ci.yml::

stages:
  - deployDev
  - deployQa
  - deployPro

deployDev:
  stage: deployDev
  script: 
        - ssh -t -t gitlab-runner@10.132.7.555 "sudo df -h > espacioDEV.txt"
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
        - scp -r ./Autogestion.war  gitlab-runner@10.132.7.772:/home/gitlab-runner
        - ssh -t -t gitlab-runner@10.132.7.772 "sudo chown tomcat.tomcat /home/gitlab-runner/Autogestion.war"
        - ssh -t -t gitlab-runner@10.132.7.772 "sudo systemctl stop tomcat.service"
        - ssh -t -t gitlab-runner@10.132.7.772 "sudo cp -dp /home/gitlab-runner/Autogestion.war /opt/tomcat/webapps/"
        - ssh -t -t gitlab-runner@10.132.7.772 "sudo rm -rf /home/gitlab-runner/Autogestion.war"
        - ssh -t -t gitlab-runner@10.132.7.772 "sudo rm -rf /opt/tomcat/work"
        - ssh -t -t gitlab-runner@10.132.7.772 "sudo systemctl start tomcat.service"
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