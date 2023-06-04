##  AWS Load  Balancer Controller - Ingress - SSL Redirect

### In this demo we are going to register a DNS name in ROUTE53 , i have already created a hosted zone wih name `jasmeetdevops.com` and then will create a AWS Certificate for that DNS name (*.jasmeetdevops.com). By adding SSL related annotations in Ingress service then it will route the traffic http to https. After creating all  manifest files , we have register a new record set for our DNS name in hosted zone and for Alias we have to choose ALB url. This will give us locked URL and our traffic will be secured by TLS/SSL. We only have to add annotations regarding SSL in our ingress service resource to make ssl redirect working.

#### I will be using/creating DNS Record set for this Demo = `ingress.jasmeetdevops.com`

### Create a EKS Cluster 
 ```
 eksctl create cluster --name test-demo-eks --region us-west-2 --nodegroup-name test-ng --node-type t3.micro --nodes 2 --node-private-networking --managed
```

### Steps to  Install Aws Load Balancer Controller

1. #### Create an IAM policy. Download an IAM policy for the AWS Load Balancer Controller that allows it to make calls to AWS APIs on your behalf.
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.7/docs/install/iam_policy.json
```

2. #### Create an IAM policy using the policy downloaded in the previous step. If you downloaded iam_policy_us-gov.json, change iam_policy.json to iam_policy_us-gov.json before running the command.

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

3. #### Create a IAM OIDC provider
 ```
 eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluter-name> \
    --approve
 ```

3. #### Create an IAM role. Create a Kubernetes service account named aws-load-balancer-controller in the kube-system namespace for the AWS Load Balancer Controller and annotate the Kubernetes service account with the name of the IAM role

```

eksctl create iamserviceaccount \
  --cluster=my-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

```

4. #### Install AWS Load Balancer Controller using HELM

`helm repo add eks https://aws.github.io/eks-charts`

`helm repo updatehelm repo update`

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller 
  --set region=
  --set vpcId=
  --set image.repository=<account>.dkr.ecr.<region-code>.amazonaws.com/amazon/aws-load-balancer-controller

```

  [Region codes](https://docs.aws.amazon.com/eks/latest/userguide/add-ons-images.html)

[Annotations for Ingress](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.5/guide/ingress/annotations/)


 #### Then you can verify the deployment 

  `kubectl -n kube-system get deployment aws-load-balancer-controller`


## Now Deploy yaml files to see working of ingress

1. First we need to create Ingress Class to specify the default controller which is ALB in this case by creating ingress Class
2. Then create all APPS deployments and services and use the names of services in ingress service yaml file under spec.
3. Create a aws certificate for hosted zone as *.jasmeetdevops.com and add ARN in annotation of certificate in ingress service 
4. Then create ingress service resource.
5. When the load balancer is in active state then create a record set in hosted zone and choose alias as load balancer
   
6. While testing you will see that this will redirect traffic from HTTP to HTTPS and will use aws certificate 
7. Now you can accsess the application using DNS record by adding prefix to the DNS name as follows
```
# HTTP URLs
http://ssl-redirect.jasmeetdevops.com/app1
http://ssl-redirect.jasmeetdevops.com/app2
http://ssl-redirect.jasmeetdevops.com/

# HTTPS URLs
https://ssl-redirect.jasmeetdevops.com/app1
https://ssl-redirect.jasmeetdevops.com/app2
https://ssl-redirect.jasmeetdevops.com/


```

8. After testing make sure to delete all  resources to save charges


