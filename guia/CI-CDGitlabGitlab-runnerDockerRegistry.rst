

CI/CD con Gitlab Gitlab-runner y con un Docker Registry
=============================================

https://github.com/cgomeznt/Docker/blob/master/guia/dockerizargitlabCentos7.rst

https://github.com/cgomeznt/Docker/blob/master/DeployRegistryServer.rst




image: registry:5000/nodejs
stages:
  - test
  - deploy

Test:
  stage: test
  tags:
  - shell-01
  script:
    - echo "write your test here...!!!"
    - echo "$CI_COMMIT_SHORT_SHA - $REPO_DEV - $CI_COMMIT_SHORT_SHA"
    - docker build -t $CI_COMMIT_SHORT_SHA .
    - docker image tag $CI_COMMIT_SHORT_SHA $REPO_DEV$CI_COMMIT_SHORT_SHA
    - docker push $REPO_DEV$CI_COMMIT_SHORT_SHA

Deploy:
  only:
    refs:
      - master
  stage: deploy
  tags:
    - shell-01
  script:
    - touch /tmp/prueba.txt
