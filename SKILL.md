---
name: film-acquisition-crm
description: "Maintain Tian's film acquisition CRM across Clients, Projects, and Contracts. Use when tracking or promoting acquisition matters, managing contract checkpoints and project files, drafting copyright documents, checking due items, or handing verified email results into the live Feishu CRM. This skill defines workflow rules only; it does not contain live CRM data or email transport code."
---

# Film Acquisition CRM

Version: `1.7.0`

The CRM is a compact action system. It should answer:

- Who is involved?
- What matter, project, or contract needs attention?
- What is the next action?
- When does it need attention again?

Email is the source record; CRM is the action summary. Do not copy full emails into CRM. Keep only current action facts, material commercial/legal information, the next action, and the next action date.

`Next action date` is the date when a matter should return to the active queue. It may remind Tian to act, chase a counterparty, check a milestone, or make a decision. It does not necessarily mean that Tian should send an email on that date.

## Action Field Semantics

- `最近处理日期` records the last meaningful internal or external action, such as sending an email, evaluating a title, making a payment, preparing a document, or checking a milestone. A CRM-only edit does not count as handling the matter.
- `下次行动日期` is the date to revisit the matter. It may represent our own task, a counterparty reminder, or a status check.
- `下一步行动` states what should happen when the matter returns to the active queue. It is not a copy of the previous email.

Use concise responsibility prefixes when they clarify what happens next:

- `我方：` for Tian's or the internal team's next task.
- `对方：` for an expected counterparty action or an overdue item that may need chasing.
- `检查：` for a milestone, availability, censorship, rights-window, or status review.
- `双方：` for a decision or step that requires both sides.

Combine prefixes for parallel work, for example `对方：发送首款发票；我方：起草版权文件`. Do not add a separate owner or action-type column.

## Operating Boundaries

- Use only the live Feishu spreadsheet configured by Tian for normal CRM reads and writes.
- Use the configured Feishu Sheets Open API client or connector. Never use browser automation or Computer Use for CRM writes without Tian's explicit permission.
- After every API write, read the exact edited range back and verify it before reporting completion.
- Treat local Excel files as offline backups only. Never substitute a local workbook for a failed live Feishu write.
- Do not hardcode workbook paths, attachment folders, or email transport methods in this workflow skill.
- Use the active approved mailbox skill for email. Draft first and send only after the mailbox confirmation flow is complete.

### CRM Target Guard

Before writing:

1. Read `.codex/film-acquisition-crm.json` in the active project and then in the user home directory.
2. If both exist, require them to agree on the primary spreadsheet and sheet IDs.
3. If local contract files are involved, read `contract_archive_root` from the same configuration files and require both values to agree.
4. Confirm that the live workbook contains exactly the core tabs `Clients`, `Projects`, and `Contracts`.
5. Read and map the header row by name. Never hand-count columns or overwrite an unchecked positional row.
6. Stop and report any target, tab, header, or archive-root mismatch.

If an approved schema migration is still pending, do not perform normal row writes against the old layout. Migrate and verify the complete tab first, or defer the write.

## Core Model

The CRM has exactly three tabs:

1. `Clients`
2. `Projects`
3. `Contracts`

There is no `Prospects` tab and no bulk-outreach contact tab in the CRM. Uncontacted mailing-list contacts remain outside the CRM. Add a company to `Clients` only after meaningful contact is established or Tian explicitly asks.

### Clients

`Clients` contains established contacts, relationship-maintenance matters, lineup requests, and specific film/package matters that have not yet been promoted into `Projects`.

One row represents one independently tracked matter. The same `客户ID` may appear on multiple rows when the client has multiple matters or action dates.

Required columns:

1. `客户ID`
2. `公司名`
3. `业务对接人`
4. `业务对接人邮箱`
5. `地区/国家`
6. `最近处理日期`
7. `下次行动日期`
8. `下一步行动`
9. `备注`

Rules:

