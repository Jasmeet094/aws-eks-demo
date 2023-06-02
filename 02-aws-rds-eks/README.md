## AWS EKS ~>> Create RDS and Use it with Kubernetes Service - External Name service

This demo will demonstrate how we can use kubernetes service called External Name Service to connect to a RDS db. We can use our RDS db which is managed by AWS to store our information related to users in it. Instead of using a mysql pod as a db , we have lots of options as we are now using a AWS managed DB. We are using Private NodeGroup  in this demo , means our ec2 instances will  be in private subnets and also our RDS will be in private subnet.

## Create a EKS Cluster 

 eksctl create cluster --name test-demo-eks --region us-west-2 --nodegroup-name test-ng --node-type t3.micro --nodes 2 --node-private-networking --managed

## First create a DB subnet  group in the same VPC as of EKS cluster and choose all private subnets. Also create a Security Group for RDS which will have Port 3306 open to all world(0.0.0.0/0) and Create a AWS RDS db of type - Free Tier with mysql db version, also select vpc , security group and database subnet group while creating rds which we created recently and  make note of DB name, Username and Password so that we can use these values as env vars in our deployment file. Use version 5.7 while creating RDS

## In mysql-external-name-service yaml file update the endpoint of the RDS db

## In secrets.yaml file encode new db password value

## update the value of RDS endpoint in service yaml file & Create the externalName service 

## Check connectivity to RDS with below command 

   kubectl run -it --rm --image=mysql:5.7.22 --restart=Never mysql-client -- mysql -h mysql.cgocd0aui2ue.us-west-2.rds.amazonaws.com -u dbadmin -pdbpassword11

## Create usermanagement deployment and service 

## Check Application URL - http://a5b527f216d014956882f1b41a2bf166-1556899445.us-west-2.elb.amazonaws.com/usermgmt/health-status

## Run commands/apis through Postman



While creating these yaml files make sure to pass correct env vars for RDS in user-management deployment file as per you  set Username and password while creating RDS.

## Yaml Files Explained 

secrets.yaml: `In this we are creating a secret for db password and using this secret as a env variable in mysql deployment`

usrmgmt-deployment.yaml: `This is user management pod deployment file which we can access through web. But to create , update and delete users in mysql  db we can use postman client`

usrmgmt-svc.yaml: `This is svc for user management pod of type loadbalancer`

mqsql-externalname-svc.yaml: `This file create a svc which will point to the RDS db`

