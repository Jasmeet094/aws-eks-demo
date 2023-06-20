
# MicroServices Architecture on AWS EKS

In this Demo we are going to use microservice architecture. 
Microservices are a software architecture approach where applications are built as a collection of small, independent services that communicate with each other through APIs. Each service focuses on a specific business capability and can be developed, deployed, and scaled independently. This modular approach enables flexibility, scalability, and easier maintenance of complex systems.

### User-app Microservice
```
This microservice can create/delete/update users. 
When it received a request through user-app-service then 
it will create a user whose info will be store in AWS RDS using external-name service and also it will send notification to the notification-app that a user has been created. This app will use environment variables in deployment file regarding RDS ike database name , user name and db password Also it will use name and port of the notification service as environment variable.
```
### Notification-app Microservice
```
When user got created by user-app then this app wil receive a notification from user-app that a user got created. Now this app will send a notification to user email address. This can do so by using SMTP service of AWS which is AWS Simple Email Service(AWS SES). We are using AWS SES service related info like server name , password  and from address as environment variables in the deployement of this service.
```

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

### Steps to follow Demo...

#### 1. Create RDS and choose VPC same as EKS Cluster and choose private subnets.
#### 2. Install AWS Load Balancer Controller and create ingress class.
#### 3. Create External DNS deployment.
#### 4. Create Notification-app service first.
#### 5. Now create external rds service using RDS enpoint as external name and then create User-app deployment and service and make sure to add env variables related to DB and notification svc in deployment file of user-app. 
#### 6. Create smtp external name service first and add external name as smtp of that region and before creating notification deloy update env variables related to smtp user and passwords. And also in notification deployment add any email which we created earlier which you want to use as from address.
#### 7. In the end create ingress service.
#### 8. Monitor all resource if all are working as expected and application will be accessible at URL `microservice.jasmeetdevops.com/usermgmt/health-status`
#### 9. After testing all things , make sure to remove/delete everything to save charges like iamserviceaccounts , pods of applications , ingress-service , external dns and alb load balancer Controller. Also delete SMTP server and credentials and RDS as well.

### Steps to check application in Postman Client...

1. Import the postman-file.json to postman client
2. create new environment with the url `https://microservices.jasmeetdevops.com` which you are using in Route 53 for this app demo.
3. Choose the newly created environment and create new user and for user's email update 2nd email address which we created in first steps.
4.  You can see that user email will received a notification from (from email address), This is the working of this demp application.