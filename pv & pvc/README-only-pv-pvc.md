How to use Persistent Volume and Persistent Claims | Kubernetes
Aug 30, 2020
· 7 min read
 ·
 · Last Modified : Dec 8, 2020 Share on:
 Author : 
Rahul Wagh
Working with kubernetes is always fun as well as challenging. The more you dive deep into the kubernetes ecosystem the more you learn.

It always bugged me when I started working with kubernetes that - How can I retain the data after the end of pod life cycle?

Answer is -

Kubernetes Persistent Volume and Persistent claims help you to retain the data of the pod even after the end of the pod life cycle

What problems does it solve?
Containers running inside the pod can not share the files with each other.

All the files inside the container are temporary which means if you terminate the container you are going to lose all your files.
Secondly if in any case, your container crashes then there is no way to recover files.
Kuberenetes provides volume plugin as Persistent Volume to address the above problems.

The lifecycle of these volumes are independent of the lifecycle of pods.

So if PODs are terminated then volumes are unmounted and detached keeping the data intact.


How to use Persistent Volume and Persistent Claims




What is Persistent Volume(PV)?
In simple terms, it's storage available within your Kubernetes cluster. This storage can be provisioned by you or Kubernetes administrator.

It's basically a directory with some data in it and all the containers running inside the pods can access it. But Persistent Volumes are independent of the POD life cycle.

So if PODs live or die, persistent volume does get affected and it can be reused by some other PODs.

Kubernetes provides many volume plugins based on the cloud service provider you are using -

awsElasticBlockStore, azureDisk, azureFile, cephfs, cinder, configMap, csi, downwardAPI, emptyDir, fc (fibre channel), flexVolume, flocker, gcePersistentDisk, gitRepo (deprecated), glusterfs, hostPath, iscsi, local, nfs, persistentVolumeClaim, projected, portworxVolume, quobyte, rbd, scaleIO, secret, storageos, vsphereVolume



How can you Create persistent volume?
There are some prerequisites before you create your persistent volume