- Keep `下一步行动` free-form and concise. It may be relationship maintenance, a lineup request, a title, a package, an internal task, or a milestone check.
- Use a separate row when the same client has another matter with a different action date.
- Reuse the same `客户ID` for all rows belonging to the same company.
- Before updating Clients, read every row with the same `客户ID` and identify the exact matter from its current `下一步行动` and `备注`. Do not use the changing action text as a stable key, and do not overwrite another matter belonging to the same client. Ask Tian when the target matter remains ambiguous.
- When the main contact changes, update all relevant active rows for that client so repeated contact details remain consistent.
- Put flexible background, secondary contacts, title history, lineup metadata, and non-actionable context in `备注`.
- An active client matter should normally have a next action date. After a verified inbound reply, a blank date is not allowed unless Tian explicitly decides that the matter is permanently dormant. For a reply that needs no immediate response, use a review date and write `检查：定期查看片单/项目` (or an equivalent concrete review action) instead of silently dropping the matter from the queue.

### Promoting A Client Matter To Projects

Promote a specific Clients matter when it becomes a tracked acquisition project. Examples include a title or package entering evaluation, price discussion, offer preparation, or negotiation.

Use this atomic order:

1. Create a new Projects row and assign a new `项目ID`.
2. Copy the `客户ID`, company, relevant business contact, and title/package.
3. Set the project next action and action date.
4. Read the new Projects row back and verify it.
5. Delete only the corresponding Clients matter row.

Do not write `已转Projects` in Clients and do not retain a duplicate Clients row for the same matter. Never delete unrelated rows that share the same `客户ID`.

### Projects

`Projects` contains tracked acquisition projects, including:

- Projects ready for evaluation.
- Projects in negotiation.
- Projects waiting for a screener, rights window, price information, or a distant future date after they have already been promoted.

Once a matter enters `Projects`, keep it there until it is closed or converted into a contract. Do not move it back to Clients merely because it becomes immature, delayed, or long-term.

Required columns:

1. `项目ID`
2. `客户ID`
3. `公司名`
4. `业务对接人`
5. `片名或片包名`
6. `最近处理日期`
7. `下次行动日期`
8. `下一步行动`
9. `备注`

Rules:

- Keep `公司名` and `业务对接人` for direct readability and reliable email preparation even though `客户ID` links back to Clients.
- There is no project-status column.
- Put evaluation stage, negotiation status, offers, rights, price, risks, history, and other flexible facts in `备注`.
- Use `下一步行动` for the next action, question, check, or decision, not the full previous email history.
- `下次行动日期` must never be blank.
- Use `YYYY-MM-DD` for active projects, including long-term reminders.
- Use `关闭` when the project will no longer be pursued.
- Use `已转合同` after the project is converted into Contracts.

### Contracts

Contracts begin after both sides confirm the core commercial terms, such as price, rights, territory, and license term. A contract must have both a `客户ID` and a `项目ID`.

Contracts is an evidence-based checkpoint table. One contract occupies one row. Do not create separate rows for invoices, copyright documents, censorship, or delivery tasks.

Required columns:

1. `合同ID`
2. `客户ID`
3. `项目ID`
4. `公司名`
5. `业务对接人`
6. `合同/项目名称`
7. `双签合同电子版`
8. `版权文件`
9. `首款`
10. `尾款`
11. `介质及公证费`
12. `报审进度`
13. `海牙公证文件`
14. `介质/物料`
15. `最近处理日期`
16. `下次行动日期`
17. `下一步行动`
18. `备注`

Allowed checkpoint values:

- `双签合同电子版`: `有` or `无`.
- `版权文件`: `有` or `无`.
- `首款`, `尾款`, and `介质及公证费`: `无发票`, `有发票未付`, `已付`, or `不适用`.
- `报审进度`: `待报审`, `报审中`, `报审完成`, or `无需报审`.
- `海牙公证文件`: `有` or `无`.
- `介质/物料`: `有` or `无`.

The checkpoint columns contain only these exact values. Put negotiation detail, partial delivery, special payment terms, single-signed contract status, version history, and exceptions in `备注` or the current `下一步行动`.

Use these evidence meanings:

