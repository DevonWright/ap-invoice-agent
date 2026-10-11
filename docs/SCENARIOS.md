# Scenarios

Each scenario is a situation the agent must handle, with the correct outcome. These drive the tool list, the schema, and the seed data, and they become the labeled eval set later.

Format: **Setup** (what's true in the ERP), **Invoice** (what arrives), **Expected** (what the agent should do and must not do).

## Rules these scenarios rely on

**Agent approval threshold**
- The agent may pay invoices with a total **under $1,000**.
- **$1,000.00 and above** requires human approval: the agent creates an approval request (status `pending`). The payment is created only after approval.
- If an invoice has an exception, no approval request is created, regardless of amount. The exception must be resolved first.

**Price tolerance** (only overbilling counts; underbilling is never an exception)
- **Line level:** invoice unit price must exceed the PO unit price by **less than 2%**.
- **Header level:** invoice total must exceed the expected total by **less than the smaller of 2% of the expected total or $50**.
- Header compares actual totals, so underbilled lines **offset** overbilled lines.
- All boundaries are **strict**: exactly 2% fails, exactly at the header limit fails.
- Within tolerance, pay the **invoice** amount.
- Any failure is a **price mismatch** exception.

**A consequence of these rules:** if every line is under 2% over, the total is too. So the header check can only catch what the lines missed through the **$50 cap**, which only applies when the expected total is above $2,500. Header failures (S5) and netting that changes the outcome (S6) can therefore only happen above the agent threshold.

**Exceptions** contain structured fields (type, invoice, PO, failing line(s), PO price, invoice price, variance, limit) plus a plain-language summary so the rep knows immediately what to look at.

---

## S1. Clean match, under agent threshold
**Setup:** PO-1001 from Acme Industrial (active): 100 widgets at $5.00. Receipt: 100 received.
**Invoice:** Acme Industrial, PO-1001, 100 widgets at $5.00. Total $500.00.
**Expected:**
- Match passes; $500.00 is under $1,000.
- Create payment for $500.00 (status `scheduled`).
- Must not flag an exception. Must not create an approval request.

## S2. Clean match, over agent threshold
**Setup:** PO-1002 from Acme Industrial (active): 1,000 widgets at $10.00. Receipt: 1,000 received.
**Invoice:** Acme Industrial, PO-1002, 1,000 widgets at $10.00. Total $10,000.00.
**Expected:**
- Match passes; $10,000.00 is at or above $1,000.
- Create approval request for $10,000.00 (status `pending`).
- On approval: create payment for $10,000.00 (status `scheduled`).
- Must not flag an exception. Must not create a payment before approval.

## S3. Price variance within tolerance
**Setup:** PO-1003 from Harbor Fasteners (active): 50 bolts at $10.00 ($500.00). Receipt: 50 received.
**Invoice:** Harbor Fasteners, PO-1003, 50 bolts at $10.10. Total $505.00.
**Checks:**
- Line: $10.10 vs $10.00 is 1.0% over, under 2%. Pass.
- Header: $5.00 over; limit is min(2% of $500 = $10, $50) = $10. Pass.
**Expected:**
- Treated as a clean match. Pay the invoice amount.
- Create payment for $505.00 (status `scheduled`); under threshold, no approval needed.
- Must not flag an exception.

## S4. One line over 2% (line-level price mismatch)
**Setup:** PO-1004 from Delta Supply Co (active):
- Line 1: 100 gaskets at $3.00 ($300.00)
- Line 2: 25 valves at $20.00 ($500.00)
Expected total $800.00. Receipt: all quantities received.
**Invoice:** Delta Supply Co, PO-1004:
- Line 1: 100 gaskets at $3.10 ($310.00)
- Line 2: 25 valves at $20.00 ($500.00)
Total $810.00.
**Checks:**
- Line 1: $3.10 vs $3.00 is 3.33% over. **Fail.**
- Line 2: no variance. Pass.
- Header: $10.00 over; limit min(2% of $800 = $16, $50) = $16. Pass.
**Expected:**
- Flag price mismatch exception: line 1, PO price $3.00, invoice price $3.10, 3.33% over (limit 2%), $10.00 impact. Summary explains this in plain language.
- Must not create a payment. Must not create an approval request.

## S5. All lines within 2%, header over limit (header-level price mismatch)
Necessarily above the agent threshold (see "A consequence of these rules").
**Setup:** PO-1005 from Pinecrest Equipment (active):
- Line 1: 100 pumps at $20.00 ($2,000.00)
- Line 2: 100 filters at $10.00 ($1,000.00)
- Line 3: 50 hoses at $20.00 ($1,000.00)
Expected total $4,000.00. Receipt: all quantities received.
**Invoice:** Pinecrest Equipment, PO-1005:
- Line 1: 100 pumps at $20.30 ($2,030.00)
- Line 2: 100 filters at $10.15 ($1,015.00)
- Line 3: 50 hoses at $20.30 ($1,015.00)
Total $4,060.00.
**Checks:**
- Each line 1.5% over. Pass.
- Header: $60.00 over; limit min(2% of $4,000 = $80, $50) = $50. **Fail.**
**Expected:**
- Flag price mismatch exception at header level: expected $4,000.00, invoiced $4,060.00, $60.00 over (limit $50.00). Summary notes every line is individually within tolerance.
- Must not create a payment. Must not create an approval request (the exception takes precedence over the threshold).

## S6. Over and under lines net out (clean only because of netting)
Necessarily above the agent threshold (see "A consequence of these rules"), so the clean path includes approval.
**Setup:** PO-1006 from Acme Industrial (active):
- Line 1: 200 sensors at $20.00 ($4,000.00)
- Line 2: 100 cables at $10.00 ($1,000.00)
Expected total $5,000.00. Receipt: all quantities received.
**Invoice:** Acme Industrial, PO-1006:
- Line 1: 200 sensors at $20.30 ($4,060.00)
- Line 2: 100 cables at $9.70 ($970.00)
Total $5,030.00.
**Checks:**
- Line 1: 1.5% over ($60.00). Pass. Line 2: under ($30.00). Ignored.
- Header: $30.00 over (net); limit min(2% of $5,000 = $100, $50) = $50. Pass.
- Without netting, line 1's $60.00 overage alone would fail the $50 header limit. This scenario proves the header nets.
**Expected:**
- Treated as a clean match; $5,030.00 is at or above $1,000.
- Create approval request for $5,030.00 (status `pending`).
- On approval: create payment for $5,030.00 (status `scheduled`).
- Must not flag an exception. Must not create a payment before approval.

---

## To write next
- Over threshold, approver **rejects** (S2 setup)
- Threshold boundary: invoice at $999.99 (agent pays) and $1,000.00 (approval request)
- Line boundary: exactly 2.00% over (fails)
- Header boundary: exactly at the limit (fails)
- Partial delivery
- Duplicate invoice
- Unknown or inactive vendor
- Invoice references a nonexistent PO
- Multiple problems on one invoice (e.g. two different exception types)
