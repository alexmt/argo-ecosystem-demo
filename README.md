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
argocd admin dashboard --namespace argocd
```

* Create application set:

```bash
kubectl apply -f https://raw.githubusercontent.com/alexmt/argo-ecosystem-demo/master/appset.yaml
```

![image](https://user-images.githubusercontent.com/426437/136109471-136fe2ce-da67-47fa-b31a-de6b159bda72.png)

* Enable automatic image updates:

For demo purposes, let's stop auto syncing:

```bash
kubectl patch applicationset demo-apps -p '{"spec":{"template":{"spec":{"syncPolicy":{"automated":null}}}}}' --type merge -n argocd
```

```bash
curl https://raw.githubusercontent.com/alexmt/argo-ecosystem-demo/master/appset-patch.json > /tmp/patch.json && \
    kubectl patch applicationset demo-apps --patch-file /tmp/patch.json  -n argocd --type merge
```