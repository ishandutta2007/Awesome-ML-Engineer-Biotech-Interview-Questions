# 5. Infrastructure, Compute & Cost

Compute and storage decisions in biotech ML have real cost stakes given the scale of genomic, imaging, and simulation data.

---

### Q: How would you decide between using on-premise HPC infrastructure versus cloud computing for a biotech company's ML/computational workloads? 🟡

**Answer:**
Key tradeoffs:
- **Workload predictability and utilization:** on-premise HPC infrastructure has high upfront capital cost but can be cheaper per unit of compute if utilization is consistently high (e.g., a steady, predictable stream of genomic pipeline jobs) — cloud computing's pay-as-you-go model tends to be more cost-effective for bursty, unpredictable, or rapidly-scaling workloads where on-premise infrastructure would otherwise sit underutilized much of the time.
- **Data gravity and transfer costs:** if large volumes of data are already generated and stored on-premise (e.g., from in-house sequencing/imaging instruments), the cost and time of transferring this data to the cloud for processing can be a real, sometimes underestimated factor favoring on-premise or hybrid approaches, at least for initial processing stages close to data generation.
- **Specialized hardware availability:** certain workloads (e.g., large-scale deep learning training requiring the latest GPU generations) may be more readily and flexibly accessible via cloud providers than through on-premise procurement cycles, which can lag behind the latest hardware availability and require significant lead time to provision.
- **Compliance and data governance requirements:** some biotech data (particularly clinical or patient data) may have specific data residency, security, or compliance requirements that influence or constrain the choice between cloud and on-premise infrastructure, or dictate using specific compliant cloud service configurations rather than a completely free choice.

Many organizations land on a **hybrid approach**: on-premise infrastructure for steady-state, predictable workloads (and data close to instruments), with cloud bursting for peak/variable demand (e.g., large-scale training runs, or large batch reprocessing jobs) — I'd frame this as an ongoing cost/workload-pattern analysis rather than a one-time, permanent decision.

**Follow-ups:**
- How would you model and compare the true cost of on-premise versus cloud infrastructure for a specific, real workload, accounting for factors beyond just raw compute hourly rates?

---

### Q: How would you approach GPU resource scheduling/allocation in a shared research environment where multiple scientists and ML engineers compete for limited GPU resources? 🟡

**Answer:**
- **Implement a fair-share scheduling policy** (e.g., using a cluster scheduler with configurable priority/fair-share policies) so that resource-hungry, long-running jobs from one team or individual don't indefinitely starve others of access, while still allowing legitimate high-priority or time-sensitive work to be appropriately prioritized when justified.
- **Support job preemption for lower-priority workloads** where feasible (e.g., allowing a high-priority job to preempt a lower-priority one that can be safely checkpointed and resumed later) to improve overall utilization without permanently blocking access for any single job.
- **Provide visibility into current utilization and queue status** to users, so they can make informed decisions about when to submit jobs or how much resource to request, rather than experiencing scheduling as an opaque black box that leads to frustration and workarounds (e.g., people requesting more resources than needed "just in case," which further worsens contention).
- **Set and enforce reasonable resource request practices** (e.g., requiring realistic time/resource estimates, flagging or limiting excessively large requests without justification) to prevent a small number of jobs or users from disproportionately consuming shared resources.
- **Consider workload-appropriate infrastructure segmentation** — e.g., reserving a portion of GPU capacity for interactive/exploratory work (where scientists need quick iteration) separate from a queue for large, long-running training or batch jobs, since these have very different usage patterns and blocking one behind the other creates poor experiences for both types of users.

**Follow-ups:**
- How would you handle a situation where one team consistently over-requests GPU resources "just in case," degrading overall cluster utilization and fairness for other teams?

---

### Q: What storage architecture considerations are specific to handling genome-scale or large imaging datasets efficiently in an ML pipeline? 🟡

**Answer:**
- **Choose file formats and storage layouts optimized for the actual access patterns** — e.g., using indexed, randomly-accessible formats (like indexed BAM/CRAM for genomic data, or chunked formats for large imaging arrays) rather than formats requiring full sequential reads, when downstream processing needs to efficiently access specific regions/subsets of very large files.
- **Consider data locality relative to compute** — for genome-scale data especially, moving large files repeatedly between storage and compute can become a significant bottleneck and cost driver; architectures that colocate compute close to where data is stored (or use efficient, high-throughput networked storage designed for HPC-style access patterns) can meaningfully outperform naive setups where data is repeatedly transferred over slower general-purpose network storage.
- **Use appropriate compression** balancing storage cost savings against the computational cost of decompression during frequent access — highly compressed formats save storage space but can introduce meaningful CPU overhead if data needs to be decompressed repeatedly for frequent access patterns, so this tradeoff should be evaluated empirically for the specific access pattern rather than assumed.
- **Design a clear data lifecycle/tiering policy** (as discussed in the Data Engineering section) so that storage costs scale sensibly with actual data value/access frequency over time, rather than treating all data as equally "hot" indefinitely.

**Follow-ups:**
- You notice that a genomic pipeline's runtime is dominated by I/O/data transfer time rather than actual computation. What architectural changes would you investigate to address this?

---

### Q: How would you approach cost optimization for a large-scale model training or inference workload without compromising scientific reproducibility or model quality? 🔴

**Answer:**
- **Right-size compute resources based on empirical profiling**, not assumption — e.g., confirming whether a training job is actually compute-bound (justifying more/faster GPUs) or bottlenecked elsewhere (data loading, I/O), since throwing more expensive compute at a bottleneck that isn't actually compute-bound wastes money without proportional benefit.
- **Use spot/preemptible instances for interruption-tolerant workloads** (e.g., large batch scoring jobs, or training jobs with robust checkpointing that can resume cleanly after an interruption) to meaningfully reduce cost, while reserving more expensive guaranteed/on-demand compute for latency-sensitive or interruption-intolerant workloads.
- **Cache and reuse expensive intermediate computations** (e.g., precomputed molecular/sequence features or embeddings that don't need to be recomputed for every single training run or experiment) rather than redundantly recomputing them, which is a common source of avoidable compute cost in iterative research/ML workflows.
- **Maintain reproducibility discipline even while optimizing cost** — e.g., ensuring that cost-driven infrastructure choices (like using spot instances) don't compromise the ability to exactly reproduce a given experiment's results later, by making sure checkpointing/logging captures everything needed to reproduce or resume a run regardless of the underlying compute infrastructure's variability.
- **Build cost visibility and attribution into the platform** (e.g., tracking compute cost per project/team/experiment) so that cost tradeoffs become a visible, informed part of research decision-making rather than an invisible, centrally-absorbed cost that individual researchers have no visibility into or incentive to consider.

**Follow-ups:**
- How would you design a training pipeline's checkpointing strategy specifically to make it robust to running on preemptible/spot compute instances that can be interrupted at any time?
