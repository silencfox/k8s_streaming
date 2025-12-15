# Sealed Secrets – Instalación y Uso en Kubernetes

Este documento describe cómo instalar **Sealed Secrets**, realizar el **backup de la clave privada**, y crear y sellar secretos de forma segura para Kubernetes.

---

## 1. Instalación del controlador Sealed Secrets (Helm)

Agregar el repositorio de Bitnami:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Instalar Sealed Secrets en el namespace `kube-system`:

```bash
helm install sealed-secrets bitnami/sealed-secrets \
  --namespace kube-system
```

Verificar que el controlador esté en ejecución:

```bash
kubectl get pods -n kube-system | grep sealed
```

---

## 2. Instalación del cliente kubeseal (Linux)

```bash
wget https://github.com/bitnami-labs/sealed-secrets/releases/latest/download/kubeseal-linux-amd64
chmod +x kubeseal-linux-amd64
sudo mv kubeseal-linux-amd64 /usr/local/bin/kubeseal
```

---

## 3. Backup de la clave privada (CRÍTICO)

> **Advertencia:**  
> Si pierdes esta clave, **no podrás desencriptar los secretos existentes**.

```bash
kubectl get secret -n kube-system sealed-secrets-key -o yaml > sealed-secrets-key-backup.yaml
```

Guarda este archivo en un lugar seguro y **fuera del repositorio Git**.

---

## 4. Crear un Secret estándar

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=SuperSecret123 \
  --namespace default \
  --dry-run=client -o yaml > secret.yaml
```

---

## 5. Sellar el Secret

```bash
kubeseal \
  --controller-name sealed-secrets \
  --controller-namespace kube-system \
  --format yaml < secret.yaml > sealed-secret.yaml
```

Aplicar el SealedSecret al cluster:

```bash
kubectl apply -f sealed-secret.yaml
```

Verificar que el Secret fue creado correctamente:

```bash
kubectl get secret db-secret -n default
```

---

## 6. Datos adicionales – Tipos de alcance (scope)

### Namespace-scoped (default)
Solo funciona en el mismo namespace.

```yaml
spec:
  template:
    metadata:
      name: db-secret
      namespace: default
```

---

### Cluster-wide
Funciona en cualquier namespace del cluster.

```yaml
spec:
  scope: cluster-wide
```

---

### Strict (más seguro)
Requiere que el nombre y el namespace coincidan exactamente.

```yaml
spec:
  scope: strict
```

---

## 7. Recomendaciones finales

- Respaldar la clave privada regularmente
- Versionar **solo** los SealedSecrets, nunca los Secrets reales
- Usar `strict` o `namespace-scoped` en entornos productivos
- Integrar el despliegue con CI/CD o GitOps
