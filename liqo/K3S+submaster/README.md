# Note
Cluster component creation order seems to affect the behaviour of the cluster. <br />
Those tests aim to show those different behaviour.
# Test 1 
## Approach 
In this test I create the whole tree, composed of 2 clusters first then install Liqo on both of them.
## Results
Problems appear on both clusters
### First cluster
Liqo is not installed completly due to some pods stuck in pending state.
### Second cluster 
Metrics-server/Traefik/core-dns pods are stuck in CrashLoopBackOff state.
## Requierements on both clusters
kubectl : https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/ <br />
liqoctl : https://doc.liqo.io/installation/
## Steps
### First cluster creation
```
curl -sfL https://get.k3s.io | sh -s -
```
#### Add 1 node to the first cluster
```
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=XXX sh -s - --docker --lb-server-port=6544
```
### Create the second cluster (containerized)
```
kubectl apply -f submaster.yaml
```
#### Add 1 node to the second cluster
```
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=XXX sh -
```
##### Can also be done with Docker
```
docker-compose up 
```
### Install liqo on the first cluster (note: your kubectl must point to your first cluster)
```
liqoctl install k3s --generate-name
```
# Test 2 
## Approach 
In this test I create the first cluster and install Liqo on it then do the same for the second one.
## Results
The overall behaviour of the clusters seems better than in the first test. However some problems were still encountered.
### First cluster 
When the second cluster is created one of the Liqo pods get stuck in the CrashLoopBackOff.
Pod concerned : liqo-capsule-controller-manager
Error message : Liveness probe failed: Get "http://10.42.1.6:10080/healthz": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
### Peering step
Trying to peer from the firest cluset to the second one leads to access error (container's ip is not reachable).
Trying to peer from the second cluster to the first leads to authentification established and peering stuck in pending.
If I don't create a slave node or destroy it if created (on the second cluster) the gateway is moved from slave to submaster. However this configuration doesn't work due to the fact that the submaster is the gateway on both clusters. This means that the submaster has 2 pods that are on the same port, leading to problems.
## TO DO
Make the second cluster slave reachable from outside (Docker HostPort:ContainerPort).
The gateway port seems to be able to be configured. It could be dug as a non-definitive option.
## Steps 
### First cluster creation
```
curl -sfL https://get.k3s.io | sh -s -
```
#### Add 1 node to the first cluster
```
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=XXX sh -s - --docker --lb-server-port=6544
```
### Install liqo on the first cluster (note: your kubectl must point to your first cluster)
```
liqoctl install k3s --generate-name
```
### Create the second cluster (containerized)
```
kubectl apply -f submaster.yaml
```
#### Add 1 node to the second cluster
```
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=XXX sh -
```
##### Can also be done with Docker
```
docker-compose up 
```
### Install liqo on the second cluster (note: your kubectl must point to your second cluster)
```
liqoctl install k3s --generate-name
```
### Peering
To enable an outgoing peering from cluster A to cluster B, you must retrive cluster A credentials and add it to cluster B
#### Retrieve credentials (on cluster A)
```
liqoctl generate-add-command
```
#### Example of add command to enable peering (on cluster B)
```
liqoctl add cluster cold-brook --auth-url https://172.19.0.3:31531 --id f2606a1f-b1e5-4b10-9da3-f34b80b3633b --token 5418721a31decff5a1c373bd7039bb5e2d154d10a945b5dadcd0d1ead959268a4351e6c902f4b3149a7073674721620494e37da861a083d57ff2a133d834a178
```
