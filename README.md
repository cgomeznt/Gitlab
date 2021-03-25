## GitLab Enterprise Edition


* [Instalar GitLab Enterprise Edition](guia/instalargitlab.rst)
* [Configuracion Inicial de GitLab Enterprise Edition](guia/configuracioninicial.rst)
* [Personalizar GitLab Enterprise Edition](guia/personalizar.rst)
* [Instalar GitLab Docker images](guia/instalargitlabdocker.rst)
* [Instalar GitLab-Runner Docker images](guia/instalargitlabrunnerdocker.rst)
* [Registrando GitLab-Runner](guia/registrargitlabrunner.rst)
* [CI/CD primeros pasos con un Runner SHELL](guia/integracioncontinuaentender.rst)
* [CI/CD primeros pasos con un Runner Docker](guia/integracioncontinuaentenderDocker.rst)
* [Integración continua con GitLab CI/CD](guia/integracioncontinua.rst)

Gitlab	172.17.0.4 
Gitlab Runner	172.17.0.2
Gitlab CI/CD
Gitlab Docker Registry	172.17.0.3

Recuerda lo importante del Tag en el runner
Recuerda lo importante del Executor en el runner
Recuerda lo importante de indicarle al runner cual es su executor (Shell, Docker, SSH, etc, etc)
Verificar que el runner este operativo y eso se hace con le usuario 
Verificar que el runner este asociado y activo para el proyecto

Verificar que gitlab-registry este asociado al proyecto

I worked around this issue with volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]

docker run -d --name gitlab-runner --restart always \
  -v /home/srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest


echo -e "127.0.0.1	localhost \
::1	localhost ip6-localhost ip6-loopback \n \
fe00::0	ip6-localnet \n \
ff00::0	ip6-mcastprefix \n \
ff02::1	ip6-allnodes \n \
ff02::2	ip6-allrouters \n \
172.17.0.2	e9a36e13d351 \n \
172.17.0.3      gitlab.example.com gitlab" > /etc/hosts

echo -e "172.17.0.4      gitlab.example.com gitlab" >> /etc/hosts
echo -e "172.17.0.2      gitlab-runner" >> /etc/hosts
echo -e "172.17.0.3      gitlab-registry" >> /etc/hosts

Link:

https://docs.gitlab.com/omnibus/docker/
https://docs.gitlab.com/runner/
https://docs.gitlab.com/runner/install/docker.html
https://docs.gitlab.com/runner/register/#macos

https://blog.develat.io/pipelines-en-gitlab-el-camino-hacia-el-ci-cd-parte-i/#gitlab-runners

https://tangelov.me/posts/ic-gitlab.html
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-continuous-deployment-pipeline-with-gitlab-ci-cd-on-ubuntu-18-04
https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-socket-binding
http://vlabs.iitb.ac.in/gitlab/help/ci/docker/using_docker_build.md

https://blog.irontec.com/creando-un-docker-registry-en-nuestro-gitlab/
https://docs.gitlab.com/ee/user/packages/container_registry/
https://docs.docker.com/registry/deploying/
https://docs.gitlab.com/ee/administration/packages/container_registry.html
https://about.gitlab.com/blog/2016/05/23/gitlab-container-registry/




