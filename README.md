# sacloud-apprun-action

GitHub Action to build and deploy Python applications to Sakura AppRun.

## Features

- ✅ Python application support
- ✅ Optimized Docker builds
- ✅ Container registry integration
- ✅ Litestream SQLite backup support
- ✅ Zero-downtime deployments
- ✅ Custom requirements file path

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

### Custom Requirements File

```yaml
- name: Deploy Python App with Custom Requirements
  uses: your-username/sacloud-apprun-action@v1
  with:
    requirements-file: config/requirements.txt  # Custom path
    sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
    # ... other parameters
```

### Custom Main File

```yaml
- name: Deploy with Custom Main File
  uses: your-username/sacloud-apprun-action@v1
  with:
    main-file: server.py  # Custom main file
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
    sqlite-db-folder: /app/data
```

## Input Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
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
| `main-file` | Main Python file to run | No | `app.py` |

### Litestream Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `object-storage-bucket` | S3 bucket for SQLite backup | No | |
| `object-storage-access-key` | S3 access key | No | |
| `object-storage-secret-key` | S3 secret key | No | |
| `sqlite-db-folder` | SQLite database folder path (entire folder will be backed up as tar archive) | No | |
| `litestream-replicate-interval` | Backup interval | No | `10s` |

## Application Requirements

Your Python application should:

1. Have a requirements file with dependencies (default: `requirements.txt`)
2. Have a main entry point (default: `app.py`)
3. Listen on the port specified by the `PORT` environment variable

You can customize file paths using these parameters:
- `requirements-file`: Custom requirements file path (e.g., `config/requirements.txt`)
- `main-file`: Custom main file (e.g., `server.py`, `main.py`)

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

## Custom Dockerfile

If you prefer to use your own Dockerfile, set `use-repository-dockerfile: true` and place a `Dockerfile` in your repository root (or in the `app-dir` if specified).

## Outputs

| Output | Description |
|--------|-------------|
| `public-url` | Public URL of the deployed application |
| `app-id` | AppRun application ID |