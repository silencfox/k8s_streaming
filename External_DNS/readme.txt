Cloud Flare Token
xdjY***********************AcbA

curl "https://api.cloudflare.com/client/v4/user/tokens/verify" \
     -H "Authorization: Bearer xdjY***********************AcbA"

kubectl create secret generic cloudflare-api-key --from-literal=apiKey=xdjY***********************AcbA --from-literal=email=kelvin.ag89@gmail.com	 



helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/
helm repo update
helm upgrade --install external-dns external-dns/external-dns --values values.yaml
## helm uninstall external-dns 

kubectl create -f nginx.yaml
## check
https://www.cloudflare.com/a/dns/kdvops.com

##  Remove
kubectl delete -f nginx.yaml
kubectl delete -f externaldns.yaml