- `双签合同电子版 = 有` only after the fully signed electronic agreement has been received, identified, and archived.
- `版权文件 = 有` only after all required drafts, such as the LOA and COT, exist and are archived. This field records that the documents have been drafted; it does not mean that they have been signed, apostilled, or fully received. Record signature, return, version, and missing-document details in `下一步行动` or `备注`.
- An invoice changes a payment checkpoint only to `有发票未付`. Change it to `已付` only with Tian's confirmation, payment evidence, or another explicit verified source.
- Use `不适用` only when verified contract terms confirm that the payment category does not apply.
- `海牙公证文件 = 有` only after every required valid final apostilled document has been received and archived. An accepted electronic apostille may count when appropriate for the deal.
- `介质/物料 = 有` only after all required delivery items have been received and verified. Partial delivery remains `无`, with missing items listed in `下一步行动`.
- Never infer censorship approval, payment, or complete delivery merely from a filename, an invoice, or a counterparty promise.

### Contract Next-Action Inference

Derive the next action from objective checkpoints rather than maintaining a separate subjective contract-status field:

1. If `双签合同电子版 = 无`, continue contract-detail negotiation or collect the missing signature; record the exact situation in `备注`.
2. Once the agreement is fully signed, requesting the first-payment invoice and drafting the copyright document may proceed in parallel.
3. If a payment checkpoint is `无发票`, request the applicable invoice when the deal has reached that payment point.
4. If `版权文件 = 无`, prepare the document from verified contract terms. If it is `有`, send it for confirmation and signature when appropriate, and record the exact outstanding step in `下一步行动`.
5. For `待报审`, obtain or locate the censorship screener and prepare forms, subtitles, and lab materials. For `报审中`, track the expected decision date without claiming approval. For `报审完成`, move to the applicable final invoice, payment, apostille, and delivery actions.
6. For `无需报审`, follow the deal's agreed payment and copyright-document sequence without creating censorship tasks.
7. Request missing delivery items only after the deal's payment, censorship, and document prerequisites for delivery are satisfied.
8. Mark the contract `完结` in `下次行动日期` only when all applicable payments, censorship, copyright/apostille, and delivery obligations are complete. Use `关闭` only for a terminated contract.

When multiple actions are active, keep one contract row, use the earliest actionable date, and combine responsibility prefixes in `下一步行动`.

When converting a project:

1. Create and verify the Contracts row.
2. Initialize every checkpoint with an allowed value. Determine `待报审` versus `无需报审` from verified deal requirements; ask Tian when unknown.
3. Set a nonblank next action date and concise `下一步行动`.
4. Keep the Projects row for history.
5. Set `Projects.下次行动日期 = 已转合同`.
6. Put future execution actions only in Contracts.

For `Contracts.下次行动日期`, use `YYYY-MM-DD` while active, `完结` after successful completion, and `关闭` if terminated.

## Action Ownership

One matter has one active action owner:

1. Contracts owns post-deal execution.
2. Projects owns a promoted acquisition project.
3. Clients owns a relationship matter, lineup request, or specific matter not yet promoted.

Different matters for the same company may have different dates in different rows or tabs.

Do not duplicate the same matter:

- On Clients-to-Projects promotion, verify Projects and delete the matching Clients row.
- On Projects-to-Contracts conversion, keep Projects only as history with the terminal marker `已转合同`; Contracts owns the active date.

## Evidence-First Updates

Use this order for email-driven work:

1. Read the exact source email or Tian's instruction.
2. Save and verify explicitly requested attachments.
3. Draft the outbound email and obtain the mailbox tool's required confirmation.
4. Send once and verify Sent, or obtain Tian's explicit confirmation of delivery.
5. Update the correct CRM row with verified facts.

Do not pre-commit future states:

- Draft only: write `待我方发送` only if Tian requests a CRM update before sending.
- Ambiguous send result: do not advance the most recent processed date; verify Sent first.
- Attachment mentioned but not saved: do not record it as archived.
- Send verified: update the actual send date, next action, and next action date.

### Inbound Mail Reconciliation Gate

