# DevOps overview

This page shows where DevOps work lives and how changes and traffic move
through the system. Use the repository guides for commands and change steps.

## Repository ownership

| Repository | Owns |
| --- | --- |
| `disease-prediction-infra` | Terraform for OCI resources, including the VM, network, boot-volume backup, Vault and HSM key, access policy, quotas, and budget. Its README is the Terraform setup and change guide. |
| `disease-prediction-gitops` | K3s configuration, Argo CD, Helmfile, platform manifests, and host automation. Its bootstrap and operations guides are the cluster runbooks. |
| `disease-prediction-ci` | Shared GitHub Actions workflows and their pinned tool versions. Each application repository keeps its own pipeline file and commands. |
| Application repositories | Application source and service-specific setup. The CI workflows use the shared workflows where needed. |
| `disease-prediction-docs` | Cross-repository overviews, diagrams, and project model artifacts. |

## Provision and deploy

```mermaid
flowchart LR
  operator[Operator] -->|reviewed changes| infraRepo[Infrastructure repository]
  infraRepo --> terraform[Terraform CLI]
  terraform -->|OCI API| oci[OCI resources]
  terraform <-->|remote state| state[(Private Object Storage bucket)]
  oci --> vm[Oracle VM and K3s]

  operator -->|reviewed changes| gitRepo[GitOps repository]
  gitRepo -->|Helmfile installs| argocd[Argo CD]
  argocd -->|syncs checked-in manifests| k3s[K3s cluster]
  gitRepo -->|host manifests and Ansible| host[VM host configuration]

  appRepo[Application repositories] -->|application workflows| sharedCI[Shared CI workflows]
  sharedCI -->|checks| result[Workflow results]
  sharedCI -. image build when enabled .-> registry[Private GHCR]
```

The private, versioned remote state bucket is created separately from the
project Terraform stack. Argo CD reads the GitOps repository and syncs its checked-in cluster
configuration. The shared container workflow can build and publish images when
an application sets `CONTAINER_ENABLED` to `true` and provides a Dockerfile. It
does not mean an application is deployed. The GitOps repository currently has
no public product workload route.

K3s applies its host manifests for the Gateway API and Traefik CRDs. Argo CD
manages the rest of the checked-in cluster configuration. See the GitOps
[bootstrap guide](https://github.com/Disease-Prediction-ASET26/disease-prediction-gitops/blob/main/docs/bootstrap.md)
for the procedure.

## Administration and secrets

```mermaid
flowchart LR
  admin[Admin workstation] -->|Tailscale SSH and Kubernetes API| vm[Oracle VM]
  vm --> k3s[K3s]
  k3s --> eso[External Secrets Operator]
  eso -->|VM Instance Principal| vault[OCI Vault]
  eso -->|sync selected values| secrets[Kubernetes Secrets]
  secrets --> workloads[Workloads]
```

K3s encrypts Kubernetes Secrets at rest. The OCI Instance Principal is a VM-level
identity, not a pod identity. A pod that can reach the OCI metadata endpoint
may be able to use the VM's Vault permissions. Check this trust boundary before
you run an untrusted workload. See the GitOps [secrets runbook](https://github.com/Disease-Prediction-ASET26/disease-prediction-gitops/blob/main/docs/operations/secrets.md)
and [private access runbook](https://github.com/Disease-Prediction-ASET26/disease-prediction-gitops/blob/main/docs/operations/tailscale.md).

The OCI security list allows no inbound SSH, Kubernetes API, HTTP, or HTTPS
traffic. Use Tailscale for administration.

## Public traffic

```mermaid
flowchart LR
  browser[Browser] -->|HTTPS; TLS ends at Cloudflare| cloudflare[Cloudflare DNS and edge]
  cloudflare -->|Tunnel| tunnel[cloudflared]
  tunnel -->|HTTP inside cluster| traefik[Traefik Gateway]
  traefik --> route[Approved HTTPRoute]
  route -->|login portal| authelia[Authelia]
  route -->|native OIDC route| proxy[Argo CD web proxy]
  proxy --> argocd[Argo CD]
  route -. future approved route .-> service[Workload Service]
  service -. optional ForwardAuth .-> authelia
  externalDNS[ExternalDNS] -->|proxied CNAME and TXT; upsert-only| cloudflare
```

The Authelia portal and Argo CD UI have public hostnames. Argo CD uses native
OIDC with Authelia. No product workload route is published. Name.com is the
registrar and delegates DNS to Cloudflare. The Cloudflare Tunnel uses outbound
connectivity, so OCI exposes no inbound application ports. See the GitOps
[domain and ingress runbook](https://github.com/Disease-Prediction-ASET26/disease-prediction-gitops/blob/main/docs/operations/domain-ingress.md)
and [Authelia runbook](https://github.com/Disease-Prediction-ASET26/disease-prediction-gitops/blob/main/docs/operations/authelia.md).

## Where to make changes

- For OCI resources and Terraform state, use the
  [infrastructure guide](https://github.com/Disease-Prediction-ASET26/disease-prediction-infra/blob/main/README.md).
- For cluster bootstrap, access, secrets, routes, and recovery, use the
  [GitOps operations index](https://github.com/Disease-Prediction-ASET26/disease-prediction-gitops/blob/main/docs/operations/README.md).
- For shared workflow behavior and tool versions, use the
  [CI guide](https://github.com/Disease-Prediction-ASET26/disease-prediction-ci/blob/main/README.md)
  and [tool versions](https://github.com/Disease-Prediction-ASET26/disease-prediction-ci/blob/main/TOOL-VERSIONS.md).
