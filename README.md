# Kustomize Apply

Applies kustomize overlays to Kubernetes clusters with optional wait and verification.

## Features

- 🚀 **Direct apply** - kubectl apply with kustomize
- ⏳ **Wait for rollout** - Optional deployment monitoring
- 📦 **Namespace management** - Auto-create namespaces
- 🔍 **Dry run** - Preview changes before applying
- 📊 **Status reporting** - Detailed deployment status

## Usage

```yaml
- name: Deploy to cluster
  uses: KoalaOps/kustomize-apply@v1
  with:
    overlay_dir: deploy/overlays/production
    wait: true
    wait_timeout: 300
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `overlay_dir` | Path to kustomize overlay | ✅ | - |
| `namespace` | Target namespace | ❌ | from overlay |
| `dry_run` | Perform dry run only | ❌ | `false` |
| `wait` | Wait for resources to be ready | ❌ | `true` |
| `wait_timeout` | Wait timeout in seconds | ❌ | `300` |
| `wait_for_jobs` | Wait for jobs to complete | ❌ | `false` |
| `force` | Force apply (delete and recreate) | ❌ | `false` |
| `server_side` | Use server-side apply | ❌ | `false` |
| `validate` | Validate manifests against API schema | ❌ | `true` |
| `prune` | Prune resources not in manifests | ❌ | `false` |
| `prune_selector` | Label selector for pruning | ❌ | - |

## Outputs

| Output | Description |
|--------|-------------|
| `applied_resources` | List of applied resources |
| `namespace` | Namespace where resources were applied |
| `deployment_name` | Primary deployment name (if found) |

## Examples

### Basic deployment
```yaml
- name: Deploy application
  uses: KoalaOps/kustomize-apply@v1
  with:
    overlay_dir: deploy/overlays/staging
```

### With specific namespace
```yaml
- name: Deploy to namespace
  uses: KoalaOps/kustomize-apply@v1
  with:
    overlay_dir: deploy/overlays/dev
    namespace: development
```

### Dry run first
```yaml
- name: Preview changes
  uses: KoalaOps/kustomize-apply@v1
  with:
    overlay_dir: deploy/overlays/production
    dry_run: true

- name: Apply if approved
  if: ${{ inputs.approved == 'true' }}
  uses: KoalaOps/kustomize-apply@v1
  with:
    overlay_dir: deploy/overlays/production
```

### No wait (fire and forget)
```yaml
- name: Quick deploy
  uses: KoalaOps/kustomize-apply@v1
  with:
    overlay_dir: deploy/overlays/dev
    wait: false
```


### With pruning
```yaml
- name: Deploy with cleanup
  uses: KoalaOps/kustomize-apply@v1
  with:
    overlay_dir: deploy/overlays/staging
    prune: true
    prune_selector: app=myapp,env=staging
```

## Complete Workflow Example

```yaml
jobs:
  deploy:
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Login to cloud
        uses: KoalaOps/cloud-login@v1
        with:
          provider: aws
          region: us-east-1
          cluster: production
      
      - name: Update image
        uses: KoalaOps/kustomize-edit@v1
        with:
          overlay_dir: deploy/overlays/production
          image: backend
          new_tag: ${{ inputs.tag }}
      
      - name: Apply to cluster
        uses: KoalaOps/kustomize-apply@v1
        id: deploy
        with:
          overlay_dir: deploy/overlays/production
          wait: true
          wait_timeout: 300
      
      - name: Show status
        run: |
          echo "Deployed to: ${{ steps.deploy.outputs.namespace }}"
          echo "Deployment: ${{ steps.deploy.outputs.deployment_name }}"
```

## Resource Detection

The action automatically detects what to wait for:

1. **Deployments** - Waits for rollout to complete
2. **StatefulSets** - Waits for all pods ready
3. **DaemonSets** - Waits for desired number scheduled
4. **Jobs** - Waits for completion
5. **Services** - Waits for endpoints

## Error Handling

The action will fail if:
- Kustomize build fails
- kubectl apply fails
- Resources don't become ready within timeout

## Prerequisites

- kubectl configured with cluster access
- Appropriate RBAC permissions
- kustomize available (or kubectl 1.14+)

## Notes

- Use with cloud-login for authentication
- Supports kubectl 1.14+ (built-in kustomize)
- Respects resource ordering
- Handles CRDs correctly
- Works with Helm-generated manifests