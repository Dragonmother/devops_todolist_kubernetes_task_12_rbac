# INSTRUCTION

## RBAC configuration

This project configures Kubernetes RBAC to allow the `secrets-reader` ServiceAccount to list Secrets in the `todoapp` namespace.

## Resources

The `security/rbac.yml` manifest creates:

- **ServiceAccount:** `secrets-reader`
- **Role:** `secrets-reader` with permission to list Secrets only
- **RoleBinding:** binds the Role to the `secrets-reader` ServiceAccount

The application Deployment uses the `secrets-reader` ServiceAccount.

## Deployment and validation

### 1. Create the kind cluster

```bash
kind create cluster --config cluster.yml
```

### 2. Apply the application configuration

Apply the namespace and application resources according to the project setup. Ensure the `todoapp` namespace and its dependencies exist before applying the RBAC and Deployment manifests.

### 3. Apply RBAC configuration

```bash
kubectl apply -f security/rbac.yml
```

### 4. Apply the Deployment

```bash
kubectl apply -f .infrastructure/app/deployment.yml
```

### 5. Verify RBAC permissions

Check that the ServiceAccount can list Secrets:

```bash
kubectl auth can-i list secrets --as=system:serviceaccount:todoapp:secrets-reader -n todoapp
```

Expected result:

```text
yes
```

Check that it cannot get individual Secrets:

```bash
kubectl auth can-i get secrets --as=system:serviceaccount:todoapp:secrets-reader -n todoapp
```

Expected result:

```text
no
```

### 6. Execute the request from the Deployment Pod

Get the name of a running Deployment Pod:

```bash
kubectl get pods -n todoapp -l app=todoapp
```

Connect to the Pod (replace `<pod-name>` with the actual Pod name):

```bash
kubectl exec -it <pod-name> -n todoapp -- sh
```

Inside the Pod, retrieve the ServiceAccount token and CA certificate:

```sh
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

Execute the request to list Secrets:

```sh
curl --cacert "$CACERT" \
  -H "Authorization: Bearer $TOKEN" \
  https://kubernetes.default.svc/api/v1/namespaces/todoapp/secrets
```

Expected result: a JSON response of kind `SecretList` containing the Secrets available in the `todoapp` namespace.

### 7. Evidence

Take a screenshot showing the successful request and its `SecretList` response. Attach the screenshot to the Pull Request.