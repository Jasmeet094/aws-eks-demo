##  AWS EKS - Network Load Balancer

### In AWS EKS we can create Network Load Balancer in 2 ways. One way is to use Legacy `Cloud Provider Load Balancer Controller` in which we have to create service of type loadbalancer and use this annotation `service.beta.kubernetes.io/aws-load-balancer-type: nlb` this way we can create a Network Load Balancer. But this way does nto provide us additonal functionality of Load balancers to use. 

### 2nd way to create a Network Load balancer is use same annotation but with value is `external` = `service.beta.kubernetes.io/aws-load-balancer-type: external`. This will create a Network Load Balancer and also this will be possible through if we have installed latest AWS Load  Balancer Controller (ingress) in our cluster. By using this we can use additional functions for our load balancer like use certificates ,route traffic to https and many  more. Like we create Application Load Balancer and use different annotations regarding load balancer , in this same way we need to use same annotations in service.


#### We need to create/install AWS Load  Balancer controller , external dns , so please use prvious README form another demos to create cluster and these 2 resources..

[Annotations related to Network Load Balancer](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/guide/service/annotations/)

[AWS Network Load Balancer](https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html)

### Steps to Follow Up :- 

1. First we need to create Ingress Class to specify the default controller which is ALB in this case by creating ingress Class
2. Create a certificate for Hosted zone and use ARN of certificate in annotation related to certificate
3. Create deployment and service file
4. You will see that an NLB is created with all settings we defined in annotations and also a DNS record with name `nlb.jasmeetdevops.com`is got registered in route 53.
5. After testing make sure to delete all  resources to save charges





