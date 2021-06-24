export GITLAB_HOME="/home/cgomeznt/srv/gitlab"

docker run --detach \
  --hostname gitlab.dominio.local \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'https://gitlab.dominio.local'" \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --network app \
  gitlab/gitlab-ce:latest

==========================================

docker run --detach \
  --hostname gitlab.dominio.local \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab.dominio.local \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --network app \
  gitlab/gitlab-ce:latest

==========================================

docker run -dti \
--hostname gitlab.example.com \
--publish 443:443 \
--publish 80:80 \
--publish 22:22 \
--publish 5000:5005 \
--name gitlab \
--shm-size=4g \
--restart=on-failure \
--volume $GITLAB_HOME/config:/etc/gitlab \
--volume $GITLAB_HOME/logs:/var/log/gitlab \
--volume $GITLAB_HOME/data:/var/opt/gitlab --privileged gitlab-ce:1.0 /usr/sbin/init


--env GITLAB_OMNIBUS_CONFIG="external_url 'http://my.domain.com/'; gitlab_rails['lfs_enabled'] = true;" \

Notes:
Default admin account has been configured with following details:
Username: root
Password: You didn't opt-in to print initial root password to STDOUT.
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.

NOTE: Because these credentials might be present in your log files in plain text, it is highly recommended to reset the password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

docker exec -ti gitlab cat /etc/gitlab/initial_root_password
Password: lGACdQsozt7VrX391OIvohE5if44njlNOFZMHniCtr4=


==========================================


export GITLAB_RUNNER_HOME="/home/cgomeznt/srv/gitlab-runner"

docker run -d --name gitlab-runner.dominio.local --restart always \
    --hostname gitlab-runner.dominio.local \
    -v $GITLAB_RUNNER_HOME:/var/run/docker.sock \
    -v $GITLAB_RUNNER_HOME:/etc/gitlab-runner \
    --network app \
    gitlab/gitlab-runner:latest


docker run --rm -ti \
    -v $GITLAB_RUNNER_HOME:/var/run/docker.sock \
    -v $GITLAB_RUNNER_HOME:/etc/gitlab-runner \
    --network app \
    gitlab/gitlab-runner:latest register



==========================================

la imagen de gitlab es Ubunto focal
En el contenedor de gitlab se debe instalar docker y gitlab-runner



