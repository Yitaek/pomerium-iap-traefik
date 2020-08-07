# Identity-Aware Proxy (IAP) with Pomerium and Traefik on Kubernetes
Identity-Aware Proxy (IAP) is a secure method to provide access to internal applications without the use of VPNs. The zero-trust security model builds on Google's [BeyondCorp](https://cloud.google.com/beyondcorp/) implementation of shifting security from network perimeters to individual users, determining access from user context. 

![Context-aware access: High-level architecture](https://storage.googleapis.com/gweb-cloudblog-publish/images/enabling_beyondcorp.max-1200x1200.png)

On Google Cloud, this can easily be achieved with [Cloud IAP](https://cloud.google.com/iap/) (see an example deployment for [securing Grafana on GKE](https://medium.com/google-cloud/practical-monitoring-with-prometheus-grafana-part-iv-d4f3f995cc78)). Outside of GCP, there are two great open-source solutions:

- [Pomerium](https://github.com/pomerium/pomerium)
- [ORY Oathkeeper](https://github.com/ory/oathkeeper)

A common use case for using an IAP like Pomerium is securing access to internal applications / tools (e.g. Kubernetes dashboard, ArgoCD, Kibana). This repo will walk through an example setup of Pomerium in conjunction with Traefik to add authentication and authorization to Kubernetes dashboard.

## Prerequisites
- Kubernetes Cluster (e.g. EKS)
- Helm v3 
- DNS provider (e.g. Cloudflare)
- Ingress controller (e.g. Traefik)
- Certificate manager (e.g. cert-manager)

I'm using EKS as an example (since GKE provides a managed dashboard), but other Kubernetes clusters also work (e.g. [k3s](https://k3s.io/)). You can also replace Traefik with an ingress controller of your choice (e.g. nginx). 

## High-Level Architecture
(include blurb about proxy vs forward auth)

## Deployment Guide
Assuming a barebones EKS installation, let's start by deploying `metrics-server` and `kubernetes-dashboard`.

### Kubernetes Dashboard

Installing `metrics-server`:
```
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml
```

Installing `kubernetes-dashboard`:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
```

While those components are being deployed, create an `eks-admin` service account and cluster role binding to connect to the dashboard with a token:

`eks-admin-service-account.yaml`

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-admin
  namespace: kube-system
```

Apply the service account and cluster role binding:

```
$ kubectl apply -f eks-admin-service-account.yaml
```

Now let's verify that we can log in:

1. Start the `kubectl proxy`: 
```
$ kubectl proxy
```

2. Grab the token:
```
$ kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}') --template={{.data.token}} | base64 -D | pbcopy
```

3. Open the link and paste the token:
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login
```

### Traefik 

### Cert-Manager

### Identity-Provider

### Pomerium

### DNS Setup 