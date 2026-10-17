# The "Inbox Rules" sheet

One Google Sheet holds everything the routine reads and writes. The owner
edits it; the routine appends to it. Create the tabs in this order, freeze row
1 on each, and write the header row exactly as shown (the routine addresses
columns by letter).

| Tab | Header row (A, B, C, ...) |
| --- | --- |
| Rules | ID, Status, Action, Label, Rule, Why, Added, Hits |
| Archive log | Date, Time, From, Subject, Rule, Label, Reason, Thread ID, Received |
| Unsubscribe | Sender, Example subject, Emails seen (30 days), Method, Unsubscribe link or address, Status, Proposed, Done, Notes |
| Inbox | Thread ID, Last message, Age (days), From, Subject, Msgs, Unread, Status, Suggested action, Suggested label, Rule, Why it is here, Your decision, Notes |
| Playbook | Date, Area, Lesson or instruction, Source, Status, Notes |
| Changelog | Date, Type, What changed, Why, How to revert, # |
| Prompt history | Date, Version, Reason for the change, Full prompt before the change |

## Rules

Seed from `rules.csv`. For each row:

- **Status**: `proposed` unless the owner said in chat to start with the
  starter rules active. Proposed KEEP rules already protect mail; proposed
  ARCHIVE rules do nothing until the owner sets them to `active`.
- **Added**: today's date.
- **Hits**: the formula `=COUNTIF('Archive log'!$E:$E,A<row>)`, written with
  `USER_ENTERED` so it evaluates.

Values the owner uses in **Status**: `active`, `proposed`, `off`.

## Playbook

Seed from `playbook.csv`, with **Date** set to today. These are operational
lessons from real runs. The routine reads every `active` row at the start of
each run and appends new lessons as it learns them.

## Everything else

Archive log, Unsubscribe, Inbox, Changelog and Prompt history start empty
(header row only). Write one Changelog row at setup:
`<today>, Routine, Installed inbox-admission-control <version>, Setup, Delete the scheduled task and the sheet, 1`.

## Labels

Create these Gmail labels (the parent `Auto` first). Colors match the recap's
section colors; they must come from Gmail's label palette.

| Label | Background | Text |
| --- | --- | --- |
| Auto | #cccccc | #000000 |
| Auto/Promo | #cccccc | #000000 |
| Auto/Receipts | #16a765 | #ffffff |
| Auto/Shipping | #2da2bb | #ffffff |
| Auto/Calendar | #4a86e8 | #ffffff |
| Auto/Alerts | #ffad47 | #000000 |
| Auto/Security | #fb4c2f | #ffffff |
| Auto/Bills | #fad165 | #000000 |
| Auto/Notices | #a479e2 | #ffffff |
| Auto/Recruiting | #f691b2 | #000000 |
| Auto/Done | #43d692 | #000000 |
| Auto/Aged | #434343 | #ffffff |
| Auto/Junk | #7a4706 | #ffffff |
