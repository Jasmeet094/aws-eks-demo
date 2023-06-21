
# AWS EKS - Horizontal Pod Autoscaler (HPA)

 In Kubernetes, a pod is the smallest unit of deployment that contains one or more containers. The HPA is a feature that automatically adjusts the number of replicas (copies) of a pod based on the current workload or demand. Horizontal Pod Autoscaler (HPA) can be configured to target different metrics to determine when to scale the number of pod replicas. The targets commonly used with HPA are: CPU utilization , Memory utilization & Custom metrics. 
 
 `The Metrics Server` is a component in Kubernetes that collects resource utilization metrics from the nodes and pods within the cluster. It plays a crucial role in supporting the Horizontal Pod Autoscaler (HPA) by providing the necessary metrics for autoscaling decisions.

[Metrics Server](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html)

## In this Demo   we are going to see working of the HPA.

### 1. Install Metrics Server in your EKS Cluster.

`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

### 2. Create demo app by applying app-deployment.yaml file.

### 3. Run below command to enable the HPA for the deployment

`kubectl autoscale deployment demo-app-hpa --cpu-percent=50 --min=1 --max=10`

### 4. When all pods are in running state , create a pod which will increase the load on the app pods to see workng of the HPA.
```
kubectl run  apache-bench -i --tty --rm --image=httpd -- ab -n 500000 -c 1000 http://hpa-svc.default.svc.cluster.local/

```

### 5. Check the working of HPA
```
kubectl get hpa
kubectl  describe hpa name
```
### 6. You can observe that the target will start increasing and number of pods will also start increasing.


