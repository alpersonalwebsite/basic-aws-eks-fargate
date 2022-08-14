# AWS EKS and Fargate

Fargate is a Container Runtime for ECS and EKS.

When we use `Fargate`, we just take care about our app (pods).
When we don't, we have to handle:
* EC2
* OS
* Kubernetes client and Docker runtime.

In all cases, EKS manages care of the Core Insfrastructure.
<!-- 
  TODO:
    More about Fargate
    Pricing
-->

## Create cluster

```shell
eksctl create cluster -f eks/fargate-profile.yaml 
```

It is going to take a while (~20 minutes).

Now we can check our nodes:

```shell
kubectl get nodes
```

```shell
NAME                                                   STATUS   ROLES    AGE   VERSION
fargate-ip-192-168-101-85.us-west-1.compute.internal   Ready    <none>   39s   v1.22.6-eks-14c7a48
fargate-ip-192-168-72-103.us-west-1.compute.internal   Ready    <none>   38s   v1.22.6-eks-14c7a48
```

And the runnings pods too:

```shell
kubectl get pods --all-namespaces
```

```shell
NAMESPACE     NAME                     READY   STATUS    RESTARTS   AGE
kube-system   coredns-84bf7749-22jzn   1/1     Running   0          11m
kube-system   coredns-84bf7749-4qc2h   1/1     Running   0          11m
```

## Run NGINX

We are going to run NGINX in one of the default namespaces

```shell
kubectl run nginx --image=nginx --restart=Never
```

Output:

```shell
pod/nginx created
```

Check running pods:

```shell
kubectl get pods --all-namespaces -o wide
```

```shell
NAMESPACE     NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE                                                   NOMINATED NODE   READINESS GATES
default       nginx                    1/1     Running   0          83s   192.168.65.64    fargate-ip-192-168-65-64.us-west-1.compute.internal    <none>           <none>
kube-system   coredns-84bf7749-22jzn   1/1     Running   0          15m   192.168.101.85   fargate-ip-192-168-101-85.us-west-1.compute.internal   <none>           <none>
kube-system   coredns-84bf7749-4qc2h   1/1     Running   0          15m   192.168.72.103   fargate-ip-192-168-72-103.us-west-1.compute.internal   <none>           <none>
```

## Cleanup

### Delete cluster

This would be the same to `delete in CFN` the stack `eksctl-basic-eks-fargate-cluster-cluster`

Before doing this, be sure that there are no resources tied to the VPC: example, Security Groups.

```shell
eksctl delete cluster --name basic-eks-fargate-cluster --region us-west-1
```

Output:

```shell
2022-08-12 09:58:16 [ℹ]  deleting EKS cluster "basic-eks-fargate-cluster"
2022-08-12 09:58:16 [ℹ]  deleting Fargate profile "fargate"
2022-08-12 10:00:24 [ℹ]  deleted Fargate profile "fargate"
2022-08-12 10:00:24 [ℹ]  deleting Fargate profile "staging"
2022-08-12 10:02:32 [ℹ]  deleted Fargate profile "staging"
2022-08-12 10:02:32 [ℹ]  deleted 2 Fargate profile(s)
2022-08-12 10:02:32 [✔]  kubeconfig has been updated
2022-08-12 10:02:32 [ℹ]  cleaning up AWS load balancers created by Kubernetes objects of Kind Service or Ingress
2022-08-12 10:02:34 [ℹ]  1 task: { delete cluster control plane "basic-eks-fargate-cluster" [async] }
2022-08-12 10:02:34 [ℹ]  will delete stack "eksctl-basic-eks-fargate-cluster-cluster"
2022-08-12 10:02:34 [✔]  all cluster resources were deleted
```