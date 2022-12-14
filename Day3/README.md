# Day 3

## Login to OpenShift from command line
<pre>
(jegan@tektutor.org)$ cat ~/openshift.txt 
RedHat OpenShift CLI Login URL
++++++++++++++++++++++++++++++
https://api.ocp.tektutor.org:6443 

RedHat OpenShift web-console 
++++++++++++++++++++++++++++
https://console-openshift-console.apps.ocp.tektutor.org 

Login Credentials
++++++++++++++++++++++++++++++++++++++
+ user    | kubeadmin		     +
+ password| 6ssow-q4UhD-82pCd-FE2XL  +
++++++++++++++++++++++++++++++++++++++
(jegan@tektutor.org)$ oc login -u kubeadmin https://api.ocp.tektutor.org:6443
Authentication required for https://api.ocp.tektutor.org:6443 (openshift)
Username: kubeadmin
Password: 
Login successful.

You have access to 66 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "jegan".
</pre>

## Creating NodePort external Service
Delete any existing service for nginx
```
oc delete svc/nginx
```

Expected ouptut
<pre>
(jegan@tektutor.org)$ oc expose deploy/nginx --type=NodePort --port=8080
service/nginx exposed
(jegan@tektutor.org)$ oc get svc
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nginx   NodePort   172.30.40.251   <none>        8080:32000/TCP   3s
(jegan@tektutor.org)$ oc describe svc/nginx
Name:                     nginx
Namespace:                jegan
Labels:                   app=nginx
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       172.30.40.251
IPs:                      172.30.40.251
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32000/TCP
Endpoints:                10.128.2.16:8080,10.129.1.8:8080,10.131.0.17:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
(jegan@tektutor.org)$ virsh list
 Id    Name                           State
----------------------------------------------------
 2     ocp-lb                         running
 10    ocp-master-1                   running
 11    ocp-master-2                   running
 12    ocp-master-3                   running
 13    ocp-worker-1                   running
 14    ocp-worker-2                   running

(jegan@tektutor.org)$ oc get nodes -o wide
NAME                        STATUS   ROLES           AGE   VERSION           INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                                                        KERNEL-VERSION                 CONTAINER-RUNTIME
master-1.ocp.tektutor.org   Ready    master,worker   28h   v1.23.5+012e945   192.168.122.245   <none>        Red Hat Enterprise Linux CoreOS 410.84.202207262020-0 (Ootpa)   4.18.0-305.57.1.el8_4.x86_64   cri-o://1.23.3-11.rhaos4.10.gitddf4b1a.1.el8
master-2.ocp.tektutor.org   Ready    master,worker   28h   v1.23.5+012e945   192.168.122.121   <none>        Red Hat Enterprise Linux CoreOS 410.84.202207262020-0 (Ootpa)   4.18.0-305.57.1.el8_4.x86_64   cri-o://1.23.3-11.rhaos4.10.gitddf4b1a.1.el8
master-3.ocp.tektutor.org   Ready    master,worker   28h   v1.23.5+012e945   192.168.122.70    <none>        Red Hat Enterprise Linux CoreOS 410.84.202207262020-0 (Ootpa)   4.18.0-305.57.1.el8_4.x86_64   cri-o://1.23.3-11.rhaos4.10.gitddf4b1a.1.el8
worker-1.ocp.tektutor.org   Ready    worker          28h   v1.23.5+012e945   192.168.122.101   <none>        Red Hat Enterprise Linux CoreOS 410.84.202207262020-0 (Ootpa)   4.18.0-305.57.1.el8_4.x86_64   cri-o://1.23.3-11.rhaos4.10.gitddf4b1a.1.el8
worker-2.ocp.tektutor.org   Ready    worker          28h   v1.23.5+012e945   192.168.122.197   <none>        Red Hat Enterprise Linux CoreOS 410.84.202207262020-0 (Ootpa)   4.18.0-305.57.1.el8_4.x86_64   cri-o://1.23.3-11.rhaos4.10.gitddf4b1a.1.el8
(jegan@tektutor.org)$ curl 192.168.122.245:32000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
(jegan@tektutor.org)$ curl 192.168.122.197:32000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
</pre>

## Let's clean up all resources by deleting our project and recreate the same project
```
oc delete project jegan
oc new-project jegan
```

