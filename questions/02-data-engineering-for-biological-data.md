# 2. Data Engineering for Biological Data

The unglamorous, high-leverage plumbing work of getting biological data into a form ML systems can actually use reliably.

---

### Q: What makes biological/biotech data pipelines particularly challenging compared to typical business/product data pipelines? 🟡

**Answer:**
Several distinguishing characteristics:
- **Extreme heterogeneity of data types and formats:** a single biotech organization might need to handle genomic sequencing files, microscopy images, structured assay results from lab instruments, unstructured lab notebook text, and chemical structure data — each with different scale characteristics, formats, and domain-specific quality considerations, unlike a typical product analytics pipeline dealing with more uniform event/transaction data.
- **Data generation is physically constrained and slow:** unlike web/app data that can often be generated and collected continuously and cheaply, biological data frequently comes from real, slow, expensive wet-lab experiments — meaning pipeline design needs to treat each data point as comparatively precious, with correspondingly higher priority on not losing or corrupting data, and often needs to gracefully handle irregular, bursty data arrival patterns tied to experimental schedules rather than a steady stream.
- **Domain-specific metadata is critical and easy to lose:** a raw measurement (e.g., a gene expression value) is often close to meaningless without carefully preserved experimental metadata (sample source, processing batch, protocol version, reagent lot) — pipelines need to be designed with this metadata as a first-class citizen, not an afterthought, since it's frequently essential for later catching confounds like batch effects (see the Genomics Data Scientist and Computational Biologist companion repos).
- **Longer-lived data and evolving standards:** biological reference data (genome assemblies, ontologies, annotation databases) gets updated periodically, and pipelines need explicit strategies for handling and tracking these version transitions over the multi-year lifespans that biotech datasets and projects often have, unlike many product datasets with shorter practical relevance windows.

**Follow-ups:**
- How would you design a data schema/metadata standard that ensures experimental context (batch, protocol, reagent lot) is never accidentally lost or disconnected from the raw measurement data as it flows through a pipeline?

---

### Q: How would you design a data storage strategy for a biotech company generating large volumes of raw sequencing or imaging data, balancing cost, accessibility, and long-term retention needs? 🟡

**Answer:**
- **Tiered storage strategy:** use fast, more expensive "hot" storage for actively-used raw and processed data, and migrate older/rarely-accessed raw data to cheaper "cold"/archival storage tiers, since raw sequencing/imaging data is typically accessed frequently only around the time it's initially processed and much less frequently afterward, but often needs to be retained long-term for reproducibility, regulatory, or future-reanalysis purposes.
- **Separate raw data from derived/processed data clearly**, with clear provenance linking processed outputs back to their raw source data and the exact pipeline version used — this supports reproducibility and allows processed data to potentially be regenerated from raw data if a pipeline bug is later discovered, rather than raw data being prematurely deleted.
- **Consider compression strategies specific to the data type** — e.g., specialized compression formats for sequencing data (like CRAM instead of BAM) can meaningfully reduce storage costs at scale without full information loss, and should be evaluated rather than defaulting to generic storage without considering domain-specific options.
- **Plan retention policies explicitly and in consultation with scientific and compliance stakeholders**, since biological data retention requirements can be driven by regulatory, IP, or scientific-reproducibility considerations that aren't purely an engineering cost-optimization decision — a purely cost-driven retention policy risks conflicting with real organizational obligations.

**Follow-ups:**
- How would you decide whether to store raw instrument output files indefinitely versus deleting them once validated processed data exists, given the cost implications at scale?

---

### Q: How would you handle schema evolution in a biological data pipeline, given that experimental protocols and data formats often change over time as science and instrumentation evolve? 🟡

**Answer:**
- **Design schemas with explicit versioning from the start**, rather than assuming a fixed, unchanging schema — biological data pipelines are especially prone to evolving requirements as new assay types, instruments, or experimental protocols are introduced over a project's multi-year lifespan.
- **Use a schema design that tolerates optional/extensible fields gracefully** (e.g., structured formats that support optional fields or nested extensions) rather than a rigid schema that breaks whenever a new experimental variant introduces a field that wasn't originally anticipated.
- **Maintain backward compatibility or clear, well-documented migration paths** when schema changes are unavoidable, so that historical data doesn't become silently unreadable or misinterpreted by pipelines that assume the newer schema version.
- **Communicate schema changes proactively to downstream consumers** (other pipelines, analysis notebooks, dashboards) rather than allowing changes to propagate silently and break downstream consumers unexpectedly — this requires treating internal data schemas with some of the same discipline as a public API contract.

**Follow-ups:**
- A new experimental protocol introduces a data field that doesn't fit your existing schema, and the scientific team needs to start using it immediately. How would you handle this without blocking their work or compromising pipeline integrity?

---

### Q: How would you build a data pipeline to ingest and standardize assay results coming from multiple different lab instruments, each with its own proprietary output format? 🟡

**Answer:**
- **Build format-specific parsers/adapters for each instrument's output**, converting each into a common, standardized internal representation — rather than trying to force a single generic parser to handle all formats, which usually leads to fragile, hard-to-maintain code.
- **Define a canonical internal data model** that captures the essential information needed downstream (measurement values, units, sample identifiers, relevant metadata) independent of any specific instrument's native format, and validate that each format-specific adapter correctly maps into this canonical model.
- **Preserve original raw instrument output alongside the standardized version**, rather than discarding it after conversion — this supports debugging when something looks wrong downstream and provides a fallback if a conversion bug is later discovered.
- **Build automated validation checks for each adapter** (e.g., sanity-checking that converted values fall within biologically/technically plausible ranges) since a subtly incorrect unit conversion or field-mapping bug in one instrument's adapter can silently corrupt data in a way that's hard to detect just by inspecting the pipeline code.
- **Plan for new instrument types being added over time** by designing the adapter architecture to be easily extensible (a clear, well-documented pattern for adding a new adapter) rather than requiring significant rearchitecting each time a new instrument is introduced.

**Follow-ups:**
- One instrument's proprietary format occasionally has ambiguous or instrument-vendor-inconsistent unit labeling. How would you design your pipeline to catch and handle this rather than silently mis-converting values?

---

### Q: What role does a feature store play in an ML system built on biological data, and what unique considerations apply compared to a typical product/business feature store? 🔴

**Answer:**
A feature store centralizes the computation, storage, and serving of features used by ML models, aiming to ensure consistency between the features used during model training and those used at inference/serving time, and to enable feature reuse across multiple models/teams rather than each team recomputing similar features independently.

Unique considerations for biological data: **features are often expensive and slow to compute** (e.g., molecular representations from complex cheminformatics computations, or genomic features requiring substantial pipeline processing), so caching and efficient recomputation-avoidance matters even more than in many typical product contexts where features might be simple aggregations computed cheaply on demand; **features often need to be tied to specific reference data versions** (a genomic feature is only meaningful relative to a specific reference genome/annotation version, similar to concerns discussed in the Genomics Data Scientist companion repo), so a biotech feature store needs robust versioning not just of the feature computation logic but of the underlying reference data it depends on; and **the relevant "entity" for features is often more complex and multi-level** than a typical product feature store's simpler "user" or "item" entities — e.g., features might need to be tracked at the level of a sample, a batch, a patient, a compound, or a specific assay run, with complex relationships between these levels that the feature store's data model needs to represent faithfully.

**Follow-ups:**
- How would you design a feature store to handle the case where a feature's correct value depends on which reference genome version was used to compute it, ensuring a model doesn't inadvertently mix features computed against different reference versions?
