## Preview 
In my final project for project i will make a ***DevSecOps system***. In there will be a ***ci cd pipeline that will integrate with a gitlab, jenkins, docker, kubernetes, dan other pentesting tools, also ChatGpt will be used for assist the project process***. To make this project we need some tools, so there are the tools.

#### Some Tools That Will be Used
```
2 Vps : Server to a deployment and Running Build Project
Gitlab and the Repository : for Repository a File in the Project.
Jenkins : for Running the CI-CD Pipiline in this Project.
Docker and DockerHub : for making Image/Artefact for the Project, dan Repository of the Image. 
Kubernetes : for Container that will Running Image for Deployment Process.
Pentesing Tools : for Checking Vulnerability in Security of Project, that will be SCA, SAST, DAST Process.
ChatGPT : ChatGPT will Integrated for Help and Assist to Fixing the Vulnerability that will be Found.
```

#### Model of Proecess

![index_image_google](/images/doc_2.png) 
In the image above will see a some fase of process. That process will be used in the ci-cd pipilne, also security testing will be checked in the process above.

## Preparation
So in this topic will show u what that will be prepared for the project. 

#### Github
Github in this project will be like Repository that will be storing all file for website and the configuration. 

#### Docker
Docker will be used for making a container that will running the images. 
![index_image_google](/images/doc_5.png) 
``` bash
Installing the Docker

sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt install docker.io
sudo systemctl status docker
```

#### Jenkins
![index_image_google](/images/doc_3.png) 
Jenkins is main part for this project to making ci-cd pipeline. In the process Jenkins will be running on Docker Container. 

```
docker volume create jenkins-data
docker run -d -p 8080:8080 -v jenkins-data:/var/jenkins_home --name jenkins-container jenkins/jenkins

stop and start
docker stop jenkins-container
docker start jenkins-container

# buat docker volume backup
docker run --rm -v jenkins-data:/data -v $(pwd):/backup busybox tar czf /backup/jenkins-data-backup.tar.gz /data

#buat docker container backup (optional)
docker export jenkins-container > jenkins-container-backup.tar 

# import docker container (optional)
docker import jenkins-container-backup.tar

# import docker volume
sudo mkdir -p /var/lib/docker/volumes/jenkins-data/_data
sudo tar -xzvf jenkins-volume-backup.tar.gz -C /var/lib/docker/volumes/jenkins-data/_data/ --strip-components=1
sudo docker volume create jenkins-data

# setting permission
sudo chmod -R 777 /var/lib/docker/volumes/jenkins-data/_data

# run with backup docker volume
docker run -d -p 8080:8080 -v jenkins-data:/var/jenkins_home --name jenkins-container jenkins/jenkins

# something permission (optional)
sudo chown -R 1000:1000 $(sudo docker inspect -f '{{ range .Mounts }}{{ if eq .Destination "/var/jenkins_home" }}{{ .Source }}{{ end }}{{ end }}' <JENKINS_CONTAINER_ID>)

```

### Install Java JDK for runner
```
sudo yum install java-11-openjdk-devel
```

![index_image_google](/images/doc_6.png) 

#### Kubernetes
![index_image_google](/images/doc_7.png) 
Kuberntes will be used for making container for the deployment process. 

```
sudo snap install microk8s --classic
sudo microk8s status --wait-ready

# for not using sudo
sudo usermod -a -G microk8s $USER  
sudo chown -f -R $USER ~/.kube

sudo microk8s enable dashboard dns registry istio (optional)
sudo microk8s kubectl get all --all-namespaces
```

### Setting password GCP

Login to the Linux environment and then edit the text file found at:

```
/etc/ssh/sshd_config
```

Look for the line which reads:

```
PasswordAuthentication no
```

and change it to

```
PasswordAuthentication yes
```

Save the file.

Finally, restart SSH using:

1. If your OS is Ubuntu/Debian:

```
sudo service ssh restart
```

2. If your OS is CentOS/RedHat:

```
sudo service sshd restart
```

At this point, you will now be able to login using SSH using a userid/password pair.  
To set the password for `$USER`, do:

```bash
sudo passwd $USER
```


### Kubernetes
service-nodeport.yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: menandap
spec:
  replicas: 2
  selector:
    matchLabels:
      app: menandap
  template:
    metadata:
      name: menandap
      labels:
        app: menandap
    spec:
      containers:
        - name: menandap
          image: menandap/CI_PROJECT_NAME:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080

---

apiVersion: v1
kind: Service
metadata:
  name: menandap-service
spec:
  type: NodePort
  selector:
    app: menandap
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30000
```


### sonarqube (admin, admin default login)
docker-compose.yml

```
version: '3'

services:
  sonarqube:
    image: sonarqube:latest
    container_name: sonarqube
    ports:
      - "9000:9000" # SonarQube web interface
    environment:
      - SONARQUBE_JDBC_URL=jdbc:postgresql://sonarqube-db:5432/sonar
      - SONARQUBE_JDBC_USERNAME=sonar
      - SONARQUBE_JDBC_PASSWORD=password
    networks:
      - sonarnet
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_logs:/opt/sonarqube/logs
      - sonarqube_extensions:/opt/sonarqube/extensions

  sonarqube-db:
    image: postgres:latest
    container_name: sonarqube-db
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=sonar
    networks:
      - sonarnet
    volumes:
      - postgres_data:/var/lib/postgresql/data
networks:
  sonarnet:

volumes:
  sonarqube_data:
  sonarqube_logs:
  sonarqube_extensions:
  postgres_data:

```
### FIX SONARQUBE
root@server-main:~# cat docker-compose.yml 

version: "3"
services:
  sonarqube:
    image: sonarqube:community
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"
volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:


## BACKUP
docker run --rm -v root_sonarqube_data:/volume -v $(pwd):/backup busybox tar -czf /backup/sonarqube_data_backup.tar.gz -C /volume .
docker run --rm -v root_sonarqube_extensions:/volume -v $(pwd):/backup busybox tar -czf /backup/sonarqube_extensions_backup.tar.gz -C /volume .
docker run --rm -v root_sonarqube_logs:/volume -v $(pwd):/backup busybox tar -czf /backup/sonarqube_logs_backup.tar.gz -C /volume .

## RESTORE
docker volume create sonarqube_data
docker volume create sonarqube_extensions
docker volume create sonarqube_logs

docker run --rm -v root_sonarqube_data:/volume -v $(pwd):/backup busybox tar -xzf /backup/sonarqube_data_backup.tar.gz -C /volume

## BASH 

docker volume create --name sonarqube_data
docker volume create --name sonarqube_extensions
docker volume create --name sonarqube_logs

docker run \
    -v root_sonarqube_data:/opt/sonarqube/data \
    -v root_sonarqube_extensions:/opt/sonarqube/extensions \
    -v root_sonarqube_logs:/opt/sonarqube/logs \
    --name="sonarqube" -p 9000:9000 sonarqube:community



