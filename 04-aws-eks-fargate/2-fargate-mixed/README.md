##  AWS EKS - FARGATE ==> Mixed Workloads (Deploying resouces on Managed Nodes and Fargate)

### AWS EKS (Elastic Kubernetes Service) Fargate is a compute engine for Amazon Elastic Kubernetes Service (EKS) that allows you to run containers without having to provision and manage the underlying infrastructure. It provides a serverless experience for running containers on Kubernetes. In this demo we are creating a simple demo  application which will be deploy on Fargate. In Fargate  1 Pod = 1 Node , so suppose in our deployment we have declared replicas = 3 then there will be 3 nodes get deployed. We need to  keep in mind while using Fargate profiles in EKS that we have to deploy/create resources in the namespace which is declared in Fargate profile yaml file. So in the end app1 related resources will deployed on managed nodes (ec2-instances) and app2 and ums app resoucres will be deployed on fargate.


#### I will be using/creating DNS Record set for this Demo 
```
app1.jasmeetdevops.com
app2.jasmeetdevops.com
ums.jasmeetdevops.com
```

### Create a EKS Cluster 
 ```
 eksctl create cluster --name test-demo-5 --region us-west-2 --nodegroup-name test-ng --node-type t3.micro --nodes 2 --node-private-networking --managed
```
### Create a Fargate Profile using `fargate-profile.yaml`

`kubectl apply -f fargate-profile.yaml`


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
    --cluster test-demo-5 \
    --approve
 ```

4. #### Create an IAM role. Create a Kubernetes service account named aws-load-balancer-controller in the kube-system namespace for the AWS Load Balancer Controller and annotate the Kubernetes service account with the name of the IAM role

```

eksctl create iamserviceaccount \
  --cluster=test-demo-5 \
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
  --set clusterName=test-demo-5 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \ 
  --set region=us-west-2 \
  --set vpcId=vpc-03e0386e74528e0a8 \
  --set image.repository=602401143452.dkr.ecr.us-west-2.amazonaws.com/amazon/aws-load-balancer-controller 

```

[Region codes](https://docs.aws.amazon.com/eks/latest/userguide/add-ons-images.html)

[Annotations for Ingress](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.5/guide/ingress/annotations/)

[Fargate Information](https://eksctl.io/usage/fargate-support/)


### Steps to install External DNS

1. #### Create a IAM Policy so that Service account of external dns get access to Route53
  
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:ListResourceRecordSets"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

2. Use this policy to create an IAM role and service account for external dns:

```
eksctl create iamserviceaccount \
    --name external-dns \
    --namespace default \
    --cluster test-demo-5 \
    --attach-policy-arn arn:aws:iam::307436399520:policy/AllowExternalDns_eks \
    --approve \
    --override-existing-serviceaccounts
```

3. #### Then create `External DNS` by applying `external-dns.yaml`

4. #### You should see that external  dns deployment will running having SA attached which we created earlier.




 #### Then you can verify the deployment 

  `kubectl get deployment | grep -i external`


## Now Deploy yaml files to see working of External Dns

1. First we need to create Ingress Class to specify the default controller which is ALB in this case by creating ingress Class
2. Create namespace `app2` & `ums`
3. Then Create External-dns resource with service account and IAM policy.
4. You can declare different hostname/dns name for any service under spec as shown in ingress-service yaml file.
5. All Apps will have their differnet ingress resources and ALB and DNS
6. Then create all APPS deployments and services and use the names of services in ingress service yaml file under spec.
7. Create ACM certificate for hosted zone and then Create Ingress Service resource 
8. You should see a dns record named `fargate.jasmeetdevops.com` got created in Route 53
9. This will redirect traffic from HTTP to HTTPS and will use aws certificate automatically as we didn't  defined the annotaion related to aws certificate arn.

10. After testing make sure to delete all  resources to save charges



