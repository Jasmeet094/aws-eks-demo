##  AWS Load  Balancer Controller - Ingress - Context Path Based Routing

### In this demo we are going to use Aws Load balncer Controller - Ingress with Context based routing rules. AWS Load Balancer Controller Ingress is a Kubernetes Ingress Controller designed specifically for AWS load balancers. It enables automatic provisioning and configuration of ALBs and NLBs to route traffic to Kubernetes services.It integrates with Kubernetes ingress resources to define routing rules, SSL/TLS termination, and other load balancing configurations.The controller streamlines the deployment and management of AWS load balancers within Kubernetes clusters, enhancing scalability and high availability for applications.

## Create a EKS Cluster 

 ```
 eksctl create cluster --name test-demo-eks --region us-west-2 --nodegroup-name test-ng --node-type t3.micro --nodes 2 --node-private-networking --managed
```



## Steps to  Install Aws Load Balancer Controller

1. #### Create an IAM policy. Download an IAM policy for the AWS Load Balancer Controller that allows it to make calls to AWS APIs on your behalf.
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.7/docs/install/iam_policy.json
```

2. ####  Create an IAM policy using the policy downloaded in the previous step. If you downloaded iam_policy_us-gov.json, change iam_policy.json to iam_policy_us-gov.json before running the command.

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

4. #### Create an IAM role. Create a Kubernetes service account named aws-load-balancer-controller in the kube-system namespace for the AWS Load Balancer Controller and annotate the Kubernetes service account with the name of the IAM role


```
eksctl create iamserviceaccount \
  --cluster=my-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

5. #### Install AWS Load Balancer Controller using HELM

`helm repo add eks https://aws.github.io/eks-charts`

`helm repo update`
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=test-demo-3 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-west-2 \
  --set vpcId=vpc-0ee6a775330b191ab \
  --set image.repository=602401143452.dkr.ecr.us-east-1.amazonaws.com/amazon/aws-load-balancer-controller  
```

  [Region codes](https://docs.aws.amazon.com/eks/latest/userguide/add-ons-images.html)

[Annotations for Ingress](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.5/guide/ingress/annotations/)


#### Then you can verify the deployment 

  `kubectl -n kube-system get deployment aws-load-balancer-controller`


## Now Deploy yaml files to see working of ingress

1. First we need to create Ingress Class to specify the default controller which is ALB in this case by creating ingress Class
2. Then create all APPS deployments and services and use the names of services in ingress service yaml file under spec.
3. Now Create Ingress service Resource 
4. Now go to Route53 and under hosted zone create a record and choose alias as load balancer which is got created now
5. In Aws certificate manager create a certificate for hosted zone by declaring certificate as *.jasmeetdevops.com.
6. Now you can accsess the application using URL of the alb by adding prefix /app1 , /app1
and /

7. After testing make sure to delete all  resources to save charges




