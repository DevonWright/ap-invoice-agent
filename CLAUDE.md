# AP Invoice Agent

An accounts payable agent that handles three-way match exceptions (invoice vs purchase order vs delivery receipt). This is a **learning project**: the goal is hands-on experience with MCP, LangGraph, and Langfuse. Understanding matters more than speed.

## Stack
- Python 3.12, managed with **uv** (never use pip or manually create venvs)
- Postgres in Docker (the "ERP": vendors, purchase orders, receipts, payments)
- MCP: custom ERP server (Python MCP SDK) + off-the-shelf filesystem MCP server for the invoice inbox
- LangGraph for orchestration, Postgres checkpointer for durable state
- Langfuse for tracing (cloud free tier first, self-hosted later)

## Commands
- Add a dependency: `uv add <package>`
- Run a script: `uv run <file>`
- Start the database: `docker compose up -d`
- Stop the database: `docker compose down`

## How to work with me (important)
**You build:** Docker and compose files, database setup, the seed data generator, invoice PDF generation, test scaffolding, config, and other plumbing.

**I build, you tutor:** MCP tool definitions and the LangGraph graph (state, nodes, edges, routing, interrupts, checkpointer). For these:
- Explain the concept and point me to the relevant docs
- Review my code and suggest improvements, but do not write the implementation unless I explicitly ask
- If I'm stuck, give a hint before giving the answer

Other rules:
- Use plan mode for anything touching more than a couple of files, and let me review the plan first
- I use voice dictation; read past transcription errors (e.g. "MPC" means MCP)
- Keep changes small and explain what you changed and why
## Session workflow
- At the start of a session, read `docs/PLAN.md` to see where we are
- At the end of a session, update `docs/PLAN.md` (check off finished items) and add any new decisions to `docs/DECISIONS.md`, then commit with a clear message

## Project docs
- `docs/PLAN.md`: phases and checklist
- `docs/DECISIONS.md`: log of design decisions and the reasoning behind them
