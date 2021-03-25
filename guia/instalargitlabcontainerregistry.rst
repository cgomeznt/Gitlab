Instalar el Gitlab Container Registry Imagen
==============================


https://docs.gitlab.com/ee/administration/packages/container_registry.html

https://docs.docker.com/registry/deploying/


docker run -d -p 5000:5000 --restart=always --name registry registry:2



root@debian:~# docker run -d -p 5000:5000 --restart=always --name registry registry:2
Unable to find image 'registry:2' locally
2: Pulling from library/registry
e95f33c60a64: Pull complete 
4d7f2300f040: Pull complete 
35a7b7da3905: Pull complete 
d656466e1fe8: Pull complete 
b6cb731e4f93: Pull complete 
Digest: sha256:da946ca03fca0aade04a73aa94b54ff0dc614216bdd1d47585f97b4c1bdaa0e2
Status: Downloaded newer image for registry:2
1e107f084df7d87243b083b2831e76bb22519c7e076797d79858ecf4960470f1
root@debian:~# docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED             STATUS                       PORTS                                                          NAMES
1e107f084df7   registry:2                    "/entrypoint.sh /etc…"   8 seconds ago       Up 3 seconds                 0.0.0.0:5000->5000/tcp                                         registry
cdd705343b9b   gitlab/gitlab-runner:latest   "/usr/bin/dumb-init …"   40 minutes ago      Up 17 minutes                                                                               gitlab-runner
8b9b57a69c4f   gitlab/gitlab-ce:latest       "/assets/wrapper"        About an hour ago   Up About an hour (healthy)   0.0.0.0:22->22/tcp, 0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   gitlab
root@debian:~# docker images
REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
gitlab/gitlab-runner   latest    d7e996f04ec9   3 days ago     819MB
gitlab/gitlab-ce       latest    1139cc795e5f   7 days ago     2.19GB
debian                 latest    dc2eddc15825   13 days ago    114MB
hello-world            latest    d1165f221234   2 weeks ago    13.3kB
registry               2         5c4008a25e05   4 weeks ago    26.2MB
alpine                 latest    28f6e2705743   5 weeks ago    5.61MB
centos                 latest    300e315adb2f   3 months ago   209MB
root@debian:~# netstat -nat | grep -i listen
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:57115           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:36775           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:41227         0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:35595           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:41041           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:29754         0.0.0.0:*               LISTEN     
tcp6       0      0 :::39455                :::*                    LISTEN     
tcp6       0      0 :::2049                 :::*                    LISTEN     
tcp6       0      0 :::48073                :::*                    LISTEN     
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::34585                :::*                    LISTEN     
tcp6       0      0 :::59801                :::*                    LISTEN     
root@debian:~# docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED             STATUS                       PORTS                                                          NAMES
1e107f084df7   registry:2                    "/entrypoint.sh /etc…"   3 minutes ago       Up 3 minutes                 0.0.0.0:5000->5000/tcp                                         registry
cdd705343b9b   gitlab/gitlab-runner:latest   "/usr/bin/dumb-init …"   43 minutes ago      Up 21 minutes                                                                               gitlab-runner
8b9b57a69c4f   gitlab/gitlab-ce:latest       "/assets/wrapper"        About an hour ago   Up About an hour (healthy)   0.0.0.0:22->22/tcp, 0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   gitlab
root@debian:~# docker images
REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
gitlab/gitlab-runner   latest    d7e996f04ec9   3 days ago     819MB
gitlab/gitlab-ce       latest    1139cc795e5f   7 days ago     2.19GB
debian                 latest    dc2eddc15825   13 days ago    114MB
hello-world            latest    d1165f221234   2 weeks ago    13.3kB
registry               2         5c4008a25e05   4 weeks ago    26.2MB
alpine                 latest    28f6e2705743   5 weeks ago    5.61MB
centos                 latest    300e315adb2f   3 months ago   209MB
root@debian:~# docker tag centos:latest localhost:5000/my-centos
root@debian:~# docker push localhost:5000/my-centos
Using default tag: latest
The push refers to repository [localhost:5000/my-centos]
2653d992f4ef: Pushed 
latest: digest: sha256:dbbacecc49b088458781c16f3775f2a2ec7521079034a7ba499c8b0bb7f86875 size: 529
root@debian:~# docker images
REPOSITORY                 TAG       IMAGE ID       CREATED        SIZE
gitlab/gitlab-runner       latest    d7e996f04ec9   3 days ago     819MB
gitlab/gitlab-ce           latest    1139cc795e5f   7 days ago     2.19GB
debian                     latest    dc2eddc15825   13 days ago    114MB
hello-world                latest    d1165f221234   2 weeks ago    13.3kB
registry                   2         5c4008a25e05   4 weeks ago    26.2MB
alpine                     latest    28f6e2705743   5 weeks ago    5.61MB
centos                     latest    300e315adb2f   3 months ago   209MB
localhost:5000/my-centos   latest    300e315adb2f   3 months ago   209MB
root@debian:~# docker rmi 300e315adb2f
Error response from daemon: conflict: unable to delete 300e315adb2f (must be forced) - image is referenced in multiple repositories
root@debian:~# docker rmi localhost:5000/my-centos
Untagged: localhost:5000/my-centos:latest
Untagged: localhost:5000/my-centos@sha256:dbbacecc49b088458781c16f3775f2a2ec7521079034a7ba499c8b0bb7f86875
root@debian:~# docker images
REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
gitlab/gitlab-runner   latest    d7e996f04ec9   3 days ago     819MB
gitlab/gitlab-ce       latest    1139cc795e5f   7 days ago     2.19GB
debian                 latest    dc2eddc15825   13 days ago    114MB
hello-world            latest    d1165f221234   2 weeks ago    13.3kB
registry               2         5c4008a25e05   4 weeks ago    26.2MB
alpine                 latest    28f6e2705743   5 weeks ago    5.61MB
centos                 latest    300e315adb2f   3 months ago   209MB
root@debian:~# curl -X GET https://localhost:5000/v2/_catalog
curl: (35) error:1408F10B:SSL routines:ssl3_get_record:wrong version number
root@debian:~# curl -X GET http://localhost:5000/v2/_catalog
{"repositories":["my-centos"]}