## Understanding dry-run
```
oc create deploy nginx --image=bitnami/nginx:latest --replicas=3 --dry-run=client -o yaml
```
Expected output
<pre>
(jegan@tektutor.org)$ oc create deploy nginx --image=bitnami/nginx:latest --replicas=3 --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: bitnami/nginx:latest
        name: nginx
        resources: {}
status: {}
</pre>

## Creating a nginx deployment 
```
oc create deploy nginx --image=bitnami/nginx:latest --replicas=3 --dry-run=client -o yaml > nginx-deploy.yml
```

## Scale up nginx deployment from 3 Pods to 6 Pods using imperative style
```
oc scale deploy/nginx --replicas=6
```

Expected ouput
<pre>
(jegan@tektutor.org)$ <b>oc scale deploy/nginx --replicas=6</b>
deployment.apps/nginx scaled
(jegan@tektutor.org)$ oc get po
NAME                     READY   STATUS              RESTARTS   AGE
nginx-78644964b4-2w2rs   1/1     Running             0          4m33s
nginx-78644964b4-4j469   0/1     ContainerCreating   0          3s
nginx-78644964b4-hgtwm   1/1     Running             0          4m33s
nginx-78644964b4-swv9w   0/1     ContainerCreating   0          3s
nginx-78644964b4-vtvb9   0/1     ContainerCreating   0          3s
nginx-78644964b4-z4znv   1/1     Running             0          4m33s
</pre>

## Scale up nginx deploying from 6 Pods to 8 Pods using declarative style
Edit nginx-deploy.yml and update the replicas count from 6 to 8
```
oc apply -f nginx-deploy.yml
```

<pre>
(jegan@tektutor.org)$ vim nginx-deploy.yml 
(jegan@tektutor.org)$ oc apply -f nginx-deploy.yml 
deployment.apps/nginx configured
(jegan@tektutor.org)$ oc get po
NAME                     READY   STATUS              RESTARTS   AGE
nginx-78644964b4-2w2rs   1/1     Running             0          6m27s
nginx-78644964b4-4j469   1/1     Running             0          117s
nginx-78644964b4-fjhsc   0/1     ContainerCreating   0          3s
nginx-78644964b4-hgtwm   1/1     Running             0          6m27s
nginx-78644964b4-swv9w   1/1     Running             0          117s
nginx-78644964b4-vtvb9   1/1     Running             0          117s
nginx-78644964b4-wpsfs   0/1     ContainerCreating   0          3s
nginx-78644964b4-z4znv   1/1     Running             0          6m27s
</pre>

## Creating ClusterIP Service using declarative style
<pre>
(jegan@tektutor.org)$ <b>oc expose deploy/nginx --port=8080 --dry-run=client -o yaml > nginx-clusterip-svc.yml</b>
(jegan@tektutor.org)$ </b>ls</b>
<b>nginx-clusterip-svc.yml</b>  nginx-deploy.yml  README.md
(jegan@tektutor.org)$ oc get svc
No resources found in jegan namespace.
(jegan@tektutor.org)$ <b>oc apply -f nginx-clusterip-svc.yml</b>
service/nginx created
(jegan@tektutor.org)$ <b>oc get svc</b>
NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
nginx   ClusterIP   172.30.150.178   <none>        8080/TCP   5s
(jegan@tektutor.org)$ <b>oc describe svc/nginx</b>
Name:              nginx
Namespace:         jegan
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                172.30.150.178
IPs:               172.30.150.178
Port:              <unset>  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.128.0.111:8080,10.128.2.17:8080,10.128.2.18:8080 + 5 more...
Session Affinity:  None
Events:            <none>
</pre>

## Deleting the clusterip service using declarative style
<pre>
(jegan@tektutor.org)$ <b>oc get svc</b>
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
nginx   ClusterIP   172.30.150.49   <none>        8080/TCP   3s
(jegan@tektutor.org)$ <b>oc delete -f nginx-clusterip-svc.yml</b>
service "nginx" deleted
(jegan@tektutor.org)$ <b>oc get svc</b>
No resources found in jegan namespace.
</pre>

## Creating NodePort Service using declarative style
<pre>
(jegan@tektutor.org)$ <b>oc expose deploy/nginx --type=NodePort --port=8080 --dry-run=client -o yaml > nginx-nodeport-svc.yml</b>
(jegan@tektutor.org)$ <b>ls</b>
nginx-clusterip-svc.yml  nginx-deploy.yml  <b>nginx-nodeport-svc.yml</b>  README.md

