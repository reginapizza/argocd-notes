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
ifconfig
```
For more help with this step, see [here for Linux](https://www.cyberciti.biz/faq/bash-shell-command-to-find-get-ip-address/) or [here for MacOS](https://www.wikihow.com/Find-Your-IP-Address-on-a-Mac).  

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
6. Make sure you are in your argocd namespace 
```
kubectl config set-context k3d-my-cluster --namespace=argocd
```

7. Run `make start` to start your instance of ArgoCD.

8. Access the ArgoCD UI at [localhost:4000](http://localhost:4000/). Yay!

9. To delete your cluster, run `k3d cluster stop my-cluster` and then `k3d cluster delete my-cluster`. 

--- 

##### Common Error Messages & How to Fix Them:

1. 
```
rm: cannot remove '/tmp/argocd-local/gpg/source': Permission denied
rm: cannot remove '/tmp/argocd-local/gpg/keys': Permission denied
make: *** [Makefile:403: start-local] Error 1
make: *** [Makefile:395: start] Error 2
```
To fix these errors you will have to change the owner of these files. Run `sudo chown $USER /tmp/argocd-local/gpg/source`, `sudo chown $USER /tmp/argocd-local/gpg/keys`.

2. `Unable to connect to the server: x509: certificate is valid for 0.0.0.0, 10.43.0.1, 127.0.0.1, 172.25.0.2, not 192.168.0.14` 
This error means that the IP address is wrong. Open your `~/.kube/config` and make sure that `server:` says `https://[your ip address]:[port]` instead of `https://0.0.0.0:[port]`. 

2. `The container name "/argocd-test-server" is already in use by container "45d5d062b84f8816a39c226363afc5aadb7cd0c120584a096231e51a69c5c9de". You have to remove (or rename) that container to be able to reuse that name.`

This will require you to stop the argcd-test-server. This error usually occurs if you tried to run `make start` before and something went wrong, but it still made the argocd-test-server. You can delete it with `docker stop [long number/letter container from above]` and then trying `make start` again.
