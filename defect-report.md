Summary: Check-in time displayed in UTC instead of receptionist's local (Kathmandu) timezone

Type: Functional

Description: The spec states all times should be displayed in the receptionist's local timezone. The backend correctly serializes checked_in_at as an ISO 8601 string with timezone info, but when registering a visitor near midnight Kathmandu time (UTC+5:45), the frontend displayed 18:13 instead of the correct local time. This indicates the frontend is not converting the UTC timestamp to Asia/Kathmandu before rendering it.

Steps to Reproduce:

Set system clock to Asia/Kathmandu timezone, at a time near midnight local time
Register a new visitor via the check-in form
Observe the check-in time shown in the active visitor list

Expected Result: Check-in time should display in Kathmandu local time (e.g. ~00:1x for a registration made just after midnight).

Actual Result: Check-in time displayed as 18:13, matching the raw UTC value rather than the converted local time.

Summary: Deactivated visitors remain visible in the active visitor list

Type: Functional

Description: The active visitor list only filters on checked_out_at IS NULL and does not exclude visitors with active: false. As a result, a deactivated visitor who hasn't been checked out still appears in the active list.

Steps to Reproduce:

Register a visitor (e.g. via POST /api/visitors)
Deactivate that visitor (PATCH /api/visitors/:id/deactivate)
Fetch the active visitor list (GET /api/visitors?page=1)
Observe the deactivated visitor is still present

Expected Result: Deactivated visitors should not appear in the active visitor list, per spec.

Actual Result: Visitor with active: false (e.g. id 1, Ana Smith) is included in the response.

Summary: Deactivated visitors remain selectable via search for repeat visit registration

Type: Functional

Description: The /api/visitors/search endpoint filters only by full_name LIKE, with no active scope, so deactivated visitors are returned and would be selectable when registering a repeat visit.

Steps to Reproduce:

Register and deactivate a visitor (e.g. "Ana Smith")
Query GET /api/visitors/search?q=Ana
Observe the deactivated visitor appears in the results

Expected Result: Deactivated visitors should be excluded from search results used for repeat-visit selection.

Actual Result: Ana Smith (id 1, active: false) is returned in the search results.
Assumptions / Open Questions

Unpermitted parameter warning on visitor creation: Server logs show "Unpermitted parameter: :visitor" on POST /api/visitors — the frontend sends both flat parameters (full_name, company_name, etc.) and a redundant nested visitor: {...} object. The flat parameters are correctly permitted and used, so this does not currently cause data loss, but it suggests an inconsistency between the frontend's request shape and the backend's expected contract. Flagging as an assumption/observation rather than a confirmed defect, since behavior is currently correct despite the warning.