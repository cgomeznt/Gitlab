Backup y Restore en GitLab
=============================

Primero vamos a hacer los cambios de configuración necesarios para hacer backup. 

Para ello editamos el fichero /etc/gitlab/gitlab.rb::

	# vi /etc/gitlab/gitlab.rb
	
buscamos la palabra Backup Settings que nos llevará directamente a la sección.

En esta sección podemos ver dónde se almacenarán localmente los backups generados en la propiedad «backup_path». Por defecto, en «/var/opt/gitlab/backups»

En la propiedad «backup_keep_time» podemos definir en segundos el tiempo de vida de los ficheros de backup a fin de no llenar el disco de la máquina. Por defecto este valor se establece a 604800 segundos que son 7 días.

Para que los cambios de configuración tengan efecto, tenemos que ejecutar::

	# gitlab-ctl reconfigure
	
Hecho esto en cualquier momento que queramos generar un fichero de backup solo tendremos que ejecutar el siguiente comando::

	# gitlab-rake gitlab:backup:create
	
Al ejecutar este comando se generará el archivo.

**Nota:** Por razones de seguridad los ficheros: /etc/gitlab/gitlab.rb y /etc/gitlab/gitlab-secrets.json no se incluyen dentro del fichero generado y tienen que ser salvados de forma independiente.

Ahora si queremos que el backup se realice de forma automática todos los días a las 2 AM, creamos un crontab::

	0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1

Una vez tenemos salvado el fichero con el backup, vamos a ver cómo podemos utilizarlo para restaurar nuestra instancia en caso de que sea necesario.

Lo primero que necesitamos es que GitLab esté corriendo y al menos hayamos ejecutado una vez un reconfigure.

Nota: En caso de no estar corriendo no se puede restaurar el backup, así que al menos necesitamos una instancia limpia con la misma versión de GitLab corriendo.

El siguiente paso sería copiar el fichero de backup deseado en la ruta de la propiedad backup_path que por defecto es /var/opt/gitlab/backups/

Ahora vamos a parar los servicios «unicorn» y «sidekiq» únicamente, el resto de servicios tienen que estar corriendo::

	# gitlab-ctl stop unicorn
	# gitlab-ctl stop sidekiq
	
Podemos comprobar que solo estos servicios están parados ejecutando::

	# gitlab-ctl status
	
Ahora ejecutamos el comando de resturación indicando el timestamp del fichero de backup::

	# gitlab-rake gitlab:backup:restore BACKUP=1534804928_2018_08_21_11.1.4_gitlab
	
Hecho esto de forma satisfactoria es el momento de restaurar los ficheros /etc/gitlab/gitlab.rb y /etc/gitlab/gitlab-secrets.json y reiniciar::

	/etc/gitlab/gitlab.rb y /etc/gitlab/gitlab-secrets.json
	
Reiniciamos::

	# gitlab-ctl restart
	
Podemos comprobar que todos los procesos están correctamente con el comando::

	# gitlab-rake gitlab:check SANITIZE=true