##  AWS Load  Balancer Controller - Ingress - Host Based Routing

### In this Demo we are going to use ALB Scheme type as Internal which means now this alb will be route traffic internally to the services and should no be accessible from outside.Our Ingress file is very specific because we are using internal facing alb to route traffic internally to the services.

### We are creating a Curl pod to test the traffic internally and we will exec into the pod and access the endpoint of the Load Balancer with prefixes related to apps.

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



1. First we need to create Ingress Class to specify the default controller which is ALB in this case by creating ingress Class
2. We have to change the value of the scheme-type annotation to Internal
3. We will route traffic internally
4. This ALB will not be accesible from outside
5. We are using a curl pod to test the routing of traffic 
6. Now you can accsess the application using DNS record by adding prefix to the DNS name  /app1 , /app1 and / through curl pod





