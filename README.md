## kops-kubernetes-cluster-configuration
## Jensen Kokou Edits & Adds 

## Setting up Kubernetes (K8s) Cluster on AWS Using KOPS

1.kops is a software use to create production ready k8s cluster in a cloud provider like AWS.

2. kOPS SUPPORTS MULTIPLE CLOUD PROVIDERS

3. Kops compete with managed kubernestes services like EKS, AKS and GKE

4. Kops is cheaper than the others.

5. Kops create production ready K8S.

6. KOPS create resources like: LoadBalancers, ASG, Launch Configuration, woker node Master node (CONTROL PLANE.

7. KOPS is IaaC

#!/bin/bash
## 1) Create Ubuntu EC2 instance in AWS

## 2a) create kops user
``` sh
 sudo adduser kops
 sudo echo "kops  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/kops
 sudo su - kops
 ```
 ##  2a) install AWSCLI using the apt package manager
  ```sh
 sudo apt install awscli -y 
 ```
 ## or 2b) install AWSCLI using the script below
 ```sh
 sudo apt update -y
 sudo apt install unzip wget -y
 sudo curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
 sudo apt install unzip python -y
 sudo unzip awscli-bundle.zip
 sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
 ```
## 3) Install kops software on an ubuntu instance by running the commands below:
 	sudo apt install wget -y
 	sudo wget https://github.com/kubernetes/kops/releases/download/v1.22.0/kops-linux-amd64
 	sudo chmod +x kops-linux-amd64
 	sudo mv kops-linux-amd64 /usr/local/bin/kops
 
## 4) Install kubectl kubernetes client if it is not already installed
```sh
 sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
 sudo chmod +x ./kubectl
 sudo mv ./kubectl /usr/local/bin/kubectl
```
## 5) Create an IAM role from AWS Console or CLI with the below Policies. 

	AmazonEC2FullAccess 
	AmazonS3FullAccess
	IAMFullAccess 
	AmazonVPCFullAccess

Then Attach IAM role to ubuntu server from Console Select KOPS Server --> Actions --> Instance Settings --> Attach/Replace IAM Role --> Select the role which
You Created. --> Save.

## 6) create an S3 bucket
## Execute the commands below in your KOPS control Server. use unique s3 bucket name. If you get bucket name exists error.
	1. aws s3 mb s3://class30kops <- Give Custom Name ( CANT EXIST ALREADY ) 
	2. aws s3 ls # to verify
	
 ## 6b) create an S3 bucket    
	Expose environment variable:
    # Add env variables in bashrc
    
       vi .bashrc
       
	# Give Unique Name And S3 Bucket which you created.
 
	export NAME=class30.k8s.local <- Cluster Name
 
	export KOPS_STATE_STORE=s3://class30kops <- S3 Bucket Name 
 
      source .bashrc  
	
### 7) Create sshkeys before creating cluster
```sh
    ssh-keygen
 ```

# 8) Create kubernetes cluster definitions on S3 bucket
```sh
kops create cluster --zones us-east-1a --networking weave --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 ${NAME}
```
# Explnation of Above) Explanation of Command in #8 
```sh

The command kops create cluster is used to create a Kubernetes cluster using Kops, a tool for managing Kubernetes clusters on cloud infrastructure. Let's break down the flags and options in this command:

--zones us-east-1a: This specifies the availability zone where your cluster's instances will be deployed. In this case, it's "us-east-1a," which is a specific region's zone.

--networking weave: This sets up the networking solution for your cluster. "Weave" is a popular networking addon for Kubernetes that facilitates communication between pods.

--master-size t2.medium: This defines the instance type for the Kubernetes master node. In this example, "t2.medium" is chosen as the instance type for the master node.

--master-count 1: Specifies that there should be 1 master node in the cluster. A single master is a common setup for smaller clusters.

--node-size t2.medium: Similar to the master node, this defines the instance type for the worker nodes in the cluster.

--node-count=2: Sets the number of worker nodes in the cluster to 2. These nodes run your applications.

${NAME}: This is a placeholder for the name you've chosen for your Kubernetes cluster. It will be replaced with the actual name when you execute the command.

In summary, this command creates a Kubernetes cluster in the specified availability zone with specific instance types for master and worker nodes, using the "Weave" networking solution. The cluster will have one master node and two worker nodes. The placeholder ${NAME} will be replaced with your chosen cluster name.
```


# 8b) copy the sshkey into your cluster to be able to access your kubernetes node from the kops server
```sh 
kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub
```
# 9) Initialize your kops kubernetes cluser by running the command below
```sh
kops update cluster ${NAME} --yes
```
# 10a) Validate your cluster(KOPS will take some time to create cluster ,Execute below commond after 3 or 4 mins)

kops validate cluster
	   
	   Suggestions:
 * validate cluster: kops validate cluster --wait 10m
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa ubuntu@api.class.k8s.local
 * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
 * read about installing addons at: https://kops.sigs.k8s.io/operations/addons.

## 10b - Export the kubeconfig file to manage your kubernetes cluster from a remote server. For this demo, Our remote server shall be our kops server 
```sh
 kops export kubecfg $NAME --admin
```
# Explantion of Above) Command explanation for #10b 
```sh
The command kops export kubecfg is used to export a Kubernetes configuration file (kubeconfig) that allows you to interact with your Kops-managed cluster using the kubectl command-line tool. Let's break down the flags and options in this command:

kops export kubecfg: This is the main command that tells Kops to export the kubeconfig.

$NAME: This refers to the name of your Kubernetes cluster. It's a placeholder that should be replaced with the actual name of your cluster.

--admin: This flag specifies that the exported kubeconfig should have administrative privileges. An admin kubeconfig allows you to perform cluster management tasks.

In simple terms, this command creates a kubeconfig file that contains the necessary information for connecting to and interacting with your Kops-managed Kubernetes cluster. The exported kubeconfig will grant you administrative privileges, so you can manage and configure the cluster using kubectl commands.

After running this command, you can use the exported kubeconfig with kubectl to perform tasks on your cluster as an administrator.
```
## 11a) To list nodes and pod to ensure that you can make calls to the kubernetes apiSAerver and run workloads
	  kubectl get nodes 

### 11b) Alternative you can ssh into your kubernetes master server using the command below and manage your cluster from the master
    sh -i ~/.ssh/id_rsa ubuntu@ipAddress
    ssh -i ~/.ssh/id_rsa ubuntu@18.222.139.125
    ssh -i ~/.ssh/id_rsa ubuntu@172.20.58.124

### 11b. Alternative, Enable PasswordAuthentication in the master server and assign passwd
```sh
sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo service sshd restart
sudo passwd ubuntu
```

### 11c) To list nodes

	  kubectl get nodes 
 
## 12) To Delete Cluster

   kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes  

   Example Command - kops delete cluster --name=juicekops.k8s.local --state=s3://juicekops --yes | With info added. 
   
====================================================================================================


13 # IF you want to SSH to Kubernetes Master or Nodes Created by KOPS. You can SSH From KOPS_Server

sh -i ~/.ssh/id_rsa ubuntu@ipAddress
ssh -i ~/.ssh/id_rsa ubuntu@3.90.203.23
  
``
