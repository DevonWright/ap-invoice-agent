# Decisions

A running log of design choices and why. Newest at the bottom. Format: what was decided, why, and what was considered instead.

---

### Use case: accounts payable three-way match
**Why:** One of the most common real enterprise agent deployments. Runs entirely on infrastructure I own, so nothing expires. Each technology does real work.
**Considered:** A Salesforce-based agent (scratch orgs expire, hard to keep running); an incident response agent (would need simulated logs and metrics).

### Python 3.12 with uv
**Why:** uv handles Python versions, virtual environments, and dependencies in one tool and is the modern default. 3.12 is well supported by every library in the stack.
**Considered:** pip + venv (slower, more manual); newest Python version (more risk of library incompatibility).

### Langfuse over LangSmith for observability
**Why:** Open source and self-hostable, so it can run alongside the rest of the stack. Framework-agnostic. Matches data residency needs common in enterprise clients. I already have LangSmith experience from my RAG build, so this adds a second tool.
**Considered:** LangSmith (lowest friction with LangGraph, but closed and SaaS-first).
**Plan:** Start on Langfuse cloud free tier, move to self-hosted in Phase 4.

### Claude Code as the coding agent, no Copilot for now
**Why:** Agent workflow suits multi-file plumbing and can act as an MCP client to test my own server. Skipping inline autocomplete so I actually write the MCP tools and graph myself.

### Postgres 17 in Docker Compose for the ERP
**Why:** Pinning the major version keeps the setup reproducible. A named volume keeps data across `docker compose down`. The port is bound to 127.0.0.1 only, so the database isn't exposed on the network. A `pg_isready` healthcheck gives a clear "ready" signal, which later scripts can wait on. Credentials live in `.env` (gitignored), and `.env.example` is the committed template.
**Considered:** Installing Postgres natively on Windows (harder to reset and less portable); a bind-mounted data folder (Windows file permission issues).

### Design order: scenarios first, schema later
**Why:** The schema only exists to serve the scenarios. Defining scenarios and expected outcomes first, then the questions and actions each needs, then tools, then schema, avoids guessing. The scenarios also become the eval set.
**Considered:** Schema first, as originally planned (risks designing tables that miss what a scenario needs, such as tolerance or line-level receipts).

### Agent approval threshold: under $1,000 (originally $5,000)
**Why:** Treat the agent like an AP clerk with delegated authority. Clean invoices under $1,000 are paid without a human, which reflects real "touchless" AP processing. $1,000.00 and above requires human approval. A single threshold is enough to learn the interrupt pattern. Lowered from $5,000 to $1,000 shortly after the first decision; scenarios were rescaled so each still tests the same rule.
**Considered:** Human approval before every payment (safer, but unrealistic for small clean invoices); a tiered approval matrix (more realistic, more scope; moved to future features).

### Approval requests are records; payment is created only after approval
**Why:** Mirrors real ERP workflow items. The record is the business side (visible, auditable, shows who approved what and when); the LangGraph interrupt and checkpointer are the agent side. The approval request links to the graph thread so approving it resumes the right run. Creating the payment only after approval keeps the write tool unreachable until a human says yes.
**Considered:** Creating the payment first in a `pending_approval` status (simpler data model, but the payment exists before anyone approves it).

### Exceptions vs approval requests are separate concepts
**Why:** An exception means something is wrong and needs fixing (an error). An approval request means nothing is wrong but someone with authority must sign off (a control). They can both apply to one invoice. Keeping them separate keeps routing and reporting clear.

### Price tolerance rules
**Decided:**
- Line level: invoice unit price must exceed PO unit price by less than 2%.
- Header level: invoice total must exceed expected total by less than the smaller of 2% or $50.
- Only overbilling counts; underbilling is never an exception.
- Header compares actual totals, so underbilled lines offset overbilled lines.
- Boundaries are strict: exactly at a limit fails.
- Within tolerance, pay the invoice amount.
- Any failure is a price mismatch exception.
**Why:** Two-level tolerance matches how major ERPs work: the line check catches a single bad price, the header check catches many small variances that add up. The $50 cap limits dollar exposure on larger invoices; 2% governs smaller ones. Netting at the header reflects what is actually paid. A single global tolerance is enough for now.
**Considered:** Applying the $50 cap at line level (rejected; wanted it on the total only); summing only overages at the header (stricter, but netting was judged acceptable); per-vendor tolerances (future feature).

### Exceptions carry structured fields plus a summary
**Why:** The plain-language summary tells the rep immediately what to look at. Structured fields (type, invoice, PO, failing lines, prices, variance, limit) support filtering, reporting, and exact assertions in evals, since LLM-written summaries vary run to run.

### Exceptions take precedence over the approval threshold
**Why:** No one should be asked to approve an invoice that is known to be wrong. If an invoice has an exception, the agent flags it and does not create an approval request, whatever the amount. Approval only happens once the invoice matches.

### Vendor query drafting deferred
**Why:** Keeps scope focused on the core learning goals. Moved to future features; a natural "what would you add next" item.
