Upgra de Gitlab
==================

Este es el plan de trabajo para actualizar Gitlab en el siguiente caso:

Tenemos una versión de Gitlab muy obsoleta que no permite hacer los upgrade progresivos y ya posee data y CI/CD. Vamos a instalar en un servidor la ultima versión de Giltab y vamos a migrar todo:


* Instalar Gitlab

* Instalar Gitlab-runner

* Registrar el Gitlab-runner en Gitlap

* Crearlos runner

* Personalizacion del Gitlab

* relación confianza entre los server

* Configurar el Gitlap para que conecte al Registry por SSL/TLS

* Configurar el Gitlap para el LDAP

* Crear los grupos

* Exportar los proyectos

* Crear las variables de CI-CD

* Las variables no pueden estar protegidas

* Recordar qeu las variables CI-CD deben ser publicas.

* Cambiar el valor de los artefactos para que sea de 8G

* echo 524288 | sudo tee -a /proc/sys/fs/inotify/max_user_watches
