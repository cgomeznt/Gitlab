image: 10.134.0.252:4443/ccrnodes

stages:
  - install
  - test
  - registerDev
  - registerQa
  - registerPro
  - deployGitlab
  - deployDev
  - deployQa
  - deployPro
  - deployPiloto

install:
  stage: install
  script:
    - npm config set proxy http://10.132.0.10:8080
    - npm config set strict-ssl false
    - npm install
  tags:
    - docker
  artifacts:
    expire_in: 1h
    paths:
      - node_modules/
  cache:
    paths:
      - node_modules/

registerDev:
  stage: registerDev
  dependencies:
    - install
  script:
    - sudo docker build -t $CI_PROJECT_NAME$CI_PIPELINE_ID .
    - sudo docker image tag $CI_PROJECT_NAME$CI_PIPELINE_ID $REPO_DEV$CI_PROJECT_NAME$CI_PIPELINE_ID
    - sudo docker push $REPO_DEV$CI_PROJECT_NAME$CI_PIPELINE_ID
    #- sudo docker run --name $CI_PROJECT_NAME$CI_PIPELINE_ID -p PORT_APP:PORT_CONTAINER -d $REPO_DEV$CI_PROJECT_NAME$CI_PIPELINE_ID
  tags:
    - shell
  only:
    - develop

deployDev:
  stage: deployDev
  dependencies:
    - registerDev
  script:
    - ssh -t -t $USER@$HOSTMASTER "sudo sh /home/jsuarez/service_script.sh 31201"
    - ssh -t -t $USER@$HOSTMASTER "sudo kubectl create deployment $CI_PROJECT_NAME$CI_PIPELINE_ID --image=$REPO_DEV$CI_PROJECT_NAME$CI_PIPELINE_ID"
    - ssh -t -t $USER@$HOSTMASTER "sudo kubectl create service nodeport $CI_PROJECT_NAME$CI_PIPELINE_ID --tcp=$PORT_APP:$PORT_APP --node-port=$PORT_KUBERNETES"
  tags:
    - shell
  only:
    - develop

registerQa:
  stage: registerQa
  dependencies:
    - install
  script:
    - sudo docker build -t $CI_PROJECT_NAME$CI_PIPELINE_ID .
    - sudo docker image tag $CI_PROJECT_NAME$CI_PIPELINE_ID $REPO_QA$CI_PROJECT_NAME$CI_PIPELINE_ID
    - sudo docker push $REPO_QA$CI_PROJECT_NAME$CI_PIPELINE_ID
    #- sudo docker run --name $CI_PROJECT_NAME$CI_PIPELINE_ID -p PORT_APP:PORT_CONTAINER -d $REPO_DEV$CI_PROJECT_NAME$CI_PIPELINE_ID
  tags:
    - shell
  only:
    - qa

deployQa:
  stage: deployQa
  dependencies:
    - registerQa
  script:
    - ssh -t -t $USER@$HOSTMASTERQA "sudo sh /home/jsuarez/service_script.sh 31201"
    - ssh -t -t $USER@$HOSTMASTERQA "sudo kubectl create deployment $CI_PROJECT_NAME$CI_PIPELINE_ID --image=$REPO_QA$CI_PROJECT_NAME$CI_PIPELINE_ID"
    - ssh -t -t $USER@$HOSTMASTERQA "sudo kubectl create service nodeport $CI_PROJECT_NAME$CI_PIPELINE_ID --tcp=$PORT_APP:$PORT_APP --node-port=$PORT_KUBERNETES"
  tags:
    - shell
  only:
    - qa

registerPro:
  stage: registerPro
  dependencies:
    - install
  script:
    - sudo docker build -t $CI_PROJECT_NAME$CI_PIPELINE_ID .
    - sudo docker image tag $CI_PROJECT_NAME$CI_PIPELINE_ID $REPO_PROD$CI_PROJECT_NAME$CI_PIPELINE_ID
    - sudo docker push $REPO_PROD$CI_PROJECT_NAME$CI_PIPELINE_ID
    #- sudo docker run --name $CI_PROJECT_NAME$CI_PIPELINE_ID -p PORT_APP:PORT_CONTAINER -d $REPO_DEV$CI_PROJECT_NAME$CI_PIPELINE_ID
  tags:
    - shell
  only:
    - master

