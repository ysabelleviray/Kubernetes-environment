# Kubernetes Production Deployment Guide

This guide provides a step-by-step process for deploying and managing a production environment using Kubernetes. It covers creating namespaces, setting up `ConfigMaps` and `Secrets`, deploying a production application, and exposing it via a service.

## Steps to Set Up the Production Environment

### 1. **Create the Production Namespace**

```bash
kubectl create namespace production
```
### 2. **Configure configmaps**

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --namespace=production
```
