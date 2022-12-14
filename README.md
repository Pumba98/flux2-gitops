[//]: # "renovate: datasource=github-releases depName=k3s-io/k3s"
[![k3s](https://img.shields.io/badge/k8s-v1.24.9+k3s1-orange?style=for-the-badge&logo=kubernetes)](https://k3s.io/)
[![flux](https://img.shields.io/badge/GitOps-Flux-blue?style=for-the-badge&logo=git)](https://fluxcd.io/)
[![renovate](https://img.shields.io/badge/renovate-enabled-brightgreen?style=for-the-badge&logo=renovatebot)](https://github.com/renovatebot/renovate)

# k8s clusters backed by Flux v2

Kubernetes clusters using the [GitOps](https://www.weave.works/blog/what-is-gitops-really) tool [Flux](https://fluxcd.io/).  
The Git repository is the driving the state of the Kubernetes clusters.  
The awesome [Flux SOPS integration](https://toolkit.fluxcd.io/guides/mozilla-sops/) is used to encrypt secrets with gpg.

## 📂 Repository structure

The Git repository contains the following directories:

```sh
📁
├─📁 apps
│  ├─📁 all          # apps available for intallation
│  └─📁 production   # kustomization and overlays for app installations per cluster
├─📁 base
│  ├─📁 flux-system  # flux & gitops operator
│  └─📁 production   # flux configuration per cluster
├─📁 charts          # helm chart repos
├─📁 config          # configs per cluster
└─📁 crds            # custom resources required by apps
```

## :computer:&nbsp; Software

The following apps are installed on the clusters.

| Software                                                                          | Purpose                                                       |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| [Flux2](https://fluxcd.io)                                                        | GitOps Tool managing the cluster                              |
| [Longhorn](https://longhorn.io)                                                   | Persistent Block Storage Provisioner                          |
| [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx)            | Cluster Ingress controller                                    |
| [MetalLB](https://metallb.universe.tf)                                            | Bare metal LoadBalancer                                       |
| [Cert-Manager](https://cert-manager.io)                                           | Letsencrypt certificates with Cloudflare DNS                  |
| [ExternalDNS](https://github.com/kubernetes-sigs/external-dns)                    | Configure Cloudflare DNS Servers                              |
| [kube-vip](https://github.com/kube-vip/kube-vip)                                  | Virtual IP Load-Balancer for Control Plane High Availability  |
| [Kube-Prometheus Stack](https://github.com/prometheus-operator/kube-prometheus)   | Prometheus & Exporters to monitor the cluster                 |
| [Grafana](https://grafana.com)                                                    | Monitoring & Logging Dashboard                                |
| [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager)           | Monitoring Alerts                                             |
| [Grafana Loki](https://grafana.com/oss/loki)                                      | Log aggregation system                                        |
| [System Upgrade Controller](https://github.com/rancher/system-upgrade-controller) | Automated k3s upgrades                                        |
| [Weave GitOps](https://www.weave.works/product/gitops/)                           | Powerful WebUI extension to Flux for deployment insights      |
| [MinIO](https://min.io/)                                                          | Amazon S3 compatible high Performance Object Storage          |
| [Authelia](https://www.authelia.com)                                              | SSO & 2FA authentication server for Cluster Web Apps          |
| [Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx)                   | Document management system                                    |
| [Rancher](https://rancher.com/products/rancher)                                   | Kubernetes Management Dashboard                               |
| [Homer](https://github.com/bastienwirtz/homer)                                    | Static dashboard for the cluster applications                 |
| [Pod-Gateway](https://github.com/k8s-at-home/pod-gateway)                         | Route traffic through a VPN gateway                           |
| [Descheduler](https://github.com/kubernetes-sigs/descheduler)                     | Evicts pods to optimize scheduling                            |
| [Goldilocks](https://github.com/FairwindsOps/goldilocks)                          | Utility to help identifying good resource requests and limits |
| [X.509 Certificate Exporter](https://github.com/enix/x509-certificate-exporter)   | A Prometheus exporter to monitor x509 certificates            |
| [SMB CSI Driver](https://github.com/kubernetes-csi/csi-driver-smb)                | CSI Driver for SMB                                            |
| [kured](https://github.com/kubereboot/kured)                                      | Kubernetes Reboot Daemon (only used for Monitoring)           |
| [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server)    | Source of container resource metrics for Kubernetes           |

## :robot:&nbsp; Automation

[Renovate](https://www.whitesourcesoftware.com/free-developer-tools/renovate) Bot makes sure the components are never outdated.

It creates PullRequests when Helm charts or Docker images have newer versions available and even keeps Flux and k3s up-to-date.

## :handshake:&nbsp; Thanks

Big shout out to [k8s@home](https://github.com/k8s-at-home) and everyone from [awesome-home-kubernetes](https://github.com/k8s-at-home/awesome-home-kubernetes) for the inspiration :heart:

## :open_book:&nbsp; Notes

<details>
    <summary>:round_pushpin: Installation Notes</summary>
<br>

**tl;dr**
```
kubectl create namespace flux-system --dry-run=client -o yaml | kubectl apply -f -
sops -d ./base/flux-system/init/flux-sops-gpg-secret.sops.yaml | kubectl apply -f -
sops -d ./base/flux-system/init/flux-secret.sops.yaml | kubectl apply -f -
kubectl apply --kustomize=./base/flux-system
kubectl apply --kustomize=./base/staging
```

1. Pre-create the `flux-system` namespace

```sh
kubectl create namespace flux-system --dry-run=client -o yaml | kubectl apply -f -
```

4. Add the Flux GPG key in-order for Flux to decrypt SOPS secrets

```sh
sops -d ./base/flux-system/init/flux-sops-gpg-secret.sops.yaml | kubectl apply -f -
```

5. (Optional) Add the Flux SSH key in-order for Flux to pull private git repositories

```sh
sops -d ./base/flux-system/init/flux-secret.sops.yaml | kubectl apply -f -
```

5. Install Flux

```sh
kubectl apply --kustomize=./base/flux-system
```

6. Configure Flux

```sh
kubectl apply --kustomize=./base/staging
```

</details>