(jegan@tektutor.org)$ <b>oc get svc</b>
No resources found in jegan namespace.
(jegan@tektutor.org)$ <b>oc apply -f nginx-nodeport-svc.yml</b>
service/nginx created
(jegan@tektutor.org)$ <b>oc get svc</b>
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nginx   NodePort   172.30.252.33   <none>        8080:31405/TCP   2s
(jegan@tektutor.org)$ <b>oc describe svc/nginx</b>
Name:                     nginx
Namespace:                jegan
Labels:                   app=nginx
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       172.30.252.33
IPs:                      172.30.252.33
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31405/TCP
Endpoints:                10.128.0.111:8080,10.128.2.17:8080,10.128.2.18:8080 + 5 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
</pre>

## Deleting NodePort service in declarative style
<pre>
(jegan@tektutor.org)$ <b>oc get svc</b>
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nginx   NodePort   172.30.252.33   <none>        8080:31405/TCP   2m30s
(jegan@tektutor.org)$ <b>oc delete -f nginx-nodeport-svc.yml</b>
service "nginx" deleted
(jegan@tektutor.org)$ <b>oc get svc</b>
No resources found in jegan namespace.
</pre>

## Creating LoadBalancer Service using declarative style
<pre>
(jegan@tektutor.org)$ <b>oc expose deploy/nginx --type=LoadBalancer --port=8080 --dry-run=client -o yaml > nginx-lb-svc.yml</b>
(jegan@tektutor.org)$ <b>ls</b>
nginx-clusterip-svc.yml  nginx-deploy.yml  <b>nginx-lb-svc.yml</b>  nginx-nodeport-svc.yml  README.md
</pre>

## Configuring MetalLB in OpenShift
Refer my medium blog
<pre>
https://medium.com/tektutor/using-metallb-loadbalancer-with-bare-metal-openshift-onprem-4230944bfa35
</pre>

## What is Persistent Volume(PV)?
- System Administrators can creates Volume of
   - sizes ( 1Gib, 2 Gib, 5 Gib, etc )
   - different types(storage class) ( aws EBS, NFS Storage, etc., )
   - Permission(access mode) - Whether all Pods in every node can access or All Pods in same Node alone access to storage
 
## What is Persistent Volume Claim(PVC)?
- Application Developers they can request the storage requirement for their applications by creating Persistent Volume Claim
- When PVC should mention
   - the access mode
   - size
   - storage class ( optional )
   
## OpenShift Storage Controller
- When you deploy an applcation that requires some PVC(storage)
- Storage Controller will search the OpenShift cluster for Persistent volumes that matches the requirements of PVC
- Things that should match between PV and PVC
    1. Size
    2. Access Mode
    3. Storage Class(if mentioned)
    4. Labels (if mentioned)


## Clone the TekTutor openshift repository if you haven't done already
```
cd ~
git clone https://github.com/tektutor/openshift-aug-2022.git
cd openshift-aug-2022
```

## Pull dela changes if you have already clone the TekTutor Openshift GitHub Repository
```
cd ~
cd openshift-aug-2022
git pull
```

## Creating the Persistent Volume
Make sure you find and replace 'jegan' with your name before deploying.

```
cd ~/openshift-aug-2022
git pull
cd Day3/mysql

oc apply -f mysql-pv.yml
```
The persistent volume manifest file content will look like below
<pre>
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-jegan 
  labels:
     name: jegan
spec:
  capacity:
    storage: 100Mi 
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /mnt/mysql
    server: 192.168.1.80
</pre>

In the above file, 
 - you need to replace 'jegan' with your name
 - replace path /mnt/mysql with /mnt/userxx
 - replace NFS Server IP with your RPS Lab Machine IP address.
 
In case your username is user05 then you should replace the path as /mnt/user05
 

## Creating the Persistent Volume claim
Make sure you find and replace 'jegan' with your name before deploying.
```
cd ~/openshift-aug-2022
git pull
cd Day3/mysql

oc apply -f mysql-pvc.yml
```

## Deploying mysql database
Make sure you find and replace 'jegan' with your name before deploying.
```
cd ~/openshift-aug-2022
git pull
cd Day3/mysql

oc apply -f mysql-deploy.yml
```

## Understanding Persistent Volume and Claim in a funny way
<pre>
https://youtube/YIQa-LNBcJA"/>
</pre>

