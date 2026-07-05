# 4. MLOps & Production Deployment

Getting biotech ML models into reliable, monitored production use — and keeping them there.

---

### Q: How would you design a model versioning and deployment strategy for a biotech ML system where scientists need to be able to trace exactly which model version produced a given historical prediction? 🟡

**Answer:**
- **Version everything that affects a prediction, not just model weights** — the model artifact itself, the exact preprocessing/feature computation logic and its version, the training data snapshot/version, and any reference data (e.g., reference genome version) the model depends on — since in biotech contexts, reproducing or explaining a past prediction often matters as much as generating new ones (for scientific credibility, debugging, and sometimes regulatory/audit purposes).
- **Log the full versioned context alongside every prediction served**, not just the prediction output itself, so that any historical prediction can later be traced back to exactly which model and pipeline configuration produced it — this is a heavier logging/storage burden than many typical product ML systems adopt by default, but is often a real, non-negotiable requirement in this domain.
- **Design deployment to support easy rollback** to a previous model version, given that a newly deployed model might later be found to have an issue (discovered through downstream monitoring or scientist feedback) that requires quickly reverting to a known-good previous version while the issue is investigated.
- **Make model version information visible to end users** (scientists) in whatever interface they interact with predictions through, rather than hiding it as pure backend metadata — scientists reasonably want to know which model version generated a given result, especially when comparing results generated at different points in time.

**Follow-ups:**
- A scientist reports that a prediction they got today differs from a seemingly identical query they ran last month. How would your versioning/logging design help you investigate this?

---

### Q: What would you monitor in production for an ML model making predictions on biological data, beyond standard software/infrastructure metrics (latency, uptime)? 🟡

**Answer:**
- **Input data distribution monitoring:** track whether incoming data (e.g., new compound structures, new sample characteristics) looks similar to the model's training distribution, flagging significant drift — since biological data pipelines often evolve (new experimental protocols, new sample types) in ways that can silently push production data outside a model's validated applicability domain (see the AI Drug Discovery Scientist and Computational Biologist companion repos for the domain-science framing of this concept).
- **Prediction distribution monitoring:** sudden shifts in the model's output distribution can be an early warning sign of an upstream data or pipeline issue, often catchable before downstream users even notice something is wrong.
- **Reference/dependency version monitoring:** if the model's inference pipeline depends on external reference data or databases (e.g., an annotation database, a reference genome), monitor for unexpected/unplanned version changes in these dependencies that could silently alter model behavior even with no code or model changes.
- **Scientist feedback and override rate:** track how often scientists disagree with, override, or flag a model's predictions as clearly wrong — this qualitative/semi-quantitative signal is often as valuable as automated statistical monitoring for catching real-world model quality issues in a domain where "ground truth" for a given prediction may not be available for a long time (until an expensive experiment confirms or refutes it).
- **End-to-end pipeline health**, not just the model-serving component in isolation — since biotech ML systems are often long, multi-stage pipelines (data ingestion → feature computation → model inference → business logic), monitoring needs to cover the full chain, since a failure or silent data-quality issue anywhere in this chain can produce a confidently-wrong final output even if the model-serving component itself is technically healthy.

**Follow-ups:**
- How would you design an alerting system that distinguishes a genuine, actionable model-quality regression from normal, expected variation in incoming biological data?

---

### Q: How would you design a CI/CD pipeline for ML models in a biotech context, given that model changes often need more rigorous validation than typical software changes before deployment? 🟡

**Answer:**
- **Automate the standard software CI checks** (unit tests, integration tests, code style/linting) as a baseline, same as any production software system.
- **Layer in ML-specific automated validation gates** beyond standard software tests — e.g., automatically running the candidate model against a held-out validation/regression set with domain-appropriate splitting (see section 3) and requiring it to meet or exceed defined performance thresholds (and not regress on specific known-important edge cases) before it's eligible for deployment, rather than relying purely on manual, ad hoc review of a model's reported metrics.
- **Include a staged/canary rollout process** for model updates rather than an immediate full cutover — e.g., running a new model version alongside the existing one on a subset of traffic/use cases first, comparing outputs, before fully promoting it to replace the previous production model.
- **Build in a required human review/sign-off step for model changes** that's more substantive than a typical code review — ideally involving a domain scientist reviewing the validation results and understanding what changed and why, not just an engineer approving that the code compiles and tests pass, given that a subtly wrong model can cause real scientific/business harm that automated tests alone might not catch.
- **Maintain a clear, fast rollback mechanism** as a core part of the pipeline design (see the versioning discussion above), since even with rigorous pre-deployment validation, real-world issues can still emerge after deployment that weren't caught by any validation gate.

**Follow-ups:**
- How would you decide what specific automated validation thresholds should block a model deployment, versus what should just generate a warning for human review?

---

### Q: How would you design model serving infrastructure for a use case with highly variable, unpredictable load (e.g., large batch scoring jobs interspersed with occasional interactive queries from scientists)? 🟡

**Answer:**
- **Separate infrastructure/serving paths for batch and interactive/real-time use cases** where their latency and throughput requirements genuinely differ — a single serving architecture optimized for low-latency interactive queries may be poorly suited (and unnecessarily expensive) for large batch scoring jobs that instead benefit from maximizing throughput and parallelism, and vice versa.
- **Use autoscaling infrastructure for interactive serving** to handle unpredictable interactive query volume cost-effectively, scaling down during idle periods (common in many biotech ML use cases where scientist query patterns are bursty rather than continuous) rather than provisioning for peak load at all times.
- **Use a job-queue-based architecture for batch scoring workloads**, allowing large batch jobs to be scheduled and processed efficiently (potentially using cheaper, preemptible/spot compute resources where the workload tolerates some interruption/retry) rather than requiring the same always-on, low-latency infrastructure used for interactive queries.
- **Monitor and set appropriate expectations with scientist users about latency for each path** — e.g., being explicit that interactive queries return in seconds while batch jobs may take hours, rather than leaving users uncertain about what performance to expect from a system that serves both use cases.

**Follow-ups:**
- How would you decide when a batch scoring job is large enough that it should be routed to the batch infrastructure path rather than processed through the interactive serving path?

---

### Q: How would you handle a situation where a model's dependency (e.g., an external annotation database or reference dataset) gets updated by a third party, potentially changing model behavior without any code or model change on your end? 🔴

**Answer:**
- **Pin exact versions of all external dependencies explicitly** in the production configuration, rather than always pulling "the latest" version of an external database/resource automatically — an update to an external dependency should be a deliberate, tracked, and tested change, not something that happens silently and unpredictably in production.
- **Build a regression testing process specifically for dependency updates** — before adopting a new version of an external reference dataset/database in production, run it through the same kind of validation regression suite used for model changes (section above), comparing outputs against the previous dependency version to catch unexpected behavior shifts before they reach production.
- **Monitor for and alert on unexpected drift in outputs** even without a known dependency version change, as a backstop for cases where a dependency update happens outside your direct control or awareness (e.g., a cloud-hosted external API silently updating its underlying data without a clear versioning contract) — treat this as one of the "unknown unknowns" that production monitoring specifically needs to be able to catch.
- **Maintain clear documentation of all external dependencies and their versions** as part of the model's overall version/provenance tracking (see the model versioning discussion above), so that if unexpected behavior is observed, dependency version changes are one of the first things that can be quickly checked and ruled in or out as a cause.

**Follow-ups:**
- You discover that a widely-used external annotation database your production system depends on was silently updated last month, and you're not sure whether this has affected your model's outputs. How would you investigate and respond?
