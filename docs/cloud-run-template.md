# Cloud Run Mapping

## Configuration Variables (Non-Secret)

- **`DATABASE_URL`**: Non-sensitive database connection string. Configured in Google Cloud Run using runtime environment variables via `--set-env-vars`.
- **`LOG_LEVEL`**: Non-sensitive logging level (e.g. `info`). Configured in Google Cloud Run using runtime environment variables via `--set-env-vars`.

### Example Command
```bash
gcloud run deploy envlab-service \
  --image gcr.io/your-project/envlab:latest \
  --set-env-vars DATABASE_URL="mongodb://localhost:27017/app_db",LOG_LEVEL="info"
```

---

## Secrets

- **`API_KEY`**: Sensitive authentication credential. Stored in GCP Secret Manager and injected securely into the container environment at runtime via `--set-secrets`. Never baked into Dockerfile image layers or stored in source repositories.

### Example Secret Manager Integration
```bash
# 1. Create secret in Google Secret Manager
gcloud secrets create API_KEY --replication-policy="automatic"
echo -n "your-secret-api-key" | gcloud secrets versions add API_KEY --data-file=-

# 2. Deploy service mapping secret to environment variable
gcloud run deploy envlab-service \
  --image gcr.io/your-project/envlab:latest \
  --set-env-vars DATABASE_URL="mongodb://localhost:27017/app_db",LOG_LEVEL="info" \
  --set-secrets API_KEY=API_KEY:latest
```
