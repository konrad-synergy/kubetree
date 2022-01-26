# Purpose
This test aims to combine Submariner and my previous work on containerized submasters <br />
Submariner : https://submariner.io/ <br />
Previous work : https://github.com/olivierdg2/Kubernetes_submaster
# Results 
Services are accessible across clusters
# Steps
## First cluster creation
```
POD_CIDR=10.44.0.0/16
SERVICE_CIDR=10.45.0.0/16
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--cluster-cidr $POD_CIDR --service-cidr $SERVICE_CIDR" sh -s -
```
### Add 1 node to the first cluster
```
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=XXX sh -s - --docker --lb-server-port=6544
```
### Ensure that all nodes allow TCP traffic on port 8080 (for submariner health check)
#### Can be done with 
```
sudo ufw allow 8080
```
### Set endpoint and context of the first cluster kubeconfig
```
sudo cp /etc/rancher/k3s/k3s.yaml kubeconfig.cluster-a
sudo chown $(id -u):$(id -g) kubeconfig.cluster-a
export IP=$(hostname -I | awk '{print $1}')
yq -i eval \
'.clusters[].cluster.server |= sub("127.0.0.1", env(IP)) | .contexts[].name = "cluster-a" | .current-context = "cluster-a"' \
kubeconfig.cluster-a
```
## Install subctl 
```
curl -Ls https://get.submariner.io | bash
export PATH=$PATH:~/.local/bin
echo export PATH=\$PATH:~/.local/bin >> ~/.profile
```
## Use the first cluster as the broker
```
subctl deploy-broker --kubeconfig kubeconfig.cluster-a
```
## Join the first cluster to the broker
```
subctl join --kubeconfig kubeconfig.cluster-a broker-info.subm --clusterid cluster-a --natt=false
```
## Create the second cluster (containerized)
```
kubectl apply -f submaster.yaml
```
### Add 1 node to the second cluster
```
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=XXX sh -
```
#### Can also be done with Docker
```
docker-compose up 
```
### Extract kubeconfig from pod container
```
kubectl cp podID:output/kubeconfig.yaml /home/user/kubeconfig.cluster-b
```
### Set endpoint and context of the second cluster kubeconfig
```
sudo chown $(id -u):$(id -g) kubeconfig.cluster-b
export IP=$(hostname -I | awk '{print $1}')
yq -i eval \
'.clusters[].cluster.server |= sub("127.0.0.1", env(IP)) | .contexts[].name = "cluster-b" | .current-context = "cluster-b"' \
kubeconfig.cluster-b
```
## Join the second cluster to the broker
```
subctl join --kubeconfig kubeconfig.cluster-b broker-info.subm --clusterid cluster-b --natt=false
```
### If any health check porblem is detected use 
```
subctl join --kubeconfig kubeconfig.cluster-b broker-info.subm --clusterid cluster-b --natt=false --health-check=false
```
## Display submariner informations 
```
subctl show all 
```
