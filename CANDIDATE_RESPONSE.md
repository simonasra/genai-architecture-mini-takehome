# Candidate Response

> **Instructions:** Complete all sections below. Keep your total response to ~500 words (one page max).
>
> **Time limit:** 20 minutes recommended, 30 minutes hard stop.

---

## Your Name

Osvaldas Ulevičius

---

## Section A: Top 5 Risks (Ranked)

*Identify the five most critical risks in the proposed design. Rank them from highest to lowest priority. For each risk, provide a brief description (1-2 sentences).*

1. **No authorization / query-scoping**
   Nothing enforces the role -> data matrix; the role is in the session token but never used, so users can request data their role forbids.

2. **LLM generates free-form SQL**
   The SQL Generator gets the full schema and executes model-written queries directly against the warehouse, opening the door to unauthorized data access, injection, and runaway cost/latency.

3. **Full interaction logging**
   Prompts, RAG context, SQL, and complete result sets (including PII) are persisted to a broadly-readable central log store, creating a durable copy of sensitive data that bypasses the role model.

4. **Prompt injection**
   Retrieved RAG chunks and user text are placed above the system prompt, so poisoned documentation or crafted input can override instructions and coerce the model into unauthorized queries or actions.

5. **Designed-in fabrication fallback**
   On missing data the system invents benchmark-based numbers presented as real answers, which is a trust/correctness failure in a decision-support tool.

---

## Section B: Mitigations

*For each risk identified above, propose one concrete, implementable mitigation. Be specific.*

1. **Mitigation for Risk 1:**
   Add deterministic server-side middleware in the orchestrator that uses the session role to gate which tools the LLM may call and what data is returned (including PII column filtering); deny disallowed requests rather than letting the LLM decide.

2. **Mitigation for Risk 2:**
   Replace LLM-authored SQL with a fixed catalog of predefined, parameterized query tools (MCP-style); validate and bound parameters, and refuse out-of-catalog questions instead of falling back to raw SQL.

3. **Mitigation for Risk 3:**
   Log trace/audit metadata (user, role, tool, query ID, row counts, errors) only—enough for debugging and GDPR audit needs—but do not persist raw result sets, PII, or full prompt/RAG content; restrict access to logs.

4. **Mitigation for Risk 4:**
   Put the system prompt first and clearly delimit/label retrieved and user content as untrusted data; treat authorization as deterministic middleware (per Risk 1) so the model can never grant access regardless of prompt content.

5. **Mitigation for Risk 5:**
   Remove the benchmark-estimation behavior; require every reported number to be grounded in a real query/tool result, otherwise return an explicit "no data available" response and clearly label any estimate as such.

---

## Section C: First Architecture Change

*If you could implement only ONE change to improve this design, what would it be and why?*

**Change:**  Replace LLM-authored free-form SQL with a fixed catalog of predefined, parameterized query tools.

**Rationale:** This eliminates the largest attack surface—an LLM emitting arbitrary SQL against the warehouse (unauthorized data, injection, and runaway cost/latency)—and is shippable without re-architecting identity or sessions. It also creates a single controlled choke point trough which role-based authorization, audit logging, and PII filtering can then be enforced, so it both protects immediately and unblocks the other fixes. Role-based gating on those tools is the natural next step.

---

## Section D: Clarifying Questions

*What two questions would you ask stakeholders before implementing or further reviewing this design?*

1. The role matrix is defined in business terms ("aggregate metrics", "full analytics but no PII"), but the design exposes concrete tables/columns. How should each role map to specific tables/columns?

2. When a request is unauthorized or the data can't be found, what is the desired behavior: refuse, explain why, or escalate to someone who is permitted to see it?

---

## Section E: Success Metrics (Optional)

*How would you measure whether the chatbot is working correctly and safely? List 1-3 metrics.*

- **Accuracy** — How accurate the answers are (% of answers where the reported number matches ground truth).
- **Token usage (input/output)** — As a proxy for cost per conversation against the $0.50 budget.
- **Security incidents** — Count of cases where the wrong person got data they shouldn't, target is zero (maps to risks 1-3).

---

*End of response.*
