
# AWS EKS - Cluster Autoscaler

Cluster autoscaler is a component of Kubernetes that automatically adjusts the size of a cluster by adding or removing nodes based on the current resource demands of the running workloads. It helps ensure that your cluster has enough capacity to handle the workload efficiently while optimizing resource utilization.
 
 `The Metrics Server` is a component in Kubernetes that collects resource utilization metrics from the nodes and pods within the cluster. It plays a crucial role in supporting the Horizontal Pod Autoscaler (HPA) by providing the necessary metrics for autoscaling decisions.

[Metrics Server](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html)

[Cluster Autoscaler Image Version](https://github.com/kubernetes/autoscaler/releases)
## First Check if Worker Nodes have policy attached to the IAM role related to ASG if not attached below policy to the IAM Role attached to worker nodes.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeScalingActivities",
        "autoscaling:DescribeTags",
        "ec2:DescribeInstanceTypes",
        "ec2:DescribeLaunchTemplateVersions"
      ],
      "Resource": ["*"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup",
        "ec2:DescribeImages",
        "ec2:GetInstanceTypesFromInstanceRequirements",
        "eks:DescribeNodegroup"
      ],
      "Resource": ["*"]
    }
  ]
}
```
## After this go to ASG of EKS cluster and set minimum/desired numb to 2 and maximum to 5.


## In this Demo we are going to see working of the Cluster Autoscaler

### 1. Install Metrics Server in your EKS Cluster.

`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

### 2.Install  Cluster autoscaler 
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

```

### 3. Annotate cluster autoscaler deployment with below annotation
```
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
```



### 4. Now edit the deployment of the cluster autoscaler to add our cluster name as per below example. and also add last 2 more flags
```
   spec:
      containers:
      - command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/eksdemo1
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false

```

### 5. Set the Image of the deployment as per our cluster version 
```
kubectl -n kube-system set image deployment.apps/cluster-autoscaler cluster-autoscaler=us.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler:v1.16.5
```
### 6. Now check if pods related to cluster autoscaler are working fine
```
kubectl get pods -n kube-system
```

### 7. After this deploy demp app 
### 8. Now increase the replicas to 30


```
kubectl scale --replicas=30 deploy 
```

### 9. Now in few mins you can observe that nodes are getting added to the cluster.

### 10. To revert back sclae down the replicas to 1.
