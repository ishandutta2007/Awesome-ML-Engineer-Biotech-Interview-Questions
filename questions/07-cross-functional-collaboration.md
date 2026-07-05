# 7. Cross-Functional Collaboration

Translating between scientific needs and engineering systems is core to this role — arguably more central here than in many other ML engineering contexts.

---

### Q: How would you approach gathering requirements from a computational biology or wet-lab science team who may not think in terms of typical software/ML engineering requirements? 🟡

**Answer:**
- **Start from the scientific question or workflow pain point, not a pre-assumed technical solution** — scientists often describe what they need in terms of the biological problem they're trying to solve or the manual process they're trying to speed up, rather than in engineering terms (APIs, data schemas, latency requirements), and it's the ML engineer's job to translate this into concrete technical requirements, not to expect scientists to already speak in engineering terminology.
- **Ask about current workarounds and manual processes** — understanding exactly how a scientist currently accomplishes a task manually (e.g., a specific spreadsheet-based workflow, or a series of manual tool invocations) often reveals concrete, specific requirements (edge cases, required output formats, decision criteria) that a more abstract requirements conversation would miss.
- **Prototype early and iterate based on concrete feedback**, rather than trying to extract a complete, precise specification upfront through requirements interviews alone — scientists (like most users) are often much better at reacting to something concrete ("this isn't quite right because...") than specifying abstract requirements from scratch.
- **Clarify the actual scale, frequency, and urgency of the need** explicitly, since scientific stakeholders may not naturally think to volunteer this information but it's critical for making sound engineering tradeoffs (e.g., is this a one-off analysis need or a system that needs to support ongoing, repeated use by many scientists).

**Follow-ups:**
- A scientist describes a need in a way that seems technically ambiguous or potentially inconsistent. How would you approach clarifying this without making them feel like their request was unclear or wrong?

---

### Q: How do you communicate technical/engineering constraints or tradeoffs (e.g., why a requested feature would take much longer than expected, or why a certain approach isn't feasible) to a scientific stakeholder who may not have an engineering background? 🟡

**Answer:**
- **Frame constraints in terms of the impact on their actual goals**, not abstract technical jargon — e.g., "this approach would mean results take hours instead of minutes, which would slow down your experimental iteration cycle" is more useful and persuasive to a scientist than a purely technical explanation of why an approach is computationally expensive.
- **Use concrete analogies or examples relevant to their domain** where helpful, similar to how technical explanations are calibrated to non-technical stakeholders in other cross-functional AI contexts (see the AI PM companion repo's discussion of this general skill).
- **Offer alternatives rather than just explaining why the original request isn't feasible** — proposing a scoped-down version, a different approach that achieves most of the value more feasibly, or a phased plan demonstrates genuine partnership rather than simply delivering bad news.
- **Be honest about genuine uncertainty in engineering estimates** rather than presenting a single confident timeline that then slips — biotech engineering work often has real uncertainty (e.g., dealing with messy, inconsistent legacy data), and communicating a realistic range with the key sources of uncertainty called out builds more durable trust than false precision.

**Follow-ups:**
- A stakeholder pushes back and insists the timeline you've proposed is too long, given that "it's just a database query." How would you handle this conversation?

---

### Q: How would you build trust with a scientific team that has been burned before by an engineering team delivering a tool that didn't actually fit their real workflow? 🟡

**Answer:**
- **Start with a small, well-scoped pilot delivering genuine, tangible value quickly**, rather than proposing a large, comprehensive system upfront — demonstrating concrete value early is far more persuasive than promises about a more ambitious future system, especially to a team that's understandably skeptical based on past experience.
- **Involve the scientific team actively throughout development**, not just at initial requirements-gathering and final delivery — regular check-ins with working, even partial, functionality lets them course-correct early rather than discovering a fundamental mismatch only at the end.
- **Explicitly ask about and address the specific ways past tools failed to fit their workflow**, rather than assuming your approach will naturally avoid the same pitfalls — understanding the specific history of what went wrong before is valuable diagnostic information, not just something to move past.
- **Be transparent about your own learning curve** in understanding their specific scientific domain and workflow, and demonstrate genuine curiosity about their work — scientists are often (reasonably) more trusting of engineers who show real interest in understanding the biology/chemistry, not just treating it as an abstract data source.

**Follow-ups:**
- Partway through a project, you realize your initial understanding of the scientific workflow was incomplete or somewhat wrong. How would you handle course-correcting without losing the team's trust?

---

### Q: How would you decide how much of a scientific team's existing workflow/toolset to preserve versus how aggressively to redesign it when introducing a new ML-powered system? 🔴

**Answer:**
- **Weigh the cost of workflow disruption against the value of the new capability** — scientists often have deeply ingrained, efficient habits around their existing tools (even ones that look inefficient or outdated from an engineering perspective), and forcing an abrupt, comprehensive workflow change carries real adoption risk and productivity cost that needs to be weighed against the genuine benefit the new system provides.
- **Favor integrating with existing tools/workflows where reasonably possible**, rather than requiring scientists to abandon familiar tools entirely — e.g., building the new ML capability as an enhancement accessible from within their existing analysis environment (e.g., a familiar notebook interface) rather than requiring them to learn and switch to an entirely new, separate interface, unless there's a strong reason the new capability genuinely requires a different interaction paradigm.
- **Pilot with a subset of enthusiastic early adopters first** before a broader rollout, gathering real usage feedback that can inform how much of the existing workflow genuinely needs to change versus can be preserved, rather than making this design decision purely speculatively upfront.
- **Recognize that some workflow disruption may genuinely be necessary and worth it** if the existing workflow has fundamental limitations that a more significant redesign is needed to address — the goal is a deliberate, evidence-informed tradeoff, not a default bias toward either extreme (never disrupting existing workflows, or always assuming a clean-slate redesign is best).

**Follow-ups:**
- How would you handle strong pushback from part of the scientific team who prefer their existing (if less efficient) workflow, while other parts of the team are enthusiastic about the new system?

---

### Q: Describe how you'd approach a situation where a scientific collaborator wants a feature/capability that you believe is technically feasible but scientifically unsound or likely to be misused/misinterpreted. 🔴

**Answer:**
- **Understand the underlying scientific goal behind the request** before pushing back, since the specific requested feature might just be one (flawed) way to achieve a legitimate underlying goal that could be addressed differently and more soundly.
- **Raise the concern directly and specifically**, grounded in the technical/scientific reasoning (e.g., "this approach would risk [specific failure mode], because..."), rather than a vague or purely deferential objection — this requires genuine engagement with the domain-science reasoning, not just an engineering-only perspective, similar to the collaborative-disagreement skills discussed in the Computational Biologist companion repo.
- **Propose an alternative that addresses the underlying need more soundly**, demonstrating that the pushback comes from wanting to genuinely help achieve their goal well, not just being an obstacle.
- **Escalate appropriately if the disagreement persists and the stakes are meaningful** — e.g., looping in a more senior scientific or technical stakeholder for a second opinion, rather than either unilaterally building something you believe is unsound or unilaterally refusing without further discussion.

**Follow-ups:**
- The collaborator has more scientific seniority/authority than you and insists on proceeding despite your technical concerns. How would you handle this while maintaining a good working relationship?
