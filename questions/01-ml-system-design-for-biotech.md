# 1. ML System Design for Biotech

End-to-end system design questions specific to building ML systems around biological data and scientific workflows.

---

### Q: Design an ML system to prioritize which compounds a drug discovery team should synthesize and test next. What components would you build? 🟡

**Answer:**
Key components:
1. **Data ingestion layer:** pulling in structured assay results (potency, ADMET data) from lab information management systems (LIMS), plus compound structure data, into a consistent, queryable format.
2. **Feature/representation pipeline:** computing molecular representations (fingerprints, graph features, or embeddings — see the AI Drug Discovery Scientist companion repo for the modeling-methods side of this) from compound structures.
3. **Model serving layer:** one or more property-prediction models (potency, ADMET properties) that can score candidate molecules, ideally with uncertainty estimates alongside point predictions.
4. **Ranking/optimization layer:** combining multi-property predictions into a prioritized ranking, incorporating business constraints (synthesis feasibility, budget) — this is often a separate business-logic layer on top of raw model outputs, not baked directly into the models themselves.
5. **Feedback loop:** capturing new experimental results as they come in and feeding them back into model retraining (active learning), which requires a robust, low-friction pipeline connecting lab data systems back to the training data store.
6. **Interface layer:** a way for chemists to view and interact with rankings/predictions (dashboard, notebook integration, or embedded in existing lab software) — as an ML engineer, you're responsible for making this genuinely usable by non-ML scientists, not just exposing a raw API.

As the ML engineer, I'd emphasize that the **feedback loop and data pipeline reliability** are usually the highest-leverage, most failure-prone parts of this system in practice — a great model fed by a broken or delayed data pipeline delivers no real value.

**Follow-ups:**
- How would you design this system to gracefully handle a period where the LIMS integration is temporarily unavailable, without silently serving stale rankings as if they were current?

---

### Q: How would you design an ML system for a genomics company that needs to run variant-calling and downstream analysis at the scale of hundreds of thousands of patient samples? 🔴

**Answer:**
Key design considerations:
- **Pipeline orchestration at scale:** use a workflow management system (e.g., Nextflow, Cromwell/WDL) capable of scheduling and parallelizing many independent per-sample jobs across a compute cluster or cloud infrastructure, rather than a monolithic script — per-sample parallelism is a natural fit here since most steps (alignment, variant calling) operate independently per sample.
- **Cost-aware infrastructure choices:** genomic pipelines are compute- and storage-intensive at this scale, so decisions about spot/preemptible instances, storage tiering (hot vs. cold storage for raw versus processed data), and compute right-sizing per pipeline stage have material cost implications that need active management, not just a "throw more compute at it" default.
- **Data provenance and reproducibility:** at this scale, with samples processed over a long time period potentially using evolving pipeline versions, rigorous tracking of exactly which pipeline version, reference genome version, and parameters were used for each sample is essential — inconsistent processing across a large cohort can silently introduce artifacts if any of these change mid-stream without proper tracking.
- **Fault tolerance:** at hundreds of thousands of samples, some non-trivial fraction of jobs will fail for various reasons (corrupted input, transient infrastructure issues) — the system needs automated retry logic, clear failure alerting, and the ability to resume/reprocess failed samples without re-running the entire cohort.
- **Downstream aggregation and querying:** design a data storage/query layer (e.g., a queryable database or data warehouse of variant calls and sample metadata) that supports the actual downstream analytical use cases (e.g., cohort-level association studies) efficiently, since raw per-sample VCF files alone aren't practically queryable at this scale.

**Follow-ups:**
- How would you design monitoring/alerting so that a systematic issue affecting a subset of samples (e.g., a specific sequencing batch) is caught quickly, rather than being discovered only much later during downstream analysis?

---

### Q: How would you approach designing an ML system when the "ground truth" labels come from noisy, expensive wet-lab experiments, and the team wants to minimize the number of experiments needed? 🟡

