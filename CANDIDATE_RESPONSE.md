# Candidate Response

> **Instructions:** Complete all sections below. Keep your total response to ~500 words (one page max).
>
> **Time limit:** 20 minutes recommended, 30 minutes hard stop.

---

## Your Name

Ignas Ausiejus

---

## Section A: Top 5 Risks (Ranked)

*Identify the five most critical risks in the proposed design. Rank them from highest to lowest priority. For each risk, provide a brief description (1-2 sentences).*

1. No role-based access control
   A SupportAgent can run query 2 or 3 and the PII data would be returned to him. Role is in the JWT but it's not checked.

2. Prompt injection
   Raw user message goes straight in and RAG docs are placed above the system prompt. Prompt can modify instructions of the system itself

3. Fallback response by guessing while estimating industry benchmarks
   There is a high chance users would get false/made-up data.

4. Logs include sensitive data, no audit log
   Full prompts and complete result sets are stored for 90 days even the limit is 30 days. Also no audit trail of who accessed which data (GDPR requirement).

5. Vulnerable admin rerun endpoint
   Knowing the ID of the query is enough to get someones else results

---

## Section B: Mitigations

*For each risk identified above, propose one concrete, implementable mitigation. Be specific.*

1. **Mitigation for Risk 1:**
   Pass the role from the JWT and run queries with a matching read-only Snowflake role that can see only approved views

2. **Mitigation for Risk 2:**
  Provide system prompt first and then RAG + user input wrapped as untrusted data

3. **Mitigation for Risk 3:**
   If there is no data, point to the closest source and say so. Remove any guestimated fallback.

4. **Mitigation for Risk 4:**
   30 days log retention, log metadata (user, role, latency, SQL, result row count)

5. **Mitigation for Risk 5:**
   Rerun should be possible only after SSO + admin role on the endpoint.

---

## Section C: First Architecture Change

*If you could implement only ONE change to improve this design, what would it be and why?*

**Change:** Implement a permission-aware data access layer between LLM and DBs so that only the views allowed for the user are taken into account.

**Rationale:** Avoids the chance of prompting around the permissions. Smaller schema.

---

## Section D: Clarifying Questions

*What two questions would you ask stakeholders before implementing or further reviewing this design?*

1. Is 3s p95 required for every question or only simplier lookups? 
2. What should happen if OpenAI is down?

---

## Section E: Success Metrics (Optional)

*How would you measure whether the chatbot is working correctly and safely? List 1-3 metrics.*

- Response accuracy
- No unauthorized queries and PII leaks
- Audit log coverage

---

*End of response.*
