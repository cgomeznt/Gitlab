1. Actualización de GitLab-CE
===============================

1.2. Actualizar los paquetes
----------------------------

Actualizar la lista de paquetes::

	sudo dnf clean all
	sudo dnf makecache

Actualizar GitLab-CE::

	sudo dnf upgrade gitlab-ce


1.3. Reconfigurar GitLab
-----------------------------

Reconfigurar después de la actualización::

	sudo gitlab-ctl reconfigure

Reiniciar servicios::

	sudo gitlab-ctl restart
	
1.4. Verificar la actualización
----------------------------------

Verificar estado::

	sudo gitlab-ctl status

2. Actualización de GitLab Runner
------------------------------------

Esto se obtine desde el portal de GitLab, 

boton en la parte inferior izquierda "Admin"

En el menu "CI/CD" y en "Runners"

Luego en los cuatro botones en la parte superior derecha "Show Runner Installation and Registration Instructions"

Esto es más o menos lo que muestra::

	# Download the binary for your system
	sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

	# Give it permission to execute
	sudo chmod +x /usr/local/bin/gitlab-runner

	# Create a GitLab Runner user
	sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

	# Install and run as a service
	sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
	sudo gitlab-runner start

	# If using a `deb` package based distribution
	curl -s https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
	apt install -y gitlab-runner

	# If using an `rpm` package based distribution
	curl -s https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
	dnf install -y gitlab-runner


