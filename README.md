# My work at LiteLLM (YC W23)

**Backend & Forward Deployed Engineer · Dec 2025 – Apr 2026**

[LiteLLM](https://github.com/BerriAI/litellm) is an open-source AI gateway. Companies put it between their apps and 100+ LLM providers (OpenAI, Anthropic, Bedrock, Vertex…) to get one API with auth, budgets, guardrails, load balancing and logging on top.

I spent four months on the team behind that gateway: fixing production issues, building features, owning the **guardrails** area and helping **ship releases**.

> **317 commits in `main`** · **220 PRs** · guardrails owner · release engineering · security fixes · 4 observability integrations · all public in [BerriAI/litellm](https://github.com/BerriAI/litellm/commits/main/?author=Harshit28j)

<!-- TODO (optional): add one diagram of the LiteLLM request path (app → proxy → auth → guardrails → router → provider → logging) highlighting your areas -->

---

## ⚡ If you only read one section

| | What happened | PR |
|---|---|---|
| 🔐 | Fixed an **unauthenticated remote code execution** (sandbox escape) in the Custom Code Guardrail: reproduced the reported exploit, then closed it with sandbox hardening, admin-only endpoints and safe auth defaults | [#22095](https://github.com/BerriAI/litellm/pull/22095) |
| 🧱 | **~50% of SSO logins failed** on multi-pod Kubernetes setups because PKCE state lived in one pod's memory. Moved it to Redis so any replica can finish the login | [#20314](https://github.com/BerriAI/litellm/pull/20314) |
| 🚦 | Users were being **rate-limited at ~25% of their configured TPM limit**. Traced it to a priority-reservation default and fixed it | [#19092](https://github.com/BerriAI/litellm/pull/19092) |
| 🗄️ | Proxies on AWS RDS with IAM auth **lost their DB connection every 15 minutes**. Replaced fire-and-forget reconnection with a proactive token refresh | [#18795](https://github.com/BerriAI/litellm/pull/18795) |
| 💰 | Built a **Vantage integration** that exports LLM spend as FOCUS 1.2 cost reports, so AI spend sits next to cloud spend | [#23333](https://github.com/BerriAI/litellm/pull/23333) |
| 🛡️ | Made **Presidio PII masking** survive streaming, Anthropic's strict schemas and multi-entity edge cases, and stopped PII tokens leaking to providers | [#21949](https://github.com/BerriAI/litellm/pull/21949) [#22627](https://github.com/BerriAI/litellm/pull/22627) |

---

## 🎩 The hats I wore

One title didn't cover it:

- **Backend engineer.** Most of my 317 commits: the Python proxy, router, auth, DB layer and integrations.
- **Forward deployed engineer.** Worked directly with users to understand problems, reproduce issues and turn them into fixes.
- **Guardrails owner.** Owned the guardrails subsystem (Presidio, OpenAI Moderation, Bedrock, PANW Prisma AIRS, DynamoAI, custom code guardrails).
- **Security.** Fixed an unauthenticated RCE, credential leaks in error responses and traces, and Critical/High CVEs in the Docker images.
- **Release engineering.** Cut and patched releases, bumped versions and wrote the security scan report for release notes.
- **DevOps.** The Helm chart, non-root Docker images, and Prisma migrations in multi-pod deployments.
- **Open-source maintainer.** Fixed 56 issues reported by the community, reviewed teammates' PRs, wrote docs, and built automated duplicate-issue detection for the repo.

---

## 🛡️ Guardrails (owned this area)

Guardrails are the checks that run before and after every LLM call: PII masking, content moderation, prompt injection detection. When they break, they either block real traffic or quietly let bad content through, so both directions matter.

**What I did**
- **Presidio PII masking, hardened end to end.** Fixed streaming crashes on raw byte chunks, added an entity deny list so tokens like `S3`, `EKS` and `C4` stop getting masked as PII, fixed offset drift when masking several entities, and moved `pii_tokens` into metadata so Anthropic stops rejecting requests. [#21949](https://github.com/BerriAI/litellm/pull/21949) [#22283](https://github.com/BerriAI/litellm/pull/22283) [#22627](https://github.com/BerriAI/litellm/pull/22627) [#22084](https://github.com/BerriAI/litellm/pull/22084) [#19714](https://github.com/BerriAI/litellm/pull/19714)
- **Team-based isolation.** Teams can only see and manage their own guardrails, admins see everything, and it's enforced in the backend rather than just hidden in the UI. [#19889](https://github.com/BerriAI/litellm/pull/19889)
- **Real streaming for OpenAI Moderation.** Checks run on sampled chunks in parallel with the stream instead of adding latency. [#20718](https://github.com/BerriAI/litellm/pull/20718)
- **Fixed model-level guardrails that never ran.** The hook fired before the router attached deployment guardrails. [#18895](https://github.com/BerriAI/litellm/pull/18895)
- **Flexible modes.** List-valued modes per tag (for example, run `pre_call` + `post_call` only for `claude-cli` traffic) and default-mode lists. [#23020](https://github.com/BerriAI/litellm/pull/23020) [#22676](https://github.com/BerriAI/litellm/pull/22676)
- New and fixed integrations: PANW Prisma AIRS custom messages, DynamoAI registration, Bedrock content blocking. [#19272](https://github.com/BerriAI/litellm/pull/19272) [#23752](https://github.com/BerriAI/litellm/pull/23752) [#20606](https://github.com/BerriAI/litellm/pull/20606)

<p align="center">
<img src="https://github.com/user-attachments/assets/c96f3228-b3a6-4a38-aa13-d6180682f60a" alt="Team A vs Team B guardrail views" width="800">
<br><sub>Team A and Team B each see only their own guardrails · <a href="https://github.com/BerriAI/litellm/pull/19889">#19889</a></sub>
</p>

<p align="center">
<img src="https://github.com/user-attachments/assets/e006a008-d883-4ab0-bb58-7419f5333245" alt="Presidio streaming and deny-list before/after flow" width="800">
<br><sub>Before/after: streaming crash fixed (left), deny list stops false PII masking (right) · <a href="https://github.com/BerriAI/litellm/pull/21949">#21949</a></sub>
</p>

<details>
<summary><b>17 merged guardrails PRs</b></summary>

| PR | What | Merged |
|---|---|---|
| [#18978](https://github.com/BerriAI/litellm/pull/18978) | fix: properly handle custom guardrails parameters | 2026-01 |
| [#18895](https://github.com/BerriAI/litellm/pull/18895) | fix: model-level guardrails not taking effect (#18363) | 2026-01 |
| [#19272](https://github.com/BerriAI/litellm/pull/19272) | feat(panw_prisma_airs): add custom violation message support | 2026-01 |
| [#19151](https://github.com/BerriAI/litellm/pull/19151) | fix/bedrock-inconsistent-postcall-hook | 2026-01 |
| [#19480](https://github.com/BerriAI/litellm/pull/19480) | fix: add case-insensitive support for guardrail mode and actions | 2026-01 |
| [#19714](https://github.com/BerriAI/litellm/pull/19714) | fix(presidio): resolve runtime error by handling asyncio loops in bac | 2026-01 |
| [#19901](https://github.com/BerriAI/litellm/pull/19901) | fix: guardrails issues streaming-response regex | 2026-01 |
| [#19889](https://github.com/BerriAI/litellm/pull/19889) | feat(guardrails): implement team-based isolation guardrails mgmnt | 2026-02 |
| [#20606](https://github.com/BerriAI/litellm/pull/20606) | fixes content blocked by bedrock | 2026-02 |
| [#20718](https://github.com/BerriAI/litellm/pull/20718) | fix: openai moderation guardrails | 2026-02 |
| [#21949](https://github.com/BerriAI/litellm/pull/21949) | fix: presidio streaming, false positives | 2026-02 |
| [#22084](https://github.com/BerriAI/litellm/pull/22084) | fix(guardrails): prevent presidio crash on non-json responses | 2026-02 |
| [#22283](https://github.com/BerriAI/litellm/pull/22283) | Litellm presidio stream v3 | 2026-02 |
| [#22676](https://github.com/BerriAI/litellm/pull/22676) | feat: guardrail-mode-default-list | 2026-03 |
| [#23020](https://github.com/BerriAI/litellm/pull/23020) | feat: support list of modes in tag-based guardrails | 2026-03 |
| [#22627](https://github.com/BerriAI/litellm/pull/22627) | fix: presidio edge case with antropic handle on pii token leak | 2026-03 |
| [#23752](https://github.com/BerriAI/litellm/pull/23752) | fix: Register DynamoAI guardrail initializer and enum entry | 2026-03 |

</details>

---

## 🔐 Security & Auth

An AI gateway holds every provider API key a company has, so its security bugs are expensive.

**What I did**
- **Unauthenticated RCE / sandbox escape** in the Custom Code Guardrail: closed it. [#22095](https://github.com/BerriAI/litellm/pull/22095)
- **Docker image CVEs.** Cleared all Critical/High findings across the `litellm` and `non_root` images (OS libs, Node, npm transitive deps), verified with Trivy and Docker Scout. [#20683](https://github.com/BerriAI/litellm/pull/20683) [#22008](https://github.com/BerriAI/litellm/pull/22008)
- **Secret leaks.** Expired keys were echoed in plaintext in error responses. Langfuse credentials leaked into traces through auth metadata. Multi-project Langfuse traces leaked across projects. [#18860](https://github.com/BerriAI/litellm/pull/18860) [#22188](https://github.com/BerriAI/litellm/pull/22188) [#19823](https://github.com/BerriAI/litellm/pull/19823)
- **SSO PKCE on Kubernetes**: from ~50% failed logins to working across replicas. [#20314](https://github.com/BerriAI/litellm/pull/20314)
- **Key rotation.** A distributed lock so only one pod rotates keys, and AWS Secrets Manager rotation now updates the existing secret instead of creating orphans. [#23364](https://github.com/BerriAI/litellm/pull/23364) [#19455](https://github.com/BerriAI/litellm/pull/19455)
- Custom auth: budget enforcement and opt-in `common_checks`, JWT → virtual key mapping. [#22164](https://github.com/BerriAI/litellm/pull/22164) [#22678](https://github.com/BerriAI/litellm/pull/22678) [#22372](https://github.com/BerriAI/litellm/pull/22372)

<p align="center">
<img src="https://github.com/user-attachments/assets/7d4a66d8-f199-4dc1-be7d-9744cc3a789d" alt="Trivy scan before" width="700">
<img src="https://github.com/user-attachments/assets/9a9c93ed-e1dc-4184-a5e0-0e7387d275c2" alt="Trivy scan after" width="700">
<br><sub>Trivy scan before → after the CVE fixes · <a href="https://github.com/BerriAI/litellm/pull/20683">#20683</a></sub>
</p>

<details>
<summary><b>18 merged security & auth PRs</b></summary>

| PR | What | Merged |
|---|---|---|
| [#18860](https://github.com/BerriAI/litellm/pull/18860) | fix(security): prevent expired key plaintext leak in error response | 2026-01 |
| [#19455](https://github.com/BerriAI/litellm/pull/19455) | fix: ensure auto-rotation updates existing AWS secret instead of crea | 2026-01 |
| [#19538](https://github.com/BerriAI/litellm/pull/19538) | Fix/per service ssl override v2 | 2026-01 |
| [#19823](https://github.com/BerriAI/litellm/pull/19823) | fix: support multi-project keys and fix trace leakage | 2026-01 |
| [#20683](https://github.com/BerriAI/litellm/pull/20683) | fix critical CVE vulnerabliltes | 2026-02 |
| [#20481](https://github.com/BerriAI/litellm/pull/20481) | Fix authorization issues, same alias; verified working | 2026-02 |
| [#20321](https://github.com/BerriAI/litellm/pull/20321) | fix: virutal key grace period from env/UI | 2026-02 |
| [#20314](https://github.com/BerriAI/litellm/pull/20314) | fix: SSO PKCE support fails in multi-pod Kubernetes deployments | 2026-02 |
| [#22008](https://github.com/BerriAI/litellm/pull/22008) | security: fix critical/high CVEs in OS-level libs and NPM transitive | 2026-02 |
| [#22095](https://github.com/BerriAI/litellm/pull/22095) | fix Unauthenticated RCE and Sandbox Escape in Custom Code Guardrail | 2026-02 |
| [#22164](https://github.com/BerriAI/litellm/pull/22164) | fix: custom auth budget issue | 2026-02 |
| [#22662](https://github.com/BerriAI/litellm/pull/22662) | fix(proxy): allow custom auth routes to bypass route authorization ch | 2026-03 |
| [#22678](https://github.com/BerriAI/litellm/pull/22678) | fix(proxy): make common_checks opt-in for custom auth | 2026-03 |
| [#22372](https://github.com/BerriAI/litellm/pull/22372) | Litellm jwt vkey map | 2026-03 |
| [#23027](https://github.com/BerriAI/litellm/pull/23027) | feat: feature flag on validate key alias | 2026-03 |
| [#22188](https://github.com/BerriAI/litellm/pull/22188) | fix: langfuse trace leak key on model params | 2026-03 |
| [#23834](https://github.com/BerriAI/litellm/pull/23834) | fix: address Greptile review feedback on key rotation lock | 2026-04 |
| [#23364](https://github.com/BerriAI/litellm/pull/23364) | fix: add distributed lock for key rotation job | 2026-04 |

</details>

---

## ⚙️ Multi-pod reliability & infrastructure

These bugs only appear with more than one replica: two pods migrating the same database, an SSO callback landing on the wrong pod, two pods rotating the same key. Every serious deployment runs several replicas, so all of them hit production.

**What I did**
- **Redis-based migration lock.** Stops several pods from running Prisma migrations at the same time on startup. Clean rewrite of an 87-file PR into 3 files, with a multi-pod simulation test. [#19261](https://github.com/BerriAI/litellm/pull/19261)
- **Fail fast on broken migrations.** The proxy used to start with an out-of-sync schema and crash later in background jobs. Now it exits with a clear error. Also made already-applied migrations idempotent. [#23257](https://github.com/BerriAI/litellm/pull/23257) [#19281](https://github.com/BerriAI/litellm/pull/19281)
- **RDS IAM token refresh**: no more connection drops every 15 minutes. [#18795](https://github.com/BerriAI/litellm/pull/18795)
- **Spend log cleanup job**: replaced loading 10k full rows into memory with an index-only scan + a single SQL `DELETE`, which ended the DB timeouts and memory alerts. [#21930](https://github.com/BerriAI/litellm/pull/21930)
- **Helm & Docker**: rollout `strategy`, `preStop` hooks, single-file config mount, non-root read-only filesystem fixes, Prisma schema paths. [#23214](https://github.com/BerriAI/litellm/pull/23214) [#19613](https://github.com/BerriAI/litellm/pull/19613) [#19146](https://github.com/BerriAI/litellm/pull/19146) [#19449](https://github.com/BerriAI/litellm/pull/19449) [#19631](https://github.com/BerriAI/litellm/pull/19631)

<p align="center">
<img src="https://github.com/user-attachments/assets/89ce4588-8c80-4c57-95cc-5be4c7d0eebc" alt="Safe migration lock timeline" width="800">
<br><sub>Only one pod runs migrations; the others wait, then skip · <a href="https://github.com/BerriAI/litellm/pull/19261">#19261</a></sub>
</p>

<p align="center">
<img src="https://github.com/user-attachments/assets/875bdd52-0cd2-4ccc-ab66-17c0696f0f75" alt="RDS IAM token problem" width="420">
<img src="https://github.com/user-attachments/assets/4a8740d7-3bae-477d-919e-0fa6e96e4826" alt="Proactive refresh loop" width="300">
<br><sub>Problem (left): token expires mid-flight → proactive refresh loop (right) · <a href="https://github.com/BerriAI/litellm/pull/18795">#18795</a></sub>
</p>

<details>
<summary><b>21 merged infra & reliability PRs</b></summary>

| PR | What | Merged |
|---|---|---|
| [#18828](https://github.com/BerriAI/litellm/pull/18828) | fix: add idx on LOWER(user_email) for faster duplicate email checks l | 2026-01 |
| [#18795](https://github.com/BerriAI/litellm/pull/18795) | fix: proactive RDS IAM token refresh to prevent 15-min connection failed | 2026-01 |
| [#18892](https://github.com/BerriAI/litellm/pull/18892) | docs: add Redis requirement warning for high-traffic deployments | 2026-01 |
| [#19085](https://github.com/BerriAI/litellm/pull/19085) | feat(proxy): cleanup spend logs cron verification, fix, and docs | 2026-01 |
| [#19136](https://github.com/BerriAI/litellm/pull/19136) | docs(logging.md): add guide for mounting custom callbacks in Helm/K8s | 2026-01 |
| [#19146](https://github.com/BerriAI/litellm/pull/19146) | fix: mount config.yaml as single file in Helm chart | 2026-01 |
| [#19261](https://github.com/BerriAI/litellm/pull/19261) | feat: Add Redis-based migration lock with bug fixes | 2026-01 |
| [#19391](https://github.com/BerriAI/litellm/pull/19391) | Fix/prisma schema permission | 2026-01 |
| [#19449](https://github.com/BerriAI/litellm/pull/19449) | fix: resolve Read-only file system error in non-root images | 2026-01 |
| [#19467](https://github.com/BerriAI/litellm/pull/19467) | fix: UI 404 error when SERVER_ROOT_PATH is set | 2026-01 |
| [#19613](https://github.com/BerriAI/litellm/pull/19613) | feat(helm): Enable PreStop hook configuration in values.yaml | 2026-01 |
| [#19631](https://github.com/BerriAI/litellm/pull/19631) | fix(docker): use correct schema path for prisma generation | 2026-01 |
| [#19281](https://github.com/BerriAI/litellm/pull/19281) | fix: resolve 'does not exist' migration errors as applied in setup_da | 2026-01 |
| [#19843](https://github.com/BerriAI/litellm/pull/19843) | fix: add fix for migration issue and and stable image | 2026-01 |
| [#19808](https://github.com/BerriAI/litellm/pull/19808) | fix: handle deprecated 'redis_db' arg to prevent crash | 2026-01 |
| [#20635](https://github.com/BerriAI/litellm/pull/20635) | fix: add hook to handle db case | 2026-02 |
| [#20451](https://github.com/BerriAI/litellm/pull/20451) | fix: handles edge case which are blocked by Lock | 2026-02 |
| [#21009](https://github.com/BerriAI/litellm/pull/21009) | add tests for hotpath & docker container | 2026-02 |
| [#21930](https://github.com/BerriAI/litellm/pull/21930) | perf(spendlogs): optimize old spendlog deletion cron job | 2026-02 |
| [#23214](https://github.com/BerriAI/litellm/pull/23214) | feat: add strategy to deployment for helmchart | 2026-03 |
| [#23257](https://github.com/BerriAI/litellm/pull/23257) | fix: fail proxy startup if prisma migrate fails | 2026-03 |

</details>

---

## 📈 Observability

A gateway sits on every LLM call, so it's where teams look for latency, cost and failures. I worked across four observability backends: **Prometheus, OpenTelemetry, Datadog and Langfuse**.

**What I did**
- **Datadog**: Agent support for LLM Observability, a custom metrics logger (gateway vs provider latency), cost management, and `team` tags across logs, metrics and cost. [#19574](https://github.com/BerriAI/litellm/pull/19574) [#22103](https://github.com/BerriAI/litellm/pull/22103) [#19584](https://github.com/BerriAI/litellm/pull/19584) [#21449](https://github.com/BerriAI/litellm/pull/21449)
- **Prometheus**: cache hit/miss/token metrics, TPM/RPM limit metrics, client IP + user agent labels, failure metric labels. [#18755](https://github.com/BerriAI/litellm/pull/18755) [#19725](https://github.com/BerriAI/litellm/pull/19725) [#19717](https://github.com/BerriAI/litellm/pull/19717) [#20152](https://github.com/BerriAI/litellm/pull/20152)
- **OpenTelemetry**: upgraded to gen_ai semantic conventions 1.38, fixed duplicate spans and orphaned guardrail traces, fixed OTEL never starting when models load from the DB. [#18793](https://github.com/BerriAI/litellm/pull/18793) [#23001](https://github.com/BerriAI/litellm/pull/23001) [#23802](https://github.com/BerriAI/litellm/pull/23802)
- **Langfuse**: fixed Claude Code request traces and the multi-project / OTEL trace pipeline. [#23259](https://github.com/BerriAI/litellm/pull/23259) [#21309](https://github.com/BerriAI/litellm/pull/21309)

<p align="center">
<img src="https://github.com/user-attachments/assets/657e97a2-6c12-4b1f-aa3a-8705d24c1615" alt="Datadog total latency metric" width="800">
<br><sub>New <code>litellm.request.total_latency</code> metric in Datadog · <a href="https://github.com/BerriAI/litellm/pull/22103">#22103</a></sub>
</p>

<p align="center">
<img src="https://github.com/user-attachments/assets/c189eccf-7601-4258-bab7-39c6d4dceb7e" alt="Cache hit/miss to Prometheus flow" width="380">
<br><sub>Cache hits and misses now flow into Prometheus · <a href="https://github.com/BerriAI/litellm/pull/18755">#18755</a></sub>
</p>

<details>
<summary><b>30 merged observability PRs</b></summary>

| PR | What | Merged |
|---|---|---|
| [#18423](https://github.com/BerriAI/litellm/pull/18423) | fix: logging_worker handle event loop changes in multiprocessing by | 2026-01 |
| [#18755](https://github.com/BerriAI/litellm/pull/18755) | feat(prometheus): add caching metrics for cache hits, misses, and tokens | 2026-01 |
| [#18793](https://github.com/BerriAI/litellm/pull/18793) | fix(otel): update semantic conventions to 1.38 (gen_ai attributes) | 2026-01 |
| [#18982](https://github.com/BerriAI/litellm/pull/18982) | fix(langsmith.py): hoist thread grouping metadata (session_id, thread | 2026-01 |
| [#19325](https://github.com/BerriAI/litellm/pull/19325) | fix(logging): prevent duplicate StandardLoggingPayload logs | 2026-01 |
| [#19298](https://github.com/BerriAI/litellm/pull/19298) | fix(langfuse_otel): ignore service logs and fix callback shadowing | 2026-01 |
| [#19574](https://github.com/BerriAI/litellm/pull/19574) | feat(datadog): add agent support for LLM Observability | 2026-01 |
| [#19584](https://github.com/BerriAI/litellm/pull/19584) | feat: add datadog cost management support and fix startup callback issue | 2026-01 |
| [#19627](https://github.com/BerriAI/litellm/pull/19627) | fix: propagate JWT auth metadata to OTEL spans | 2026-01 |
| [#19708](https://github.com/BerriAI/litellm/pull/19708) | add callbacks and labels to prometheus | 2026-01 |
| [#19717](https://github.com/BerriAI/litellm/pull/19717) | feat: add clientip and user agent in metrics | 2026-01 |
| [#19725](https://github.com/BerriAI/litellm/pull/19725) | feat: tpm-rpm limit in prometheus metrics | 2026-01 |
| [#19691](https://github.com/BerriAI/litellm/pull/19691) | fix(prometheus): safely handle None metadata in logging to prevent At | 2026-01 |
| [#20152](https://github.com/BerriAI/litellm/pull/20152) | fixes failure metrics labels | 2026-02 |
| [#20145](https://github.com/BerriAI/litellm/pull/20145) | fix: proxy failure cases, now log ip and user agent, key hash, name | 2026-02 |
| [#20382](https://github.com/BerriAI/litellm/pull/20382) | Litellm fix langfuse otel trace | 2026-02 |
| [#20339](https://github.com/BerriAI/litellm/pull/20339) | fix(otel): guard against None message in set_attributes tool_calls | 2026-02 |
| [#20366](https://github.com/BerriAI/litellm/pull/20366) | fix:Parse embedded JSON in the message field of logs | 2026-02 |
| [#21449](https://github.com/BerriAI/litellm/pull/21449) | feat(datadog): add 'team' tag to logs, metrics, and cost management | 2026-02 |
| [#21309](https://github.com/BerriAI/litellm/pull/21309) | Litellm fix langfuse otel trace v2 | 2026-02 |
| [#22077](https://github.com/BerriAI/litellm/pull/22077) | Litellm gemini trace id missingv2 | 2026-02 |
| [#22103](https://github.com/BerriAI/litellm/pull/22103) | feat: ability to trace metrics datadog | 2026-02 |
| [#20632](https://github.com/BerriAI/litellm/pull/20632) | fix tts metrics issues | 2026-02 |
| [#23001](https://github.com/BerriAI/litellm/pull/23001) | Fix OTEL span redundancy, orphaned guardrail traces, and missing response IDs | 2026-03 |
| [#23209](https://github.com/BerriAI/litellm/pull/23209) | feat: record silent metrics | 2026-03 |
| [#23259](https://github.com/BerriAI/litellm/pull/23259) | fix: claude code req traces on langfuse | 2026-03 |
| [#23542](https://github.com/BerriAI/litellm/pull/23542) | fix: silent metrics race condition | 2026-03 |
| [#22390](https://github.com/BerriAI/litellm/pull/22390) | Fix Langfuse failure path kwargs inconsistency | 2026-03 |
| [#23649](https://github.com/BerriAI/litellm/pull/23649) | add tests for fix | 2026-03 |
| [#23802](https://github.com/BerriAI/litellm/pull/23802) | fix(proxy): instantiate OTEL callback at startup instead of deferring | 2026-03 |

</details>

---

## 💰 Cost, budgets & rate limits

Cost tracking and budgets are core LiteLLM features, so bugs here cost real money.

**What I did**
- **Vantage integration**: a generic FOCUS 1.2 export pipeline (transform → serialize → deliver) with Vantage as the first destination, including batching for large exports. [#23333](https://github.com/BerriAI/litellm/pull/23333) [#23746](https://github.com/BerriAI/litellm/pull/23746)
- **Per-model budgets & limits**, plus an opt-in `enforce_model_rate_limits` check in the router. [#20076](https://github.com/BerriAI/litellm/pull/20076) [#19230](https://github.com/BerriAI/litellm/pull/19230)
- **Fixed users getting ~25% of their TPM limit**, fixed an off-by-one in team budgets, and fixed billing for load-balanced models. [#19092](https://github.com/BerriAI/litellm/pull/19092) [#19207](https://github.com/BerriAI/litellm/pull/19207) [#18891](https://github.com/BerriAI/litellm/pull/18891)

<p align="center">
<img src="https://github.com/user-attachments/assets/e8c97d12-e367-4893-be85-51cb96f5f988" alt="LiteLLM as a custom provider in Vantage" width="800">
<br><sub>LiteLLM spend imported into Vantage as a custom provider · <a href="https://github.com/BerriAI/litellm/pull/23333">#23333</a></sub>
</p>

<details>
<summary><b>9 merged cost & rate-limit PRs</b></summary>

| PR | What | Merged |
|---|---|---|
| [#19092](https://github.com/BerriAI/litellm/pull/19092) | fix(dynamic_rate_limiter_v3): fix TPM 25% limiting by ensuring priori | 2026-01 |
| [#18891](https://github.com/BerriAI/litellm/pull/18891) | fix: models loadbalancing billing issue by filter | 2026-01 |
| [#19207](https://github.com/BerriAI/litellm/pull/19207) | fix: correct budget limit validation operator (>=) for team members | 2026-01 |
| [#19696](https://github.com/BerriAI/litellm/pull/19696) | fix: token calculations and refactor | 2026-01 |
| [#19230](https://github.com/BerriAI/litellm/pull/19230) | feat: enforce model-level TPM/RPM limits (enforce_model_rate_limits) | 2026-01 |
| [#19220](https://github.com/BerriAI/litellm/pull/19220) | fix: models loadbalancing billing issue by filter (#18891) | 2026-01 |
| [#20076](https://github.com/BerriAI/litellm/pull/20076) | feat: Per Model Budgets and Limits | 2026-01 |
| [#23333](https://github.com/BerriAI/litellm/pull/23333) | Add Vantage integration for FOCUS CSV export | 2026-03 |
| [#23746](https://github.com/BerriAI/litellm/pull/23746) | feat: vantage export format | 2026-03 |

</details>

---

## 🧭 Routing, SDK & platform features

**What I did**
- **Sticky sessions.** Requests with the same `session_id` go to the same deployment, which keeps prompt caching effective. [#21763](https://github.com/BerriAI/litellm/pull/21763)
- **Silent experiments (traffic mirroring).** Mirror requests to a second model to evaluate it without adding latency to the real response. [#19544](https://github.com/BerriAI/litellm/pull/19544) [#23209](https://github.com/BerriAI/litellm/pull/23209)
- **Smarter retries.** Stopped retrying 4xx client errors and added `retry_delay`, exponential backoff and jitter to `completion()`. [#19275](https://github.com/BerriAI/litellm/pull/19275) [#19371](https://github.com/BerriAI/litellm/pull/19371)
- **Project management APIs**: projects with tags, end-to-end. [#21078](https://github.com/BerriAI/litellm/pull/21078) [#22216](https://github.com/BerriAI/litellm/pull/22216)
- Fixed streaming tool calls getting lost in the OpenAI → Anthropic adapter. [#18316](https://github.com/BerriAI/litellm/pull/18316)

<p align="center">
<img src="https://github.com/user-attachments/assets/d0835864-53a2-4e06-b5c2-09ce10e18039" alt="Session affinity routing flowchart" width="500">
<br><sub>Same <code>x-litellm-session-id</code> → same deployment · <a href="https://github.com/BerriAI/litellm/pull/21763">#21763</a></sub>
</p>

<details>
<summary><b>32 merged routing, SDK & platform PRs</b></summary>

| PR | What | Merged |
|---|---|---|
| [#18316](https://github.com/BerriAI/litellm/pull/18316) | fix: lost tool_calls when streaming has both text and tool_calls | 2025-12 |
| [#18872](https://github.com/BerriAI/litellm/pull/18872) | Add fix for bedrock_cache, metadata and max_model_budget | 2026-01 |
| [#18888](https://github.com/BerriAI/litellm/pull/18888) | doc: update load balancing and routing with enable_pre_call_checks | 2026-01 |
| [#18886](https://github.com/BerriAI/litellm/pull/18886) | doc: updated pass_through with guided param | 2026-01 |
| [#18955](https://github.com/BerriAI/litellm/pull/18955) | add better err handling for antropic | 2026-01 |
| [#18975](https://github.com/BerriAI/litellm/pull/18975) | fix: num_retries in litellm_params (#18968) as per config | 2026-01 |
| [#18991](https://github.com/BerriAI/litellm/pull/18991) | chore: resolve ModuleNotFoundError for Microsoft Foundry Agents | 2026-01 |
| [#19275](https://github.com/BerriAI/litellm/pull/19275) | fix(router): prevent retrying 4xx client errors | 2026-01 |
| [#19156](https://github.com/BerriAI/litellm/pull/19156) | fix(utils.py): correctly extract messages from google genai contents | 2026-01 |
| [#19371](https://github.com/BerriAI/litellm/pull/19371) | feat: add retry_delay, exponential_backoff, and jitter to completion() | 2026-01 |
| [#19277](https://github.com/BerriAI/litellm/pull/19277) | fix(proxy): add /a2a/{agent_id}/.well-known/agent-card.json to agent_ | 2026-01 |
| [#19322](https://github.com/BerriAI/litellm/pull/19322) | docs: fix bad examples from sdk | 2026-01 |
| [#19388](https://github.com/BerriAI/litellm/pull/19388) | fix(proxy_server): pass search_tools to Router during DB-triggered in | 2026-01 |
| [#19358](https://github.com/BerriAI/litellm/pull/19358) | fix(prompts): fix prompt info lookup and delete using correct IDs | 2026-01 |
| [#19441](https://github.com/BerriAI/litellm/pull/19441) | fix: add better error handling for misconfig on health check | 2026-01 |
| [#19531](https://github.com/BerriAI/litellm/pull/19531) | fix: recursive pydantic issue | 2026-01 |
| [#19420](https://github.com/BerriAI/litellm/pull/19420) | fix: pass through endpoints update registry | 2026-01 |
| [#19689](https://github.com/BerriAI/litellm/pull/19689) | docs: add litellm-enterprise requirement for managed files | 2026-01 |
| [#19720](https://github.com/BerriAI/litellm/pull/19720) | fix: args issue & refactor into helper function to reduce bloat for b | 2026-01 |
| [#19719](https://github.com/BerriAI/litellm/pull/19719) | fix: optimize logo fetching and resolve mcp import blockers | 2026-01 |
| [#19790](https://github.com/BerriAI/litellm/pull/19790) | fix: server rooth path | 2026-01 |
| [#19544](https://github.com/BerriAI/litellm/pull/19544) | feat: add feature to make silent calls | 2026-01 |
| [#19840](https://github.com/BerriAI/litellm/pull/19840) | Fix/router search tools v2 | 2026-01 |
| [#20379](https://github.com/BerriAI/litellm/pull/20379) | Litellm add project management apis v2 | 2026-02 |
| [#20689](https://github.com/BerriAI/litellm/pull/20689) | fix: type error & better error handling | 2026-02 |
| [#21402](https://github.com/BerriAI/litellm/pull/21402) | Litellm prompt registry fix | 2026-02 |
| [#21078](https://github.com/BerriAI/litellm/pull/21078) | Litellm project management apis | 2026-02 |
| [#21763](https://github.com/BerriAI/litellm/pull/21763) | feat: add session_id to have better routing | 2026-02 |
| [#21918](https://github.com/BerriAI/litellm/pull/21918) | fix: ensure arrival_time is set before calculating queue time | 2026-02 |
| [#22216](https://github.com/BerriAI/litellm/pull/22216) | feat: add tags in project | 2026-02 |
| [#22299](https://github.com/BerriAI/litellm/pull/22299) | Litellm health check tokens | 2026-02 |
| [#22384](https://github.com/BerriAI/litellm/pull/22384) | Litellm aws edge case | 2026-02 |

</details>

---

## 🚀 Release engineering & CI

Cut and patched releases (for example `1.82.0 → 1.82.1`), managed `litellm-proxy-extras` bumps (including a revert), wrote the security scan report for the v1.81.14 release notes, added a rollback safety check to the docs, and fixed CI coverage failures.

<details>
<summary><b>9 merged release & CI PRs</b></summary>

| PR | What | Merged |
|---|---|---|
| [#19218](https://github.com/BerriAI/litellm/pull/19218) | ci(github): add automated duplicate issue checker and template safegu | 2026-01 |
| [#20065](https://github.com/BerriAI/litellm/pull/20065) | fixes: ci pipeline router coverage failure | 2026-01 |
| [#20601](https://github.com/BerriAI/litellm/pull/20601) | fix conflicts with main- (this PR is from upstream/main) | 2026-02 |
| [#21743](https://github.com/BerriAI/litellm/pull/21743) | doc: add rollback safety check | 2026-02 |
| [#22385](https://github.com/BerriAI/litellm/pull/22385) | Doc: security vulnerability scan report to v1.81.14 release notes | 2026-02 |
| [#22631](https://github.com/BerriAI/litellm/pull/22631) | bump: version 1.82.0 → 1.82.1 | 2026-03 |
| [#23008](https://github.com/BerriAI/litellm/pull/23008) | bump litellm-proxy-extras 0.4.50 → 0.4.51 | 2026-03 |
| [#23017](https://github.com/BerriAI/litellm/pull/23017) | Revert "bump litellm-proxy-extras 0.4.50 → 0.4.51" | 2026-03 |
| [#23882](https://github.com/BerriAI/litellm/pull/23882) | patch dev | 2026-03 |

</details>

---

## 🧠 What this taught me

- **Trace the code, not the symptom.** The hardest bugs threw no errors: an ignored migration failure, OTEL never starting, guardrails checked before they were attached, a hidden 25% rate cap. I found each by tracing the request path until an assumption broke. [#23257](https://github.com/BerriAI/litellm/pull/23257) [#23802](https://github.com/BerriAI/litellm/pull/23802) [#18895](https://github.com/BerriAI/litellm/pull/18895) [#19092](https://github.com/BerriAI/litellm/pull/19092)
- **Design integrations to fit the codebase.** Before building, I study how the codebase already solves similar problems and extend that pattern instead of bolting something on. I built the cost-export integration as a generic pipeline with pluggable destinations, so new ones plug in without a rewrite, and moved the moderation guardrail onto the shared streaming path instead of giving it its own. [#23333](https://github.com/BerriAI/litellm/pull/23333) [#20718](https://github.com/BerriAI/litellm/pull/20718) [#19574](https://github.com/BerriAI/litellm/pull/19574)
- **Backward compatibility is a feature.** When I changed how custom auth enforces budgets, I shipped it behind an opt-in flag so existing deployments kept working. I took the same approach with model rate limits and deprecated config. [#22678](https://github.com/BerriAI/litellm/pull/22678) [#19230](https://github.com/BerriAI/litellm/pull/19230) [#19808](https://github.com/BerriAI/litellm/pull/19808)
- **Reproduce before you fix.** I reproduce every issue locally in the same environment to confirm the bug is real and see exactly how it fails, then lock it in with tests. For example, I corrupted a migration on a fresh Postgres to watch the proxy start broken, and built the migration lock against 14 tests and a multi-pod simulation. [#23257](https://github.com/BerriAI/litellm/pull/23257) [#19261](https://github.com/BerriAI/litellm/pull/19261)
- **Own the release, not just the PR.** I cut releases, rolled back a dependency bump within half an hour, and documented rollback safety. [#22631](https://github.com/BerriAI/litellm/pull/22631) [#23017](https://github.com/BerriAI/litellm/pull/23017) [#21743](https://github.com/BerriAI/litellm/pull/21743)
- **Build for real production: many pods, streamed responses.** Code that works on one server and a full response can fail at scale. I made migrations, SSO logins and key rotation safe across replicas, and made guardrails work on streamed chunks. [#19261](https://github.com/BerriAI/litellm/pull/19261) [#20314](https://github.com/BerriAI/litellm/pull/20314) [#23364](https://github.com/BerriAI/litellm/pull/23364) [#21949](https://github.com/BerriAI/litellm/pull/21949)

---

## 🤝 What I learned from Krrish, Ishaan and the team

The biggest lessons weren't technical. Working with [Krrish](https://github.com/krrish-berri-2), [Ishaan](https://github.com/ishaan-jaff) and the [LiteLLM team](https://github.com/BerriAI) taught me how to work:

- **Get clarity before writing code.** A five-minute conversation with the person who hit a bug often saved me a day of guessing at the cause.
- **Write every piece of feedback down.** Feedback that isn't written down turns into a request someone has to repeat.
- **Follow up until it's closed.** A fix isn't done when it merges; it's done when the person who reported it confirms it works.
- **Don't wait to be assigned.** If you spot the problem, own it.
- **Ship fast.** More than half of my merged PRs went from opened to merged in under 24 hours.

I still work this way, and it's the habit I'd most want to keep if I ever build something of my own.

**Thank you** to Krrish, Ishaan and the whole LiteLLM team: for trusting me with ownership of guardrails and releases early on, for letting me sit close enough to users to learn all this, and for the reviews, the patience and the fast merges. 🙏

**Want to talk about AI infra, gateways or guardrails?** Find me on [GitHub](https://github.com/Harshit28j).

*Previously: [Google Summer of Code 2024 with CloudCV](https://github.com/Harshit28j/GSoC-work)*
