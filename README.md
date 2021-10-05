Argo Ecosystem Projects Demo
----------------------------

* Provision a Kubernetes cluster using K3S or Minikube and create `argocd` namespace:

```bash
k3d cluster create --api-port 6550 -p "8081:80@loadbalancer" --agents 3 --k3s-server-arg '--no-deploy=traefik'
kubectl create ns argocd
```

* Configure Github credentaials:

```bash
kubectl create secret generic github-creds --from-literal=username=$GH_USER --from-literal=password=$GH_PASSWORD -n argocd
```

* Configure Slack credentaials:

```bash
kubectl create secret generic argocd-notifications-secret --from-literal=slack-token=$SLACK_TOKEN -n argocd
```

* Bootstrap the Argo CD:

```
kustomize build https://github.com/alexmt/argo-ecosystem-demo//argocd | kubectl apply -f - -n argocd
```


* Wait for the Argo CD to be ready and access Argo CD UI:

```
kubectl rollout status deploy/argocd-redis -n argocd
kubectl rollout status deploy/argocd-repo-server -n argocd
argocd login --core
argocd admin dashboard --namespace argocd
```

