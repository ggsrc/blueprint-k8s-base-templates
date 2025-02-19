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

These base templates are designed to be extended using Kustomize overlays. Follow these steps to integrate them into your deployment pipeline:

1. **Clone the Repository**

   ```bash
   git clone https://github.com/your-org/blueprint-k8s-base-templates.git
   cd blueprint-k8s-base-templates
   ```

2. **Create an Overlay**

   In your overlay directory (e.g., `overlays/my-app`), create a `kustomization.yaml` that references the base templates. For example:

   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   namespace: my-app-namespace

   resources:
     - ../../base

   commonLabels:
     app: my-app

   patches:
     - patch: |-
         - op: replace
           path: /metadata/name
           value: my-app
       target:
         name: APP_NAME
     - patch: |-
         - op: replace
           path: /spec/template/spec/containers/0/resources/requests/cpu
           value: "2"
         - op: replace
           path: /spec/template/spec/containers/0/resources/limits/cpu
           value: "4"
       target:
         kind: Deployment
         name: APP_NAME

   generatorOptions:
     disableNameSuffixHash: true
   ```

3. **Deploy to Kubernetes**

   Use `kubectl` along with Kustomize to deploy your configuration:

   ```bash
   kubectl apply -k overlays/my-app
   ```

## Customization

- **Placeholders:** The base templates use placeholders like `APP_NAME` and `APP_IMAGE`. Substitute these values in your overlays using patches.
- **Resource Settings:** Adjust resource requests, limits, and probe configurations as required.
- **Port Definitions:** The service configuration exposes multiple ports:
  - HTTP REST: 3000
  - HTTP: 8080
  - gRPC: 9090
  - Prometheus Metrics: 4014

## Features

- **Auto-Scaling:** The Horizontal Pod Autoscaler adjusts replicas based on your application's CPU and memory utilization.
- **Resilience:** Liveness and readiness probes ensure the health of your application by periodically checking its status.
- **Observability:** The integrated ServiceMonitor makes it easy to connect to Prometheus for monitoring purposes.
- **Traffic Management:** Istio VirtualService provides advanced routing capabilities, including retries and fault tolerance.
- **Modularity:** Base templates are designed to be easily extended and customized with overlay configurations.

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
