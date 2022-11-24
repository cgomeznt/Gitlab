stages:
  - deployDev
  - deployQa
  - deployPro

deployDev:
  stage: deployDev
  script:
    - ssh -t -t gitlab-runner@10.134.0.210 "sudo rm -Rf /opt/SMIAPI/core/.git/"
    - scp -r ./  gitlab-runner@10.134.0.210:/opt/SMIAPI/core
    - ssh -t -t gitlab-runner@10.134.0.210 "sudo supervisorctl restart smicore-api"
  tags:
    - shell
  only:
    - develop

deployQa:
  stage: deployQa
  script:
    - ssh -t -t gitlab-runner@10.133.0.82 "sudo rm -Rf /opt/SMIAPI/core/.git/"
    - scp -r ./  gitlab-runner@10.133.0.82:/opt/SMIAPI/core
    - ssh -t -t gitlab-runner@10.133.0.82 "sudo supervisorctl restart smicore-api"
  tags:
    - shell
  only:
    - staging
    
deployPro:
  stage: deployPro
  script:
    - ssh -t -t gitlab-runner@10.133.0.75 "sudo rm -Rf /opt/SMIAPI/core/.git/"
    - scp -r ./  gitlab-runner@10.133.0.75:/opt/SMIAPI/core
    - ssh -t -t gitlab-runner@10.133.0.75 "sudo supervisorctl restart smicore-api"
  tags:
    - shell
  only:
  - master