Reading a relevant inbound email is not complete until its CRM effect has been reconciled. This gate applies whenever Tian asks to check, scan, review, follow up on, or identify pending work in the mailbox, and whenever an email task also mentions CRM.

For every inbound message from a company or contact represented in the CRM:

1. Match it to the correct matter using normalized email address, thread headers, company, title/package, and the current CRM action text. If more than one matter is plausible, stop and ask Tian instead of guessing.
2. Compare the email date with the row's `最近处理日期`. A newer valid reply must be treated as an unsynchronized CRM event.
3. Classify the message before writing:
   - `有效回复`: a human response containing a decision, answer, availability, price, title list, attachment, or other action-relevant fact.
   - `自动回复`: record only the stated return date or availability when relevant; do not treat it as a substantive reply.
   - `newsletter/无关邮件`: do not update the acquisition matter.
4. For a valid reply, update the exact owning row with the inbound date, the current action fact, and a concise note. If a lineup or catalogue was sent, record `片单状态：有片单`; record `已保存` only after the attachment has actually been downloaded and verified.
5. Derive a next action and date. If no immediate outreach is needed, set a review date using the CRM defaults and an explicit check action. Never use `暂不跟进` as a reason to leave the matter untracked.
6. After writing, read the exact edited range back and verify that the sender, matter, inbound date, action, and next date are under the intended headers.

Every mailbox review must end with one of these explicit outcomes for each relevant inbound message:

- `已同步 CRM`;
- `无需同步` with the reason (for example, newsletter or duplicate); or
- `待确认` with the exact ambiguity Tian must resolve.

Do not report a mailbox review as complete while a newer valid inbound reply remains unmatched or while a row still says `待要片单` after the email clearly says that a lineup was sent.

## Contract File Evidence

The live Feishu CRM remains the action system. Tian's approved local project folders are supporting evidence and the current working archive for contract documents, invoices, copyright documents, apostilles, censorship files, media, and materials.

When checking a contract:

1. Locate the exact project folder under the user-approved archive root.
2. Inspect files read-only before changing CRM checkpoints.
3. Match evidence by project, counterparty, document content, signatures, version, and date; do not rely on filenames alone.
4. Distinguish drafts, single-signed versions, fully signed versions, invoices, signed copyright documents, apostilled documents, censorship materials, and final delivery media.
5. Report ambiguous, conflicting, password-protected, unreadable, or unmatched files instead of guessing.

When archiving a newly received file:

- Use the exact existing project folder only when the match is unambiguous.
- Preserve the original filename by default.
- Never overwrite, delete, move, or rename an existing file without Tian's explicit instruction.
- If the destination is uncertain, a same-name file differs, or the project folder is missing, stop and ask Tian.
- Verify the saved file exists and is readable before updating the CRM checkpoint.
- A file attached to an email but not yet saved does not count as archived evidence.

### Copyright Document Drafting

When `版权文件 = 无` and Tian asks Codex to prepare the documents:

1. Use the fully signed agreement as the controlling source. Use an existing project document or approved precedent only for structure.
2. Extract and cross-check the picture title, licensor, licensee, territory, term, rights, exclusivity, sublicensing, and any anti-piracy language.
3. Prepare each required document, normally the LOA and COT, separately. Do not invent missing terms or copy another project's facts.
4. Save drafts to the configured project folder without overwriting existing versions, then reopen and verify them.
5. Change `版权文件` to `有` only after all required drafts are saved and verified. Sending, signing, and apostille remain later checkpoints recorded in the matter, notes, and dedicated apostille checkpoint.

## Spreadsheet Write Rules

Expected 0-based header order:

- `Clients`: `0=客户ID`, `1=公司名`, `2=业务对接人`, `3=业务对接人邮箱`, `4=地区/国家`, `5=最近处理日期`, `6=下次行动日期`, `7=下一步行动`, `8=备注`.
- `Projects`: `0=项目ID`, `1=客户ID`, `2=公司名`, `3=业务对接人`, `4=片名或片包名`, `5=最近处理日期`, `6=下次行动日期`, `7=下一步行动`, `8=备注`.
- `Contracts`: `0=合同ID`, `1=客户ID`, `2=项目ID`, `3=公司名`, `4=业务对接人`, `5=合同/项目名称`, `6=双签合同电子版`, `7=版权文件`, `8=首款`, `9=尾款`, `10=介质及公证费`, `11=报审进度`, `12=海牙公证文件`, `13=介质/物料`, `14=最近处理日期`, `15=下次行动日期`, `16=下一步行动`, `17=备注`.

