---
date: 2026-06-02
time: 18:40
project: [[konflux-pipeline-debugging/README]]
tags: [session, konflux-pipeline-debugging, imagepullbackoff, serviceaccount]
---

# Session: Fixed ImagePullBackOff with Service Account imagePullSecrets

## What was worked on

Konflux pipeline debugging - persistent ImagePullBackOff errors preventing container builds from completing.

## Accomplishments

- Identified root cause: Service account missing imagePullSecrets configuration for internal registry
- Added secret reference to the service account
- Pipeline now successfully pulls images from internal registry

## Discoveries & Learnings

**ImagePullBackOff in Konflux pipelines is often a service account permission issue, not an image availability problem.**

The pipeline was failing because the service account used by the build pods didn't have credentials (imagePullSecrets) to authenticate with the internal container registry. Adding the imagePullSecrets reference to the service account resolved the issue immediately.

## Next Steps

- [ ] Document this fix in a troubleshooting guide
- [ ] Check if other service accounts need similar configuration
- [ ] Create a checklist for new pipeline setup

## Blockers

None - issue resolved.

## Related

- Commands: `kubectl describe sa <service-account-name> -n <namespace>`
- Commands: `kubectl edit sa <service-account-name> -n <namespace>`
- Fix: Added `imagePullSecrets` section to service account YAML
