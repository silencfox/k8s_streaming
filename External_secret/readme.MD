# External Secrets Operator con Azure Key Vault en Kubernetes

Este documento describe paso a paso cómo integrar **Azure Key Vault** con **Kubernetes** utilizando **External Secrets Operator (ESO)** para sincronizar secretos de forma segura.

---

## Requisitos

- Kubernetes funcionando
- `kubectl`
- `helm`
- Azure CLI (`az`)
- Azure Key Vault creado
- Permisos para crear recursos en el clúster

---

## 1. Instalar Helm

```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verificar instalación:

```bash
helm version
```

---

## 2. Instalar External Secrets Operator

Agregar el repositorio Helm:

```bash
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
```

Instalar el operador:

```bash
helm install external-secrets external-secrets/external-secrets   -n external-secrets   --create-namespace
```

Verificar que los pods estén corriendo:

```bash
kubectl get pods -n external-secrets
```

---

## 3. Crear Service Principal en Azure

Este Service Principal será usado por Kubernetes para autenticarse contra Azure Key Vault.

```bash
az ad sp create-for-rbac --name eso-k8s-sp --skip-assignment
```

Guarda los siguientes valores del output:

- `clientId`
- `clientSecret`
- `tenantId`

---

## 4. Crear el Secret de Azure en Kubernetes

Este Secret almacena las credenciales del Service Principal.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: azure-sp
  namespace: external-secrets
type: Opaque
stringData:
  client-id: "<CLIENT_ID>"
  client-secret: "<CLIENT_SECRET>"
```

Aplicar:

```bash
kubectl apply -f azure-sp.yaml
```

---

## 5. Definir el SecretStore (Azure Key Vault)

> Si utilizas **Managed Identity (AKS)**, esta sección se simplifica y no requiere `clientSecret`.

```yaml
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
```

Aplicar:

```bash
kubectl apply -f secretstore.yaml
```

---

## 6. Crear el ExternalSecret

Este recurso sincroniza secretos desde Azure Key Vault hacia Kubernetes.

```yaml
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
```

Aplicar:

```bash
kubectl apply -f externalsecret.yaml
```

---

## 7. Verificar el Secret creado

```bash
kubectl get secret app-secrets
```

---

## 8. Consumir el Secret en un Deployment

### Como variables de entorno

```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secrets
      key: DB_PASSWORD
```

### Como volumen

```yaml
volumes:
- name: secrets
  secret:
    secretName: app-secrets
```

---

## 9. Rotación automática de secretos

- ESO sincroniza periódicamente según `refreshInterval`
- Si el secreto cambia en Azure Key Vault:
  - El Secret en Kubernetes se actualiza
  - **El Pod no se reinicia automáticamente**

### Opciones para aplicar cambios:
- `kubectl rollout restart deployment <nombre>`
- Usar un reloader como:
  - `stakater/reloader`

---

## 10. Buenas prácticas (recomendadas)

- Un `SecretStore` por proveedor
- Usar `ClusterSecretStore` si múltiples namespaces comparten secretos
- No versionar secretos en Git
- Preferir **Managed Identity en AKS**
- Limitar permisos del Service Principal (principio de mínimo privilegio)

---

## Referencias

- https://external-secrets.io/
- https://learn.microsoft.com/azure/key-vault/
- https://learn.microsoft.com/azure/aks/

---

Autor: DevOps / Kubernetes
