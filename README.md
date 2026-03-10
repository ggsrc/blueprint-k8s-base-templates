# blueprint-k8s-base-templates

Welcome to the blueprint-k8s-base-templates repository! This repository provides foundational Kustomize configurations for deploying Kubernetes-hosted applications. Use these base files as a starting point for creating environment-specific overlays that ensure consistent, scalable, and secure deployments across your infrastructure.

## Overview

This repository offers a collection of reusable resources designed with best practices in mind. The base templates include:

- **Deployment**: Configured with health checks (liveness and readiness probes), resource limits, and environment variable sourcing using ConfigMaps.
- **Horizontal Pod Autoscaler (HPA)**: Automatically scales your application based on CPU and memory utilization.
- **Service**: Exposes your application with proper port configurations (HTTP REST, HTTP, gRPC, and Prometheus metrics).
- **Istio VirtualService**: Provides advanced traffic routing and retry policies to enhance resilience.
- **Service Account**: Implements RBAC for secure application access.
- **ConfigMap**: Allows dynamic configuration injection.
- **ServiceMonitor**: For Prometheus integration and monitoring of application metrics.

## Usage

Reference the base templates remotely in your `kustomization.yaml` using a pinned version tag:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: my-app-namespace

resources:
  - github.com/ggsrc/blueprint-k8s-base-templates//base?ref=v1.1.1

commonLabels:
  app: my-app

patches:
  - patch: |-
      - op: replace
        path: /metadata/name
        value: my-app
    target:
      name: APP_NAME
```

Always pin to a specific version tag (e.g. `?ref=v1.1.1`) to avoid unexpected changes.

## Customization

- **Placeholders:** The base templates use placeholders like `APP_NAME` and `APP_IMAGE`. Substitute these values in your overlays using patches.
- **Resource Settings:** Adjust resource requests, limits, and probe configurations as required.
- **Port Definitions:** The service configuration exposes multiple ports:
  - HTTP REST: 3000
  - HTTP: 8080
  - gRPC: 9090
  - Prometheus Metrics: 4014

## Security Context (v1.1.1+)

The deployment template enforces pod-level security by default:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 65532      # distroless nonroot user
  runAsGroup: 65532
  fsGroup: 65532
containers:
  - securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: [ALL]
```

**Note:** Services that need a writable filesystem (e.g. Node.js) must add an emptyDir volume for `/tmp`:

```yaml
- op: add
  path: /spec/template/spec/volumes
  value: [{name: tmp, emptyDir: {}}]
- op: add
  path: /spec/template/spec/containers/0/volumeMounts
  value: [{name: tmp, mountPath: /tmp}]
```

## Versions

| Tag | Changes |
|-----|---------|
| v1.0.0 | Initial base templates |
| v1.1.0 | Add `runAsNonRoot: true` (pod securityContext) |
| v1.1.1 | Add `runAsUser/runAsGroup/fsGroup: 65532`, container securityContext |

## Features

- **Security:** Pod and container security contexts enforced by default (non-root, read-only filesystem, dropped capabilities)
- **Auto-Scaling:** HPA adjusts replicas based on CPU and memory utilization
- **Resilience:** Liveness and readiness probes on port 8080
- **Observability:** ServiceMonitor for Prometheus integration
- **Traffic Management:** Istio VirtualService with routing and retries
- **Modularity:** Designed for remote references with version pinning

## Requirements

- Kubernetes v1.16 or higher
- Kustomize v3.0 or higher
- Istio (for VirtualService and service mesh capabilities)
- Prometheus Operator (for ServiceMonitor integration)

## Contributing

Contributions are welcome! If you have suggestions or improvements, please open an issue or submit a pull request. Make sure to follow the repository's coding and contribution guidelines.

## License

This project is licensed under the Apache License 2.0. For more details, please see the [LICENSE](LICENSE) file.

## Additional Resources

- [Kustomize Documentation](https://kubectl.docs.kubernetes.io/)
- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Istio Documentation](https://istio.io/latest/docs/)
