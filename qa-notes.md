# QA Notes

## Highest-Risk Area

The highest-risk area of this feature is the **deactivation flow as a whole**, rather than any single defect in isolation. Testing surfaced three related issues that, together, mean the "deactivate a visitor" capability is effectively non-functional end-to-end:

1. There is no way to deactivate a visitor from the UI at all — the backend endpoint (`PATCH /api/visitors/:id/deactivate`) works correctly when called directly, but no button, link, or admin view exposes it to a real user.
2. Even when a visitor is deactivated (e.g. via direct API call), they still appear in the active visitor list, since the list query only filters on `checked_out_at IS NULL` and never checks the `active` flag.
3. Deactivated visitors also remain selectable in search results when registering a repeat visit, since the search query has no `active` filter at all.

This matters because deactivation is presumably meant to serve as an access-control or safety mechanism — for example, blocking a visitor who caused a problem from being re-registered. As it stands, deactivating a visitor has no observable effect anywhere in the application: they're not blocked from search, not removed from the active list, and there's no way to even trigger deactivation without going around the UI entirely. A receptionist or administrator using this system as designed would have no way to know the feature isn't working, since nothing in the interface indicates deactivation failed or is unavailable.

## Question for the Product Owner

For the double-checkout scenario (see TC-008 in qa-suite.md): when a visitor who has already been checked out is checked out again, the system currently overwrites `checked_out_at` with a new timestamp silently, with no error or confirmation.

**Should this be blocked, silently ignored (no change to the existing timestamp), or is overwriting the intended behavior?**

This affects whether the checked-out timestamp can be relied on as an accurate audit trail — if overwriting is unintentional, a receptionist accidentally double-clicking "Check Out" could quietly erase the visitor's true departure time with no way to recover it.
