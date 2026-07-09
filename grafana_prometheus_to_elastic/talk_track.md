# Talk Track: From Prometheus + Grafana to Unified AI-Native Observability
**Target length: ~30 minutes (16 slides, ~1–3 min each) + Q&A**

Source reference for stats/claims: [Elastic Observability Labs — "Elasticsearch: best-in-class for logs, now best-in-class for metrics"](https://www.elastic.co/observability-labs/blog/prometheus-metrics-elasticsearch-faster-cheaper-datadog)

---

## Slide 1 — Title (1 min)

> "Thanks Abhijeet. What Abhijeet has presented is an overview of how Elasticsearch can be used for Observability use cases in general. What I would like to do is to talk about what pain points are customers facing and why they are looking to migrate to Elasticsearch. I will specifically talk about migrating from Promethues + Grafana to Elasticsearch. 

If you're an SRE, in IT Ops, or DevOps, you've probably lived some version of this story: Prometheus scrapes everything, Grafana visualizes it, and it works great — until it doesn't. Today I want to talk about why that combination breaks down at scale, and what a genuinely unified, AI-native alternative looks like. This isn't a slideware pitch — everything I'm showing you today shipped as of late June 2026, so we'll be clear throughout about what's GA versus what's still in preview."

Set expectations: ~30 minutes, live demos where possible, Q&A at the end.

---

## Slide 2 — Agenda (1 min)

> "We'll start with the pain everyone in this room may already know and experiencing. Then we'll walk through how Elastic unifies metrics, logs, and traces on one backend — including running your existing PromQL unchanged. From there we'll get into performance and cost numbers, the AI-native layer — natural language dashboards and agentic investigations — Kubernetes content out of the box, and finally migration: how you'd actually get from where you are today to this, without a rebuild project."

Quick read-through of the two columns — no need to dwell.

---

## Slide 3 — The Problem (2.5 min)

> "Let's call the pain out directly. Cardinality is the first one — every new Kubernetes label, every new pod, every new OTel dimension multiplies your series count, and most backends make you pay for that growth non-linearly. Second, if you're running Prometheus at real scale, you're probably also running Mimir, Thanos, or Cortex for long-term storage — and now you're operating a distributed systems project just to keep your metrics around longer than a few weeks.
>
> Third — and this is the one that shows up at 2am — your signals are siloed. Metrics in one tool, logs in another, traces in a third. During an incident, you're not debugging, you're tab-switching, manually correlating timestamps across three UIs before you've even formed a hypothesis.
>
> And fourth, if you've looked at Grafana Cloud to solve some of this, you've probably noticed the best features — the ones that would actually help — are gated behind hosted plans, which is a nonstarter for a lot of self-managed or regulated environments."

Optional pause: *"Does this sound familiar to anyone in the room?"* — light audience check-in, not a hard ask.

---

## Slide 4 — Platform Overview: One Backend (2.5 min)

> "So here's the core architectural idea we're going to spend the rest of the talk unpacking: Elasticsearch now stores Prometheus and OpenTelemetry metrics natively, in the same backend that already holds your logs and traces. Not a separate metrics product bolted on — the same store.
>
> That means Prometheus Remote Write points at Elasticsearch instead of Mimir, OTLP lands directly, and everything — metrics, logs, traces — is queryable through one language, ES|QL, and visualized in Kibana. No translation layer in between.
>
> And critically, this isn't cloud-only. It runs self-managed, on Elastic Cloud, or fully serverless — including air-gapped environments, which is something Grafana Cloud only partially offers."

Transition: *"Let's get concrete about what 'no translation layer' actually means, starting with PromQL."*

---

## Slide 5 — PromQL Native in ES|QL (2.5–3 min, demo-friendly)

> "This is the slide I'd ask you to actually read carefully, because it's the detail that removes the biggest migration objection. If your team has PromQL queries, dashboards, and alert rules today — for example, a rate-of-5xx-errors query — that query runs completely unchanged against Elasticsearch as the backend. Same syntax, same result. No rewrite, no retraining.
>
> On the right, that same data is also queryable through ES|QL, Elastic's own query language, which uses a `TS` command purpose-built for time series — counter rates, gauge averages, multilevel aggregations across high-cardinality dimensions. The reason this matters: ES|QL can join that metric against logs and traces in the same query. So the CPU spike you just found in PromQL — in the next line, you pull the deployment event from logs that happened five seconds earlier. That correlation used to mean opening a second tool."

**[If doing a live demo: this is the natural place to show a real PromQL query running in Kibana side by side with the ES|QL equivalent.]**

Transition: *"That's the compatibility story. Now let's talk about what changed under the hood to make this fast and cheap."*

---

## Slide 6 — TSDB Enhancements: Up to 2.6x Efficiency (2.5 min)

> "Here are the numbers, and I want to be precise about where they come from — these aren't marketing round-ups, they're published benchmark results from Elastic's Observability Labs team, comparing directly against Prometheus, Mimir, and ClickHouse.
>
> Storage: Elasticsearch's columnar TSDS engine stores metrics up to two-and-a-half times more efficiently than Prometheus, and about twice as efficiently as ClickHouse. Query performance: ES|QL time-series queries run up to 30 times faster than Prometheus and Mimir on gauge averages and counter rates — and that number holds even under high-cardinality workloads, which is exactly where competing systems tend to fall over.
>
> The architectural reason is worth mentioning: Elasticsearch's metrics engine doesn't maintain a per-series in-memory state that scales with cardinality. So when you add a thousand new Kubernetes pod labels, you're not paying a memory tax for it the way you would with a lot of TSDB architectures. Practically, that means no cardinality limits, no penalty for detailed instrumentation, and a materially more predictable cost curve as your label cardinality grows."

Transition: *"So it's fast and it's efficient. Let's move into the part of the platform that's newest — the AI-native layer, starting with how you actually build things day to day."*

---

## Slide 7 — Natural Language Dashboards (2 min)

> "Building a dashboard has traditionally meant: pick a visualization type, write a query, wire it to the panel, repeat twenty times. Elastic now lets you describe what you want in plain language — 'show me p95 latency by service for the last 24 hours, grouped by cluster' — and it generates the underlying ES|QL query and renders the panel.
>
> Two things I want to flag: the generated query is visible and editable, so this isn't a black box you have to trust blindly — you can always drop down and refine it by hand. And because it's pulling from the same unified backend, these prompts aren't limited to metrics; you can ask for panels that mix in log volume or trace latency in the same dashboard."

Optional live demo: build one panel via natural language on stage — high engagement moment.

---

## Slide 8 — Managed OTLP Endpoint (1.5 min)

> "If Prometheus is your metric source, and If your team is on the OpenTelemetry path to instrument applications for traces — or thinking about it — this removes a whole category of infrastructure work. Instead of standing up and scaling your own OTel Collector fleet, you point your OTel SDKs directly at a managed OTLP endpoint that elastic cloud provides. No need of a collector that needs installation, patching, sizing, or scaling. Managed OTLP endpoint handles scaling automatically.
>
> And this lands in the exact same store as your Prometheus data, so if you're mid-migration from Prometheus instrumentation to OTel — which a lot of teams are — you're not running two separate backends during that transition."

---

## Slide 9 — Agentic Investigations (3 min — this is a centerpiece slide, give it room)

> "This is the part of the talk that's genuinely new territory, so let's slow down here. Because metrics, logs, and traces already share one backend and one schema, when an alert fires, Elastic can automatically correlate across all three — no engineer manually opening three tabs and lining up timestamps. ML anomaly detection runs against infrastructure metrics automatically, so you're not just getting a threshold breach, you're getting a scored anomaly with context: what's typical here, what changed, how severe is this compared to baseline.
>
> Compare that to a Grafana LGTM stack, where you're stitching Loki, Mimir, and Tempo together by hand during an incident.
>
> And here's the part I think this audience will actually use day to day: this isn't locked inside a Kibana UI. Elastic ships an Observability MCP App and a library of Agent Skills. That means if your team already lives in Claude, Cursor, or VS Code, you can ask those tools to pull infrastructure health, service dependency graphs, or blast-radius analysis directly into your existing workflow — as interactive views in the conversation, not a link you have to click out to. Thats very powerful and the idea is that we bring elasticseatch to where you are, instead of asking you to come to elasticsearch for triaging and troubleshooting."

Transition: *"Let's ground this in something concrete — Kubernetes, since that's daily reality for most of you."*

---

## Slide 10 — Kubernetes Out of the Box (1.5 min)

> "Nobody wants to spend their first month on a new observability platform building basic Kubernetes dashboards from scratch. Elastic's Kubernetes integration ships with hierarchical dashboards, alert rule templates, ML anomaly detection jobs, and the prompts needed for AI-assisted root cause analysis — all active the moment data starts flowing. This is applicable to not just Kubernetes, but to all the infrastructure and application tech stack."

---

## Slide 11 — Migration Strategy: Best Practices & Pitfalls (2 min)

> "Now, the question I get asked most in conversations like this: 'okay, but how do we actually move?' I would like to highlight a few best practices that matter. 1) Inventory your existing PromQL queries and alert rules before you start — you want a checklist, not just a vibe check. 2) Migrate read paths first: get dashboards validated against production before you touch alerting. This is important, to make sure that there are artefacts to visualize, triage and troubleshoot before we get an alert. 3) Consider running Prometheus and Elastic side by side during cutover rather than a hard switch, and 4) right-size your retention now that storage is meaningfully cheaper.

> On the pitfall side — some of the things that you must avoid - the biggest one I see is teams migrating alerting before dashboards are fully validated, which means you find out your thresholds are wrong during a real incident. Don't assume every cardinality pattern you built to work around Prometheus's limitations still needs to exist once those limitations are gone. And always have a rollback plan — treat this as a phased migration, not a big-bang cutover."

---

## Slide 12 — Automated Migration Platform (2 min)

> "For teams with years of dashboards and alert rules already built in Grafana or Datadog, Elastic's Observability Migration Platform automates the actual conversion. You point a CLI — or Claude or Cursor using Elastic's agent skills — at your Datadog org or Grafana instance, and it converts supported dashboards, alert rules, and PromQL queries into Kibana-native equivalents. It also tells you honestly what fully migrated cleanly, what needs manual tweaks, and what it couldn't handle — which matters, because a migration tool that silently drops edge cases is worse than no tool at all.
>
> This is currently tech preview, so I'd treat it as something to pilot on a non-critical team first, not something to bet a production cutover on yet."

---

## Slide 13 — Benchmarks & Trade-offs (2 min)

> "Let's put real comparison numbers on the table. Query speed: up to 30x faster than raw Prometheus or Mimir. Storage: up to 2.5x more efficient than Prometheus, roughly 2x more efficient than ClickHouse. On cardinality: no hard limits, versus cardinality being the single most common complaint I hear about Prometheus and Mimir at scale. On cost: in published list-price comparisons, Elastic runs at roughly half of Datadog's cost for comparable metrics workloads — and that gap widens specifically for high-cardinality, densely instrumented environments like Kubernetes, which is exactly where Datadog's custom-metric billing hurts most.
>
> I'll be direct about the trade-off, though: none of this erases the effort of a migration, and if your team has deep PromQL and Grafana muscle memory, there's a real onboarding curve to ES|QL even if PromQL itself still works unchanged. Plan a phased rollout, not a weekend cutover."

---

## Slide 14 — Deployment Flexibility (1.5 min)

> "One more structural point worth making explicit: you're not locked into a single deployment model. Self-managed for full control, including air-gapped environments if you're in a regulated industry. Elastic Cloud if you want a managed service on AWS, GCP, or Azure. Or fully serverless if you want zero ops and automatic scaling. Datadog doesn't offer an on-prem option at all, and Grafana Cloud reserves a lot of its best functionality for hosted deployments — so this flexibility is a genuine differentiator, not a checkbox."

---

## Slide 15 — GA vs. Tech Preview (1.5 min)

> "I promised to be straight with you about what's actually production-ready today versus what's still maturing, so here's the honest breakdown. GA — meaning you can rely on it in production right now: the columnar metrics engine, ES|QL time series support, native PromQL in Kibana, Prometheus Remote Write ingest, and the Kubernetes out-of-the-box dashboards, alerts, and ML jobs. Also GA: Agent Skills.
>
> Still in tech preview: the Observability MCP App, AWS infrastructure content, and the Observability Migration Platform. I'd encourage you to pilot those rather than bet a production migration on them today — but they're moving fast, and worth watching."

----
DEMO
----

    - One-click migration of dashboards and alerts from Grafana to Elastic
    - Prometheus remote write support (Add Data)
    - 

## Slide 16 — Closing (1.5 min)

> "So to bring it back to where we started: the Prometheus-plus-Grafana model breaks down at scale because cardinality gets expensive, long-term storage is its own ops project, and your signals live in silos. The alternative isn't a rip-and-replace — your PromQL keeps working, your OTel pipeline gets simpler, and what you gain is a single backend where AI can do the first pass on an investigation before anyone's paged.
>
> I've put some links up here — the Elastic Observability site, the Prometheus monitoring deep-dive, Observability Labs for more technical write-ups, and the Agent Skills repo on GitHub if you want to see what's under the hood.
>
> With that, I'd love to open it up — what questions do you have?"

---

## Timing Summary

| Slide | Topic | Minutes |
|---|---|---|
| 1 | Title | 1.0 |
| 2 | Agenda | 1.0 |
| 3 | The Problem | 2.5 |
| 4 | Platform Overview | 2.5 |
| 5 | PromQL in ES\|QL | 2.5–3.0 |
| 6 | TSDB Enhancements | 2.5 |
| 7 | NL Dashboards | 2.0 |
| 8 | Managed OTLP | 1.5 |
| 9 | Agentic Investigations | 3.0 |
| 10 | Kubernetes OOTB | 1.5 |
| 11 | Migration Strategy | 2.0 |
| 12 | Migration Platform | 2.0 |
| 13 | Benchmarks | 2.0 |
| 14 | Deployment Flexibility | 1.5 |
| 15 | GA vs. Preview | 1.5 |
| 16 | Closing | 1.5 |
| **Total** | | **~30 min** |

## Delivery Notes
- **Live demo candidates** (if you want to cut talk time to make room): Slide 5 (PromQL vs ES|QL side-by-side), Slide 7 (build a dashboard via natural language), Slide 12 (migration platform conversion).
- **Q&A landmines to prepare for:** "Is this actually GA or is it slideware?" (answer via Slide 15), "What happens to my Grafana dashboards?" (answer: Grafana can stay, point it at Elasticsearch via the native Prometheus API — mention this even though it isn't its own slide), "How does pricing actually compare?" (the ~50% vs. Datadog figure is from published list-price comparisons — say so explicitly, don't oversell it as guaranteed for their specific workload).
- **Tone:** Confident on what's shipped and benchmarked; explicitly honest about tech-preview items and migration effort. This audience (SRE/DevOps) will trust the talk more, not less, if you flag the caveats yourself before someone asks.