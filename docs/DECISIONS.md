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
