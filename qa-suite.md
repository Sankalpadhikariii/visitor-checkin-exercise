# QA Test Suite — Visitor Check-in

Marker legend: `[ ]` not yet executed, `[pass]` passed, `[fail]` failed.

## Happy Path

**TC-001: Register a visitor with all required fields**
- Preconditions: Receptionist is on the registration form; at least one host employee exists.
- Steps:
  1. Enter a full name, company name, select a host employee, and enter a purpose of visit.
  2. Submit the registration form.
- Expected Result: Visitor is created and appears in the active visitor list with the current check-in time.
- Status: [ ]

**TC-002: Newly registered visitor shows correct check-in time**
- Preconditions: A visitor has just been registered.
- Steps:
  1. Confirm the check-in time shown in the active visitor list matches the current time in the receptionist's local (Kathmandu) timezone.
- Expected Result: Displayed check-in time matches current local time.
- Status: [ ] (Known related defect: see Defect 1 in defect-report.md — expect this to fail when tested near a UTC-offset boundary, e.g. late evening/midnight KTM.)

**TC-003: Check out an active visitor**
- Preconditions: At least one visitor is present in the active visitor list.
- Steps:
  1. Select a visitor from the active list.
  2. Trigger the check-out action for that visitor.
  3. Confirm the active visitor list.
- Expected Result: The visitor no longer appears in the active visitor list.
- Status: [ ]

**TC-004: Deactivate a visitor record as an administrator**
- Preconditions: At least one visitor record exists; tester has access to the deactivate action.
- Steps:
  1. Trigger the deactivate action for a visitor.
  2. Check the visitor's record.
- Expected Result: Visitor record is marked inactive (`active: false`).
- Status: [ ]

**TC-005: Register a repeat visitor by searching existing records**
- Preconditions: At least one active (non-deactivated) prior visitor exists.
- Steps:
  1. On the registration form, search for the existing visitor's name.
  2. Select the matching result.
  3. Submit the registration.
- Expected Result: A new check-in record is created for that visitor, reusing their existing details.
- Status: [ ]

## Negative Cases

**TC-006: Attempt to register a visitor with a missing full name**
- Preconditions: Receptionist is on the registration form.
- Steps:
  1. Leave the full name field blank.
  2. Fill in the remaining fields and submit.
- Expected Result: Submission is rejected with a validation error; no visitor record is created.
- Status: [ ]

**TC-007: Attempt to register a visitor with no host employee selected**
- Preconditions: Receptionist is on the registration form.
- Steps:
  1. Leave the host employee field unselected.
  2. Fill in the remaining fields and submit.
- Expected Result: Submission is rejected with a validation error; no visitor record is created.
- Status: [ ]

**TC-008: Attempt to check out a visitor who is already checked out**
- Preconditions: A visitor has already been checked out.
- Steps:
  1. Attempt to trigger check-out again for the same visitor (e.g. by repeating the API call or action).
- Expected Result: Unspecified in the spec — flag as an open question rather than a defect. Confirm whether the app errors, no-ops, or overwrites the checkout time.
- Status: [ ]

**TC-009: Deactivated visitor does not appear in active visitor list**
- Preconditions: A visitor has been deactivated but not checked out.
- Steps:
  1. Check the active visitor list for the deactivated visitor.
- Expected Result: Deactivated visitor should not appear in the active list.
- Status: [fail] (Confirmed defect — see Defect 2 in defect-report.md.)

**TC-010: Deactivated visitor is not selectable when registering a repeat visit**
- Preconditions: A visitor has been deactivated.
- Steps:
  1. On the registration form, search for the deactivated visitor's name.
- Expected Result: Deactivated visitor should not appear in search results.
- Status: [fail] (Confirmed defect — see Defect 3 in defect-report.md.)

## Boundary Cases

**TC-011: Active visitor list shows exactly 20 records on page 1**
- Preconditions: At least 21 active visitors exist in the system.
- Steps:
  1. Confirm the active visitor list page 1 shows exactly 20 records.
- Expected Result: Page 1 contains 20 records, no more.
- Status: [ ]

**TC-012: Active visitor list page 2 shows the 21st+ record**
- Preconditions: At least 21 active visitors exist.
- Steps:
  1. Navigate to page 2 of the active visitor list.
  2. Confirm the remaining record(s) appear here.
- Expected Result: Page 2 shows record 21 onward.
- Status: [ ]

**TC-013: Active visitor list with exactly 20 active visitors shows no page 2**
- Preconditions: Exactly 20 active visitors exist (no more, no fewer).
- Steps:
  1. Check whether pagination controls offer a page 2.
- Expected Result: No second page should be available/necessary.
- Status: [ ]

**TC-014: Empty active visitor list**
- Preconditions: No active visitors currently checked in (all checked out or none registered).
- Steps:
  1. Confirm the active visitor list view.
- Expected Result: List renders an empty state without errors.
- Status: [ ]

**TC-015: Search with a very short query (1 character)**
- Preconditions: Multiple visitor records exist.
- Steps:
  1. Enter a single character in the registration search box.
- Expected Result: Unspecified in spec — confirm the app handles this gracefully (e.g. returns broad matches or requires a minimum length) without erroring.
- Status: [ ]

**TC-016: Search with no matching results**
- Preconditions: Search term does not match any visitor.
- Steps:
  1. Enter a name that does not exist in any visitor record.
- Expected Result: Search returns an empty result set without erroring.
- Status: [ ]

---

## Regression Subset — Minor Registration Form Update

Scenario: the registration form receives a minor update (e.g. a new optional field is added, such as a phone number).

**Include in regression:**
- TC-001 (core registration still succeeds) — the primary happy path is the first thing any form change can break.
- TC-006, TC-007 (required-field validation) — new fields can accidentally alter existing validation logic or field ordering.
- TC-005 (repeat visitor registration via search) — form changes risk breaking the pre-fill/search integration.
- TC-010 (deactivated visitor not selectable) — since this touches the same form, worth re-confirming the fix/behavior wasn't affected.

**Exclude from regression:**
- TC-002 (timezone display) — unrelated to form fields; this is a display-layer concern tied to the visitor list, not the form itself.
- TC-003, TC-004 (check-out, deactivate) — these are separate actions/views not touched by a registration form update.
- TC-011–TC-014 (pagination) — pagination logic lives in the list view, not the registration form.
- TC-008, TC-009 — unrelated to the registration form specifically.

