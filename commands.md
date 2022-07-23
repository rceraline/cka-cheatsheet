# Discover the api version for a resource

`kubectl explain daemonset`
Retrieve information about DaemonSet.

# POD

Create an NGINX Pod

`kubectl run nginx --image=nginx`

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

`kubectl run nginx --image=nginx --dry-run=client -o yaml`

# Deployment

Create a deployment

`kubectl create deployment --image=nginx nginx`

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

Generate Deployment with 4 Replicas

`kubectl create deployment nginx --image=nginx --replicas=4`

You can also scale a deployment using the kubectl scale command.

`kubectl scale deployment nginx --replicas=4`

Another way to do this is to save the YAML definition to a file and modify

`kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml`

You can then update the YAML file with the replicas or any other field before creating the deployment.

# Service

### Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

(This will automatically use the pod's labels as selectors)

Or

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml` (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

### Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the kubectl expose command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

# Upgrade cluster with kubeadm

## Upgrade the master node

Scenario: upgrade from v1.11 to v1.12

1. Run a plan to have details on the version on the cluster.

```
kubeadm upgrade plan
```

1. Updgrade kudeadm tool

```
apt-get upgrade -y kubeadm=1.12.0-00
```

1. Upgrade the cluster

```
kudeadmn upgrade apply v1.12.0
```

Upgrade the kubelets

```
apt-get upgrade -y kubelet=1.12.0-00

systemctl restart kubelet
```

## Worker node upgrade

On worker nodes, we upgrade kudeadm, kubelet & kubectl.
To move all pods from a different node in order to upgrade the node, run the command

```
kubectl drain node-name
```

Then on the node, run

```
apt-get upgrade -y kudeadm=1.12.0-00
apt-get upgrade -y kubelet=1.12.0-00
kudeadm upgrade node config --kubelet-version v1.12.0
systemctl restart kubelet
```

To make the node schedulable after, run

```
kubectl uncordon node-name
```

## List all resources

```
kubectl api-resources --namespaced=true

kubectl api-resources --namespaced=false
```

## List all processes

```
ps -aux
```
