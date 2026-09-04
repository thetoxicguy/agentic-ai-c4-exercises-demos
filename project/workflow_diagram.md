# Beaver's Choice Paper Company — Multi-Agent Workflow

Mermaid source for the two workflow diagrams (rendered to `workflow_diagram.png` and
`sequence_diagram.png` via `mermaid-cli`, which is the image deliverable). Framework:
`smolagents` `ToolCallingAgent` x4, model `gpt-4o-mini` via the OpenAI-compatible
Vocareum proxy.

## 1. Architecture diagram (`workflow_diagram.png`)

Shows all four agents, their tool bindings (and which starter-code helper function each
tool wraps), and the data flow between agents, the customer, and the database.

```mermaid
flowchart TB
    Customer(["Customer<br/>free-text request + date"])

    Orchestrator["<b>Orchestrator Agent</b><br/>parses request -> matches items to<br/>catalog (match_catalog_item) -><br/>delegates -> replies with rationale"]

    Inventory["<b>Inventory Agent</b><br/>stock levels · supplier lead time<br/>restock orders (cash-gated)<br/><br/><i>tools: get_all_inventory, get_stock_level,<br/>get_supplier_delivery_date,<br/>create_transaction (stock_orders),<br/>get_cash_balance, match_catalog_item</i>"]

    Quoting["<b>Quoting Agent</b><br/>maps items to catalog · historical<br/>pricing lookup · bulk discounts<br/><br/><i>tools: match_catalog_item,<br/>search_quote_history, get_stock_level</i>"]

    Sales["<b>Sales / Fulfillment Agent</b><br/>stock + delivery check -><br/>finalize sale or reject<br/><br/><i>tools: match_catalog_item, get_stock_level,<br/>get_supplier_delivery_date,<br/>create_transaction (sales),<br/>get_cash_balance, generate_financial_report</i>"]

    DB[("Munder Difflin SQLite DB<br/>transactions · inventory ·<br/>quotes · quote_requests")]

    Customer -- "request text + request date" --> Orchestrator
    Orchestrator -- "final quote / decision + rationale" --> Customer

    Orchestrator -- "ask_inventory_agent (optional)" --> Inventory
    Inventory -. result .-> Orchestrator
    Orchestrator -- "ask_quoting_agent" --> Quoting
    Quoting -. result .-> Orchestrator
    Orchestrator -- "ask_sales_agent" --> Sales
    Sales -. result .-> Orchestrator

    Inventory --- DB
    Quoting --- DB
    Sales --- DB

    classDef agent fill:#3b6ea5,stroke:#1f3f5c,color:#ffffff,font-weight:bold;
    classDef store fill:#6b6b6b,stroke:#3a3a3a,color:#ffffff,font-weight:bold;
    classDef actor fill:#5a5a5a,stroke:#333333,color:#ffffff,font-weight:bold;
    class Orchestrator,Inventory,Quoting,Sales agent;
    class DB store;
    class Customer actor;
```

**Tool bindings (helper function → agent):**

| Helper function (starter code) | Agent(s) using it |
| --- | --- |
| `get_all_inventory` | Inventory Agent |
| `get_stock_level` | Inventory Agent, Quoting Agent, Sales Agent |
| `get_supplier_delivery_date` | Inventory Agent, Sales Agent |
| `create_transaction` | Inventory Agent (`stock_orders`), Sales Agent (`sales`) |
| `get_cash_balance` | Inventory Agent, Sales Agent |
| `generate_financial_report` | Sales Agent (pre-fulfillment health check on large orders) |
| `search_quote_history` | Quoting Agent |

All database access goes through these starter-code helper functions — no raw SQL
from agent code. `match_catalog_item` is a deterministic (non-LLM) fuzzy-matching
tool shared by the Orchestrator, Inventory Agent, Quoting Agent, and Sales Agent to
map free-text item descriptions to exact catalog names, avoiding item-name
hallucination — each agent re-validates the name itself as a safety net, and every
DB-touching tool also rejects any `item_name` that isn't an exact catalog match.

## 2. Sequence diagram (`sequence_diagram.png`)

Shows the temporal flow and data exchanged for a single customer request, including
the branch between a fulfilled order and a rejected one.

```mermaid
sequenceDiagram
    actor C as Customer
    participant O as Orchestrator Agent
    participant Q as Quoting Agent
    participant S as Sales / Fulfillment Agent
    participant DB as Munder Difflin DB

    C->>O: request text + request date
    O->>O: extract request date, needed-by date
    O->>O: match_catalog_item(item) for each<br/>requested item -> exact catalog names

    O->>Q: ask_quoting_agent("Request date: ...", matched items)
    Q->>Q: match_catalog_item(item) (double-check)
    Q->>DB: search_quote_history(keywords)
    DB-->>Q: comparable past quotes
    Q->>DB: get_stock_level(item, date)
    DB-->>Q: current stock
    Q-->>O: quoted total + explanation

    O->>S: ask_sales_agent("Request date: ...", "Needed by: ...", items, quoted total)
    S->>S: match_catalog_item(item) (double-check)
    S->>DB: get_stock_level(item, request date)
    DB-->>S: current stock
    S->>DB: get_supplier_delivery_date(order date, qty)
    DB-->>S: estimated delivery date

    alt stock sufficient and delivery meets deadline
        opt large order
            S->>DB: generate_financial_report(date)
            DB-->>S: cash / inventory / assets snapshot
        end
        S->>DB: create_transaction(item, "sales", qty, price, date)
        DB-->>S: transaction id
        S-->>O: order finalized + delivery estimate
    else insufficient stock or missed deadline
        S-->>O: rejected + plain-language reason
    end

    O-->>C: final quote / decision + rationale
```
