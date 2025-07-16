# Installing and Configuring AWS EBS CSI Driver for Kubernetes Cluster with Dynamic Provisioning of EBS Volumes
This guide provides the steps to install and configure the AWS EBS CSI Driver on a Kubernetes cluster to enable the use of Amazon Elastic Block Store (EBS) volumes as persistent volumes.

## Prerequisites
- A running Kubernetes cluster.
- A Kubernetes version of 1.20 or greater.
- An AWS account with access to create an IAM user and obtain an access key and secret key.

## EKS cluster

================================================================== Setup Kubernetes using eksctl =================================================================
### Install AWS CLI

  sudo apt update
  sudo apt install python3-pip unzip
  sudo pip3 install awscli
  sudo ./aws/install
  aws --version
  aws configure

Access key: AKIAU6GD253
Secret access : nzlucqrMTuE/Zoz/Zy4B47PRI8RJ+

------------------------- 

### Installing kubectl

Refer--https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

$ sudo su
$ curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
$ ll , $ chmod +x ./kubectl  //Gave executable permisions
$ mv kubectl /bin   //Because all our executable files are in /bin
$ kubectl version --output=yaml

-------------------------------------------

### Installing  eksctl
 
Refer--https://github.com/eksctl-io/eksctl/blob/main/README.md#installation

$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ cd /tmp
$ ll
$ sudo mv /tmp/eksctl /bin
$ eksctl version

-------------------------------------------

eksctl create cluster --name=devops --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup

eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster devops --approve

eksctl create nodegroup \
  --cluster=devops \
  --region=us-east-1 \
  --name=devops-ng-private \
  --node-type=t3.medium \
  --nodes-min=1 \
  --nodes-max=1 \
  --node-volume-size=20 \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access \
  --node-private-networking
  
aws eks update-kubeconfig --region us-east-1 --name devops

AND

eksctl delete nodegroup --cluster=devops --region=us-east-1 --name=devops-ng-private

eksctl delete cluster --name=devops --region=us-east-1


----------------------------- or -------------------------

## Installing Helm
Helm is a package manager for Kubernetes that simplifies the deployment and management of applications.

- Run the following commands to install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Installing AWS EBS CSI Driver
- Create a secret to store your AWS access key and secret key using the following command:
```
kubectl create secret generic aws-secret \
    --namespace kube-system \
    --from-literal "key_id=${AWS_ACCESS_KEY_ID}" \
    --from-literal "access_key=${AWS_SECRET_ACCESS_KEY}"
```

- Add the AWS EBS CSI Driver Helm chart repository:
```
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm repo update
```

- Deploy the AWS EBS CSI Driver using the following command:
```
helm upgrade --install aws-ebs-csi-driver \
    --namespace kube-system \
    aws-ebs-csi-driver/aws-ebs-csi-driver
```

- Verify that the driver has been deployed and the pods are running:
```
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver
```

## Provisioning EBS Volumes
- Create a storageclass.yaml file and apply the storageclass.yaml file using the following command:
```
kubectl apply -f storageclass.yaml
```

- Create a pvc.yaml file and apply the pvc.yaml file using the following command:
```
kubectl apply -f pvc.yaml
```

- Create a pod.yaml file and apply the pod.yaml file using the following command:
```
kubectl apply -f pod.yaml
```

- Verify that the EBS volume has been provisioned and attached to the pod:
```
kubectl exec -it app -- df -h /data
```
The output of the above command should show the mounted EBS volume and its available disk space.

## Conclusion
In this guide, we have shown you how to install and configure the AWS EBS CSI Driver on your Kubernetes cluster and how to use it for dynamic provisioning of EBS volumes.
