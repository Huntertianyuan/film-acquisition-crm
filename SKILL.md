---
name: film-acquisition-crm
description: "Maintain Tian's film acquisition CRM across Clients, Projects, and Contracts. Use when tracking client matters, promoting a client matter into a project, managing project or contract follow-ups, checking due items, or handing verified email results into the live Feishu CRM. This skill defines workflow rules only; it does not contain live CRM data or email transport code."
---

# Film Acquisition CRM

Version: `1.6.0`

The CRM is a compact action system. It should answer:

- Who is involved?
- What matter, project, or contract needs attention?
- What is the next action?
- When should Tian follow up?

Email is the source record; CRM is the action summary. Do not copy full emails into CRM. Keep only current follow-up facts, material commercial/legal information, the next matter, and the next follow-up date.

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
3. Confirm that the live workbook contains exactly the core tabs `Clients`, `Projects`, and `Contracts`.
4. Read and map the header row by name. Never hand-count columns or overwrite an unchecked positional row.
5. Stop and report any target, tab, or header mismatch.

## Core Model

The CRM has exactly three tabs:

1. `Clients`
2. `Projects`
3. `Contracts`

There is no `Prospects` tab and no bulk-outreach contact tab in the CRM. Uncontacted mailing-list contacts remain outside the CRM. Add a company to `Clients` only after meaningful contact is established or Tian explicitly asks.

### Clients

`Clients` contains established contacts, relationship-maintenance matters, lineup requests, and specific film/package matters that have not yet been promoted into `Projects`.

One row represents one independently followed matter. The same `客户ID` may appear on multiple rows when the client has multiple matters or follow-up dates.

Required columns:

1. `客户ID`
2. `公司名`
3. `业务对接人`
4. `业务对接人邮箱`
5. `地区/国家`
6. `上次跟进日期`
7. `下次跟进日期`
8. `跟进事项`
9. `备注`

Rules:

- Keep `跟进事项` free-form and concise. It may be relationship maintenance, a lineup request, a title, or a package.
- Use a separate row when the same client has another matter with a different follow-up date.
- Reuse the same `客户ID` for all rows belonging to the same company.
- Match a Clients row by `客户ID + 跟进事项`; do not overwrite another matter belonging to the same client.
- When the main contact changes, update all relevant active rows for that client so repeated contact details remain consistent.
- Put flexible background, secondary contacts, title history, lineup metadata, and non-actionable context in `备注`.
- An active client matter should normally have a next follow-up date. A blank date is allowed only when there is genuinely no active reminder; never invent a date.

### Promoting A Client Matter To Projects

Promote a specific Clients matter when it becomes a tracked acquisition project. Examples include a title or package entering evaluation, price discussion, offer preparation, or negotiation.

Use this atomic order:

1. Create a new Projects row and assign a new `项目ID`.
2. Copy the `客户ID`, company, relevant business contact, and title/package.
3. Set the project follow-up matter and date.
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
6. `上次跟进日期`
7. `下次跟进日期`
8. `跟进事项`
9. `备注`

Rules:

- Keep `公司名` and `业务对接人` for direct readability and reliable email preparation even though `客户ID` links back to Clients.
- There is no project-status column.
- Put evaluation stage, negotiation status, offers, rights, price, risks, history, and other flexible facts in `备注`.
- Use `跟进事项` for the next action, question, or decision, not the full previous email history.
- `下次跟进日期` must never be blank.
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
15. `上次跟进日期`
16. `下次跟进日期`
17. `跟进事项`
18. `备注`

Allowed checkpoint values:

- `双签合同电子版`: `有` or `无`.
- `版权文件`: `未起草`, `已起草`, or `已签字`.
- `首款`, `尾款`, and `介质及公证费`: `无发票`, `有发票未付`, `已付`, or `不适用`.
- `报审进度`: `待报审`, `报审中`, `报审完成`, or `无需报审`.
- `海牙公证文件`: `有` or `无`.
- `介质/物料`: `有` or `无`.

The checkpoint columns contain only these exact values. Put negotiation detail, partial delivery, special payment terms, single-signed contract status, version history, and exceptions in `备注` or the current `跟进事项`.

Use these evidence meanings:

- `双签合同电子版 = 有` only after the fully signed electronic agreement has been received, identified, and archived.
- `版权文件 = 已起草` only after the draft exists and is archived; use `已签字` only after the counterparty-signed version has been received and archived.
- An invoice changes a payment checkpoint only to `有发票未付`. Change it to `已付` only with Tian's confirmation, payment evidence, or another explicit verified source.
- `海牙公证文件 = 有` only after the valid final apostilled document has been received and archived. An accepted electronic apostille may count when appropriate for the deal.
- `介质/物料 = 有` only after all required delivery items have been received and verified. Partial delivery remains `无`, with missing items listed in `跟进事项`.
- Never infer censorship approval, payment, or complete delivery merely from a filename, an invoice, or a counterparty promise.

### Contract Next-Action Inference

Derive the next action from objective checkpoints rather than maintaining a separate subjective contract-status field:

