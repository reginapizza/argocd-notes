### Instructions for Setting Up Development Environment for ArgoCD with K3D Clusters
Note: This is specific to Fedora 32, no guarantees it works for other Linux distributions.

##### Prerequisites: 
- Install [Go](https://golang.org/doc/install)
- Fork and clone the [ArgoCD repo](https://github.com/argoproj/argo-cd). This should be placed somewhere in your $GOPATH.
- Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- Install [K3D](https://k3d.io/) (prerequisite for this is to also have [Docker](https://docs.docker.com/get-docker/) installed)

Now that all that's done...

1. Get your IP address
```
ip addr
```
For more info with this step, see [here](https://tecadmin.net/check-ip-address-fedora-desktop/).  

2. Start your k3d cluster.
You will need to replace $HOST_IP with your IP address from the step above. (ex: `--api-port 192.168.1.240:6550`). Replace `my-cluster` with whatever you want your cluster to be called.
```
k3d cluster create my-cluster --wait  --k3s-server-arg '--disable=traefik' --api-port $HOST_IP:6550 -p 443:443@loadbalancer
```
3. Create your argocd namespace within your new cluster.
```
kubectl create namespace argocd
```

4. If you aren't already in your `argo-cd` project directory...
```
cd ~/path/to/argo-cd
```
5. Install ArgoCD resources to your cluster.
```
kubectl apply -n argocd --force -f manifests/install.yaml
```
6. 
