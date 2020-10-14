## Using ArgoCD with an OpenShift cluster
##### Note: this is only tested using Fedora 32 with an OpenShift 4.6 cluster

##### Prerequisites:
- [oc tool](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

##### Instructions: 
1. If you have to create a cluster, read [this documentation](https://docs.openshift.com/container-platform/4.5/welcome/index.html) that will tell you how to get started. 

1. Once you have access to a cluster, login to the cluster and type in the login command in your terminal. You can get this command generated for you automatically if you navigate to the user menu in the top right corner and click on "Copy Login Command".
```
oc login --token=[token] --server=[cluster url]
```

2. Create the argocd namespace on the cluster and switch into it
```
oc create namespace argocd
oc project argocd
```
3. Make sure you're in your argocd folder (`cd ~/path/to/file/argo-cd`) and install the argo-cd manifests in the argocd namespace
```
oc apply -n argocd --force -f manifests/install.yaml
```

4. Scale down any ArgoCD instance in your cluster
```
kubectl -n argocd scale deployment/argocd-application-controller --replicas 0
kubectl -n argocd scale deployment/argocd-dex-server --replicas 0
kubectl -n argocd scale deployment/argocd-repo-server --replicas 0
kubectl -n argocd scale deployment/argocd-server --replicas 0
kubectl -n argocd scale deployment/argocd-redis --replicas 0
```

5.   Launch ArgoCD in your preferred code editor and navigate to the Makefile at the project root. Update VOLUME_MOUNT (around line 15) from 
`VOLUME_MOUNT=$(shell if test "$(go env GOOS)" = "darwin"; then echo ":delegated"; elif test selinuxenabled; then echo ":delegated"; else echo ""; fi)` to `VOLUME_MOUNT=$(shell if test "$(go env GOOS)" = "darwin"; then echo ":delegated"; elif test selinuxenabled; then echo ":Z"; else echo ""; fi)` and then save. 

6. Check that selinux mode is set to Disabled by typing ‘getenforce’ in terminal.

If its in  in enforcing mode, Change the SELINUX value to SELINUX=disabled in file /etc/selinux/config.
More(https://www.thegeekdiary.com/how-to-disable-or-set-selinux-to-permissive-mode/)

6. Run `make dep-ui`
	Note: might hang for a little while on `Not installing Tiller due to 'client-only' flag having been set`... just wait on it

7. Run `make mod-download`

8. Run `make mode-vendor`

9. Run `make build`

10. Run `make test`

11. Run `make start`

12. Access the ArgoCD UI runing locally at [localhost:4000](http://localhost:4000/)
