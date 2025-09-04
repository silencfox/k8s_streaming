#helm install \
#  cert-manager oci://quay.io/jetstack/charts/cert-manager \
#  --version v1.18.2 \
#  --namespace cert-manager \
#  --create-namespace \
#  --set prometheus.enabled=false \
#  --set webhook.timeoutSeconds=4 \
#  --set crds.enabled=true


helm repo add jetstack https://charts.jetstack.io --force-update

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.18.2 \
  --set crds.enabled=true


# helm uninstall cert-manager -n cert-manager