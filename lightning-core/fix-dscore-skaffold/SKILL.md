---
name: fix-dscore-skaffold
description: Fix dscore skaffold image mismatch by creating a manifest override with the correct local image
---

# Fix dscore skaffold

Fixes the issue where dscore skaffold fetches a deployment manifest from the cluster with a production image, preventing skaffold from replacing it with your locally built image.

## Usage

```bash
/fix-dscore-skaffold <service-name>
```

Example:

```bash
/fix-dscore-skaffold tooth-recognition-worker-20250401
```

## What this does

1. Runs `dscore skaffold` (may fail initially - this is expected)
2. Extracts the temporary manifest location and generated image tag from logs
3. Copies the temp manifest to `services/<service-name>/remotedev-manifest.yaml`
4. Updates the worker container image to use the locally built sandbox image
5. Re-runs dscore skaffold with `--manifest-override` to use the corrected manifest

## Steps

### 1. Parse the service name

Extract the service name from the arguments (required).

### 2. Run dscore skaffold

Run dscore skaffold in the background and capture both stdout and stderr:

```bash
dscore skaffold <service-name> -c lightning-d2 -n core -a kn 2>&1
```

This command may or may not fail. If it does not fail and continues running, kill the process before proceeding to the next step. The goal is to capture the output (which contains the temp manifest path and image tag) and then stop the process.

### 3. Extract from the output

Parse the output to find:
- Temporary manifest path: Look for `Temporary manifests written to: <path>`
- Generated sandbox image tag: Look for the pattern `artifactory.dentsplysirona.com/dpns-docker-local-sandbox/ephemeral/<service-name>:remotedev-<hash>`

### 4. Find the service directory

Extract the base service name (remove version suffix like "-20250401").

Service directory is: `services/<base-service-name>/`

### 5. Copy and fix the manifest

Copy the temporary manifest to the service directory:

```bash
cp <temp-manifest-path> services/<base-service-name>/remotedev-manifest.yaml
```

Then edit `remotedev-manifest.yaml` to replace the worker container image:
- Find the Deployment kind in the YAML
- Locate the container with `name: <base-service-name>` (not sidecar containers)
- Replace its `image:` field with the sandbox image tag from step 3

### 6. Run dscore with manifest override

Run dscore skaffold with the corrected manifest:

```bash
dscore skaffold <service-name> -c lightning-d2 -n core -a kn --manifest-override services/<base-service-name>/remotedev-manifest.yaml
```

This should now work correctly with the local image being deployed.

### 7. Cleanup (when user asks to stop skaffolding)

If the user asks to stop the skaffold process, you must also clean up the cluster resources:

1. Stop the background task (if running in background)
2. Delete the deployment and service from the cluster:

```bash
kubectl --context lightning-d2 delete deployment <service-name>-kn -n core
kubectl --context lightning-d2 delete service <service-name>-kn -n core
```

**Important**: Simply stopping the process with TaskStop does NOT trigger dscore's cleanup handlers (like Ctrl+C would). You must manually delete the resources from the cluster.

## Notes

- The manifest override file (`remotedev-manifest.yaml`) is a workaround and doesn't need to be committed
- This issue occurs because dscore fetches the existing deployment manifest from the cluster instead of generating it fresh from kustomize
- The fetched manifest has production registry images that don't match the local sandbox registry images
- By fixing the image name in the manifest, skaffold can properly match and replace it
