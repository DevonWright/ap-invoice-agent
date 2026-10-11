# Plan

## Phase 0: Setup
- [x] Create project with `uv init` and push to a GitHub repo
- [x] Add CLAUDE.md, docs/PLAN.md, docs/DECISIONS.md
- [x] docker-compose.yml with Postgres; confirm it starts and accepts connections
- [x] `.env.example` for database and API credentials (real `.env` gitignored)

## Phase 1: Data and MCP server
Work backward from scenarios: scenarios, then the questions and actions each needs, then tools, then schema, then seed data.
- [ ] Define scenarios and expected outcomes in `docs/SCENARIOS.md` (in progress: clean match and price mismatch done)
  - [x] Clean match under and over agent threshold
  - [x] Price mismatch (line level, header level, within tolerance, netting)
  - [ ] Approval rejected, threshold and tolerance boundaries
  - [ ] Partial delivery
  - [ ] Duplicate invoice
  - [ ] Unknown or inactive vendor
  - [ ] Invoice references a nonexistent PO
  - [ ] Multiple problems on one invoice
- [ ] List the questions and actions each scenario needs
- [ ] Design the ERP schema: vendors, purchase orders, PO line items, receipts, invoices, payments, exceptions, approval requests
- [ ] Seed script that generates realistic fake data
- [ ] Seed data covers every scenario in `docs/SCENARIOS.md`
- [ ] Generate invoice PDFs into an `inbox/` folder
- [ ] Record the expected outcome for each invoice (future eval set)
- [ ] MCP server tools (I write these): `get_purchase_order`, `get_receipts`, `check_duplicate`, `flag_exception`, `request_approval`, `create_payment`
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
- [ ] Match node: three-way match against PO and receipts, applying the price tolerance rules
- [ ] Conditional routing by outcome: clean, price mismatch, partial delivery, duplicate, unknown vendor, over threshold
- [ ] Over threshold ($1,000 and above): create approval request, then human approval interrupt before `create_payment`
- [ ] Link each approval request to its graph thread so approving it resumes the right run
- [ ] Postgres checkpointer; stop the app mid-approval and resume it
- [ ] Exceptions include structured fields plus a plain-language summary

## Phase 4: Hardening
- [ ] `create_payment` only reachable after a passing match, and after approval when at or above threshold
- [ ] Handle unreadable PDFs and missing records gracefully
- [ ] Move the MCP server to HTTP transport with auth
- [ ] Move Langfuse to self-hosted in Docker
- [ ] README with architecture diagram and demo vs production notes

## Future features (out of scope for now)
- Draft vendor query for price mismatches
- Tiered approval matrix (e.g. controller, CFO) instead of a single threshold
- Per-vendor tolerance settings

## Next build
- Evals, using the labeled invoices from Phase 1
