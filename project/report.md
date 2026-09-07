# Beaver's Choice Paper Company — Multi-Agent System Report

## 1. System Architecture

The system is a 4-agent pipeline built with `smolagents` `ToolCallingAgent`, running on
`gpt-4o-mini` via the OpenAI-compatible Vocareum proxy already configured in `.env`
(`OPENAI_API_KEY` / `OPENAI_BASE_URL`). Full diagrams and a helper-function-to-agent
table are in [`workflow_diagram.md`](workflow_diagram.md) (rendered to
`workflow_diagram.png` and `sequence_diagram.png` via Mermaid).

**Agents (4, within the 5-agent maximum):**

- **Orchestrator Agent** — entry point. Extracts the request date and any customer
  delivery deadline, maps every requested item to an exact catalog name (via
  `match_catalog_item`), delegates to the three workers below, and returns one
  customer-facing message with a clear rationale.
- **Inventory Agent** — stock questions and restocking. Wraps `get_all_inventory`,
  `get_stock_level`, `get_supplier_delivery_date`, `create_transaction`
  (`stock_orders`), and `get_cash_balance` (afford-to-reorder check).
- **Quoting Agent** — pricing. Wraps `search_quote_history` (to price consistently
  with comparable past orders) and `get_stock_level`, and applies bulk discounts for
  large quantities.
- **Sales / Fulfillment Agent** — finalizes or rejects. Wraps `get_stock_level`,
  `get_supplier_delivery_date`, `create_transaction` (`sales`), `get_cash_balance`,
  and `generate_financial_report` (a pre-fulfillment health check gated on large
  orders).

The Orchestrator delegates via three thin `@tool`-wrapped closures
(`ask_inventory_agent`, `ask_quoting_agent`, `ask_sales_agent`), each calling the
corresponding sub-agent's `.run()` — the pattern used throughout this course's own
`lesson-6` multi-agent material, rather than `smolagents`' `ManagedAgent`.

This covers all 7 required helper functions and cleanly maps onto the three required
capabilities (inventory management, quote generation, sales processing) plus
orchestration. `get_all_inventory`, `get_cash_balance`, and `generate_financial_report`
are all bound to the Inventory and/or Sales agents, per the project instructions, even
though they are used for internal checks rather than being customer-facing.

## 2. Design Decisions

**Deterministic catalog matching, not LLM guessing.** Early testing showed the model
would paraphrase item names instead of copying them exactly (e.g. turning "Glossy
paper" into "high-quality glossy paper" between agent hops, or matching "A4 glossy
paper" to a combined name that doesn't exist in the catalog). Since `create_transaction`
and `get_stock_level` require exact string matches against the `inventory` table, this
silently broke stock checks and, in one case, let the model invent both an item name
*and* a unit price for a restock order. I addressed this in two layers:

1. **`match_catalog_item`** — a small, non-LLM fuzzy matcher (substring check + a
   stopword-filtered Jaccard similarity over word sets) shared by the Orchestrator,
   Inventory Agent, Quoting Agent, and Sales Agent, used to resolve free text like
   "heavy cardstock (white)" to the exact catalog name `Cardstock`.
2. **Hard validation in every DB-touching tool** — `check_item_stock`,
   `check_stock_for_order`, `finalize_sale`, and `reorder_stock` all reject any
   `item_name` that isn't an exact catalog name (looked up against the same
   `CATALOG_PRICES` dict backing `match_catalog_item`) rather than silently
   proceeding. `reorder_stock` also no longer accepts a model-supplied `unit_price` —
   it looks the price up from the catalog itself. This turned a silent data-integrity
   bug into a caught error the agent can recover from by calling `match_catalog_item`.

**Explicit, repeated date-passing.** The model would occasionally default to a
training-cutoff date (e.g. `2023-10-06`) instead of the actual request date once that
date was a few delegation hops away from the original customer message. Every agent's
system prompt now states a strict contract: the query it receives will contain a
literal `"Request date: YYYY-MM-DD"` (and, where relevant, `"Needed by: YYYY-MM-DD"`)
line, and it must use exactly that date for every `as_of_date` / `order_date` tool
argument rather than inventing one.

**Framework and model.** `smolagents` was chosen for consistency with this course's
own demos, its lightweight `@tool` decorator, and its built-in `ToolCallingAgent`
(more reliable structured tool calls than free-form code generation for this kind of
deterministic business logic). The existing `OPENAI_API_KEY` / `OPENAI_BASE_URL`
Vocareum proxy was kept rather than switching to a different model provider, and
`gpt-4o-mini` was used for cost/latency given the 20-request evaluation loop.

