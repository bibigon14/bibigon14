Multi-cloud SRE. Kubernetes-first, opinionated about observability. Currently exploring new roles.

- Portfolio: [sre.dstepanov.dev](https://sre.dstepanov.dev)
- Live dashboards (kiosk rotation): [grafana.dstepanov.dev](https://grafana.dstepanov.dev/playlists/play/adhxnsb?kiosk)

## Recent portfolio work

Each demo is designed for same-day apply/demo/destroy, with full walkthroughs, Mermaid architecture diagrams, screenshots, and CI-verified end-to-end runs.

<table>
<thead>
<tr>
  <th width="240">Project</th>
  <th width="240">Stack</th>
  <th>Highlights</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="https://github.com/bibigon14/ebpf-tcp-observer">ebpf-tcp-observer</a></td>
  <td>Go, cilium/ebpf, WireGuard, Prometheus</td>
  <td>Kernel-level TCP retransmit observer via a kprobe on <code>tcp_retransmit_skb</code>, scraped from homelab Prometheus over a reverse-initiated WireGuard tunnel from an Oracle Cloud Ampere A1 edge VM</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/chaos-scheduler-operator">chaos-scheduler-operator</a></td>
  <td>Go, kubebuilder, controller-runtime, Prometheus</td>
  <td>K8s operator running scheduled chaos with a Prometheus SLO guardrail that aborts when the error budget is already burning</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/terraform-eks-platform">terraform-eks-platform</a></td>
  <td>AWS, EKS, Terraform, OIDC</td>
  <td>EKS bootstrap with GitHub OIDC and a hand-rolled IRSA module</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/terraform-gke-platform">terraform-gke-platform</a></td>
  <td>GCP, GKE, Terraform, WIF</td>
  <td>GKE with Workload Identity Federation; real race-condition bug caught by CI</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/terraform-vault-platform">terraform-vault-platform</a></td>
  <td>AWS, Vault, Raft, KMS</td>
  <td>HA Vault cluster on EC2 with KMS auto-unseal and Raft integrated storage</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/k8s-blue-green-deploy">k8s-blue-green-deploy</a></td>
  <td>k3s, Argo Rollouts, Go</td>
  <td>Blue-green with prePromotionAnalysis smoke tests on a Raspberry Pi cluster</td>
</tr>
</tbody>
</table>
