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
  <td>GCP BigQuery billing export (Standard + Detailed + FOCUS), Terraform, Grafana, SRE burn-rate</td>
  <td>SRE playbook ported to cloud cost. Monthly budget = cost SLO, unspent budget = error budget, <code>burn_rate = (consumption_fraction / window_elapsed_fraction)</code>: <code>&gt;=14.4</code> pages (budget exhausts in ~2 days), <code>&gt;=6</code> tickets (~5 days), <code>&gt;=3</code> warns (~10 days) - the SRE Workbook multi-window multi-burn-rate shape from <a href="https://github.com/bibigon14/inference-slo-stack">inference-slo-stack</a> applied to spend. Three parallel BigQuery exports enabled on the live billing account: Standard (per-SKU monthly rollup, drives the burn alert), Detailed (per-SKU per-day + resource_id + GKE <code>goog-k8s-namespace</code>/<code>goog-k8s-cluster-name</code> labels, drives cost-per-namespace attribution), and <strong>FOCUS usage cost (Preview)</strong> - the FinOps Foundation Open Cost and Usage Specification 1.0, so the same query runs against AWS CUR 2.0, Azure Cost Export and OCI Usage Report in v0.2+. Five validated reference SQL queries (MTD-by-service, 30-day trend, cost-by-namespace, FOCUS-format MTD, burn-rate math with <code>SAFE_DIVIDE</code> + month-boundary arithmetic). Reference Grafana dashboard JSON (MTD stat, burn-rate with threshold at 1.0/1.5, projected month-end, bar by service, stacked 30-day timeseries) and Grafana unified-alerting <code>BudgetBurnRate_FastWindow</code> rule with on-call runbook. Terraform scope is honest: the billing-export sink has no first-class GCP Terraform resource (console + REST API only as of 2026), so <code>main.tf</code> pins the dataset via <code>data</code> block and documents the out-of-band dependency rather than hiding it behind a fragile <code>local-exec</code>. v0.2 adds AWS CUR 2.0 via Athena as a second datasource in the same Grafana; v0.3 OCI + Azure; v0.4 cost-per-K8s-namespace; v0.5 z-score anomaly detection</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/gpu-observability-gke">gpu-observability-gke</a></td>
  <td>GCP, GKE, NVIDIA L4 (Ada Lovelace, 24 GB), DCGM, vLLM, Prometheus</td>
  <td>GCP mirror of gpu-observability-eks on a newer GPU generation. Two money-shots side by side on a real L4 Spot: idle (vLLM resident in 19.6 GB of framebuffer, <code>GPU_UTIL=0%</code>, 38 W) vs under 20-stream load (same 19.6 GB, <code>GPU_UTIL=87%</code>, 70.5 W near the 72 W TDP, ~1000 generation tok/s). <code>FB_USED</code> does not move between the two states; every real signal does - that gap is what the <code>GPUAllocatedButIdle</code> alert catches, on both clouds. Four GKE-specific quirks documented inline: the <code>dcgm-exporter</code> Helm chart&apos;s hard-coded securityContext conflicts with GKE COS&apos;s CDI runtime (shipped as a raw DaemonSet instead); the auto-applied <code>gke-resource-quotas</code> ResourceQuota blocks <code>system-node-critical</code> PriorityClass outside kube-system; DCGM profiling fields fail on Spot L4 so a custom counters CSV keeps only the NVML fields; and the Cluster Autoscaler marks a GPU node with <code>min=0</code> as DeletionCandidate within ~10 min</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/chaos-scheduler-operator">chaos-scheduler-operator</a></td>
  <td>Go, kubebuilder, controller-runtime, Prometheus</td>
  <td>Scheduled fault injection with an SLO-aware abort. Custom <code>ChaosExperiment</code> CRD (group <code>chaos.dstepanov.dev/v1alpha1</code>) with a cron schedule, label selector, and optional Prometheus guardrail: before each run the reconciler queries a PromQL expression and aborts if the scalar exceeds the configured threshold, so chaos never fires while a service is already burning budget. Deployed to a single-node k3s homelab via ArgoCD (Application manifest lives in the operator repo itself), actively killing pods across two namespaces on offset schedules against a real availability SLO (<code>1 - avg(avg_over_time(up{job=~...}[5m]))</code>), with three custom Prometheus metrics (<code>chaos_experiment_runs_total</code>, <code>chaos_experiment_last_run_timestamp</code>, <code>chaos_experiment_guardrail_check_duration_seconds</code>) and a 6-panel Grafana dashboard shipped in <code>docs/grafana/</code>. <strong>Milestone 5</strong> (Nov 2026): validating admission webhook rejects a CR when the schedule does not parse as 5-field cron, the target namespace is <code>kube-system</code>/<code>kube-public</code>/<code>kube-node-lease</code>, the label selector is empty (would match every pod), or <code>podKill.count &lt; 1</code>; errors aggregate so a multi-field edit surfaces everything at once in a single admission response. Native-arm64 multi-arch CI (push-by-digest + <code>buildx imagetools create</code> merge; dropped build from ~25 min under QEMU to ~3 min) and three MADR decision records documenting the deploy, GitOps, and CI trade-offs</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/incident-timeline-generator">incident-timeline-generator</a></td>
  <td>Go CLI, stdlib only, kubectl + Prometheus ALERTS + git log ingesters</td>
  <td>Built from the lesson writing <a href="https://github.com/bibigon14/inference-slo-stack/tree/main/docs/incidents">two production post-mortems</a> last week - the mechanical timeline assembly step (which pod crashed when, which alert fired, which commit went in just before) was pure typing across three terminals. This CLI runs the three queries over the same time window, merges by timestamp, and renders the standard post-mortem Markdown shape with the Timeline table pre-populated and Root cause / Lessons / Follow-up blank for the human. Three independently-optional ingesters: <code>kubectl get events -o json</code> with window filter, <code>ALERTS{alertstate=&quot;firing&quot;}</code> over <code>/api/v1/query_range</code> with firing-episode detection (gap &gt; 2x step), and <code>git log --since --until --format</code> with custom separator. Pure stdlib, no client-go dependency, runs wherever kubectl/git/curl-reachable Prometheus already do. Unit tests on timeline merge and markdown rendering</td>
</tr>
<tr>
  <td><a href="https://github.com/bibigon14/inference-slo-stack">inference-slo-stack</a></td>
  <td>OCI Ampere A1, k3s, VictoriaMetrics (VMRule + VMAgent + VMAlert), Grafana, Go vLLM-compatible mock, k6</td>
  <td>SLO + burn-rate alerting for LLM inference endpoints. Three orthogonal SLIs (availability, p99 TTFT, p99 TPOT) because latency on a chat completion is the sum of prompt-processing and per-token decode - lumping them hides which half of the pipeline is in trouble. Six burn-rate alerts (3 SLOs x 2 severities) using the Google SRE Workbook multi-window multi-burn-rate pattern (1h x 14.4 fast, 6h x 6 slow). Go vLLM-compatible mock emits the exact <code>vllm:*</code> Prometheus metric families real vLLM 0.6.x does, so the rules and dashboards port 1:1 when the mock is swapped for a real GPU-backed runtime. Live run demonstrated TPOT burn at 94.3x against threshold while availability and TTFT stayed quiet - the <em>which SLO is on fire</em> story worked exactly as designed</td>
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
