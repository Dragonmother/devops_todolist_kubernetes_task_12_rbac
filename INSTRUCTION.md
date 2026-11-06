# INSTRUCTION

## RBAC configuration

This project configures Kubernetes RBAC to allow the `secrets-reader` ServiceAccount to list Secrets in the `todoapp` namespace.

## Resources

The `security/rbac.yml` manifest creates:

- **ServiceAccount:** `secrets-reader`
- **Role:** `secrets-reader` with permission to get and list Secrets
- **RoleBinding:** binds the Role to the ServiceAccount

The Deployment uses the `secrets-reader` ServiceAccount.

## Deployment and validation

Apply the RBAC configuration:

```bash
kubectl apply -f security/rbac.yml
```

Apply the application Deployment:

```bash
kubectl apply -f .infrastructure/app/deployment.yml
```

Verify that the ServiceAccount has permission to list Secrets:

```bash
kubectl auth can-i list secrets --as=system:serviceaccount:todoapp:secrets-reader -n todoapp
```

Expected result:

```text
yes
```

Create a test Pod using the same ServiceAccount:

```bash
kubectl apply -f security/rbac-test.yml
```

Connect to the test Pod:

```bash
kubectl exec -it rbac-test -n todoapp -- sh
```

Retrieve the ServiceAccount token and CA certificate:

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