1. If `双签合同电子版 = 无`, continue contract-detail negotiation or collect the missing signature; record the exact situation in `备注`.
2. Once the agreement is fully signed, requesting the first-payment invoice and drafting the copyright document may proceed in parallel.
3. If a payment checkpoint is `无发票`, request the applicable invoice when the deal has reached that payment point.
4. If `版权文件 = 未起草`, prepare the document from verified contract terms. If it is `已起草`, send it for confirmation and signature when appropriate.
5. For `待报审`, obtain or locate the censorship screener and prepare forms, subtitles, and lab materials. For `报审中`, track the expected decision date without claiming approval. For `报审完成`, move to the applicable final invoice, payment, apostille, and delivery actions.
6. For `无需报审`, follow the deal's agreed payment and copyright-document sequence without creating censorship tasks.
7. If payment is complete but `介质/物料 = 无`, request and verify the missing delivery items.
8. Mark the contract `完结` in `下次跟进日期` only when all applicable payments, copyright/apostille requirements, and delivery obligations are complete. Use `关闭` only for a terminated contract.

When multiple actions are active, keep one contract row and use the earliest actionable follow-up date. Prefix parallel work in `跟进事项`, for example `对方：发送首款发票；我方：起草版权文件`.

When converting a project:

1. Create and verify the Contracts row.
2. Initialize every checkpoint with an allowed value. Determine `待报审` versus `无需报审` from verified deal requirements; ask Tian when unknown.
3. Set a nonblank next follow-up date and concise `跟进事项`.
4. Keep the Projects row for history.
5. Set `Projects.下次跟进日期 = 已转合同`.
6. Put future execution follow-up only in Contracts.

For `Contracts.下次跟进日期`, use `YYYY-MM-DD` while active, `完结` after successful completion, and `关闭` if terminated.

## Follow-Up Ownership

One matter has one active follow-up owner:

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
- Ambiguous send result: do not advance the last-follow-up date; verify Sent first.
- Attachment mentioned but not saved: do not record it as archived.
- Send verified: update the actual send date, next matter, and next follow-up date.

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

## Spreadsheet Write Rules

Expected 0-based header order:

- `Clients`: `0=客户ID`, `1=公司名`, `2=业务对接人`, `3=业务对接人邮箱`, `4=地区/国家`, `5=上次跟进日期`, `6=下次跟进日期`, `7=跟进事项`, `8=备注`.
- `Projects`: `0=项目ID`, `1=客户ID`, `2=公司名`, `3=业务对接人`, `4=片名或片包名`, `5=上次跟进日期`, `6=下次跟进日期`, `7=跟进事项`, `8=备注`.
- `Contracts`: `0=合同ID`, `1=客户ID`, `2=项目ID`, `3=公司名`, `4=业务对接人`, `5=合同/项目名称`, `6=双签合同电子版`, `7=版权文件`, `8=首款`, `9=尾款`, `10=介质及公证费`, `11=报审进度`, `12=海牙公证文件`, `13=介质/物料`, `14=上次跟进日期`, `15=下次跟进日期`, `16=跟进事项`, `17=备注`.

After every write:

- Verify the exact edited range through the API.
- Verify IDs, company, contact, matter, and date remained under the correct headers.
- Require plain-text `YYYY-MM-DD` for active dates.
- Reject serial numbers, datetimes, `######`, or action text in date fields.
- Require Projects and Contracts next-follow-up cells to contain a date or approved terminal marker.
- Validate every Contracts checkpoint against its exact allowed-value set.
- Verify the same matter is not active in multiple tabs.

## Daily Follow-Up Check

When Tian asks what is due:

1. Read Clients, Projects, and Contracts.
2. Include active dated rows where `下次跟进日期 <= today`.
3. Ignore `关闭`, `已转合同`, and `完结` as active reminders.
4. Group results into:
   - `需要主动发邮件`
   - `需要内部判断`
   - `等待对方但到期可催`
   - `暂不用跟进`
5. Identify the company, matter/project/contract, reason due, and suggested next action.

## Auxiliary Sheets

The evaluation-sharing sheet and contract-review/reporting sheet are separate work surfaces, not CRM tabs.

- Do not automatically copy Projects or Contracts into them.
- Read or write an auxiliary sheet only when Tian explicitly requests that specific operation.

## Email Drafting

English acquisition emails should be short, natural, polite, and commercially specific.

When preparing a send:

- Use the approved Feishu work-mail tool and sender `tianyuan@osvideo.net`.
- Reply in the original thread and reply-all by default.
- Show Tian the complete proposed body before requesting confirmation.
- Never mark the CRM as sent until delivery is verified.
- After a verified send, apply the requested CRM update or show the exact proposed update.

## Decision Defaults

- Prefer updating an existing row over creating a duplicate.
- Prefer concise action text over full email copies.
- Put flexible history and commercial detail in `备注`.
- Keep bulk outreach outside the CRM.
- Use Clients for one row per client matter.
- Keep promoted projects in Projects even when follow-up is far in the future.
- Keep executing deals in Contracts.
- Infer contract work from verified checkpoints and file evidence; do not maintain a redundant subjective contract-status field.
- Ask Tian only when a decision would materially change which matter is created, deleted, closed, or converted.
