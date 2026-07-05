# 3. Model Development at Scale

Training and developing models where the data has biology-specific challenges — scale, imbalance, multi-modality, and scarcity of labels.

---

### Q: How would you approach distributed training for a large model on biological data (e.g., a protein language model or a large genomic sequence model), and what unique challenges arise compared to distributed training in other domains? 🔴

**Answer:**
General distributed training principles apply (data parallelism, model parallelism, or a combination, depending on model and dataset size), but a few considerations are especially salient for biological sequence models:
- **Sequence length variability and extremes:** biological sequences (protein sequences, genomic regions) can vary enormously in length, and some biologically important sequences (e.g., full chromosomes, or very long proteins) can be far longer than typical NLP sequence lengths — this has real implications for batching strategy (padding/bucketing to minimize wasted compute) and for whether standard attention mechanisms are computationally feasible without architecture modifications (e.g., sparse or linear-attention variants) for very long sequences.
- **Data loading bottlenecks specific to biological file formats:** efficiently streaming and parsing large-scale genomic or sequence data during training (rather than assuming data is already in a simple, training-ready tabular format) can become a real bottleneck if not engineered carefully, and this is a common, underappreciated source of underutilized GPU compute in practice — it's important to profile and confirm the training pipeline isn't data-loading-bound before focusing purely on compute optimization.
- **Dataset scale and deduplication concerns:** large biological sequence databases often contain substantial near-duplicate or highly redundant sequences (e.g., many closely related protein sequences from related species) — training on this data without appropriate deduplication or reweighting can bias the model and also affect the validity of any held-out evaluation split (similar to the general splitting concerns raised in the AI Drug Discovery Scientist and Computational Biologist companion repos).

**Follow-ups:**
- How would you profile a distributed training job to determine whether it's compute-bound or data-loading-bound, and what would you do differently depending on the answer?

---

### Q: How would you approach fine-tuning a pretrained foundation model (e.g., a protein language model or a general-purpose scientific foundation model) for a specific, narrower biotech application with limited labeled data? 🟡

**Answer:**
- **Start with the smallest viable approach** — e.g., using the pretrained model's frozen embeddings as input features to a much simpler downstream model (like a lightweight classifier/regressor), rather than immediately committing to full fine-tuning of the entire foundation model, since full fine-tuning is more compute-intensive and more prone to overfitting when labeled data is limited.
- **Consider parameter-efficient fine-tuning approaches** (e.g., adapter layers, or other lightweight fine-tuning techniques) as a middle ground that can achieve much of full fine-tuning's benefit at a fraction of the compute and overfitting risk, particularly valuable when labeled data is scarce.
- **Rigorously validate using an appropriate held-out split** for the specific biological domain (e.g., a scaffold split for molecules, or a phylogenetically-aware split for sequences, per the evaluation rigor discussed in the AI Drug Discovery Scientist and Computational Biologist companion repos) rather than a naive random split, since foundation models can appear to generalize well under a lenient split while actually just leveraging memorized similarity to training data.
- **Be realistic about the actual size of labeled data needed** — foundation model fine-tuning can meaningfully reduce, but doesn't eliminate, the need for labeled data, and setting appropriate expectations with stakeholders about how much labeled data is actually needed for a reliable result is an important part of scoping the project correctly upfront.

**Follow-ups:**
- How would you decide between frozen-embedding-plus-simple-classifier versus parameter-efficient fine-tuning versus full fine-tuning, given a specific labeled dataset size?

---

### Q: How would you engineer a training pipeline to handle severe class imbalance, which is extremely common in biological datasets (e.g., rare disease cases, rare positive screening hits)? 🟡

