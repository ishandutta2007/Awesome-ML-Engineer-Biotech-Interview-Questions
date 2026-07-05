# 8. Behavioral & Case Studies

Real-world scenarios blending engineering judgment with biotech-specific context. Many interviews include a live system design exercise — practice narrating tradeoffs, not just naming components.

---

### Q: Tell me about a time you had to build something for a scientific team under significant ambiguity about the exact requirements. How did you handle it? 🟡

**What a strong answer includes:**
- A specific example showing how ambiguity was reduced through concrete action (a quick prototype, a focused clarifying conversation, examining an existing manual workflow) rather than either guessing and building blind, or stalling indefinitely waiting for perfectly specified requirements.
- Evidence of appropriate iteration — building something, getting real feedback, and adjusting, rather than committing to a large upfront design based on incomplete understanding.
- A concrete, honest outcome, including any misjudgments made along the way and how they were corrected.

**Follow-ups:**
- Looking back, what would you have done differently to reduce the ambiguity earlier in the project?

---

### Q: Describe a time a production ML system you built or maintained had a real, discovered failure (a bug, a data quality issue, a model regression) that impacted scientific work. Walk me through your response. 🟡

**What a strong answer includes:**
- Honesty that production issues happen — interviewers are wary of candidates who claim their systems have never had a real issue in production.
- A clear, systematic incident response process: understanding the scope and severity of impact first, communicating transparently with affected stakeholders, and fixing the immediate issue before addressing root cause.
- A specific root-cause investigation and a concrete systemic fix (e.g., a new monitoring check, an automated test, a process change) implemented afterward to reduce the chance of recurrence — not just a one-off patch.
- Evidence of appropriately transparent communication with scientific stakeholders about what happened and its potential impact on their work/conclusions, rather than downplaying or hiding the issue.

**Follow-ups:**
- How did you determine which past results or decisions, if any, needed to be revisited given the discovered issue?

---

### Q: Walk me through how you'd design a system to let a biotech company's chemists submit new molecules for property prediction and get results back, from a completely blank slate. 🟡

**Live exercise structure — a strong candidate would:**
1. **Clarify scale and usage patterns** — how many molecules per submission, how frequently, how many concurrent users, and what latency expectations are reasonable (interactive versus batch).
2. **Clarify what "property prediction" means concretely** — which specific properties, using which existing or new models, and what output format/detail level chemists actually need (a single score, or more detailed supporting information like confidence and similar known compounds).
3. **Sketch the key components:** an ingestion/validation layer (checking submitted molecular structures are well-formed), a feature computation layer, a model-serving layer, and a results-delivery mechanism (synchronous API response for small/fast requests, or an asynchronous job-based flow with notification for larger/slower batch requests).
4. **Discuss versioning and provenance** — ensuring users can see which model version produced a given result, given the importance of this in a scientific context (see the MLOps section).
5. **Discuss how this integrates into chemists' existing tools/workflow** rather than assuming a brand new, standalone interface is automatically the right choice (see the Cross-Functional Collaboration section).
6. **Address monitoring and feedback** — how will you know if predictions are trusted/used, and how will corrections/experimental follow-up results feed back into future model improvement.

**Follow-ups:**
- How would your design change if you learned chemists need results within 5 seconds for interactive design work, versus if this were purely a nightly batch process?

---

### Q: Tell me about a time you had to push back on a proposed engineering timeline or approach because you believed it didn't adequately account for the validation rigor a biotech use case required. 🔴

**What a strong answer includes:**
- A specific, concrete example connecting to real validation concepts (e.g., inadequate held-out evaluation, insufficient consideration of applicability domain, a missing prospective validation step) rather than a vague, generic caution.
- Evidence of communicating this concern in terms of concrete risk/consequence (what could go wrong, and how likely/severe) rather than an abstract, hard-to-act-on objection.
- A clear articulation of how the disagreement was resolved, including whether the added rigor was ultimately deemed worth the timeline cost, and what happened as a result.

**Follow-ups:**
- How do you calibrate how much validation rigor is "enough" for a given project, balancing genuine risk against realistic delivery timelines?

---

### Q: Describe a situation where you had to make a build-vs-buy (or build-vs-adopt-existing-open-source-tool) decision for a piece of biotech ML infrastructure. How did you approach it? 🟡

**What a strong answer includes:**
- A structured decision framework considering: how well an existing tool/vendor solution actually fits the specific need versus requiring significant customization; the total cost of ownership (including integration, maintenance, and any licensing costs) versus the engineering cost/time of building a custom solution; and how core/differentiating this capability is to the organization's competitive advantage (similar in spirit to the build-vs-buy reasoning discussed in the AI PM companion repo, applied to an infrastructure/engineering decision).
- A concrete example with a clear articulated outcome and, ideally, honest reflection on whether the decision looks right in hindsight.
- Awareness of biotech-specific considerations that might tip this decision (e.g., data sensitivity/compliance requirements that make a third-party vendor solution less attractive, or a genuinely commoditized capability where building custom clearly isn't worth the engineering investment).

**Follow-ups:**
- What would make you revisit a build-vs-buy decision after it's already been made and implemented?

---

### Q: Tell me about a time you had to balance moving fast (e.g., to support an urgent scientific deadline) against maintaining engineering rigor and code/pipeline quality. 🟡

**What a strong answer includes:**
- A specific, honest account of what corners (if any) were deliberately cut, and a clear rationale for why that tradeoff was acceptable given the specific stakes and timeline (versus being reckless).
- Evidence of communicating the tradeoff transparently to stakeholders — e.g., explicitly flagging any accepted technical debt or reduced validation rigor, rather than silently cutting corners without disclosure.
- A concrete plan (ideally followed through on) for addressing any technical debt incurred once the urgent deadline pressure passed, rather than letting shortcuts silently become permanent.

**Follow-ups:**
- How do you personally decide where the line is between an acceptable, temporary shortcut under time pressure and a change that's too risky to make even under deadline pressure?
