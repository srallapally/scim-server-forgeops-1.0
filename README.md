# SCIM 2.0 Server for PingOne Advanced Identity Cloud - ForgeOps Deployment Package

## Package Contents

This package contains everything needed to deploy the SCIM 2.0 Server into a ForgeOps-managed PingIdentity platform.

**Included Files:**
- `scim-server-1.0.0.tar` - Docker image (load this first)
- `kustomize/` - Kubernetes manifests
- `docs/` - Detailed documentation

## Quick Start for Your DevOps Team

### Step 1: Load the Docker Image
```bash
# Load the Docker image into your cluster's container registry
docker load -i scim-server-1.0.0.tar

# Verify the image loaded
docker images | grep scim-server

# Tag for your registry (if using a private registry)
docker tag scim-server:1.0.0 your-registry.com/scim-server:1.0.0

# Push to your registry
docker push your-registry.com/scim-server:1.0.0
```

### Step 2: Copy Manifests to ForgeOps
```bash
# Assuming you're in the scim-server-deployment directory
# and your ForgeOps repo is at /path/to/forgeops

# Copy base manifests
cp -r kustomize/base/scim-server /path/to/forgeops/kustomize/base/

# Copy dev overlay
cp -r kustomize/overlays/dev/scim-server /path/to/forgeops/kustomize/overlay/dev/

# Copy prod overlay
cp -r kustomize/overlays/prod/scim-server /path/to/forgeops/kustomize/overlay/prod/
```

### Step 3: Configure OAuth Client

**You MUST create an OAuth App with Client Credentials grant in your Alpha realm tenant before deploying.**

@TODO See `docs/OAUTH_SETUP.md` for detailed instructions.

### Step 4: Update Secrets

Edit the file: `/path/to/forgeops/kustomize/overlay/dev/scim-server/kustomization.yaml`

Find this section:
```yaml
secretGenerator:
  - name: scim-server-secrets
    behavior: replace
    literals:
      - oauth-client-id=CUSTOMER_WILL_REPLACE_THIS
      - oauth-client-secret=CUSTOMER_WILL_REPLACE_THIS
```

Replace `CUSTOMER_WILL_REPLACE_THIS` with your actual OAuth client ID and secret.

### Step 5: Update URLs

Edit the same file and update these URLs for your environment:
```yaml
configMapGenerator:
  - name: scim-server-config
    behavior: replace
    literals:
      - pingidm-base-url=http://idm.dev.svc.cluster.local:80/openidm
      - oauth-token-url=http://am.dev.svc.cluster.local:80/am/oauth2/realms/root/access_token
      - scim-server-base-url=https://scim.dev.yourcompany.com
```

Change:
- `idm.dev.svc.cluster.local` to your IDM service name
- `am.dev.svc.cluster.local` to your AM service name
- `scim.dev.yourcompany.com` to your public SCIM URL

### Step 6: Update Ingress Hostname

In the same file, find:
```yaml
patchesJson6902:
  - target:
      ...
    patch: |-
      - op: replace
        path: /spec/rules/0/host
        value: scim.dev.example.com
```

Change `scim.dev.example.com` to your actual hostname.

### Step 7: Deploy with Skaffold
```bash
cd /path/to/forgeops

# If using your own registry, update kustomize/base/scim-server/kustomization.yaml
# Change the image name under the 'images' section

# Deploy to dev
skaffold run -p dev

# Or for continuous development
skaffold dev -p dev
```

### Step 8: Verify Deployment
```bash
# Check if pod is running
kubectl get pods -n dev -l app=scim-server

# Check logs
kubectl logs -n dev -l app=scim-server -f

# Port forward to test locally
kubectl port-forward -n dev svc/scim-server 8080:8080

# Test health endpoint
curl http://localhost:8080/health

# Test SCIM endpoint
curl http://localhost:8080/scim/v2/ServiceProviderConfig
