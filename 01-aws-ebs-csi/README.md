## AWS EKS - EBS CSI Driver Working

This demo explains about the working of AWS EBS CSI driver. 
EBS CSI (Elastic Block Store Container Storage Interface) driver is a Kubernetes Container Storage Interface (CSI) driver provided by AWS for managing Amazon Elastic Block Store (EBS) volumes in Kubernetes clusters. We are creating Private NodeGroup in this demo.

## Create a EKS Cluster 

 `eksctl create cluster --name test-demo-eks --region us-west-2 --nodegroup-name test-ng --node-type t3.micro --nodes 2 --node-private-networking --managed`



=================================================================================================================================
### Steps to Install EBS CSI Driver

#### 1. Attach a IAM Policy to worker node IAM role - EbsCsiDriver policy
    
    `ARN of Policy =  arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy`

#### 2. Install EBS CSI driver using Helm : (If helm is not installed , install it first:- curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash )

Helm

~ 1. Add the aws-ebs-csi-driver Helm repository.

     helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver

~ 2. Update Helm repository

     helm repo update

~ 3. Install the latest release of the driver.

     helm upgrade --install aws-ebs-csi-driver --namespace kube-system aws-ebs-csi-driver/aws-ebs-csi-driver
    
=================================================================================================================================
## Check the working of application in Postman Client 

1. Install the postman client
2. Import postman.json to postman client
3. create new environment and paste the URL/DNS name of the CLB 
4. Select the created environment form right top and select imported file from left side and select first api which is         UserManagement-HealthStatus 
5. You can now create ,  update delete user using all APIs
6. You can check user which you created using postman by accessing myswl db directly by using below command

     kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -u root -pdbpassword11

7. Use below myswl commands 

 mysql> show schemas;
 mysql> use usermgmt;
 mysql> show tables;
 mysql> select * from users;

==================================================================================================================================
## CHECK application URL =  http://a5b527f216d014956882f1b41a2bf166-1556899445.us-west-2.elb.amazonaws.com/usermgmt/health-status
=================================================================================================================================


We are creating a demo usermanagement application here. In which  we can create/delete/update users in mysql database.


=================================================================================================================================

mysql-configmap.yaml: `In this yaml file we are creating a config map which will be used by mysql db to create a db named mysql on startup`

mysql-deployment.yaml: `This file will create a deployment wuth 1 replica for mysql database. Which will use pvc to create a volume in AWS`

mysql-pvc.yaml: `This is the pvc we are attaching to mysql pod. This will create a 4 gb volume in AWS which  we can manage by using EBS CSI driver`

mysql-service.yaml: `This is the file for mysql db service of clusterIp type to access mysql  pod internally`

secrets.yaml: `In this we are creating a secret for db password and using this secret as a env variable in mysql deployment`

storageclass.yaml: `This is storage class file which  will use provisioner of ebs csi`

usrmgmt-deployment.yaml: `This is user management pod deployment file which we can access through web. But to create , update and delete users in mysql  db we can use postman client`

usrmgmt-svc.yaml: `This is svc for user management pod of type loadbalancer`


## Apply all the yaml files and when all pod will in running state you can check the status of pvc will be in bound state and there will be a volume created in AWS of size 4gb.