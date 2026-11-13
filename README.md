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
  <td><a href="https://github.com/bibigon14/multi-cloud-finops">multi-cloud-finops</a></td>
  <td>GCP BigQuery billing export (Standard + Detailed + FOCUS), AWS CUR 2.0 via Athena + SCAD pod-level attribution, Azure Cost Management via Grafana Infinity + OAuth2, OCI Usage API via custom HTTP Signatures in Cloudflare Workers, Cloudflare Analytics Engine SQL, Terraform, Grafana multi-datasource, 5-cloud FinOps, SRE burn-rate</td>
  <td><strong>Five clouds in one Grafana</strong>, SRE burn-rate alerts applied to spend against a <code>$10</code>/month portfolio budget. Five billing APIs in one dashboard: GCP BigQuery (Standard + Detailed + FOCUS), AWS CUR 2.0 via Athena with SCAD pod-level attribution, Azure ARM Cost Management via Infinity + OAuth2, OCI custom draft-cavage HTTP Signatures in <a href="https://github.com/bibigon14/sre-worker">sre-worker</a>'s WebCrypto (~130 lines, because Infinity doesn't speak the OCI auth scheme), Cloudflare Workers Paid paired to Analytics Engine SQL. K8s namespace attribution row across GCP + AWS + Azure (GKE Cost Allocation, AWS SCAD, Azure <code>ResourceGroupName</code> fallback - honest limitation inline). Roadmap: v0.6 z-score anomaly detection; v0.7 Databricks federation for Azure namespace.</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/gpu-observability-gke">gpu-observability-gke</a></td>
  <td>GCP, GKE, NVIDIA L4 (Ada Lovelace, 24 GB), DCGM, vLLM, Prometheus</td>
  <td>GCP mirror of gpu-observability-eks on a newer GPU. Two money-shots on a real L4 Spot: idle (vLLM resident 19.6 GB, <code>GPU_UTIL=0%</code>, 38 W) vs 20-stream load (same 19.6 GB, <code>GPU_UTIL=87%</code>, 70.5 W near the 72 W TDP, ~1000 tok/s). <code>FB_USED</code> does not move between states; every real signal does - the gap the <code>GPUAllocatedButIdle</code> alert catches. Four GKE quirks inline: <code>dcgm-exporter</code> securityContext vs GKE COS CDI (raw DaemonSet), <code>gke-resource-quotas</code> blocks <code>system-node-critical</code> outside kube-system, DCGM profiling fields fail on Spot L4 (NVML-only counters), Cluster Autoscaler marks <code>min=0</code> GPU nodes as DeletionCandidate within ~10 min.</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/chaos-scheduler-operator">chaos-scheduler-operator</a></td>
  <td>Go, kubebuilder, controller-runtime, Prometheus</td>
  <td>Scheduled fault injection with an SLO-aware abort. Custom <code>ChaosExperiment</code> CRD (<code>chaos.dstepanov.dev/v1alpha1</code>) with cron schedule, label selector, and optional Prometheus guardrail - the reconciler queries a PromQL expression before each run and aborts if the scalar crosses threshold, so chaos never fires while a service is already burning budget. Deployed to single-node k3s homelab via ArgoCD, killing pods across two namespaces on offset schedules. Validating admission webhook rejects bad CRs (bad cron, system namespaces, empty selector); native-arm64 multi-arch CI with buildx push-by-digest + imagetools merge (~25 min under QEMU down to ~3 min).</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/incident-timeline-generator">incident-timeline-generator</a></td>
  <td>Go CLI, stdlib only, kubectl + Prometheus ALERTS + git log ingesters</td>
  <td>Mechanical post-mortem timeline assembly - the typing-across-three-terminals step from writing <a href="https://github.com/bibigon14/inference-slo-stack/tree/main/docs/incidents">two production post-mortems</a>. CLI runs three queries over the same time window, merges by timestamp, renders the standard post-mortem Markdown with the Timeline table pre-populated and Root cause / Lessons blank for the human. Three independently-optional ingesters: <code>kubectl get events -o json</code>, <code>ALERTS{alertstate=&quot;firing&quot;}</code> over Prometheus <code>/query_range</code> with firing-episode detection (gap &gt; 2x step), <code>git log</code>. Pure stdlib, no client-go, runs wherever kubectl/git/curl-reachable Prometheus do. Unit tests on timeline merge and markdown rendering.</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/inference-slo-stack">inference-slo-stack</a></td>
  <td>OCI Ampere A1, k3s, VictoriaMetrics (VMRule + VMAgent + VMAlert), Grafana, Go vLLM-compatible mock, k6</td>
  <td>SLO + burn-rate alerting for LLM inference endpoints. Three orthogonal SLIs (availability, p99 TTFT, p99 TPOT) because chat-completion latency is prompt-processing + per-token decode - lumping them hides which half of the pipeline is in trouble. Six burn-rate alerts (3 SLOs x 2 severities) using the Google SRE Workbook multi-window multi-burn-rate pattern (1h x 14.4 fast, 6h x 6 slow). Go vLLM-compatible mock emits the exact <code>vllm:*</code> Prometheus metric families real vLLM 0.6.x does, so rules + dashboards port 1:1 to GPU-backed runtime. Live run showed TPOT burn at 94.3x while availability + TTFT stayed quiet - the <em>which SLO is on fire</em> story worked as designed.</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/otel-logs-traces-stack">otel-logs-traces-stack</a></td>
  <td>OCI Ampere A1, k3s, Grafana Loki 3.3.2, Grafana Tempo 2.6.1, OpenTelemetry Collector 0.113.0</td>
  <td>Logs + traces half of the observability triad on the same OCI VM as the Prometheus and VictoriaMetrics stacks - one Grafana pane with metrics / logs / traces all pointing at the same pod. OTEL Collector <code>k8sattributes</code> processor enriches every signal with pod/ns/node/deployment labels; Grafana <code>tracesToLogsV2</code> and <code>derivedFields</code> on <code>trace_id</code> make Explore walk the correlation. 532 MiB total RSS on a 12 GiB shared host</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/vm-observability">vm-observability</a></td>
  <td>OCI Ampere A1, k3s, vmoperator, VMSingle, VMAgent, VMAlert, Grafana</td>
  <td>Operator-pattern VictoriaMetrics stack running side-by-side with kube-prometheus-stack on the same OCI VM; measured ~28% less RSS for the identical 24-target scrape surface; vmoperator's Prometheus Converter auto-translates existing PodMonitors so no manual VMPodScrape is needed to adopt existing instrumentation</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/kafka-slo-observability">kafka-slo-observability</a></td>
  <td>OCI Ampere A1, k3s, Strimzi 0.47, Kafka 3.9 (KRaft), Prometheus Operator, Grafana</td>
  <td>Three-replica KRaft Kafka with multi-window multi-burn-rate SLO alerts from the Google SRE Workbook; two silent dashboard traps (<code>strimzi_io_name</code> relabel, Kafka 3.9 JVM metric-source overlap) documented end to end</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/cilium-l7-observability">cilium-l7-observability</a></td>
  <td>OCI Ampere A1, k3s, Cilium 1.17, Hubble</td>
  <td>Identity-based (not IP-based) L4 <code>CiliumNetworkPolicy</code> with per-flow verdicts in Hubble UI; three Oracle Cloud deployment gotchas documented end-to-end (INPUT reject on pod CIDR, missing SNAT masquerade, Alpine musl DNS flakes against CoreDNS)</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/gpu-observability-eks">gpu-observability-eks</a></td>
  <td>AWS, EKS, NVIDIA T4, DCGM, vLLM, Prometheus</td>
  <td>GPU allocated-but-idle cost signal on EKS: DCGM profiling counters catch a T4 reporting ~85% "utilization" while its tensor cores sit at ~13% (memory-bound, not compute-bound). Standalone device-plugin + dcgm-exporter on the accelerated AMI, with a blameless <a href="https://github.com/bibigon14/gpu-observability-eks/blob/main/docs/postmortem.md">postmortem</a> of the driver/AMI matrix</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/terraform-aks-platform">terraform-aks-platform</a></td>
  <td>Azure, AKS, Terraform, Workload Identity</td>
  <td>AKS with federated Managed Identity and Azure RBAC; five distinct gotchas caught by CI (OIDC subject drift, k8s version deprecation, VM allowlist, kubelogin, RBAC bootstrap)</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/ebpf-tcp-observer">ebpf-tcp-observer</a></td>
  <td>Go, cilium/ebpf, WireGuard, Prometheus</td>
  <td>Kernel-level TCP retransmit observer via a kprobe on <code>tcp_retransmit_skb</code>, scraped from homelab Prometheus over a reverse-initiated WireGuard tunnel from an Oracle Cloud Ampere A1 edge VM</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/k8s-blue-green-deploy">k8s-blue-green-deploy</a></td>
  <td>k3s, Argo Rollouts, Go</td>
  <td>Blue-green with prePromotionAnalysis smoke tests on a Raspberry Pi cluster</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/terraform-gke-platform">terraform-gke-platform</a></td>
  <td>GCP, GKE, Terraform, WIF</td>
  <td>GKE with Workload Identity Federation; real race-condition bug caught by CI</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/terraform-eks-platform">terraform-eks-platform</a></td>
  <td>AWS, EKS, Terraform, OIDC</td>
  <td>EKS bootstrap with GitHub OIDC and a hand-rolled IRSA module</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/terraform-vault-platform">terraform-vault-platform</a></td>
  <td>AWS, Vault, Raft, KMS</td>
  <td>HA Vault cluster on EC2 with KMS auto-unseal and Raft integrated storage</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/homelab-k3s">homelab-k3s</a></td>
  <td>k3s, Traefik, ArgoCD, cert-manager, Blackbox</td>
  <td>Production homelab on a Raspberry Pi 5. <strong>ArgoCD v3.4.4 as the sole control plane</strong>: 15 <code>Application</code> manifests under <code>argocd/</code>, every workload Healthy + Synced (<a href="https://github.com/bibigon14/homelab-k3s/blob/main/docs/screenshots/argocd-apps.png">screenshot</a>); <code>syncPolicy.automated: {prune: true, selfHeal: true}</code> on every app; Argo CD Notifications wires sync failures to the same Telegram channel as Alertmanager. Traefik ingress with self-signed wildcard, end-to-end Blackbox probing, Velero daily backups to R2. Operational artifacts include the <a href="https://github.com/bibigon14/homelab-k3s/blob/main/docs/runbooks/ingress-health-check.md">ingress health check runbook</a></td>
</tr>
</tbody>
</table>
