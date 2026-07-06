# Code Review Notes

Fill this in as you work through the milestones. Each section mirrors the structure of a real GitHub pull request review.

---

## PR #1 — Bulk Purchase (`pr1_bulk_purchase.py`)

### Summary
*What does this PR do? (1–2 sentences in your own words)*

> This PR adds a bulk purchase feature that marks all currently unpurchased items on a list as purchased in one action, and records who purchased each item.

### Issues

For each issue you find, note: where it is (file + function), what's wrong, and why it matters in production.

**Issue 1**
- Location: [pr1_bulk_purchase.py](prs/pr1_bulk_purchase.py#L30-L36) (`purchase_all_items`)
- What's wrong: The query `Item.query.filter_by(list_id=list_id).all()` pulls in *every* item on the list, not just the ones that haven't been purchased yet. It doesn't filter on `is_purchased=False`, so items that were already purchased (possibly by a different user, at an earlier time) get swept up and overwritten by the bulk action.
- Why it matters: This silently destroys existing purchase history. If item A was already marked purchased by Alice yesterday, and Bob runs bulk-purchase today, item A's `purchased_by` and `purchased_at` get stomped with Bob's info — Alice's original purchase record is gone with no way to recover it. In production this means the list's purchase audit trail can't be trusted, and users could get credited/blamed for purchases they didn't make.
- Suggested fix: Add `is_purchased=False` to the `filter_by()` call so the query only touches unpurchased items: `Item.query.filter_by(list_id=list_id, is_purchased=False).all()`.

**Issue 2**
- Location: [pr1_bulk_purchase.py](prs/pr1_bulk_purchase.py#L51-L55) (`purchase_all` route)
- What's wrong: `user_id` is read from the request body but never validated — a request with no `user_id` (or an empty body, e.g. `curl -d '{}'`) still succeeds and marks every item purchased with `purchased_by = None`.
- Why it matters: In production you lose the audit trail of who actually bought the items, and callers get a false "success" response (`{"purchased": 8}`) instead of a clear error telling them the request was malformed.
- Suggested fix: Validate `user_id` before calling the service — e.g. `if not user_id: return jsonify({"error": "user_id is required"}), 400`.

**Issue 3** *(if found)*
- Location: [pr1_bulk_purchase.py](prs/pr1_bulk_purchase.py#L51-L55) (`purchase_all` route)
- What's wrong: The incorrect number of items are returned that were newly marked as purchased
- Why it matters: The output is inaccurate and includes previously purchased items
- Suggested fix: change the query for how we arrive at items, which an above issue will automatically fix. 

### Questions for the Author
*Things you're uncertain about — design choices that could be intentional or bugs depending on intent.*

>

### Verdict
- [ ] Approve — ship it
- [x] Request Changes — needs fixes before merging
- [ ] Comment — needs discussion before a verdict

**Rationale** *(1–2 sentences)*:

> The bulk purchase query overwrites already-purchased items' history, and the endpoint accepts requests with no `user_id` and still reports success. Both are straightforward fixes, but as written this would corrupt purchase data in production, so it needs changes before merging.

---

## PR #2 — List Stats (`pr2_list_stats.py`)

### Summary
*What does this PR do? (1–2 sentences in your own words)*

>

### Issues

**Issue 1**
- Location:
- What's wrong:
- Why it matters:
- Suggested fix:

**Issue 2**
- Location:
- What's wrong:
- Why it matters:
- Suggested fix:

**Issue 3** *(if found)*
- Location:
- What's wrong:
- Why it matters:
- Suggested fix:

### Questions for the Author
*A good code review often surfaces design questions, not just bugs. What would you want to clarify before approving?*

>

### Verdict
- [ ] Approve — ship it
- [ ] Request Changes — needs fixes before merging
- [ ] Comment — needs discussion before a verdict

**Rationale** *(1–2 sentences)*:

>

---

## Reflection

*Answer after completing both reviews.*

**1.** Which issue was hardest to spot, and why?

>

**2.** Which issues do you think an LLM reviewer (like Claude reviewing its own code) would most likely miss? Why?

>

**3.** One thing you'd add to a code review checklist for AI-generated backend code:

>