## ??????????????? Lab - Deploying Wordpress and mysql multi-pod application

#### Clone the repo in case you haven't done already
```
cd ~
git clone https://github.com/tektutor/openshift-aug-2022.git
cd openshift-aug-2022
```

#### Pull the delta changes after you have cloned the repo
```
cd ~
cd openshift-aug-2022
git pull
```

Create the mysql persistent volume
```
cd Day3/wordpress

oc apply -f mysql-pv.yml
```

Create the mysql persistent volume claim
```
cd Day3/wordpress
oc apply -f mysql-pvc.yml
```

Deploy the mysql db server
```
cd Day3/wordpress
oc apply -f mysql-deployment.yml
```

Create clusterip service for mysql deployment
```
cd Day3/wordpress
oc apply -f mysql-service.yml
```

Create the wordpress persistent volume
```
cd Day3/wordpress
oc apply -f wordpress-pv.yml
```

Create the wordpress persistent volume claim
```
cd Day3/wordpress
oc apply -f wordpress-pvc.yml
```

Create the wordpress deployment
```
cd Day3/wordpress
oc apply -f wordpress-deployment.yml
```

Create the wordpress service
```
cd Day3/wordpress
oc apply -f wordpress-service.yml
```

Create the wordpress route
```
cd Day3/wordpress
oc apply -f wordpress-route.yml
```

## What is HELM?
- HELM is a package manager for Kubernetes/OpenShift Cluster
- with this tool, you can package/bundle your application as Charts
- Helm packaged application is called as Charts
- using this tool, you can install/uninstall/upgrade your application with easy commands

## Installing Helm in your system
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Checking Helm version after installing Helm
<pre>
(jegan@tektutor.org)$ helm version
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jegan/.kube/config
version.BuildInfo{Version:"v3.9.0", GitCommit:"7ceeda6c585217a19a1131663d8cd1f7d641b2a7", GitTreeState:"clean", GoVersion:"go1.17.5"}
</pre>

## Creating Custom wordpress Helm chart
```
cd ~/openshift-aug-2022
git pull
cd Day3/HELM

helm create wordpress
```

Expected output
<pre>
(jegan@tektutor.org)$ helm create wordpress
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jegan/.kube/config
Creating wordpress
(jegan@tektutor.org)$ tree wordpress/
wordpress/
????????? charts
????????? Chart.yaml
????????? templates
??????? ????????? deployment.yaml
??????? ????????? _helpers.tpl
??????? ????????? hpa.yaml
??????? ????????? ingress.yaml
??????? ????????? NOTES.txt
??????? ????????? serviceaccount.yaml
??????? ????????? service.yaml
??????? ????????? tests
???????     ????????? test-connection.yaml
????????? values.yaml

3 directories, 10 files
</pre>

## Creating wordpress helm chart
```
cd ~/openshift-aug-2022
git pull

cd Day3/HELM
helm package wordpress
```

Expected output
<pre>
(jegan@tektutor.org)$ <b>helm package wordpress</b>
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jegan/.kube/config
Successfully packaged chart and saved it to: /home/jegan/openshift-aug-2022/Day3/HELM/wordpress-0.1.0.tgz
(jegan@tektutor.org)$ <b>ls</b>
wordpress  <b>wordpress-0.1.0.tgz</b>
</pre>

## Deploying wordpress using Helm Chart that we created
```
cd ~/openshift-aug-2022
git pull

cd Day3/HELM
helm install wordpress wordpress-0.1.0.tgz
```

Expected output
<pre>
(jegan@tektutor.org)$ helm install wordpress wordpress-0.1.0.tgz 
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jegan/.kube/config
NAME: wordpress
LAST DEPLOYED: Wed Aug 10 18:18:52 2022
NAMESPACE: jegan
STATUS: deployed
REVISION: 1
TEST SUITE: None
</pre>

## List the helm charts installed into your OpenShift cluster
```
helm list
```

Expected output
<pre>
(jegan@tektutor.org)$ helm list
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jegan/.kube/config
NAME     	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART          	APP VERSION
wordpress	jegan    	1       	2022-08-10 18:18:52.224094827 +0530 IST	deployed	wordpress-0.1.0	1.16.0     
</pre>

## Testing the wordpress deployment
- Go to your OpenShift webconsole
- Switch to Developer view
- Switch to your project
- Select the topology
- Click the route upward arrow shown on the wordpress
- you should be able to access wordpress web page
