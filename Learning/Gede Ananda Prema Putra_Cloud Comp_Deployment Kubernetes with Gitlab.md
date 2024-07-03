## Documentation Kubernetes CI/CD by Gede Ananda Prema Putra
---
For the repo you can access https://gitlab.com/menandap/cloud_kubernetes_ci_cd


# What is CI/CD Project
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
![index_image_google](/images/kk_1.png)

## 2. CI/CD Models

### a. Edit your Gitlab Project
After that you can clone the project in your local/pc. And then you can upload/push some file what we need for make this project. There are like this :
![index_image_google](/images/kk_2.png)
In this repo there are :
- Website files
- Configuration

For Website files you can add hmtl, css, js, images, and etc in ***One Folder*** in this case named ***website*** on the menandap's repo.

For Configuration files there are .gitlab-ci.yaml, .gitlab-deploy.sh  .

***.gitlab-ci.yml***

```yml
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
docker build -t menandap/kubernetesgitlab:$CI_COMMIT_BRANCH .
docker login --username=$DOCKER_USER --password=$DOCKER_PASS
docker push menandap/kubernetesgitlab:$CI_COMMIT_BRANCH
echo $CI_PIPELINE_ID

elif [ "$1" == "DEPLOY" ];then
echo "[*] Tag $WAKTU"
echo "[*] Deploy to production server in version $CI_COMMIT_BRANCH"
echo '[*] Generate SSH Identity'
HOSTNAME=`hostname` ssh-keygen -t rsa -C "$HOSTNAME" -f "$HOME/.ssh/id_rsa" -P "" && cat ~/.ssh/id_rsa.pub
echo '[*] Execute Remote SSH'
# ssh -i key.pem -o "StrictHostKeyChecking no" root@20.213.164.175 "sudo microk8s.kubectl delete -f service-nginx-nodeport.yaml"
ssh -i key.pem -o "StrictHostKeyChecking no" root@20.213.164.175 "sudo microk8s.kubectl create -f service-nginx-nodeport.yaml"
echo $CI_PIPELINE_ID
f
```
***Dockerfile***
```Dockerfile
FROM nginx:latest
COPY website/ /usr/share/nginx/html
```

***service-nginx-nodeport.yaml***
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: menandap
spec:
  replicas: 3
  selector:
    matchLabels:
      name: menandap
  template:
    metadata:
      name: menandap
      labels:
        name: menandap
    spec:
      containers:
        - name: menandap
          image: menandap/kubernetesgitlab:main
          ports:
            - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: menandap-service
spec:
  type: NodePort
  selector:
    name: menandap
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30003
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

And then ***fill data variables*** in your Gitlab Project by fill the username, password for docker, and vps's public key. You can go to ***Setting->CI/CD->Variables*** like this.
![index_image_google](/images/k2.png)

***Dont forget*** to copy ***service-nginx-nodeport.yaml*** file to your VPS in /root directory.

Before we running CI/CD with Kubernetes. We need to install Kubernetes in our VPS. In this case i use ***Microk8s***.
```bash
sudo snap install microk8s --classic
```

## 3. Deploy Kubernetets to VPS with CI/CD

With all component completed, we can run the pipline of the gitlab project. In this project we use shared runner because all thing is prepared by gitlab runner and ready to use.

![index_image_google](/images/kk_3.png)

You can run build and deploy by running the pipeline project. Go to ***CI/CD*** menu -> ***Pipeline***, and then click ***Run Pipeline*** buttton.

![index_image_google](/images/kk_4.png)

If everything work corrently we can see the pipelines run like this. The build and the deploy stage passed. You can see if build stage passed, there will a docker image builded by the runner and uplouded in DockerHub otomaticly.

![index_image_google](/images/kk_5.png)
 
If deploy stage passed, there will a proccess of the runner will conect to VPS and pull images from the DockerHub and then running the ***microk8s.kubectl services*** by using ***.yaml configuration file we create before (service-nginx-nodeport.yaml)***. In this file there will pull image from dockerhub and then crated kubectl service in port 30003.

![index_image_google](/images/kk_9.png)

![index_image_google](/images/kk_6.png)

After that you can check if kubernetes pod can be access in our VPS, you can accsess ***IP:port*** in this case 20.213.164.175:30003. So that all about making Kubernetes CI/CD on Gitlab :) 