deployProd:
  stage: deployPro
  dependencies:
    - registerPro
  script:
    - ssh -t -t $USER@$HOSTMASTERQA "sudo sh /home/jsuarez/service_script.sh 31201"
    - ssh -t -t $USER@$HOSTMASTERQA "sudo kubectl create deployment $CI_PROJECT_NAME$CI_PIPELINE_ID --image=$REPO_PROD$CI_PROJECT_NAME$CI_PIPELINE_ID"
    - ssh -t -t $USER@$HOSTMASTERQA "sudo kubectl create service nodeport $CI_PROJECT_NAME$CI_PIPELINE_ID --tcp=$PORT_APP:$PORT_APP --node-port=$PORT_KUBERNETES"
  tags:
    - shell
  only:
    - master

registerPro:
  stage: registerPro
  dependencies:
    - install
  script:
    - sudo docker build -t $CI_PROJECT_NAME$CI_PIPELINE_ID .
    - sudo docker image tag $CI_PROJECT_NAME$CI_PIPELINE_ID $REPO_PROD$CI_PROJECT_NAME$CI_PIPELINE_ID
    - sudo docker push $REPO_QA$CI_PROJECT_NAME$CI_PIPELINE_ID
    #- sudo docker run --name $CI_PROJECT_NAME$CI_PIPELINE_ID -p PORT_APP:PORT_CONTAINER -d $REPO_DEV$CI_PROJECT_NAME$CI_PIPELINE_ID
  tags:
    - shell
  only:
    - master

deployPro:
  stage: deployPro
  dependencies:
    - registerPro
  script:
    - ssh -t -t $USER@$HOSTMASTERPROD "sudo sh /home/jsuarez/service_script.sh 31201"
    - ssh -t -t $USER@$HOSTMASTERPROD "sudo kubectl create deployment $CI_PROJECT_NAME$CI_PIPELINE_ID --image=$REPO_PROD$CI_PROJECT_NAME$CI_PIPELINE_ID"
    - ssh -t -t $USER@$HOSTMASTERPROD "sudo kubectl create service nodeport $CI_PROJECT_NAME$CI_PIPELINE_ID --tcp=$PORT_APP:$PORT_APP --node-port=$PORT_KUBERNETES"
  tags:
    - shell
  only:
  - master


deployPiloto:
  stage: deployPiloto
  dependencies:
    - registerDev
  script:
    - ssh -t -t $USER@$HOSTMASTERQA "sudo sh /home/jsuarez/service_script.sh 31301"
    - ssh -t -t $USER@$HOSTMASTERQA "sudo kubectl create deployment $CI_PROJECT_NAME$CI_PIPELINE_ID --image=$REPO_DEV$CI_PROJECT_NAME$CI_PIPELINE_ID"
    - ssh -t -t $USER@$HOSTMASTERQA "sudo kubectl create service nodeport $CI_PROJECT_NAME$CI_PIPELINE_ID --tcp=$PORT_APP:$PORT_APP --node-port=31301"
  tags:
    - shell
  only:
    - release/piloto

#deployGitlab:
#  stage: deployGitlab
# # dependencies:
# #   - build
#  script:
#    - sudo docker build -t  $CI_PROJECT_NAME$CI_PIPELINE_ID .
#    - sudo docker run --name $CI_PROJECT_NAME$CI_PIPELINE_ID -p $PORT_CONTAINER:$PORT_APP -d $CI_PROJECT_NAME$CI_PIPELINE_ID
#  tags:
#    - shell
#  except: ['master']
#  except: ['develop']

#tests:
#  stag  e: test
#  variables:
#    CHROME_BIN: google-chrome    
#  dependencies:
#    - install
#  before_script:
#    - apt-get update && apt-get install -y apt-transport-https
#    - wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
#    - sh -c 'echo "deb https://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'
#    - apt-get update && apt-get install -y google-chrome-stable
#  script: 
#    - npm run test:ci
#  coverage: '/Statements.*?(\d+(?:\.\d+)?)%/'  
