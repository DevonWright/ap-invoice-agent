# Plan

## Phase 0: Setup
- [x] Create project with `uv init` and push to a GitHub repo
- [x] Add CLAUDE.md, docs/PLAN.md, docs/DECISIONS.md
- [x] docker-compose.yml with Postgres; confirm it starts and accepts connections
- [x] `.env.example` for database and API credentials (real `.env` gitignored)

## Phase 1: Data and MCP server
- [ ] Design the ERP schema: vendors, purchase orders, PO line items, receipts, invoices, payments, exceptions
- [ ] Seed script that generates realistic fake data
- [ ] Deliberate exceptions mixed in: price mismatch, partial delivery, duplicate invoice, unknown vendor, over approval threshold
- [ ] Generate invoice PDFs into an `inbox/` folder
- [ ] Record the expected outcome for each invoice (future eval set)
- [ ] MCP server tools (I write these): `get_purchase_order`, `get_receipts`, `check_duplicate`, `flag_exception`, `create_payment`
- [ ] Test every tool in the MCP Inspector
- [ ] Connect the server to Claude Code and try it conversationally

## Phase 2: Simple agent
- [ ] Add LangGraph and Langfuse
- [ ] Load tools from both MCP servers (ERP + filesystem) via `langchain-mcp-adapters`
- [ ] Basic tool-calling agent attempts to process the inbox
- [ ] Langfuse tracing on from the first run
- [ ] Write down the failure modes I observe (this motivates Phase 3)

## Phase 3: Real workflow (the core)
- [ ] Define graph state
- [ ] Intake node: pick up an invoice
- [ ] Extraction node: structured output into a Pydantic model
- [ ] Match node: three-way match against PO and receipts
- [ ] Conditional routing by outcome: clean, price mismatch, partial delivery, duplicate, unknown vendor, over threshold
- [ ] Human approval interrupt before `create_payment`
- [ ] Postgres checkpointer; stop the app mid-approval and resume it
- [ ] Draft vendor query for mismatches; escalation path for over-threshold

## Phase 4: Hardening
- [ ] Write tools only reachable after approval
- [ ] Handle unreadable PDFs and missing records gracefully
- [ ] Move the MCP server to HTTP transport with auth
- [ ] Move Langfuse to self-hosted in Docker
- [ ] README with architecture diagram and demo vs production notes

## Next build
- Evals, using the labeled invoices from Phase 1