**A deterministic sanitization layer as a last line of defense.** Despite every
agent being told never to reveal cash balances or profit margins, a review of an
earlier `test_results.csv` found several responses stating things like "the company
has a cash balance of $0.00" or "insufficient cash balance" - the prompt instruction
alone was not reliably followed. `sanitize_customer_response()` runs on every
Orchestrator reply before it's returned: a small table of regex patterns rewrites
cash-balance and "funds"-based phrasing (both were observed leaking) into neutral,
customer-appropriate language ("limited restocking capacity") without touching
legitimate figures like quoted order totals. This is deliberately a post-processing
guard rather than another prompt instruction - prompt-only fixes proved unreliable
for this exact failure mode, so removing dependence on the model getting it right
every time was the more robust fix.

**A hard stock check inside `finalize_sale`, not just in the Sales Agent's prompt.**
Auditing a run turned up a sale that pushed `Cardstock` to -490 units: the Sales
Agent's instructions say to call `check_stock_for_order` before `finalize_sale`, but
nothing enforced that it actually did. `finalize_sale` now re-checks
`get_stock_level` itself and refuses to record a sale that would oversell -
overselling is now structurally impossible regardless of whether the model follows
the prescribed step order. Note: an earlier attempt to fix a much smaller date-
labeling issue in `reorder_stock` by adding more prompt text to the Inventory Agent
backfired badly - it dropped that agent's `reorder_stock` calls from 12 in a run to
0 in the next two. That edit was reverted. The lesson generalizes: a tool-level hard
check (like this one, and like the catalog-name validation) is safe to add, because
it only ever fires when something has already gone wrong; an added prompt sentence
can measurably shift a model's overall willingness to act, in ways that are hard to
predict in advance and only showed up by comparing tool-call counts across runs.

## 3. Evaluation Results

Running `python project_starter.py` processes all 20 rows of
`quote_requests_sample.csv` in request-date order and writes `test_results.csv`.
Summary:

| Metric | Result |
| --- | --- |
| Requests processed | 20 / 20 |
| Requests that changed the cash balance | 9 (rubric requires ≥ 3) |
| Requests fulfilled (fully or partially) | 5 (rubric requires ≥ 3) |
| Requests rejected/unfulfilled with a stated reason | 15 (rubric requires ≥ 1) |
| Cash balance | $45,059.70 → $45,331.25 |
| Inventory value | $4,940.30 → $4,631.25 |

**Strengths.** Every rejection includes a concrete, customer-appropriate reason
(insufficient stock of a named item, an item not carried, or limited restocking
capacity). Every response was checked - by a regex sweep for cash-balance/funds/
profit-margin/stack-trace patterns and by a full manual read of all 20 rows - and
none leak internal financial figures; this is enforced by `sanitize_customer_response`
rather than by prompt instructions alone (see Section 2). Cross-checking
`test_results.csv` against the raw `transactions` table confirms every fulfilled
sale used an exact catalog item name at the correct catalog unit price, was backed
by real recorded stock, and never drove any item's stock negative (enforced by the
hard stock check now in `finalize_sale`) - zero invalid item names, zero bogus $0
transactions, zero overselling across the full run.

