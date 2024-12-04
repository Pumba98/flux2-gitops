[//]: # "renovate: datasource=github-releases depName=k3s-io/k3s"
[![k3s](https://img.shields.io/badge/dynamic/yaml?url=https%3A%2F%2Fraw.githubusercontent.com%2FPumba98%2Fflux2-gitops%2Fmain%2Fapps%2Fsystem-upgrade-controller%2Fplans%2Fagent-plan.yaml&query=%24.spec.version&style=for-the-badge&logo=kubernetes&label=k8s&color=orange)](https://k3s.io/)
[![flux](https://img.shields.io/badge/GitOps-Flux-blue?style=for-the-badge&logo=git)](https://fluxcd.io/)
[![renovate](https://img.shields.io/badge/renovate-enabled-brightgreen?style=for-the-badge&logo=renovatebot)](https://github.com/renovatebot/renovate)

# k8s clusters backed by Flux v2

Kubernetes clusters using the [GitOps](https://www.weave.works/blog/what-is-gitops-really) tool [Flux](https://fluxcd.io/).  
The Git repository is the driving the state of the Kubernetes clusters.  
The awesome [Flux SOPS integration](https://toolkit.fluxcd.io/guides/mozilla-sops/) is used to encrypt secrets with [age](https://age-encryption.org/).

## 📂 Repository structure

The Git repository contains the following directories:

```sh
📁
├─📁 apps            # apps available for intallation
├─📁 cluster-apps    # kustomization and overlays for app installations per cluster
│  ├─📁 staging
│  └─📁 production
├─📁 charts          # helm chart repos
├─📁 configs         # configs per cluster
└─📁 base
   ├─📁 flux-system  # flux & gitops operator
   ├─📁 staging      # flux configuration per cluster
   └─📁 production   # flux configuration per cluster
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
| [Tandoor-Recipes](https://github.com/TandoorRecipes/recipes)                      | Recipe Manager and Meal Planner                               |
| [SFTPGo](https://sftpgo.com/)                                                     | Full-featured SFTP Server                                     |
| [Immich](https://immich.app/)                                                     | Photo and video management solution                           |
| [Memos](https://www.usememos.com/)                                                | A privacy-first, lightweight note-taking service              |
| [Syncthing](https://syncthing.net/)                                               | Continuous file synchronization service                       |
| [Radicale](https://radicale.org/)                                                 | CalDAV and CardDAV Server                                     |
| [Rancher](https://rancher.com/products/rancher)                                   | Kubernetes Management Dashboard                               |
| [Homer](https://github.com/bastienwirtz/homer)                                    | Static dashboard for the cluster applications                 |
| [Bind](https://www.isc.org/bind/)                                                 | Full-featured DNS System                                      |
| [Blocky](https://0xerr0r.github.io/blocky/latest/)                                | Fast and lightweight DNS proxy as ad-blocker                  |
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
sops -d ./base/flux-system/init/flux-sops-age-secret.sops.yaml | kubectl apply -f -
sops -d ./base/flux-system/init/flux-secret.sops.yaml | kubectl apply -f -
kubectl apply --kustomize=./base/flux-system
kubectl apply --kustomize=./base/staging
```

1. Pre-create the `flux-system` namespace

```sh
kubectl create namespace flux-system --dry-run=client -o yaml | kubectl apply -f -
```

4. Add the Flux age key in-order for Flux to decrypt SOPS secrets

```sh
sops -d ./base/flux-system/init/flux-sops-age-secret.sops.yaml | kubectl apply -f -
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

<details>
    <summary>:construction: PostgreSQL Major Version Upgrade</summary>
<br>

I now use a [tianon/postgres-upgrade](https://github.com/tianon/docker-postgres-upgrade) init-container for PostgreSQL Major Upgrades.

**Always take backups, dataloss is possible**. Old data gets removed and replaced by output of pg_upgrade.

```yaml
## Init Container for Major PostgreSQL Upgrades, not needed permanently
initContainers:
- name: pg-upgrade
  image: tianon/postgres-upgrade:15-to-16
  securityContext:
    runAsUser: 0
    runAsGroup: 0
  volumeMounts:
  - name: data
    mountPath: /bitnami/postgresql
  env:
  - name: "PG_OLD"
    value: "15"
  - name: "PG_NEW"
    value: "16"
  command:
  - /bin/bash
  - -c
  - |
    if [[ $(< /bitnami/postgresql/data/PG_VERSION) -eq $PG_NEW ]]; then echo "PostgreSQL is already up2date"; exit 0; fi
    if [[ "$PGBINOLD" != "/usr/lib/postgresql/$PG_OLD/bin" ]]; then echo "Wrong postgres-upgrade image"; exit -1; fi
    if [[ "$PGBINNEW" != "/usr/lib/postgresql/$PG_NEW/bin" ]]; then echo "Wrong postgres-upgrade image"; exit -1; fi
    echo "Upgrading PostgreSQL from $PG_OLD to $PG_NEW"
    cp -r /bitnami/postgresql/data /var/lib/postgresql/$PG_OLD
    usermod -u 1001 postgres
    groupmod -g 1001 postgres
    chown -R postgres:postgres /var/lib/postgresql
    su postgres -c 'PGDATA="$PGDATANEW" eval "initdb $POSTGRES_INITDB_ARGS"'
    cp -p $PG_NEW/data/postgresql.conf $PG_NEW/data/pg_hba.conf $PG_OLD/data/
    chmod 700 $PG_OLD/data
    gosu postgres pg_upgrade
    gosu postgres pg_ctl -D /var/lib/postgresql/$PG_NEW/data -l logfile start
    gosu postgres /usr/lib/postgresql/$PG_NEW/bin/vacuumdb --all --analyze-in-stages
    gosu postgres pg_ctl -D /var/lib/postgresql/$PG_NEW/data -l logfile stop
    rm /var/lib/postgresql/$PG_NEW/data/pg_hba.conf /var/lib/postgresql/$PG_NEW/data/postgresql.conf
    rm -rf /bitnami/postgresql/data
    mv /var/lib/postgresql/$PG_NEW/data /bitnami/postgresql/
```

<details>
    <summary>:construction: Previous Manual Upgrade Approach</summary>
<br>

Based on [https://github.com/bitnami/charts/issues/1798#issuecomment-699056263](https://github.com/bitnami/charts/issues/1798#issuecomment-699056263)

1. Preparation

```sh
export NAMESPACE=selfhosted
export APPLICATION_DEPLOYMENT=paperless-ngx
export POSTGRES_DEPLOYMENT=${APPLICATION_DEPLOYMENT}-postgresql
export POSTGRES_MAJOR_VERSION=12
export POSTGRES_PVC_SIZE=4Gi
export POSTGRES_DB=${APPLICATION_DEPLOYMENT}
export POSTGRES_USERNAME=${APPLICATION_DEPLOYMENT}
export POSTGRES_PASSWORD=yourSecretPassword
```

Note: `POSTGRES_MAJOR_VERSION` is the helm version, not postgresql version.

2. Scale down application that uses the database

```sh
kubectl scale deployment ${APPLICATION_DEPLOYMENT} -n ${NAMESPACE} --replicas 0
```

3. Deploy new major version of the database

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install ${POSTGRES_DEPLOYMENT}-upgrade bitnami/postgresql --version ${POSTGRES_MAJOR_VERSION} -n ${NAMESPACE} --wait \
    --set auth.username=${POSTGRES_USERNAME} \
    --set auth.password=${POSTGRES_PASSWORD} \
    --set auth.database=${POSTGRES_DB} \
    --set primary.persistence.size=${POSTGRES_PVC_SIZE}
```

4. Migrate data to new postgresql deployment

```sh
kubectl exec -it ${POSTGRES_DEPLOYMENT}-upgrade-0 -n ${NAMESPACE} -- bash -c "export PGPASSWORD=${POSTGRES_PASSWORD}; time pg_dump -h ${POSTGRES_DEPLOYMENT} -U ${POSTGRES_USERNAME} | psql -U ${POSTGRES_USERNAME}"
```

From here on you have multiple possibilities, e.g. just use your app with the new db deployment.

I personally prefer to backup the volume of the new DB, uninstall both database deployments & delete their PVCs.

After that I restore the backup of the PVC with the name of the old database deployment & upgrade my Helmrelease version.

```sh
helm uninstall ${POSTGRES_DEPLOYMENT}-upgrade -n ${NAMESPACE}
kubectl scale sts ${POSTGRES_DEPLOYMENT} -n ${NAMESPACE} --replicas 0
```

The volume deletion and restore is done in Longhorn UI. Afterwards helm upgrade for the postgresql deployments can be done.

5. Scale up application that uses the database

```sh
kubectl scale deployment ${APPLICATION_DEPLOYMENT} -n ${NAMESPACE} --replicas 1
```

</details>
</details>