Copy an image from Docker Hub to your registry
You can pull an image from Docker Hub and push it to your registry. The following example pulls the ubuntu:16.04 image from Docker Hub and re-tags it as my-ubuntu, then pushes it to the local registry. Finally, the ubuntu:16.04 and my-ubuntu images are deleted locally and the my-ubuntu image is pulled from the local registry.

Pull the ubuntu:16.04 image from Docker Hub.

$ docker pull ubuntu:16.04
Tag the image as localhost:5000/my-ubuntu. This creates an additional tag for the existing image. When the first part of the tag is a hostname and port, Docker interprets this as the location of a registry, when pushing.

$ docker tag ubuntu:16.04 localhost:5000/my-ubuntu
Push the image to the local registry running at localhost:5000:

$ docker push localhost:5000/my-ubuntu
Remove the locally-cached ubuntu:16.04 and localhost:5000/my-ubuntu images, so that you can test pulling the image from your registry. This does not remove the localhost:5000/my-ubuntu image from your registry.

$ docker image remove ubuntu:16.04
$ docker image remove localhost:5000/my-ubuntu
Pull the localhost:5000/my-ubuntu image from your local registry.

$ docker pull localhost:5000/my-ubuntu



For the latest (as of 2015-07-31) version of Registry V2, you can get this image from DockerHub:

docker pull distribution/registry:master
List all repositories (effectively images):

curl -X GET https://localhost:5000/v2/_catalog
> {"repositories":["redis","ubuntu"]}
List all tags for a repository:

curl -X GET https://myregistry:5000/v2/ubuntu/tags/list
> {"name":"ubuntu","tags":["14.04"]}
