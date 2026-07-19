---
project: [[konflux-pipeline-debugging/README]]
tags: [lessons, konflux-pipeline-debugging]
---

# Lessons Learned: Konflux Pipeline Debugging

{Discoveries and learnings will be documented here as debugging work progresses}

## ImagePullBackOff Caused by Missing Service Account imagePullSecrets - 2026-06-02

Konflux pipelines failed with ImagePullBackOff errors when trying to pull images from the internal registry. The root cause was that the service account used by build pods didn't have imagePullSecrets configured for registry authentication.

**Context:** Spent 2 hours debugging pipeline failures. Initially suspected image availability or network issues, but the real problem was service account permissions.

**Impact:** This is a common setup oversight that can block all container builds. Adding imagePullSecrets to the service account configuration resolves it immediately.

**Related:** [[01_Projects/konflux-pipeline-debugging/Sessions/2026-06-02-imagepullbackoff-serviceaccount-fix|Session notes]]

^discovery-20260602
