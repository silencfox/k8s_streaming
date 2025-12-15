curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

helm repo add external-secrets https://charts.external-secrets.io
helm repo update

helm install external-secrets external-secrets/external-secrets \
  -n external-secrets \
  --create-namespace

kubectl get pods -n external-secrets


az ad sp create-for-rbac --name eso-k8s-sp --skip-assignment


#### CREAR SECRET DE AZURE
apiVersion: v1
kind: Secret
metadata:
  name: azure-sp
  namespace: external-secrets
type: Opaque
stringData:
  client-id: "<CLIENT_ID>"
  client-secret: "<CLIENT_SECRET>"


## 5. Definir el SecretStore (Azure Key Vault)
## Si usaras Managed Identity (AKS), esta sección se simplifica bastante.
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: azure-keyvault
  namespace: default
spec:
  provider:
    azurekv:
      tenantId: "<TENANT_ID>"
      vaultUrl: "https://<NOMBRE-KEYVAULT>.vault.azure.net/"
      authType: ServicePrincipal
      authSecretRef:
        clientId:
          name: azure-sp
          key: client-id
          namespace: external-secrets
        clientSecret:
          name: azure-sp
          key: client-secret
          namespace: external-secrets


##. Crear el ExternalSecret
## Este recurso mapea secretos de Azure → Secret de Kubernetes.
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secrets
  namespace: default
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: azure-keyvault
    kind: SecretStore
  target:
    name: app-secrets
    creationPolicy: Owner
  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: db-password
    - secretKey: API_KEY
      remoteRef:
        key: api-key

###  VERIFICAR
kubectl get secret app-secrets



## Consumir el Secret en un Deployment
## Como variables de entorno
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secrets
      key: DB_PASSWORD

## Como volumen
volumes:
- name: secrets
  secret:
    secretName: app-secrets



## Rotación automática de secretos
## ESO sincroniza periódicamente (refreshInterval)
## Si cambias el secreto en Azure Key Vault:
## Kubernetes Secret se actualiza
## El Pod no se reinicia automáticamente
## Para rotación efectiva:
## Usa rollout restart
## O un reloader (ej: stakater/reloader)


10. Buenas prácticas (recomendadas)

Un SecretStore por proveedor
Usar ClusterSecretStore si varios namespaces consumen lo mismo
No versionar secretos en Git
Usar Managed Identity en AKS si es posible
Limitar permisos del Service Principal (principio de mínimo privilegio)
