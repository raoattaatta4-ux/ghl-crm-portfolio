# Lead inquiry → appointment: CRM automation blueprint

A platform-neutral implementation plan intended for a GoHighLevel CRM build.
This is documentation, not an importable GHL snapshot or a deployed workflow.
Use synthetic test contacts when implementing it. Confirm available features and channel requirements in the target account.

## Objective
Keep new inquiries organized, assign a clear next action, and stop nurture messages once a contact replies, books, or opts out.

## Contact data
| Field | Purpose |
| --- | --- |
| Name and email | Contact identity and duplicate lookup |
| Service interest | Route the inquiry to the right service |
| Lead source and UTM values | Record campaign attribution when supplied |
| Channel consent and timestamp | Document the actual permission given; never infer consent from a visit |
| Assigned owner | Give one team member responsibility |
| Appointment status | Control booking-related transitions |
| Opt-out state | Suppress future messages on the affected channel |

Suggested pipeline: New Inquiry → Contact Attempted → Conversation Started → Appointment Booked → Appointment Completed → Won / Closed.
Use a separate Follow Up Later stage for contacts who explicitly request it.

## Routing logic
1. Validate incoming fields and normalize email for lookup.
2. Find an existing contact; update it rather than creating a duplicate.
3. Preserve existing opt-out preferences and established attribution.
4. Create or update the relevant open opportunity; do not blindly create another on every submission.
5. Assign the owner and create a follow-up task.
6. If the channel has valid permission and is not suppressed, acknowledge the inquiry.
7. Before each later message, re-check reply, appointment and opt-out status.
8. On a reply, stop automated nurture and hand the conversation to the owner.
9. On a booking, stop inquiry nurture and transition to appointment handling.
10. On an opt-out, suppress that channel immediately and cancel queued nurture.
11. On a cancellation, create an owner task; do not assume permission to restart a sequence.

## Example timing
These are configuration examples, not universal recommendations:
- At submission: route the lead and acknowledge an eligible inquiry.
- Next business day: owner task if there is no reply or booking.
- After three business days: one eligible follow-up, only after checking stop conditions.
- After seven business days: close the automated sequence and leave a manual review task.
Configure local timezone, quiet hours and business hours before enabling messaging.

## Build checklist
- Create the data fields, pipeline stages and assignment rules.
- Connect the form and map every field explicitly.
- Separate inquiry nurture from appointment reminders.
- Add checks immediately before every send, not only at entry.
- Define duplicate/re-entry behavior.
- Keep sending disabled while testing.
- Record the test results and approve actual copy before activating.

## Acceptance tests
| Scenario | Expected result |
| --- | --- |
| New inquiry | One contact, one relevant opportunity, assigned owner |
| Same email submitted twice | Existing contact updated; no duplicate open opportunity |
| No channel consent | Owner task only; no automated message |
| Contact replies during delay | Pending nurture stops; owner notified/task created |
| Contact books during delay | Inquiry nurture stops; appointment handling begins |
| Contact opts out | No later sends on the suppressed channel |
| Contact cancels | Owner task; no automatic nurture restart |
| Missing email | Validation fails; no incomplete contact created by this flow |
| Submission outside business hours | Follow-up respects configured timezone and schedule |

## Production boundary
Connectors, account credentials, actual sending, calendar events and contact records are outside this demo. No client data, account exports or credentials are included.
