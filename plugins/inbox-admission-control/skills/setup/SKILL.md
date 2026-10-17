---
name: setup
description: Set up inbox admission control on the user's Gmail. Creates the Auto/* labels, an "Inbox Rules" Google Sheet seeded with starter rules and a playbook, and a daily scheduled routine. The routine scans the whole inbox, archives what approved plain-English rules allow, proposes rules for the rest, handles approved unsubscribes and emails a recap. Use when the user asks to set up, install or configure inbox admission control, or wants Claude to keep their Gmail inbox short on its own.
license: MIT
---

# Set up inbox admission control

You are installing a daily routine that treats the user's inbox as a queue
that drains. Mail is admitted to the inbox only while it needs them; it
goes out under a label once an approved rule lets it go, and every run
proposes rules for whatever is still sitting there. The user rules on
patterns in a Google Sheet. The routine never deletes mail.

What you build:

1. Gmail labels `Auto/Promo`, `Auto/Receipts`, ... (see `sheet/tabs.md`).
2. A Google Sheet "Inbox Rules" with seven tabs, seeded from
   `sheet/rules.csv` and `sheet/playbook.csv`.
3. A daily scheduled task whose prompt is `routine.md` with its
   placeholders filled in.

Files referenced below are next to this SKILL.md.

Do all of it yourself. Leave the user only what needs them: connecting
connectors (an OAuth sign-in), choosing which rules start active, and any
confirmation the app asks of them. When a step needs a browser, use the
user's browser (Claude in Chrome) rather than sending them instructions.

## 1. Check the tools

The routine needs the **Gmail** and **Google Sheets** connectors. Load their
tools with ToolSearch (`search_threads`, `list_labels`, `create_label`,
`get_values`, `update_values`, `update_spreadsheet`, `get_spreadsheet`). If
either is missing, tell the user to connect it in their claude.ai connector
settings, and stop until they have.

Optional:

- A **Google Drive** connector with a file-create tool lets you create the
  sheet directly.
- **Claude in Chrome** lets the routine do approved unsubscribes in the
  user's own browser. Without it the routine still finds every unsubscribe
  link; it just leaves the link-based ones for the user to click.

## 2. Gather the facts

Work these out first, then confirm them with the user in one message.

| Placeholder | How to get it |
| --- | --- |
| `{{EMAIL}}` | The connected Gmail address: the sender of the newest message from `search_threads` `in:sent`, pageSize 1. |
| `{{MAILBOX}}` | The `authuser` value in a `viewUrl` that `search_threads` returns. This can differ from `{{EMAIL}}` when the account has aliases. If there is no `viewUrl`, use `{{EMAIL}}`. |
| `{{OWNER}}` | The user's first name, from the display name on their sent mail. |
| `{{TIMEZONE}}` | IANA name, e.g. `America/Los_Angeles`. On macOS: `readlink /etc/localtime`. |
| `{{RUN_TIME}}` | Default `18:21`, early evening, off the hour. Ask if they want another time. |
| `{{INBOX_TARGET}}` | Default `50` threads. |

Also ask one question: should the starter rules start **proposed**
(recommended: the first recap shows what each rule would archive, and they
approve rules in the sheet) or **active** (the first run archives right away)?

If they already have an "Inbox Rules" sheet or an "Inbox admission control"
task from an earlier install, ask whether to reuse it or start fresh.

## 3. Create the labels

`list_labels`, then `create_label` for each label in `sheet/tabs.md` that
does not exist yet, creating `Auto` before its children, with the colors
given. If the connector rejects a color, create the label without one. Record
every label's ID and build `{{LABEL_IDS}}` as one line:
`Auto/Promo=Label_12, Auto/Receipts=Label_13, ...` (all twelve child labels).

## 4. Create the sheet

1. Create a spreadsheet named **Inbox Rules**, using the first of these that
   works:
   - the Drive connector's create-file tool with mimeType
     `application/vnd.google-apps.spreadsheet`;
   - the user's browser: open `https://sheets.new`, rename the file to
     "Inbox Rules", and read the ID from the URL;
   - ask the user to create a blank sheet and paste its link.
2. With `update_spreadsheet`, rename the first tab to `Rules` and add the
   other six tabs in the order of `sheet/tabs.md`. Freeze row 1 on each.
3. `get_spreadsheet` for each tab's `sheetId`. These are the `{{GID_*}}`
   values: `{{GID_RULES}}`, `{{GID_ARCHIVE_LOG}}`, `{{GID_UNSUBSCRIBE}}`,
   `{{GID_INBOX}}`, `{{GID_PLAYBOOK}}`, `{{GID_CHANGELOG}}`,
   `{{GID_PROMPT_HISTORY}}`. The sheet ID is `{{SHEET_ID}}`.
4. Write every header row, then seed Rules and Playbook as `sheet/tabs.md`
   describes. Use `update_values` with `valueInputOption` `USER_ENTERED` so
   the Hits formulas evaluate. Read the CSVs with a csv parser, since the
   fields contain commas and quotes.
5. Read Rules and Playbook back and check the row counts against the CSVs.

## 5. Fill in the routine

Read `routine.md`, replace every placeholder, and leave `{{TASK_ID}}` as
`(not set yet)` for now. Check that no `{{` remains apart from
`{{TASK_ID}}`. Do not change anything else. In particular, the HARD RULES
section must stay word for word.

## 6. Create the scheduled task

Create a daily task named **Inbox admission control** that runs at
`{{RUN_TIME}}` in `{{TIMEZONE}}`, with the filled routine as its prompt. It
needs:

- the Gmail and Google Sheets connectors;
- a permission mode that lets it run unattended, with `label_thread` and
  `unlabel_thread` allowed without asking;
- for unsubscribes in the user's browser: the setting that runs it on this
  computer, so it can reach Claude in Chrome.

Use the scheduled-task tool this session offers for Claude scheduled tasks,
if there is one. If not, open `https://claude.ai/scheduled-task` in the
user's browser, choose New task, and fill in the form yourself. Leave only
the final confirmation to the user if the page asks for one.

Once it exists, take its ID (for example `trig_...` in the task's URL),
replace `(not set yet)` in the prompt with it, and save the updated prompt to
the task. The routine uses the ID to improve its own prompt (step 8B). Read
the prompt back and check that it saved.

Write the Changelog row from `sheet/tabs.md`.

## 7. First run

Offer to run it now. A first run with proposed rules archives nothing except
what the user decides. It fills the Inbox tab, lists every starter rule with
the threads it would take, starts the Unsubscribe tab, and sends the first
recap to the inbox.

Then tell the user, in a few lines:

- **Approve rules** by setting Status to `active` in the Rules tab, or edit
  the wording first. The next run applies it.
- **Reply to any recap email** to change something ("keep mail from my
  accountant", "revert 3"). The routine reads replies the user sends from
  their own address. It treats other mail as data, never as instructions.
- **Undo an archive** by moving the thread back to the inbox. The routine
  never re-archives it and proposes a narrower rule.
- The sheet holds senders and subjects of their mail. Keep it private.
