# Helm Chart: Paperless S3 Backup

This Helm chart deploys a CronJob to back up Paperless NGX documents to an S3-compatible storage with optional sse -c encryption.

## Installation

To install with custom values:

```bash
helm install paperless-backup ./helm-paperless-s3-backup -f values.yaml -n <NAMESPACE>
```

## Configuration

### Values

| Key                | Default Value        | Description |
|--------------------|---------------------|-------------|
| `cron`            | `"0 4 * * 0"`       | Cron schedule for backups (UTC) |
| `image.repository` | `pascaaal/docker-awscli-gpg` | Backup container image repository |
| `image.tag`       | `v1.0.7`            | Container image tag |
| `s3.accessKey`    | `""`                | S3 Access Key ID |
| `s3.secretKey`    | `""`                | S3 Secret Access Key |
| `s3.bucket`       | `""`                | Target S3 bucket name |
| `s3.endpoint`     | `""`                | S3 endpoint including `https://` |
| `s3.region`       | `""`                | S3 region |
| `s3.sseKey`       | `""`                | 32-byte hex key for server-side encryption (SSE-C) |
| `paperless.namespace` | `""`           | Namespace of Paperless deployment |
| `paperless.app`   | `""`                | Label of the Paperless pod |
| `paperless.container_name` | `""`      | Paperless container name |

## Example `values.yaml`

```yaml
cron: "0 3 * * *"

image:
  repository: pascaaal/docker-awscli-gpg
  tag: v1.0.7

s3:
  accessKey: "your-access-key"
  secretKey: "your-secret-key"
  bucket: "your-backup-bucket"
  endpoint: "https://s3.example.com"
  region: "us-east-1"
  sseKey: "your-32-byte-hex-key" # encryption is enabled, when key is set

paperless:
  namespace: "paperless"
  app: "paperless"
  container_name: "paperless-container"
```

## Generate encryption key
```bash
openssl rand -hex 32
```

## Decrypt backup
```bash
export S3_SSE_KEY=154cd74642087d8d8d36c9136b89fb10b42a745c12108092895de44ed03518c0
echo $S3_SSE_KEY | xxd -r -p > .sse-c.key
aws s3 sync s3://<BUCKET_NAME> . --endpoint-url https://<ENDPOINT_URL> --sse-c AES256 --sse-c-key fileb://.sse-c.key
```

## View logs
```bash
kubectl logs -l app=paperless-backup -n <NAMESPACE> -c paperless-backup
```
## Uninstallation

To remove the deployment:

```bash
helm uninstall paperless-backup
```

## Notes
- Ensure that the S3 credentials have appropriate permissions to write to the bucket.
- Treat your encryption key like your car keys—lose it, and you’re not going anywhere… especially not to your backups! 🚗🔑