**Answer:**
- **Implement configurable resampling and class-weighting strategies** in the training pipeline (oversampling minority class, undersampling majority class, or weighted loss functions), and make these easy to experiment with and tune rather than hardcoding a single fixed approach, since the right strategy is often dataset- and task-specific.
- **Ensure evaluation metrics and monitoring are appropriate for imbalanced data** (precision-recall AUC, per-class metrics) baked into the standard training/evaluation harness by default, so that a severely imbalanced dataset doesn't silently produce a misleadingly high aggregate accuracy metric that masks poor minority-class performance — this is as much an engineering/tooling responsibility (making the right metrics easy and default) as a modeling decision.
- **Build in safeguards against data leakage introduced by naive oversampling** — e.g., if oversampling is applied before a train/validation split (duplicating minority-class examples that then end up split across both train and validation), this can produce inflated, misleading validation performance; the pipeline should apply any resampling only within the training fold, after splitting, to avoid this.
- **Support and encourage active-learning-style workflows** where feasible (see also the ML System Design section) since, in biology specifically, class imbalance is often really a symptom of a broader "expensive to generate positive labels" problem, and an engineering solution that improves the efficiency of acquiring more positive labels can be more impactful than purely algorithmic imbalance-handling techniques applied to a fixed, static dataset.

**Follow-ups:**
- How would you build safeguards into a training pipeline to catch a case where a well-intentioned resampling step accidentally introduces train/validation leakage?

---

### Q: How would you design a model training/evaluation pipeline to support the kind of rigorous, non-standard data splitting (e.g., scaffold splits for molecules, phylogenetically-aware splits for sequences) that's often necessary for biological data, rather than defaulting to a generic random split? 🟡

**Answer:**
- **Build splitting logic as a pluggable, explicit, and well-tested pipeline component**, rather than a generic default utility function buried deep in a shared library — since the correct splitting strategy is genuinely domain- and task-dependent (as discussed extensively in the AI Drug Discovery Scientist and Computational Biologist companion repos), the pipeline should make it easy and natural for a scientist/ML practitioner to specify and swap in the appropriate domain-specific splitting strategy rather than silently defaulting to a plain random split that could produce misleadingly optimistic results.
- **Log and version exactly which splitting strategy and parameters were used** for any given training run, since this is a critical piece of provenance for correctly interpreting reported model performance later — a model's "great validation accuracy" is only meaningful in the context of knowing how rigorously the split was constructed.
- **Provide built-in support for common domain-specific splitting strategies** (e.g., scaffold splitting utilities, temporal splitting) as first-class pipeline options, reducing the friction and error-risk of a scientist having to hand-roll this logic themselves for every project.
- **Build automated checks that flag suspiciously easy splits** — e.g., a utility that reports the similarity distribution between train and test sets under a chosen split, surfacing a warning if train/test similarity looks unexpectedly high, which can catch accidental leakage or an inappropriately lenient splitting choice before it leads to an overoptimistic reported result.

**Follow-ups:**
- How would you retrofit rigorous, domain-appropriate splitting into an existing pipeline that has been using naive random splits, given that past reported model performance numbers might have been overoptimistic as a result?

---

### Q: What engineering practices would you put in place to prevent silent data leakage in a complex, multi-stage biological ML pipeline (e.g., involving multiple preprocessing, feature engineering, and modeling stages)? 🔴

**Answer:**
- **Enforce a strict, pipeline-level separation between training and evaluation data as early as possible** in the pipeline (ideally right after the initial domain-appropriate split), rather than allowing any downstream preprocessing/feature-engineering step to have access to information across the train/eval boundary — e.g., any normalization, imputation, or feature-selection step that's fit on data should be fit only on the training fold and then applied (not re-fit) to the evaluation fold.
- **Build automated tests/checks that specifically probe for leakage** — e.g., a test that intentionally checks whether any evaluation-fold sample identifiers appear anywhere in training-fold feature computation logic or lookup tables, or that measures whether a model can predict which fold ("train" vs. "eval") a sample came from suspiciously well (which can indicate the fold split correlates with an unintended artifact).
- **Make pipeline stages explicit and inspectable**, so a reviewer can trace exactly what data each stage had access to, rather than a monolithic, opaque pipeline where it's hard to audit whether leakage could have occurred at some intermediate step.
- **Treat leakage prevention as a code review and pipeline-design responsibility, not just a data science responsibility** — as the ML engineer, building pipeline architecture and tooling that makes the *correct*, leak-free approach the *easy default* path (rather than requiring every practitioner to remember to do it correctly every time) is one of the highest-leverage things you can do to prevent this class of error across an organization.

**Follow-ups:**
- How would you design an automated regression test that would have caught a real, previously-shipped data leakage bug in one of your pipelines, without knowing in advance exactly what form future leakage bugs might take?