**Weaknesses.** The fulfillment rate (5/20) is limited mainly by a structural fact,
not a system defect: `generate_sample_inventory()` stocks only 40% of the 46-item
catalog (18 items actually carry stock), so many incoming requests reference items
the company simply never had in stock this run - the system correctly identifies and
rejects these rather than fabricating availability. Separately, the underlying LLM
occasionally states something incorrect in its free-text narration even when the
actual transaction (or lack of one) is correct: one response cited "October 6, 2023"
as a restocking date, and another quoted a delivery date earlier than the request
date. Both are narration errors, not data-integrity problems - the recorded
transactions and dates in the database are correct - but they show the same
underlying limitation of composing the customer-facing message from free-form model
prose (see improvement #2 below).

## 4. Requirements Compliance Checklist

| Requirement | Status | Where |
| --- | --- | --- |
| Multi-agent system, maximum 5 agents | ✅ 4 agents | Section 1 |
| Text-based inputs/outputs only | ✅ every agent, tool, and the public entry point (`call_your_multi_agent_system`) is `str` in / `str` out | `project_starter.py` |
| Handles customer inquiries, checks inventory, provides quotations, completes transactions | ✅ Orchestrator + Inventory + Quoting + Sales agents | Section 1 |
| Workflow diagram (image file) | ✅ `workflow_diagram.png` + bonus `sequence_diagram.png`, Mermaid source in `workflow_diagram.md` | deliverable |
| Diagram shows agents, non-overlapping responsibilities, orchestration/data flow | ✅ architecture diagram | `workflow_diagram.png` |
| Diagram shows tools per agent, their purpose, and the helper function(s) wrapped | ✅ tool list per agent box + helper-function table | `workflow_diagram.md` |
| Diagram shows agent↔tool data input/output interactions | ✅ per-call arrows with request/response payloads | `sequence_diagram.png` |
| Source code = exactly one Python file | ✅ `project_starter.py` only | deliverable |
| Orchestrator + distinct worker agents (inventory / quoting / sales) | ✅ | Section 1 |
| Uses one of `smolagents` / `pydantic-ai` / `npcsh` | ✅ `smolagents` | Section 2 |
| Tools follow framework conventions | ✅ `@tool`-decorated functions, typed args, docstrings | `project_starter.py` |
| All 7 helper functions used in a tool definition (`create_transaction`, `get_all_inventory`, `get_stock_level`, `get_supplier_delivery_date`, `get_cash_balance`, `generate_financial_report`, `search_quote_history`) | ✅ verified by direct grep — each appears in ≥1 tool body | `project_starter.py` |
| `get_all_inventory` / `get_cash_balance` / `generate_financial_report` assigned to an agent even if only for internal checks | ✅ Inventory Agent (`get_all_inventory`, `get_cash_balance`); Sales Agent (`get_cash_balance`, `generate_financial_report` as a large-order health check) | Section 1 |
| Evaluated on full `quote_requests_sample.csv`, results in `test_results.csv` | ✅ 20/20 processed | Section 3 |
| ≥3 requests change cash balance | ✅ 9 | Section 3 |
| ≥3 quote requests fulfilled | ✅ 5 | Section 3 |
| Not all requests fulfilled, with reasons | ✅ 15 rejected, each with a stated reason | Section 3 |
| Reflection: diagram/roles/decisions, evaluation discussion, ≥2 improvement suggestions | ✅ | Sections 1–2, 3, 5 |
| Transparent, explainable, non-leaking customer-facing outputs | ✅ every response includes a reason; a regex sweep plus a full manual read of all 20 rows found zero cash-balance/funds/profit-margin/stack-trace leaks, enforced by `sanitize_customer_response` (Section 2) | Section 3 |
| No overselling / negative stock | ✅ enforced by a hard stock check in `finalize_sale`; verified zero negative-stock items in the final database | Section 2, 3 |
| Readable, well-commented, modular code | ✅ descriptive `snake_case` names, Google-style docstrings on every tool, logic split into catalog-matching / per-agent tool groups / agent construction / orchestrator | `project_starter.py` |

## 5. Suggestions for Further Improvement

1. **Close the loop between rejection and restocking.** Today, when the Sales Agent
   finds insufficient stock it hard-rejects the order — the Inventory Agent's
   `reorder_stock` is never invoked as part of that path. A natural extension is for
   the Orchestrator to detect an `INSUFFICIENT` result, ask the Inventory Agent to
   restock (cash permitting) and check the resulting supplier delivery date against
   the customer's deadline, and only reject if restocking genuinely can't meet it.
   This would likely raise the fulfillment rate substantially given the 40% inventory
   coverage.
2. **Make the final customer message structurally consistent with the actual outcome.**
   The narration errors noted above (a fabricated restocking date, a delivery date
   predating the order) happen because the Orchestrator free-generates its summary
   from the sub-agents' natural-language replies instead of from the tools' actual
   return values. Using `output_config.format` (structured outputs) to force the
   Sales Agent to return a typed `{status: "fulfilled" | "rejected", items: [...],
   delivery_date: str, reason: str | null}` object, and having the Orchestrator's
   final message assembled from that structure rather than re-narrated, would
   eliminate this class of error entirely.
3. **A negotiation-capable customer agent** (from the project's "stand out" suggestions)
   — a fifth agent representing the customer that can respond to a partial rejection
   (e.g. "only 2 of 3 items are available") by trying a reduced order. Several sample
   requests ask for products we simply don't carry at all (e.g. "balloons", "event
   tickets", "table napkins", "poster boards") alongside items we do — today the whole
   order is rejected in one message; a negotiation loop could still close the sale on
   the items that *are* in our catalog.