**Answer:**
This is fundamentally an active learning system design problem:
- **Model serving with uncertainty quantification:** the deployed model needs to output not just predictions but calibrated uncertainty estimates, since the active learning loop's core logic depends on identifying which unlabeled candidates the model is most uncertain about (or which would be most informative to label next).
- **A candidate selection/acquisition service:** a component that takes the model's predictions and uncertainty estimates and applies an acquisition strategy (e.g., uncertainty sampling, expected improvement) to select the next batch of candidates for experimental testing, respecting real-world constraints like experimental batch size and turnaround time.
- **A tight feedback loop with the experimental team's data systems:** since the entire value proposition of this system depends on incorporating new experimental results quickly, engineering effort should prioritize minimizing the latency and friction of getting new labels from the lab back into the training pipeline, over further optimizing initial model architecture.
- **Versioned, auditable retraining:** each round of active learning updates the model based on new data, so the system needs to track exactly which model version made which recommendation, and be able to reconstruct/audit this history — both for debugging and because scientific collaborators will reasonably want to understand why specific past recommendations were made.

**Follow-ups:**
- How would you design the system to handle the case where the experimental team can only realistically test a small batch (e.g., 20 candidates) per round, while the model could in principle rank thousands of candidates?

---

### Q: A biotech company wants to build a system that lets scientists query "which of our historical experiments are most similar to this new one" using free-text descriptions. How would you approach this? 🟡

**Answer:**
- **Clarify what "similar" should mean** for the actual use case — similar in experimental design/protocol, similar in biological question/hypothesis, or similar in observed outcome/results? Each implies a different underlying representation and matching approach, and this ambiguity needs to be resolved with the scientist stakeholders before building anything.
- **Consider an embedding-based semantic search approach:** encode experiment descriptions/metadata (and possibly structured metadata fields) into a shared embedding space using a text embedding model, and support similarity search via nearest-neighbor lookup — likely combined with structured metadata filtering (e.g., experiment type, date range) rather than relying purely on unstructured free-text similarity.
- **Consider data quality and consistency of the historical corpus** — if historical experiment descriptions were recorded inconsistently (varying levels of detail, inconsistent terminology across different scientists/teams/eras), a semantic search system trained or applied naively on this heterogeneous corpus may produce inconsistent-feeling results, and this data quality issue needs to be surfaced early as a real constraint on what's achievable, not treated as a pure modeling problem.
- **Plan for evaluation with actual scientist users** rather than only offline embedding-similarity metrics, since "relevance" here is inherently a subjective, domain-expert judgment that needs to be validated against real usage, not just an automatic proxy metric.

**Follow-ups:**
- How would you evaluate whether this semantic search system is actually useful to scientists, given that "relevance" here is hard to define with a single automatic metric?

---

### Q: How would you design a system to detect and flag potential data quality issues (e.g., mislabeled samples, contaminated assay plates) automatically, before they propagate into downstream ML models? 🔴

**Answer:**
- **Establish automated, systematic QC checks as an explicit pipeline stage**, not an ad hoc manual review step — e.g., flagging samples with metrics outside expected distributions (based on historical data), detecting statistical outliers relative to expected experimental controls, and cross-checking sample identity/metadata consistency where multiple data sources should agree.
- **Build in "sanity check" comparisons against known controls or expected reference values** built into most well-designed biological experiments — a system that automatically checks these controls against expected behavior can catch systematic issues (e.g., a contaminated reagent batch) much faster than waiting for a human to notice something looks odd in downstream results.
- **Design alerting with an appropriate signal-to-noise ratio** — overly sensitive automated QC flagging that generates constant false alarms will quickly be ignored by the scientific team, so thresholds need to be calibrated (ideally collaboratively with domain experts) to catch genuinely actionable issues without alert fatigue.
- **Make flagged issues visible and actionable**, not just logged — e.g., integrating flags into the same dashboards/interfaces scientists already use, with enough context (what triggered the flag, how severe/confident the system is) for them to quickly triage and decide whether to exclude, re-run, or manually review the flagged data.
- **Treat this as a continuously evolving system**, since new failure modes will be discovered over time that weren't anticipated in the initial QC rule set — build in a feedback mechanism so scientists can report missed issues that should inform future QC rule additions.

**Follow-ups:**
- How would you balance the tradeoff between an automated QC system that's sensitive enough to catch subtle issues versus one that generates too many false alarms and gets ignored?
