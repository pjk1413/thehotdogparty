# Hot Dog Party

Static invite site for Hot Dog Party. Deploy to Google Cloud Storage manually from the Actions tab.

## Local preview

```bash
npx --yes serve public
```

Or open `public/index.html` directly in a browser.

## Layout

```
public/
  index.html
  favicon.svg
  assets/
    stick-figure.png
.github/workflows/deploy.yml
```

## Deploy (GCS)

The workflow in `.github/workflows/deploy.yml` uploads `public/` to your bucket using a Google Cloud service account key. Run it manually: **Actions → Deploy to GCS → Run workflow**.

### One-time Google Cloud setup

1. Create a GCS bucket (this project uses `hotdogparty-ui`).
2. Enable static website hosting on the bucket (`index.html` as the main page).
3. Grant public read on objects (e.g. `allUsers` → `roles/storage.objectViewer`), or put a load balancer / CDN in front.
4. Create a deploy service account in project `kitchensinkworks`.
5. Grant it write access to the bucket (e.g. `roles/storage.objectAdmin` on `hotdogparty-ui`).
6. Create a JSON key for that service account and store it in GitHub (see below).

Example:

```bash
gcloud config set project kitchensinkworks

gcloud iam service-accounts create hotdogparty-deploy \
  --display-name="Hot Dog Party GCS deploy"

gcloud storage buckets add-iam-policy-binding gs://hotdogparty-ui \
  --member="serviceAccount:hotdogparty-deploy@kitchensinkworks.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"

gcloud iam service-accounts keys create sa-key.json \
  --iam-account=hotdogparty-deploy@kitchensinkworks.iam.gserviceaccount.com
```

Then paste the contents of `sa-key.json` into the GitHub secret and delete the local file.

### GitHub repository configuration

**Variables** (Settings → Secrets and variables → Actions → Variables):

| Name | Value |
| --- | --- |
| `GCP_PROJECT_ID` | `kitchensinkworks` |
| `GCS_BUCKET` | `hotdogparty-ui` |

**Secrets:**

| Name | Value |
| --- | --- |
| `GCP_SA_KEY` | Full JSON key for the deploy service account |

Until these are set, the deploy job will fail on auth/upload — that’s expected.

### Notes

- Uploads overwrite existing objects at the bucket root. Removed local files are not pruned automatically.
- HTTPS custom domains typically need a load balancer or Cloud CDN in front of the bucket; that wiring is outside this repo.
