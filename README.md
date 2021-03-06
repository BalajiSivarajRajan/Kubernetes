# Kubernetes

Kubernetes is an open source system for managing containerized applications across multiple hosts, providing basic mechanisms for deployment, maintenance, and scaling of applications.


In this article, I am going to setup a kubernetes cluster on AWS cloud environment up & running with dashboard. I am going to use a tool called KOPS for the cluster setup. 

What is KOPS?
kops is an opinionated provisioning system with 

- Fully automated installation
- Uses DNS to identify clusters
- Self-healing: everything runs in Auto-Scaling Groups
- Limited OS support (Debian preferred, Ubuntu 16.04 supported, early support for CentOS & RHEL)
- High-Availability support
- Can directly provision, or generate terraform manifests

Requirements
An Ubuntu or Debian instance with latest updates, AWS-CLI, S3 bucket, Hosted Zone on Route 53 and domain. I am going to use Ubuntu instance to launch my cluster. 

Installation:
Launch an AWS EC2 Ubuntu instance and update to latest packages 
	$sudo apt-get update
	$sudo apt-get -y upgrade

Download the latest version of kops and install it

	$ wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64 
	$ chmod +x kops
	$ sudo mv kops /usr/local/bin/

Download the latest version of the kubectl

	 $ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
	$ chmod +x kubectl
	$ sudo mv kubectl /usr/local/bin/

Installing AWS Cli to access the AWS from command line. 

  $ sudo apt-get install python-pip
  $ pip install --upgrade pip
  $ sudo pip install awscli 
	
On AWS console, create a new IAM user (for example kops) with full access and save the access keys as it would be used to config the AWS Cli. Copy both AWS access key ID and AWS secret access key. 

On EC2 instance, 
  - configure the newly created AWS IAM user
	$aws configure 
		AWS Access Key ID [None]: <AWS user access key ID >
		AWS Secret Access Key [None]: <AWS secret access key>
		Default region name [None]:
		Default output format [None]:
	Please enter the AWS access key ID & Secret Access Key for the newly created IAM user. Default region & output format can be left bank.
  - generate key pair for AWS EC2 user. It will be used to connect to the kubernetes cluster that we are to create. In my case its ubuntu user. 
	$ ssh-keygen 
	
Create a domain for the cluster
kops uses DNS for discovery, inside the cluster and to reach the kubernetes API server from client. Its should be vaild DNS name. You can use subdomain and I recommed to use a subdomain for the cluster configuration. 

You can use an existing domain or register a new domain. In this example, I am going to a domain hosted on dot.tk which is free domain provider. 

Domain registration on dot.tk / freenom.com:
I have registred a new domain on freenom.com with the name k8sclustersetup.tk. 

<<screen shot here >>

On AWS console, create a new Hosted zone on router 53. Login to AWS console, navigate to router53 DNS management and create new Hosted Zone. Its adviseable to create a subdomain. This creates set of name servers. Copy the name server details which will start with ns-xxx.awsdns-xx.com, ns-xxx.awsdns-xx.co.uk, ns-xxx.awsdns-xx.org, ns-xxx.awsdns-xx.net. These NS values has to be updated on domain service provider. In my case I have updated the NS details.

<< screen shot here >>

S3 bucket to store the cluster state

kops uses S3 to store the cluster details like configuration, keys, etcs. Create a new S3 bucket - kopsclusterdemo.


<<Screen shot here>>
	
Cluster creation:

$ kops create cluster --name=k8sclustersetup.tk --state=s3://kopsclusterdemo --zones=eu-west-2a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=k8sclustersetup.tk

$kops update cluster k8sclustersetup.tk --yes --state=s3://kopsclusterdemo

The above command creates one master and two node cluster with all three nodes on t2.micro instance. 

You can list the cluster details by issuing the below command
$kops get cluster --state=s3://kopsclusterdemo

NAME                  CLOUD   ZONES
k8sclustersetup.tk    aws     eu-west-2a

To list the node
$kubectl get node

NAME                                         STATUS    ROLES     AGE       VERSION
ip-xxx-xx-xx-xx.eu-west-2.compute.internal   Ready     node      2m        v1.8.7
ip-xxx-xx-xx-xx.eu-west-2.compute.internal   Ready     node      2m        v1.8.7
ip-xxx-xx-xx-xx.eu-west-2.compute.internal   Ready     master    2m        v1.8.7



