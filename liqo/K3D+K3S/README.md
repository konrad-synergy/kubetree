#requierements on both clusters
kubectl : https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/
liqoctl : https://doc.liqo.io/installation/

#Create K3D cluster
##Installation
```
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```
##Create cluster 
```
k3d cluster create mycluster
```
##Create agent node (slave)
```
k3d node create myagent --role agent --cluster mycluster
```
#Install liqo on K3D (note: your kubectl must point to your K3D cluster)
```
liqoctl install k3s --generate-name
```
#Create K3S cluster 
```
curl -sfL https://get.k3s.io | sh -
```
#Install liqo on K3S (note: your kubectl must point to your K3D cluster)
```
liqoctl install k3s --generate-name
```
#Peering
To enable an outgoing peering from cluster A to cluster B, you must retrive cluster A credentials and add it to cluster B
##Retrieve credentials (on cluster A)
```
liqoctl generate-add-command
```
## example of add command to enable peering (on cluster B)
```
liqoctl add cluster cold-brook --auth-url https://172.19.0.3:31531 --id f2606a1f-b1e5-4b10-9da3-f34b80b3633b --token 5418721a31decff5a1c373bd7039bb5e2d154d10a945b5dadcd0d1ead959268a4351e6c902f4b3149a7073674721620494e37da861a083d57ff2a133d834a178
```
