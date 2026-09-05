Multi-cloud SRE. Kubernetes-first, opinionated about observability. Currently exploring new roles.

- Portfolio: [sre.dstepanov.dev](https://sre.dstepanov.dev)
- Live dashboards (kiosk rotation): [grafana.dstepanov.dev](https://grafana.dstepanov.dev/playlists/play/adhxnsb?kiosk)

## Recent portfolio work

Each demo is designed for same-day apply/demo/destroy, with full walkthroughs, Mermaid architecture diagrams, screenshots, and CI-verified end-to-end runs.

| Project | Stack | Highlights |
|---------|-------|------------|
| [terraform-eks-platform](https://github.com/bibigon14/terraform-eks-platform) | AWS, EKS, Terraform, OIDC | EKS bootstrap with GitHub OIDC and a hand-rolled IRSA module |
| [terraform-gke-platform](https://github.com/bibigon14/terraform-gke-platform) | GCP, GKE, Terraform, WIF | GKE with Workload Identity Federation; real race-condition bug caught by CI |
| [terraform-vault-platform](https://github.com/bibigon14/terraform-vault-platform) | AWS, Vault, Raft, KMS | HA Vault cluster on EC2 with KMS auto-unseal and Raft integrated storage |
| [k8s-blue-green-deploy](https://github.com/bibigon14/k8s-blue-green-deploy) | k3s, Argo Rollouts, Go | Blue-green with prePromotionAnalysis smoke tests on a Raspberry Pi cluster |

## Homelab

Raspberry Pi 5 running k3s, self-hosted observability, and various experiments.

- [homelab-observability](https://github.com/bibigon14/homelab-observability) - textfile-collector exporters plus a blameless postmortem on a Pi-hole v5-to-v6 API break
- [homelab-k3s](https://github.com/bibigon14/homelab-k3s) - docker-compose to k3s migration on a Pi
