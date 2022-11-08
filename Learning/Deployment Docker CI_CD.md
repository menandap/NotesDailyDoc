# Member of Team 4
- Salman Abdul Jabbaar Wiharja
- Himansyah Muqorrobin
- Tio Hady Pranoto
- Benjamin Nikholas Partomuan
- Gede Ananda Prema Putra

# Documentation of CI/CD Project
What is CI/CD? That stand for Continuous integration (CI)/Continuous Deployment(CD). List what procced of deployment with Gitlab CI/CD.
- Make a Gitlab Project
- Make a CI/CD Pipline
- Deploy to a Server with CI/CD 

## 1. Gitlab Project
Before you create a project, you need an Gitlab account first. After that you can click the button ***"new project"*** in right corner.
![index_image_google](/images/g_1a.png)
After that you can select ***"Create Blank Project"***
![index_image_google](/images/g_1b.png)
Fill all data and then click ***"create project"***
![index_image_google](/images/g_2.png)

## 2. CI/CD Models

### a. Edit your Gitlab Project
After that you can clone the project in your local/pc. And then you can upload/push some file what we need for make this project. There are :
- Website files
- Configuration

For Website files you can add hmtl, css, js, images, and etc in ***One Folder*** in this case named ***website*** on the menandap's repo.

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
ssh -i key.pem -o "StrictHostKeyChecking no" DevOps@52.139.198.230 "docker login --username=$DOCKER_USER --password=$DOCKER_PASS"
ssh -i key.pem -o "StrictHostKeyChecking no" DevOps@52.139.198.230 "docker pull menandap/belajardevops:$CI_COMMIT_BRANCH"
# ssh -i key.pem -o "StrictHostKeyChecking no" DevOps@52.139.198.230 "docker stop belajardevops-$CI_COMMIT_BRANCH"
# ssh -i key.pem -o "StrictHostKeyChecking no" DevOps@52.139.198.230 "docker rm belajardevops-$CI_COMMIT_BRANCH"
ssh -i key.pem -o "StrictHostKeyChecking no" DevOps@52.139.198.230 "docker run -d -p 3003:80 --restart always --name belajardevops-$CI_COMMIT_BRANCH menandap/belajardevops:$CI_COMMIT_BRANCH"
# ssh -i key.pem -o "StrictHostKeyChecking no" DevOps@52.139.198.230 "docker exec farmnode-main sed -i 's/farmnode_staging/farmnode/g' /var/www/html/application/config/database.php"
echo $CI_PIPELINE_ID
fi

```
***Dockerfile***
```Dockerfile
FROM nginx:latest
COPY website/ /usr/share/nginx/html
```

### b. Configure your Server
In your server, you need to configure public key so in the project can login with the ***.pem*** key

```bash 
ssh-keygen -t rsa -b 2048
```

After that you can check if the key was maked by see in **.ssh** dir, in there will have 3 files authorized_key, id_rsa, and id_rsa.pub.

![index_image_google](/images/g_3.png)

***note*** : if there no authorized_keys, you can make it.
```bash
touch authorized_keys
cat id_rsa.pub > authorized_keys
```
Now u can see the id_rsa files, that contain the private key ***(id_rsa)*** that make u can login the server. Copy key inside id_rsa and then save and name it ***key.pem***.

## 3. Deploy to Server with CI/CD

With all component completed, we can run the pipline of the gitlab project. In this project we use shared runner because all thing is prepared by gitlab runner and ready to use.

![index_image_google](/images/g_4.png)

You can run build and deploy by running the pipeline project. Go to ***CI/CD*** menu -> ***Pipeline***, and then click ***Run Pipeline*** buttton.

![index_image_google](/images/g_5.png)

If everything work corrently we can see the pipelines run like this. The build and the deploy stage passed.

![index_image_google](/images/g_6.png)

You can see if build stage passed, there will a docker image builded by the runner and uplouded in DockerHub otomaticly.

![index_image_google](/images/g_7.png)
 
If deploy stage passed, there will a proccess of the runner will conect to VPS and pull images from the DockerHub and then running the cointainer of the images we build before.

![index_image_google](/images/g_8.png)

So that all about making CI/CD on Gitlab :) 