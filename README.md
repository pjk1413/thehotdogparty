# Hot Dog Party

Static invite site for Hot Dog Party. Deployed to a Google Cloud Storage bucket on every push to `main`.

## Local preview

```bash
npx --yes serve public
```

Or open `public/index.html` directly in a browser.

## Layout

```
public/
  index.html
  assets/
    stick-figure.png
.github/workflows/deploy.yml
```

## Deploy (GCS)

The workflow in `.github/workflows/deploy.yml` uploads `public/` to your bucket using Workload Identity Federation (no long-lived JSON keys).

### One-time Google Cloud setup

1. Create a GCS bucket (optionally named for your domain).
2. Enable static website hosting on the bucket (`index.html` as the main page).
3. Grant public read on objects (e.g. `allUsers` → `roles/storage.objectViewer`), or put a load balancer / CDN in front.
4. Create a deploy service account with permission to write objects to that bucket (e.g. `roles/storage.objectAdmin` on the bucket).
5. Create a Workload Identity Pool + GitHub OIDC provider, and allow this repo’s `main` workflow to impersonate the service account.

### GitHub repository configuration

**Variables** (Settings → Secrets and variables → Actions → Variables):

| Name | Example |
| --- | --- |
| `GCP_PROJECT_ID` | `my-gcp-project` |
| `GCS_BUCKET` | `thehotdogparty.com` |

**Secrets:**

| Name | Example |
| --- | --- |
| `GCP_WORKLOAD_IDENTITY_PROVIDER` | `projects/123456789/locations/global/workloadIdentityPools/github/providers/github` |
| `GCP_SERVICE_ACCOUNT` | `deploy@my-gcp-project.iam.gserviceaccount.com` |

Until these are set, the deploy job will fail on auth/upload — that’s expected.

### Notes

- Uploads overwrite existing objects at the bucket root. Removed local files are not pruned automatically.
- HTTPS custom domains typically need a load balancer or Cloud CDN in front of the bucket; that wiring is outside this repo.
