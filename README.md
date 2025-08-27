# sacloud-apprun-action

GitHub Action to build and deploy Go or Python applications to Sakura AppRun.

## Features

- ✅ Support for both Go and Python applications
- ✅ Automatic language detection
- ✅ Optimized Docker builds
- ✅ Container registry integration
- ✅ Litestream SQLite backup support (for both Go and Python)
- ✅ Zero-downtime deployments

## Supported Languages

### Go
- Automatically detected if `go.mod` or `main.go` exists
- Uses multi-stage Docker builds for optimized images
- CGO support included for SQLite

### Python
- Automatically detected if `requirements.txt`, `app.py`, or `main.py` exists
- Uses multi-stage Docker builds with Alpine Linux
- Default entry point: `python app.py`

## Usage

### Basic Usage

```yaml
name: Deploy to Sakura AppRun
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to AppRun
        uses: your-username/sacloud-apprun-action@v1
        with:
          sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
          sakura-api-secret: ${{ secrets.SAKURA_API_SECRET }}
          container-registry: myregistry.sakuracr.jp
          container-registry-user: ${{ secrets.REGISTRY_USER }}
          container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
```

### Language Specification

```yaml
- name: Deploy Python App
  uses: your-username/sacloud-apprun-action@v1
  with:
    language: python  # or 'go', defaults to 'auto'
    sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
    # ... other parameters
```

### Custom Requirements File

```yaml
- name: Deploy Python App with Custom Requirements
  uses: your-username/sacloud-apprun-action@v1
  with:
    language: python
    requirements-file: config/requirements.txt  # Custom path
    sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
    # ... other parameters
```

### With Litestream Backup

```yaml
- name: Deploy with SQLite Backup
  uses: your-username/sacloud-apprun-action@v1
  with:
    sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
    sakura-api-secret: ${{ secrets.SAKURA_API_SECRET }}
    container-registry: myregistry.sakuracr.jp
    container-registry-user: ${{ secrets.REGISTRY_USER }}
    container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
    object-storage-bucket: my-backup-bucket
    object-storage-access-key: ${{ secrets.S3_ACCESS_KEY }}
    object-storage-secret-key: ${{ secrets.S3_SECRET_KEY }}
    sqlite-db-path: /app/data/database.db
```

## Input Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `language` | Application language (`go`, `python`, or `auto`) | No | `auto` |
| `sakura-api-key` | Sakura Cloud API Key | Yes | |
| `sakura-api-secret` | Sakura Cloud API Secret | Yes | |
| `container-registry` | Container registry URL | Yes | |
| `container-registry-user` | Registry username | Yes | |
| `container-registry-password` | Registry password | Yes | |
| `app-name` | Application name | No | Repository name |
| `app-dir` | Application directory | No | Root directory |
| `port` | Application port | No | `8080` |
| `max-cpu` | Maximum CPU allocation | No | `0.4` |
| `max-memory` | Maximum memory allocation | No | `256Mi` |
| `timeout-seconds` | Request timeout | No | `300` |
| `use-repository-dockerfile` | Use existing Dockerfile | No | `true` |
| `requirements-file` | Path to requirements.txt file (relative to app-dir) | No | `requirements.txt` |

### Litestream Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `object-storage-bucket` | S3 bucket for SQLite backup | No | |
| `object-storage-access-key` | S3 access key | No | |
| `object-storage-secret-key` | S3 secret key | No | |
| `sqlite-db-path` | SQLite database file path | No | |
| `litestream-replicate-interval` | Backup interval | No | `10s` |

## Application Requirements

### Python Applications

Your Python application should:

1. Have a requirements file with dependencies (default: `requirements.txt`)
2. Have a main entry point (default: `app.py`)
3. Listen on the port specified by the `PORT` environment variable

You can specify a custom requirements file path using the `requirements-file` parameter:
```yaml
requirements-file: config/requirements.txt
```

Example `app.py`:
```python
import os
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Python!'

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 8080))
    app.run(host='0.0.0.0', port=port)
```

### Go Applications

Your Go application should:

1. Have a `go.mod` file
2. Listen on the port specified by the `PORT` environment variable

## Custom Dockerfile

If you prefer to use your own Dockerfile, set `use-repository-dockerfile: true` and place a `Dockerfile` in your repository root (or in the `app-dir` if specified).

## Outputs

| Output | Description |
|--------|-------------|
| `public-url` | Public URL of the deployed application |
| `app-id` | AppRun application ID |