Step 1- Prerequisites
Kubernetes Cluster: - You should have Kubernetes cluster up and running (If you do not know "How to setup then please refer to this article)
Docker Image/container: - A docker image that can be deployed as Kubernetes deployment. (I am going to use Spring Boot Docker image. If you do not have then please refer on how to create docker image you can follow this article)
Directory for persistent Volume storage:- Create one directory .i.e. /home/vagrant/storage inside your Linux machine for persistent volume. You can keep the directory name and path as per your need
Once you are done with all the 3 prerequisites then your ready to proceed.

Step 2 - Create a persistent volume configuration (jhooq-pv.yml)
Its fairly easy to create it. Refer to the following configuration which you can customize with your requirements -

apiVersion: v1
kind: PersistentVolume
metadata:
  name: jhooq-demo-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /home/vagrant/storage
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node1
yaml
I saved the above configuration with name jhooq-pv.yml but you can assign any name of your choice.

In step 1 we have created Persistent Volume for Local Storage, now you need to apply the configuration.

Step 3 - Apply the configuration
kubectl apply -f jhooq-pv.yml
Bash
persistentvolume/jhooq-demo-pv created
Bash
Now you have created the persistent volume with the name "jhooq-demo-pv" inside the Kubernetes cluster



Step 4 - Lets check the status of PV
kubectl get pv
Bash
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
jhooq-demo-pv   1Gi        RWO            Retain           Available           local-storage            9s
Bash
Here is the screen of the status


create kubernetes persistent volume for local storage
Great now you have created PV, let's move ahead and create a Persistent Volume claim.



What is Persistent Volume claim?
Persistent volume provides you an abstraction between the consumption of the storage and implementation of the storage.

In the nutshell you can say its a request for storage on behalf of an application which is running on cluster.

How to use Persistent Volume claim(PVC) ?
If you as an application developer wants to use/access Persistent Volume(PV) then you must create a request for storage and it can be done by creating PVC objects.

Step 1 - Alright lets create your first Persistent Volume Claim(jhooq-pvc.yml) -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jhooq-pvc
spec:
  volumeName: jhooq-demo-pv
  storageClassName: local-storage
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
yaml
Save the above configuration with file name of your choice, in mycase I am saving this file with the name jhooq-pvc.yml

Step 2 - Now apply this configuration using the following command
kubectl create -f jhooq-pvc.yml
Bash
persistentvolumeclaim/jhooq-pvc created
Bash
Step 3 - Lets check our Persistent Volume and persistent Volume Claim
kubectl get pvc
Bash
vagrant@kmaster:~$ kubectl get pvc
NAME        STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS    AGE
jhooq-pvc   Bound    jhooq-demo-pv   1Gi        RWO            local-storage   17h
Bash

create persistent volume claim in your kubernetes cluster


Create a POD using Persistent Volume claim
Now in this step we are going to create a POD using the PV and PVC from the previous steps.

We are going to deploy Spring Boot Docker image but you can use any docker application of your choice. But if you do not have any docker image with you then you can refer to - How to deploy spring boot application in Kubernetes cluster

Step 1 - Create POD configuration yml .i.e. - "jhooq-pod.yml"
apiVersion: v1
kind: Pod
metadata:
  name: jhooq-pod-with-pvc
  labels:
    name: jhooq-pod-with-pvc
spec:
  containers:
  - name: jhooq-pod-with-pvc
    image: rahulwagh17/kubernetes:jhooq-k8s-springboot
    ports:
      - containerPort: 8080
        name: www
    volumeMounts:
      - name: www-persistent-storage
        mountPath: /home/vagrant/storage
  volumes:
    - name: www-persistent-storage
      persistentVolumeClaim:
        claimName: jhooq-pvc
yaml
One point you can note over here is we are using "persistentVolumeClaim" which is "jhooq-pvc" .

Step 2 : Lets apply the pod configuration
kubectl apply -f jhooq-pod.yml
bash
pod/jhooq-pod-with-pvc created
bash

Lets check the pod status

$kubectl get pod
bash
NAME                 READY   STATUS    RESTARTS   AGE
jhooq-pod-with-pvc   1/1     Running   0          8m37s
bash
Now you have deployed your pod successfully using the Persistent Volume and Persistent Volume Claim.

Step 3 : Test the microservice deployed under the POD
In this testing step we need to access the microservice which we deployed inside the POD.

To test the POD first we need to find the IP address on which it is running.

Use the following kubectl command to find the IP address of the POD

$ kubectl get pod -o wide
bash
It should return you with (You may get different IP address) -


NAME                 READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
jhooq-pod-with-pvc   1/1     Running   0          11m   10.233.90.3   node1   <none>           <none>
bash
You should note down the IP address of the POD because we are going to use that IP address for accessing the microservice.

$ curl 10.233.90.3:8080/hello
bash
Hello - Jhooq-k8s
bash
If you are using my docker-image then it should return a message "Hello - Jhooq-k8s" but you can use any docker image or docker container of your choice.

Pros and Cons
After using the persistent volume and persistent volume claim, I must say its always beneficial to use both when you are working in the production environment because you can not just delete the data after the end of the POD cycle.

The most favorable use case which I can see setting up the database such as MySQL, Oracle, Postgress inside the persistent volume so that it is always there irrespective of your POD life cycle.

But here I collected some of the Pros and Cons of using PV and PVC -

Pros
Storing and archiving the logs
Setting up the database
Useful for application handling a large number of batch jobs
Storing configs of application
Independent from PODs life-cycle
Easy to use
Easy to backup
Cons
Can not be used to store transnational data for performance-intensive application
You need to set up your own backup policies
Couldn't be used for HA(High Availability)
Conclusion
If you are reading this part of the blog post then I must say you have at least implemented PV and PVC inside your Kubernetes cluster. But to conclude it here is recap of what we did -

Gone through the concepts of "What is Persistent Volume and Persistent Volume Claim"
Then we created a Persistent Volume .i.e - jhooq-demo-pv with 1 Gi of storage
Created the Persistent Volume Claim .i.e. - jhooq-pvc to use persistent volume jhooq-demo-pv
Finally created the POD and deployed spring boot microservice.
I hope you learned something new from this post. If you interest in lap session then you can follow me on my YouTube channel


Learn more On Kubernetes -

Setup kubernetes on Ubuntu
Setup Kubernetes on CentOs
Setup HA Kubernetes Cluster with Kubespray
Setup HA Kubernetes with Minikube
Setup Kubernetes Dashboard for local kubernetes cluster
Setup Kubernetes Dashboard On GCP(Google Cloud Platform)
How to use Persistent Volume and Persistent Volume Claims in Kubernetes
Deploy Spring Boot Microservice on local Kubernetes cluster
Deploy Spring Boot Microservice on Cloud Platform(GCP)
Setting up Ingress controller NGINX along with HAproxy inside Kubernetes cluster
CI/CD Kubernetes | Setting up CI/CD Jenkins pipeline for kubernetes
kubectl export YAML | Get YAML for deployed kubernetes resources(service, deployment, PV, PVC....)
How to setup kubernetes jenkins pipeline on AWS?
Implementing Kubernetes liveness, Readiness and Startup probes with Spring Boot Microservice Application?
How to fix kubernetes pods getting recreated?
How to delete all kubernetes PODS?
How to use Kubernetes secrets?
Share kubernetes secrets between namespaces?
How to Delete PV(Persistent Volume) and PVC(Persistent Volume Claim) stuck in terminating state?
Delete Kubernetes POD stuck in terminating state?


Posts in this series
Kubernetes Cheat Sheet for day to day DevOps operations?
Delete Kubernetes POD stuck in terminating state?
How to Delete PV(Persistent Volume) and PVC(Persistent Volume Claim) stuck in terminating state?
Share kubernetes secrets between namespaces?
How to use Kubernetes secrets?
How to delete all kubernetes PODS?
kubernetes pods getting recreated?
Implementing Kubernetes liveness, Readiness and Startup probes with Spring Boot Microservice Application?
kubectl export yaml OR How to generate YAML for deployed kubernetes resources
Kubernetes Updates
CI/CD Kubernetes | Setting up CI/CD Jenkins pipeline for kubernetes
Kubernetes cluster setup with Jenkins
How to use Persistent Volume and Persistent Claims | Kubernetes
How to fix ProvisioningFailed persistentvolume controller no volume plugin matched
Fixing – Cannot bind to requested volume: storageClasseName does not match
Fixing – pod has unbound immediate persistentvolumeclaims or cannot bind to requested volume incompatible accessmode
How to fix kubernetes dashboard forbidden 403 error – message services https kubernetes-dashboard is forbidden User
How to fix Kubernetes – error execution phase preflight [preflight]
Deploy Spring Boot microservices on kubernetes?
How to fix – ansible_memtotal_mb minimal_master_memory_mb
How to use kubespray – 12 Steps for Installing a Production Ready Kubernetes Cluster
How to setup kubernetes on CentOS 8 and CentOS 7
How to fix – How to fix - ERROR Swap running with swap on is not supported. Please disable swap
14 Steps to Install kubernetes on Ubuntu 20.04(bento/ubuntu-20.04), 18.04(hashicorp/bionic64)
Kubernetes Dashboard | Kubernetes Admin GUI | Kubernetes Desktop Client
Install Kubernetes with Minikube
