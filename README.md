# GitOps Bad Actor Demo
A repo showing how we can alert on resources not deployed via GitOps

## Workflow

### Create a KinD cluster

```
kind create cluster
```

### Install Helm

```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'      
helm init --service-account tiller --upgrade
```

### Install Flux

```
helm repo add fluxcd https://charts.fluxcd.io

kubectl create namespace flux

helm upgrade -i flux fluxcd/flux \
--set git.url=git@github.com:swade1987/gitops-bad-actor-demo \
--set git.path=kustomize/dev
--namespace flux
```

### Add deploy key to Git repo

Add the output of below as a deploy key to the repository

```
fluxctl identity --k8s-fwd-ns default
```

### Install Helm Operator

```
kubectl apply -f https://raw.githubusercontent.com/fluxcd/helm-operator/master/deploy/crds.yaml

helm upgrade -i helm-operator fluxcd/helm-operator --namespace fluxcd
```


