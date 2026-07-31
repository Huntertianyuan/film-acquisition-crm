---
name: film-acquisition-crm
description: "Maintain Tian's film acquisition CRM across Clients, Projects, and Contracts. Use when tracking client matters, promoting a client matter into a project, managing project or contract follow-ups, checking due items, or handing verified email results into the live Feishu CRM. This skill defines workflow rules only; it does not contain live CRM data or email transport code."
---

# Film Acquisition CRM

Version: `1.5.0`

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

Keep the current Contracts structure until Tian completes the separate contract-workflow redesign:

1. `合同ID`
2. `客户ID`
3. `项目ID`
4. `公司名`
5. `业务对接人`
6. `合同/项目名称`
7. `合同状态`
8. `合同文件`
9. `付款进度`
10. `版权文件`
11. `介质/物料`
12. `上次跟进日期`
13. `上次跟进内容`
14. `下一步动作`
15. `下次跟进日期`
16. `备注`

Use Contracts for contract drafting/signature, invoices and payment, copyright documents, materials, censorship-related delivery, and other post-deal execution.

When converting a project:

1. Create and verify the Contracts row.
2. Keep the Projects row for history.
3. Set `Projects.下次跟进日期 = 已转合同`.
4. Put future execution follow-up only in Contracts.

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

## Spreadsheet Write Rules

Expected 0-based header order:

- `Clients`: `0=客户ID`, `1=公司名`, `2=业务对接人`, `3=业务对接人邮箱`, `4=地区/国家`, `5=上次跟进日期`, `6=下次跟进日期`, `7=跟进事项`, `8=备注`.
- `Projects`: `0=项目ID`, `1=客户ID`, `2=公司名`, `3=业务对接人`, `4=片名或片包名`, `5=上次跟进日期`, `6=下次跟进日期`, `7=跟进事项`, `8=备注`.
- `Contracts`: `0=合同ID`, `1=客户ID`, `2=项目ID`, `3=公司名`, `4=业务对接人`, `5=合同/项目名称`, `6=合同状态`, `7=合同文件`, `8=付款进度`, `9=版权文件`, `10=介质/物料`, `11=上次跟进日期`, `12=上次跟进内容`, `13=下一步动作`, `14=下次跟进日期`, `15=备注`.

After every write:

- Verify the exact edited range through the API.
- Verify IDs, company, contact, matter, and date remained under the correct headers.
- Require plain-text `YYYY-MM-DD` for active dates.
- Reject serial numbers, datetimes, `######`, or action text in date fields.
- Require Projects and Contracts next-follow-up cells to contain a date or approved terminal marker.
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
- Ask Tian only when a decision would materially change which matter is created, deleted, closed, or converted.
