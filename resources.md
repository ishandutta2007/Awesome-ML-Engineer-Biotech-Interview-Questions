# Additional Resources

A short list of resources for going deeper on ML engineering in biotech. (Not exhaustive — PRs welcome.) This role sits at the intersection of general ML/software engineering and biotech domain context, both of which evolve quickly — cross-check anything specific against current documentation before an interview.

## Foundational Reading
- *Designing Machine Learning Systems* — Chip Huyen. Excellent general ML systems/MLOps grounding that translates well into a biotech context.
- *Designing Data-Intensive Applications* — Martin Kleppmann. Not biotech-specific, but foundational for the data engineering and infrastructure reasoning this role demands.
- Company engineering blogs from biotech/pharma organizations building ML platforms (e.g., posts on genomics pipeline infrastructure, drug discovery ML platforms) — searching for recent engineering blog posts from active biotech ML platform teams is a good way to see how these tradeoffs play out in practice.

## Companion Repos (domain science depth)
- **Computational Biologist** — core algorithms, structural/systems biology, statistical methods.
- **Genomics Data Scientist** — NGS pipelines, variant calling, statistical genomics.
- **AI Drug Discovery Scientist** — cheminformatics, generative molecule design, structure-based design.

These cover the domain science and modeling-methods side in depth; this repo focuses on the engineering/infrastructure/production side.

## Tooling (good to have working familiarity with, depending on role focus)
- A workflow orchestration system relevant to genomics/bioinformatics pipelines (e.g., Nextflow, Cromwell/WDL, Snakemake) if the role touches pipeline engineering.
- Standard MLOps tooling for experiment tracking, model registry, and deployment (many exist — familiarity with the general category matters more than any one specific product).
- Cloud provider documentation for whichever platform(s) a target company uses, particularly around HPC-style batch computing, GPU scheduling, and storage tiering options.
- Containerization and workflow reproducibility tools (Docker/Singularity, conda) as discussed in the Computational Biologist companion repo's reproducibility section.

## Communities & Discussion
- r/bioinformatics and r/MachineLearning for discussion threads.
- MLOps-focused communities and newsletters for staying current on general production ML engineering practices that translate into this domain.

## Staying Current
- Foundation models for biology (protein language models, genomic sequence models) and the infrastructure needed to train/serve them are an active, fast-moving area — following recent technical blog posts and papers from active industry/academic groups is more valuable here than any static reference.
- MLOps tooling and best practices generally evolve quickly; treat any specific tool recommendation (here or elsewhere) as a snapshot in time rather than a permanent industry standard.
