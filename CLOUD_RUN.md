# Cloud Run Mapping Guide

This document outlines how environment configurations and secret variables map from local containerized execution (`docker run`) to Google Cloud Run deployment.

## 1. Non-Secret Configuration Variables

Variables such as database connections (`DATABASE_URL`) and operational settings (`LOG_LEVEL`) contain configuration data specific to the deployment environment.

- **Local Development**: Injected via `.env` file using `docker run --env-file .env ...`
- **Cloud Run Mapping**: Injected using the `--set-env-vars` flag during deployment.

### Example CLI Mapping:
```bash
gcloud run deploy envlab-service \
  --image gcr.io/$PROJECT_ID/envlab:latest \
  --set-env-vars DATABASE_URL="mongodb://prod-db.example.com:27017/app",LOG_LEVEL="info"
```

## 2. Secrets & Sensitive Credentials

Sensitive keys such as `API_KEY`, database passwords, or third-party OAuth secrets must **NEVER** be committed to Git or built into Docker container image layers (`ENV`).

- **Local Development**: Injected at runtime via shell environment flags (`docker run -e API_KEY=$API_KEY ...`).
- **Cloud Run Mapping**: Stored securely in **Google Secret Manager** and bound to the Cloud Run container instance via the `--set-secrets` flag.

### Example CLI Mapping:
```bash
# 1. Store secret in GCP Secret Manager
gcloud secrets create API_KEY --replication-policy="automatic"
echo -n "$API_KEY" | gcloud secrets versions add API_KEY --data-file=-

# 2. Grant Secret Accessor role to service account
gcloud secrets add-iam-policy-binding API_KEY \
  --member="serviceAccount:$SERVICE_ACCOUNT_EMAIL" \
  --role="roles/secretmanager.secretAccessor"

# 3. Inject secret into Cloud Run service instance at runtime
gcloud run deploy envlab-service \
  --image gcr.io/$PROJECT_ID/envlab:latest \
  --set-env-vars DATABASE_URL="mongodb://prod-db.example.com:27017/app",LOG_LEVEL="info" \
  --set-secrets API_KEY=API_KEY:latest
```
