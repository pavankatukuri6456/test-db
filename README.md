# PostgreSQL connection checker

This standalone example starts PostgreSQL in Docker Desktop and runs a
Kubernetes Deployment that performs a real authenticated `SELECT 1` every ten
seconds. It logs `connected` when the query succeeds and `not connected` when
it fails.

Actual credential files are ignored by Git. Create them from the committed
examples and keep the username and password identical in the Docker and
Kubernetes secret files.

```bash
cp secrets/db-username.example.txt secrets/db-username.txt
cp secrets/db-password.example.txt secrets/db-password.txt
cp secrets.example.yaml secrets.yaml
```

Replace the example password in both generated password files before starting
the demo.

## 1. Start PostgreSQL in Docker Desktop

```powershell
cd C:\Users\pavan\projects\postgres-connection-checker
docker compose up -d --wait
docker compose ps
```

PostgreSQL listens only on the host loopback interface at port `55433`.

## 2. Run the checker in Docker Desktop Kubernetes

Enable Kubernetes in Docker Desktop first. Then select its context and apply
the manifests in dependency order:

```powershell
kubectl config use-context docker-desktop
kubectl apply -f namespace.yaml
kubectl apply -f secrets.yaml
kubectl apply -f deployment.yaml
kubectl logs -n db-connect-demo deployment/db-connect-checker --follow
```

Expected output:

```text
2026-08-30T21:00:00Z connected
```

To verify the failure path, stop and restart the database while following the
logs:

```powershell
docker compose stop postgres
docker compose start postgres
```

## Cleanup

```powershell
kubectl delete namespace db-connect-demo
docker compose down
```

Add `--volumes` to `docker compose down` only when you also want to delete the
local PostgreSQL data.

Kubernetes Secrets are encoded, not encrypted. Do not commit real production
credentials; use your cluster's secret-management solution for production.
