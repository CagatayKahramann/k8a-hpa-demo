# k8s-hpa-demo

## Overview

This project demonstrates how to set up and test Horizontal Pod Autoscaling (HPA) in Kubernetes by using a custom Docker image with stress tools. We create Alpine pods with stress-ng installed, configure HPA, and test resource scaling based on CPU and memory utilization.

## Description

This project shows how to deploy an Alpine pod with stress-ng to simulate load and test the HPA functionality in Kubernetes. The setup involves creating a custom Docker image with stress-ng, deploying the image as a pod, configuring HPA to scale pods based on resource usage, and observing the scaling behavior. We will use Minikube for demonstration purposes, but you can use your own Kubernetes cluster as well.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Basic Configuration of Minikube](#basic-configuration-of-minikube)
- [Setup](#setup)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Prerequisites

Before starting, ensure you have the following:

- **Kubernetes Cluster**: A running Kubernetes cluster where you can deploy the manifests. You can use Minikube for local development or any other Kubernetes cluster.

- **Kubectl**: The command-line tool for interacting with Kubernetes. Install it by following the instructions [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

- **Docker Engine**: Required for building and running custom Docker images. Ensure Docker Engine is installed and running. You can find installation instructions [here](https://docs.docker.com/get-docker/).

- **Metrics Server**: Required for resource monitoring. We will use Minikube's metrics server addon by default, but if you encounter issues, you might need to manually install and configure it. For manual installation or troubleshooting, refer to the official [Metrics Server documentation](https://github.com/kubernetes-sigs/metrics-server). Ensure to add `--kubelet-insecure-tls` to the arguments in the metrics server deployment if needed.

## Basic Configuration of Minikube

If you decide to use Minikube, follow these steps to set up your environment:

1. **Start Minikube**:
   ```bash
   minikube start
   ```
2. **Enable Metrics Server**:
   Enable the metrics server addon within Minikube:
   ```bash
   minikube addons enable metrics-server
   ```

## Setup

**Important Note**: During the setup process, it is crucial to apply the manifests in the correct order, one by one, to ensure proper configuration.

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-username/k8s-stress-testing.git
   cd k8s-stress-testing
   ```

2. **Deploy the Stress Testing Pod**:
   
   Apply the Deployment manifest to create the stress testing pods:
   ```bash
   kubectl apply -f stress-test-deployment.yaml
   ```

3. **Create the HPA**:
   
   Apply the HPA manifest to set up autoscaling:
   ```
   kubectl apply -f stress-test-hpa.yaml
   ```

4. **Install Metrics Server(Optional)**:
   
   If you have not enabled the metrics server yet, run:
   ```bash
   minikube addons enable metrics-server
   ```

5. **Start Stress Testing**:
   
   First, retrieve the name of the stress pod:
   ```bash
   kubectl get pods -l app=stress-test-deployment
   ```
   Once you have the pod name, run the following command to access the stress pod and initiate the stress testing:
   ```bash
   kubectl exec -it <stress-pod-name> -- stress-ng --cpu 2 --vm 2 --vm-bytes 128M --timeout 300s
   ```
   Here's what each argument does:

   - `--cpu 2`: Use 2 CPU stressors to stress the CPU.
   - `--vm 2`: Use 2 virtual memory stressors to stress the memory.
   - `--vm-bytes 128M`: Allocate 128 MB of memory to each virtual memory stressor.
   - `--timeout 300s`: Run the stress test for 300 seconds (5 minutes).

   For more details on these parameters and to customize the stress test, refer to the official [stress-ng documentation](https://github.com/ColinIanKing/stress-ng).

6. ## Monitor Changes

   Observe the scaling behavior and resource utilization with the following commands:

   ```bash
   kubectl get pods
   kubectl get rs
   kubectl get hpa
   ```

## Troubleshooting

- **Metrics Server Issues**: If the metrics server is not functioning, try restarting Minikube or manually redeploying the metrics server:

  ```bash
  minikube delete
  minikube start
  ```
  Or restart the metrics server deployment:
  ```bash
  kubectl rollout restart deployment/metrics-server -n kube-system
  ```
- **HPA Not Scaling**: HPA Not Scaling: Ensure that metrics are being collected properly and check the HPA configuration for correct target utilization values.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
