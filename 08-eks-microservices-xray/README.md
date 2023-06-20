
# AWS EKS - Microservices with AWS X-RAY

In this demo we are going to use AWS X-Ray service with microservices running on AWS EKS cluster. Think of AWS X-Ray as a tool that helps you see inside your applications, much like an X-ray machine helps doctors see inside your body. It helps you gain visibility into the different components and services involved in processing a request, such as web servers, databases, and external APIs. In this demo we are specifically using AWS X-Ray service for API `https://xraydemo.jasmeetdevops.com/usermgmt/notification-xray` . When the user hit this URL , the first request will go to the user-app service and then it go to the notification service and in the end it give user back response from deployment/pod of notification app that it is running. All this request flow will be traced and visualized by the aws x-ray service. We can review all details regarding the request in aws xray.


### User-app Microservice
```
This microservice can create/delete/update users. 
When it received a request through user-app-service then 
it will create a user whose info will be store in AWS RDS using external-name service and also it will send notification to the notification-app that a user has been created. This app will use environment variables in deployment file regarding RDS ike database name , user name and db password Also it will use name and port of the notification service as environment variable. Additionally we are using environment variables for the xray service like tracing name and xray service address.
```

### Notification-app Microservice
```
When user got created by user-app then this app wil receive a notification from user-app that a user got created. Now this app will send a notification to user email address. This can do so by using SMTP service of AWS which is AWS Simple Email Service(AWS SES). We are using AWS SES service related info like server name , password  and from address as environment variables in the deployement of this service.Additionally we are using environment variables for the xray service like tracing name and xray service address.
```
[Image Tags for X-Ray](https://gallery.ecr.aws/xray/aws-xray-daemon)


### Steps to Create SMTP Credentials
```
1. Go to Amazon SES service and click on SMTP Settings
2. Click on Create SMTP credentials.
3.  Now make a not of the server address ,  username and password because we have to use this info in env variables in notification service.
4. Now click on Verfied identities & click on create Verfied identities.
5.  For Identity type choose email address and use any email address here.
6. Now you should have received a email on the above address to confirm/verify this email.
7. Now crete one more verified identity to use this email as `to address`. For example example1@gmail.com will receive mail from example2@gmail.com , this is called to address and from address in the sense.


```
### Create IAM Service account for X-Ray daemonset.
```
eksctl create iamserviceaccount \
    --name xray-daemon \
    --namespace default \
    --cluster dev-ops-test1 \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSXrayWriteOnlyAccess \
    --approve \
    --override-existing-serviceaccounts
```
### Create X-Ray DaemonSet

### Steps to follow Demo...

#### 1. Create RDS and choose VPC same as EKS Cluster and choose private subnets.
#### 2. Install AWS Load Balancer Controller and create ingress class.
#### 3. Create External DNS deployment.
#### 4. Create Notification-app service first.
#### 5. Now create external rds service using RDS enpoint as external name and then create User-app deployment and service and make sure to add env variables related to DB and notification svc in deployment file of user-app. Additionally add env variables related to xray service 
#### 6. Create smtp external name service first and add external name as smtp of that region and before creating notification deloy update env variables related to smtp user and passwords. And also in notification deployment add any email which we created earlier which you want to use as from address.Additionally add env variables related to xray service 
#### 7. In the end create ingress service.
#### 8. Monitor all resource if all are working as expected and application will be accessible at URL `https://xray.jasmeetdevops.com/usermgmt/notification-xray'
#### 9. Hit URL `https://xray.jasmeetdevops.com/usermgmt/notification-xray` many times and open X-Ray service amd monitor all the traces and segments ,you can see all the request flow there.
#### 10. After testing all things , make sure to remove/delete everything to save charges like iamserviceaccounts , pods of applications , ingress-service , external dns and alb load balancer Controller. Also delete SMTP server and credentials and RDS as well.

### Steps to check application in Postman Client..

1. Import the postman-file.json to postman client
2. create new environment with the url `https://xray.jasmeetdevops.com` which you are using in Route 53 for this app demo.
3. Choose the newly created environment and create new user and for user's email update 2nd email address which we created in first steps.
4.  You can see that user email will received a notification from (from email address), This is the working of this demp application.