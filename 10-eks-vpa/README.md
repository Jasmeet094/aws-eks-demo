
# AWS EKS - Vertical Pod Autoscaler (VPA)

In Kubernetes, you run your applications in containers called "pods." Each pod requires a certain amount of resources like CPU and memory to function properly. Sometimes, you might allocate more resources than necessary, which is inefficient, or allocate too few resources, leading to poor performance.
That's where VPA comes in. VPA continuously monitors the resource usage of your application pods and analyzes their needs. It helps you automatically adjust the allocated resources based on the observed usage patterns, ensuring optimal performance and resource utilization.
 
 `The Metrics Server` is a component in Kubernetes that collects resource utilization metrics from the nodes and pods within the cluster. It plays a crucial role in supporting the Horizontal Pod Autoscaler (HPA) by providing the necessary metrics for autoscaling decisions.

[Metrics Server](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html)

## In this Demo   we are going to see working of the VPA.

### 1. Install Metrics Server in your EKS Cluster.

`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

### 2. Clone the git repo to install VPA.
`git clone https://github.com/kubernetes/autoscaler.git`

### 3. Go to below folder in cloned repo

`cd autoscaler/vertical-pod-autoscaler/`

### 4. Run below Command to deploy VPA in your cluster
```
./hack/vpa-up.sh

```

### 5. Check if pods related to VPA are running fine
```
kubectl get pods -n kube-system
```
### 6. Now Deploy test app using app-deployment.yaml

### 7. After this deploy vpa.yaml to enable vpa for  our deployment

### 8. Now increase load on the deployment to see working of vpa

```
kubectl run  apache-bench -i --tty --rm --image=httpd -- ab -n 500000 -c 1000 http://hpa-svc.default.svc.cluster.local/
```

### 9. Get VPA in our cluster
 `kubectl get vpa`

### 10. Describe VPA
`kubectl describe vpa kubengix-vpa`

### 11. You can observe now that the pods are upgraded resources by describing any pod.

