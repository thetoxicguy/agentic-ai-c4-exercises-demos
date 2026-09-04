## 🔧 What you need to fix to pass

✅ Sanitize every customer-facing response before it is returned from `Orchestrator.handle_customer_request`.

Add a small response-sanitizing function in `project_starter.py` near `call_your_multi_agent_system`, then call it from `Orchestrator.handle_customer_request`.

This keeps worker-agent reasoning available internally while preventing the customer from seeing cash balances, internal financial status, or contradictory fulfillment wording.

```python
def sanitize_customer_response(response: str) -> str:
    """Remove internal financial details from customer-facing text."""

    replacements = [
        (
            r"(?i)the company has a cash balance of \$[\d,]+(?:\.\d{2})?",
            "we are unable to restock in time",
        ),
        (
            r"(?i)cash balance of \$[\d,]+(?:\.\d{2})?",
            "restocking capacity",
        ),
        (
            r"(?i)insufficient cash balance",
            "we are unable to restock the requested items in time",
        ),
        (
            r"(?i)no available cash balance",
            "restocking is not available for this request",
        ),
        (
            r"(?i)internal issue",
            "processing issue",
        ),
    ]

    cleaned = response

    for pattern, replacement in replacements:
        cleaned = re.sub(pattern, replacement, cleaned)

    return cleaned
```

Then update the successful path in `handle_customer_request` like this:

```python
def handle_customer_request(self, request_text: str) -> str:
    try:
        raw_response = self.run(request_text)
        return sanitize_customer_response(raw_response)
    except Exception as exc:
        print(f"Error handling request: {exc}")
        return (
            "We're sorry, we could not complete this request right now. "
            "Please contact us so we can help with alternatives."
        )
```

After making that change, rerun the evaluation and audit every row in `test_results.csv`, not just row 8.

The resubmitted file should not contain customer-visible phrases such as:

- `cash balance`
- `insufficient cash balance`
- `$0.00`
- `internal issue`
- raw stack traces
- contradictory success/out-of-stock statements
---

## What can be improved
- The final output should be generated from structured state rather than free-form worker-agent prose. That would prevent contradictions like row 4: “successfully placed” while “both items are currently out of stock.”
- Audit every row after rerunning the evaluation, not only the examples below.

## ❌ What is missing or incorrect
Rows in test_results.csv leak internal company financial status to the customer. Row 8 says “the company has a cash balance of $0.00,” and rows 3, 5, 10, and 14 refer to insufficient or unavailable cash balance. Exact cash status is internal operational data and should not be included in customer-facing messages. 👉 This is a **core rubric requirement**.

Add this concrete sanitization layer in project_starter.py and call it before returning from Orchestrator.handle_customer_request:
```python
def sanitize_customer_response(response: str) -> str:
    """Remove internal financial details from customer-facing text."""
    replacements = [
        (r"(?i)the company has a cash balance of \$[\d,]+(?:\.\d{2})?", "we are unable to restock in time"),
        (r"(?i)cash balance of \$[\d,]+(?:\.\d{2})?", "restocking capacity"),
        (r"(?i)insufficient cash balance", "we are unable to restock the requested items in time"),
        (r"(?i)no available cash balance", "restocking is not available for this request"),
        (r"(?i)internal issue", "processing issue"),
    ]
    cleaned = response
    for pattern, replacement in replacements:
        cleaned = re.sub(pattern, replacement, cleaned)
    return cleaned
```

Then update handle_customer_request so the customer sees only sanitized text:
```python
def handle_customer_request(self, request_text: str) -> str:
    try:
        raw_response = self.run(request_text)
        return sanitize_customer_response(raw_response)
    except Exception as exc:
        print(f"Error handling request: {exc}")
        return "We're sorry, we could not complete this request right now. Please contact us so we can help with alternatives.
```
This fix removes cash-balance language, keeps the explanation customer-friendly, and avoids exposing internal exception wording.