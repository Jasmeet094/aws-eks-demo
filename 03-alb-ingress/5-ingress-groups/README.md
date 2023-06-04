##  AWS Load  Balancer Controller - Ingress - Host Based Routing

### This Demo explains about the working of Ingress-Groups. In previous demos we are maintaining a single ingress yaml file for all our 3 apps. It is good if we have to manage 3-5 apps but when the counting of apps is increased then it will be problem for us to  handle 1 single ingress yaml file as it will become bigger and bigger. So we have concept in ingress that we can create a different ingress files for all apps adn use a single Load Balance for all apps. That is what we going to create in this demo. We will create different ingress files for all 3 apps and in all ingress yaml files we will use different ingress resource names but for the Load balancer name and the Host name we will use similar name.  In this way we can manage different apps using 1 single load balancer.

#### I will be using/creating DNS Record set for this Demo = `ingress-groups.jasmeetdevops.com`

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

### Steps to install External DNS

1. #### Create a IAM Policy so that Servvice account of external dns get access to Route53
  
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
    --cluster eksdemo1 \
    --attach-policy-arn arn:aws:iam::180789647333:policy/AllowExternalDNSUpdates \
    --approve \
    --override-existing-serviceaccounts
```

3. #### Then create `External DNS` by applying `external-dns.yaml`

4. #### You should see that external  dns deployment will running having SA attached which we created earlier.




 #### Then you can verify the deployment 

  `kubectl get deployment | grep -i external`


## Now Deploy yaml files to see working of External Dns

1. First we need to create Ingress Class to specify the default controller which is ALB in this case by creating ingress Class
2. Then Create External-dns resource.
3. You have to declare different ingress resource names for  all ingress yaml files for 3 apps and have to use same name for the load balancer and host name.
4. Also we are adding 2 new annotation regarding ingress groups in all ingress yaml files so that all ingress resources falls under 1 common Load balancer.
5. Then create all APPS deployments and services and use the names of services in ingress service yaml file under spec.
6. This will redirect traffic from HTTP to HTTPS and will use aws certificate 
7. You should see a dns record named `external-dns.jasmeetdevops` got created in Route 53.
8. Now you can accsess the application using DNS record by adding prefix to the DNS name  /app1 , /app1 and /.




