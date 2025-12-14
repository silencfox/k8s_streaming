## install using helm
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd --namespace argocd


## install using yamls
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

## get password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

## editar deployment para usar insecure
## agregar el argumento "--insecure" dentro del spec.container.command
kubectl --namespace argocd edit deployment/argocd-server

## ArgCD CLI
https://github.com/argoproj/argo-cd/releases/download/v3.1.1/argocd-linux-arm64

## para servicio con ingress
kubectl apply -f ingress_svc.yaml

## para servicio con traefik
kubectl apply -f traefik_svc.yaml

kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 --decode


## install argo RollOut
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
kubectl apply -n argo-rollouts -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/dashboard-install.yaml



kubectl argo rollouts version

kubectl argo rollouts dashboard
