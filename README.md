## GitLab Enterprise Edition


* [Instalar GitLab Enterprise Edition](guia/instalargitlab.rst)
* [Configuracion Inicial de GitLab Enterprise Edition](guia/configuracioninicial.rst)
* [Personalizar GitLab Enterprise Edition](guia/personalizar.rst)
* [Instalar GitLab Docker images](guia/instalargitlabdocker.rst)
* [Instalar GitLab-Runner Docker images](guia/instalargitlabrunnerdocker.rst)
* [Instalar el Gitlab Container Registry Imagen](guia/instalargitlabcontainerregistry.rst)
* [Registrando GitLab-Runner](guia/registrargitlabrunner.rst)
* [CI/CD primeros pasos con un Runner SHELL](guia/integracioncontinuaentender.rst)
* [CI/CD primeros pasos con un Runner Docker](guia/integracioncontinuaentenderDocker.rst)
* [CI/CD con Gitlab Gitlab-runner y con un Docker Registry](guia/CI-CDGitlabGitlab-runnerDockerRegistry.rst)
* [Integración continua con GitLab CI/CD](guia/integracioncontinua.rst)


CI-CDGitlabGitlab-runnerDocker Registry.rst
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

    extra_hosts = ["gitlab.example.com:192.168.1.3"]

docker run -d --name gitlab-runner --restart always \
  -v /home/srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest


echo -e "127.0.0.1	localhost" > /etc/hosts
echo -e "::1		localhost ip6-localhost ip6-loopback" >>/etc/hosts
echo -e "fe00::0		ip6-localnet" Z>/etc/hosts
echo -e "ff00::0		ip6-mcastprefix" >>/etc/hosts
echo -e "ff02::1		ip6-allnodes" >>/etc/hosts
echo -e "ff02::2		ip6-allrouters" >>/etc/hosts
echo -e "172.17.0.3		cdd705343b9b" >>/etc/hosts
echo -e "172.17.0.2      gitlab.example.com gitlab" >>/etc/hosts

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




