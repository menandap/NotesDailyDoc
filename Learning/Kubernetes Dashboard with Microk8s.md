# Member of Team 4
- Salman Abdul Jabbaar Wiharja
- Himansyah Muqorrobin
- Tio Hady Pranoto
- Benjamin Nikholas Partomuan
- Gede Ananda Prema Putra

# What is Kubernetes Dashboard
The Kubernetes Dashboard is a web-based management interface that enables you to: deploy and edit containerized applications. assess the status of containerized applications. troubleshoot containerized applications.

# Using Mircrok8s
The standard Kubernetes Dashboard is a convenient way to keep track of the
activity and resource use of MicroK8s.

On all platforms, you can install the dashboard with one command:

## a. Enable the Dashboard
```bash
microk8s enable dashboard
```
After addons Mircok8s Dashboard enable, you can verify like this. You can see in this image addons for dahsboard is enabled
![index_image_google](/images/d1.png) 

## b. Create Token
To access the installed dashboard, you’ll need to follow the guide. To log in to the Dashboard, you will need the access token (unless RBAC has
also been enabled). This is generated randomly on deployment, so a few commands
are needed to retrieve it:

For MicroK8s 1.24 or newer

```bash
microk8s kubectl create token default
```

For MicroK8s 1.23 or older

```bash
token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
microk8s kubectl -n kube-system describe secret $token
```
In this case the VPS using Mircok8s with newer version, and then creating the token like this.
![index_image_google](/images/d2.png) 

## c. Port Forward
Next, you need to connect to the dashboard service. While the MicroK8s snap will
have an IP address on your VPS (the Cluster IP of the kubernetes-dashboard service),
you can also reach the dashboard by forwarding its port to a free one on your host with :

```bash
microk8s kubectl port-forward -n kube-system service/kubernetes-dashboard --address <IP ADD> 10443:443
```
![index_image_google](/images/d3.png) 

## d. Kubernetes Dashboard 
After that you can acces the ***IP:Port*** for access the Kubernetes Dashboard. In this case we using ***128.199.142.219:10443***.

![index_image_google](/images/d4.png) 
In image above we need to verify using the token we created before to access Kubernetes Dashboard. After sumbiting the token we can access the Kubernetes Dashboard like this.

![index_image_google](/images/d5.png) 
![index_image_google](/images/d6.png) 

So that all about making Dasboard on Kubernetes :) 
