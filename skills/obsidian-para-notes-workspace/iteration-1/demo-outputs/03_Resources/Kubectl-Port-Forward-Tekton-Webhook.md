---
title: Kubectl Port-Forward for Tekton Webhook
tags: [resource, kubectl, tekton, networking]
created: 2026-06-02
updated: 2026-06-02
---

# Kubectl Port-Forward for Tekton Webhook

Use port 8080 (not 80) when port-forwarding to Tekton webhook services.

## Key Points

- Tekton webhooks listen on port 8080 by default
- Using port 80 will result in connection failures
- This applies to both EventListener and Trigger services

## Examples

```bash
# Correct - forward to port 8080
kubectl port-forward svc/el-tekton-listener 8080:8080 -n tekton-pipelines

# Incorrect - port 80 will not work
kubectl port-forward svc/el-tekton-listener 8080:80 -n tekton-pipelines
```

## Related Resources

- [[Tekton Pipelines]]
- [[Kubectl Commands]]
