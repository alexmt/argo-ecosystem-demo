Argo Ecosystem Projects Demo
----------------------------

The http://github.com/argoproj-labs host a variety of ecosystem projects that support core Argo projects.

<img width="600" alt="portfolio_view" src="https://user-images.githubusercontent.com/426437/136265445-8dcd14c3-d519-4898-93d6-05cfe474c558.png">

This repository demonstrates the integration of the following projects:

* [applicationset](https://github.com/argoproj-labs/applicationset) - automates Argo CD application management.
* [argocd-image-updater](https://github.com/argoproj-labs/argocd-image-updater) - automates docker image management for Argo CD.
* [argocd-notifications](https://github.com/argoproj-labs/argocd-notifications) - provides notifications for Argo CD.
* [argocd-extensions](https://github.com/argoproj-labs/argocd-extensions) - allows packaging resource-specific Argo CD customizations including health checks,
  actions, and custom visualization in Web UI.
* [rollout-extension](https://github.com/argoproj-labs/rollout-extension) - provides Argo Rollout CRD visualization in Argo CD Web UI.

## Repository Structure

The repository contains four directories with manifests that represent four Kubernetes namespaces.

### argocd

The `argocd` directory includes Argo CD installation manifests and additionally bundles [applicationset](https://github.com/argoproj-labs/applicationset),
[argocd-image-updater](https://github.com/argoproj-labs/argocd-image-updater), [argocd-notifications](https://github.com/argoproj-labs/argocd-notifications) and 
[argocd-extensions](https://github.com/argoproj-labs/argocd-extensions) components. The installation manifests are pulled together using the
[argocd/kustomization.yaml](argocd/kustomization.yaml) file.

### argo-rollouts

The `argo-rollouts` directory includes Argo Rollouts installation manifests as well as notifications functionality configuration:
[argo-rollouts/kustomization.yaml](argo-rollouts/kustomization.yaml).

### demo-app

The `demo-app` directory includes the installation manifests of the [Argo Rollouts](https://github.com/argoproj/rollouts-demo) demo application
that is configured to use canary deployment strategy: [demo-app/kustomization.yaml](demo-app/kustomization.yaml).

## ingress-nginx

The `ingress-nginx` installs the NGINX ingress controller using off-the-shelf Helm chart: [Chart.yaml](ingress-nginx/Chart.yaml).

## Demo

### Demo Kubernetes cluster

First things first. Provision a demo Kubernetes cluster using K3S or Minikube. The following command provisons K3S Kubernetes cluster using K3D:

```bash
k3d cluster create --api-port 6550 -p "8081:80@loadbalancer" --agents 3 --k3s-server-arg '--no-deploy=traefik'
```

### Credentials

The components of the demo are going to interact with Github and Slack. So we need to create a Kubernetes secret that provides the credentials.

Use the following links links to learn how to create the Slack and Github tokens: 
[Slack](https://argocd-notifications.readthedocs.io/en/stable/services/slack/),
[Github](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).

Store the Github token in `GH_PASSWORD` env variable, your Github username in `GH_USERNAME` and the Slack token in `SLACK_TOKEN` env variable.
Next run the following commands to create Kubernetes secrets:

```bash
kubectl create ns argocd
kubectl create secret generic github-creds --from-literal=username=$GH_USER --from-literal=password=$GH_PASSWORD -n argocd
kubectl create secret generic argocd-notifications-secret --from-literal=slack-token=$SLACK_TOKEN -n argocd

kubectl create ns argo-rollouts
kubectl create secret generic argo-rollouts-notification-secret --from-literal=slack-token=$SLACK_TOKEN -n argo-rollouts
```

### Argo CD

We are going to use Argo CD to perform the demo and to observe what it happening in the Kubernetes cluster. Run the following
command to install the Argo CD:

```
kustomize build https://github.com/alexmt/argo-ecosystem-demo//argocd | kubectl apply -f - -n argocd
```

Once installation is done use the following command to access Argo CD UI:

```
argocd admin dashboard --namespace argocd
```

### Create Argo CD Applications

Argo CD is up and running. Now we need to create the applications that will be used in the demo. Instead of creating the applications
manually we are going to leverage [applicationset](https://github.com/argoproj-labs/applicationset). Run the following command to create
the `ApplicationSet` resource:

```bash
kubectl apply -f https://raw.githubusercontent.com/alexmt/argo-ecosystem-demo/master/appset.yaml
```

The command is going to create the [demo-apps](appset.yaml) application set that is configured to create one application per the directory in this repository.
Navigate to Argo CD Web UI at http://localhost:8080 to view the applications:

![image](https://user-images.githubusercontent.com/426437/136109471-136fe2ce-da67-47fa-b31a-de6b159bda72.png)

### Upgrade the Rollout Demo App

The next step is to upgrade the demo application. Again, instead of manually changing the manifest let's take advantage of the automation. We are going to leverage
the [argocd-image-updater](https://github.com/argoproj-labs/argocd-image-updater) component. Run the following commands to update the `demo-apps` application set
and apply the annotations that enable to automated image management and disable auto-syncing (for the sake of the demo):

```bash
# stop auto syncing:
kubectl patch applicationset demo-apps  --type merge -n argocd \
    -p '{"spec":{"template":{"spec":{"syncPolicy":{"automated":null}}}}}'
# add annotations
curl https://raw.githubusercontent.com/alexmt/argo-ecosystem-demo/master/appset_patch.json > /tmp/patch.json && \
    kubectl patch applicationset demo-apps --patch-file /tmp/patch.json  -n argocd --type merge
```