After every write:

- Verify the exact edited range through the API.
- Verify IDs, company, contact, matter, and date remained under the correct headers.
- Require plain-text `YYYY-MM-DD` for active dates.
- Reject serial numbers, datetimes, `######`, or action text in date fields.
- Require Projects and Contracts next-action-date cells to contain a date or an approved terminal marker. For Projects, approved terminal markers are `关闭` and `已转合同`; for Contracts, they are `关闭` and `完结`. Do not report these markers as malformed dates.
- Validate every Contracts checkpoint against its exact allowed-value set.
- Verify the same matter is not active in multiple tabs.

## CRM Health Check

When checking due items or validating a write, scan all three tabs for:

1. Blank `下次行动日期` cells on active matters.
2. Active dates earlier than today.
3. Serial numbers, datetimes, `######`, action text, or unrecognized status text in date cells.
4. The same active matter appearing in more than one tab.
5. Projects or Contracts with missing required IDs.
6. Contracts marked `完结` while applicable payments, copyright/apostille, censorship, or delivery checkpoints are still incomplete.
7. Valid inbound emails newer than the matched row's `最近处理日期`.
8. Rows whose notes or action imply that a lineup/catalogue was received while the recorded lineup state still says `待要片单`.
9. Client matters with a substantive inbound reply but no next action date or review action.

Treat `关闭`, `已转合同`, and `完结` as valid terminal markers according to the tab rules above. Report health-check findings separately from actual due items, and do not change data during a read-only check unless Tian explicitly asks.

## Daily Action Check

When Tian asks what is due:

1. Read Clients, Projects, and Contracts.
2. Include active dated rows where `下次行动日期 <= today`.
3. Ignore `关闭`, `已转合同`, and `完结` as active reminders.
4. Group results into:
   - `需要我方行动`
   - `等待对方，到期可催`
   - `到期检查节点`
   - `需要双方推进`
   - `暂不用处理`
5. Identify the company, matter/project/contract, reason due, and suggested next action.

## Action Date Defaults

When Tian does not specify a date, suggest the following defaults; never override an explicit date:

- Routine outreach, lineup requests, sample requests, or offer follow-up: 7 days.
- Contract, invoice, copyright, apostille, censorship, or delivery matters: 3 to 5 days.
- Automatic replies: 5 days after the stated return date.
- Relationship maintenance or general lineup refresh: 30 days.
- Received lineup with no immediate title to discuss: 30 days for a review action, unless Tian specifies another interval.
- Rights or availability that depend on a known future date: use that actual trigger date rather than inventing a short reminder.

## Auxiliary Sheets

The evaluation-sharing sheet and contract-review/reporting sheet are separate work surfaces, not CRM tabs.

- Do not automatically copy Projects or Contracts into them.
- Read or write an auxiliary sheet only when Tian explicitly requests that specific operation.

## Email Drafting

English acquisition emails should be short, natural, polite, and commercially specific.

The approved mailbox skill owns sender identity, thread/reply-all behavior, confirmation, sending, and Sent verification. This CRM skill owns only the resulting evidence handoff: never record an email as sent before delivery is verified, and update the CRM only with verified facts.

## Decision Defaults

- Prefer updating an existing row over creating a duplicate.
- Prefer concise action text over full email copies.
- Put flexible history and commercial detail in `备注`.
- Keep bulk outreach outside the CRM.
- Use Clients for one row per client matter.
- Keep promoted projects in Projects even when the next action date is far in the future.
- Keep executing deals in Contracts.
- Infer contract work from verified checkpoints and file evidence; do not maintain a redundant subjective contract-status field.
- Ask Tian only when a decision would materially change which matter is created, deleted, closed, or converted.
