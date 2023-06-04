##  AWS Load  Balancer Controller - Ingress - External  DNS

### In Our previous demo  we have created our DNS record set manually in Route 53.  By using `External DNS` resource in kubernetes we can create DNS records in Route 53 automatically by adding just a annotation in our ingress service resource.  External DNS will deploy as a deployment and it will create/delete/manage dns records for us. It will also use same certificate which we have mentioned in ssl annotations so that our traffic will be https

#### I will be using/creating DNS Record set for this Demo = `external-dns.jasmeetdevops.com`

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

`helm repo updatehelm repo update`

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=test-demo-3 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \ 
  --set region=us-west-2 \
  --set vpcId=vpc-0ee6a775330b191ab \
  --set image.repository=602401143452.dkr.ecr.us-west-2.amazonaws.com/amazon/aws-load-balancer-controller  

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
2. Then Create External-dns resource with service account and IAM policy.
3. Then create all APPS deployments and services and use the names of services in ingress service yaml file under spec.
4. Create ingress service resource
5. You should see a dns record named `external-dns.jasmeetdevops` got created in Route 53.
6. This will redirect traffic from HTTP to HTTPS and will use aws certificate 
7. Now you can accsess the application using DNS record by adding prefix to the DNS name :
```
# HTTPS URLs
https://external-dns.jasmeetdevops.com/app1
https://external-dns.jasmeetdevops.com/app2
https://external-dns.jasmeetdevops.com/


```

8. After testing make sure to delete all  resources to save charges





