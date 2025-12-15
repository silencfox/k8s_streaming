helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update


helm install sealed-secrets bitnami/sealed-secrets \
  --namespace kube-system


kubectl get pods -n kube-system | grep sealed


wget https://github.com/bitnami-labs/sealed-secrets/releases/latest/download/kubeseal-linux-amd64
chmod +x kubeseal-linux-amd64
sudo mv kubeseal-linux-amd64 /usr/local/bin/kubeseal


##### Backup de la clave privada (CRÍTICO),  Si pierdes esta clave, no podrás desencriptar los secretos existentes.
kubectl get secret -n kube-system sealed-secrets-key -o yaml > sealed-secrets-key-backup.yaml



###### CREAR UN SECRET
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=SuperSecret123 \
  --namespace default \
  --dry-run=client -o yaml > secret.yaml


####### SELLAR EL SECRET
kubeseal \
  --controller-name sealed-secrets \
  --controller-namespace kube-system \
  --format yaml < secret.yaml > sealed-secret.yaml

kubectl apply -f sealed-secret.yaml

kubectl get secret db-secret -n default



####################  DATOS ADICIONALES   ############
### Tipos de alcance (scope)
### Namespace-scoped (default)
### Solo funciona en el mismo namespace.
spec:
  template:
    metadata:
      name: db-secret
      namespace: default

### Cluster-wide
### Funciona en cualquier namespace.
spec:
  scope: cluster-wide

### Strict (más seguro)
###  Nombre + namespace obligatorios.
spec:
  scope: strict
