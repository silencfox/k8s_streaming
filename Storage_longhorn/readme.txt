helm repo add longhorn https://charts.longhorn.io
helm repo update
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --version 1.9.1
kubectl -n longhorn-system get pod


USER=<USERNAME_HERE>; PASSWORD=<PASSWORD_HERE>; echo "${USER}:$(openssl passwd -stdin -apr1 <<< ${PASSWORD})" >> auth
kubectl -n longhorn-system create secret generic basic-auth --from-file=auth
kubectl -n longhorn-system apply -f longhorn-ingress.yml