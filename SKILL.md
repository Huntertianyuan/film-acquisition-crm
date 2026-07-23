---
name: film-acquisition-crm
description: "Personal workflow for maintaining Tian's film acquisition CRM across Prospects, Clients, Projects, and Contracts. Use when tracking outreach leads, promoting responsive contacts, updating film-buying contacts or projects, managing contract follow-ups, setting next follow-up dates, or drafting acquisition emails from user instructions or email context. This skill defines workflow rules only; it does not contain live CRM data, fixed file paths, email transport code, or spreadsheet scripts."
metadata:
  version: "1.3.1"
---

# Film Acquisition CRM

This is Tian's personal workflow for film acquisition follow-up.

The CRM is an action system, not a complete database. Its core job is to answer:

- Who is involved?
- What opportunity or contract is being discussed?
- Who owes the next reply?
- What should Tian do next?
- When should Tian be reminded?

Do not store live customer, project, or contract facts in this skill. Treat the CRM workbook and current emails as the source of truth.

Email is the source record; CRM is the action summary. Do not copy full emails into CRM. Extract only current actionable state, next owner, next step, next follow-up date, and material commercial/legal facts that affect future action.

## Operating Boundaries

- Do not hardcode the CRM workbook path. The user or runtime environment must provide it.
- Do not hardcode contract-file root folders. The user or runtime environment must provide them. Example paths may be mentioned only as examples, not defaults.
- Do not include or assume a specific email access method. Use the runtime's approved mailbox skill or tool when available.
- Draft first and send only after user confirmation unless the active mailbox tool has a stricter approval rule.
- Do not add scripts, automations, or transport-specific logic to this workflow skill.
- Do not expose internal tool details in user-facing responses unless the user asks.
- Respect the runtime's file-write and email-send approval mode. If a CRM update, file write, or email send requires user approval, ask for approval rather than bypassing safety controls.

### Canonical Workbook Guard

Before any CRM write:

1. Obtain the workbook path from Tian or the active runtime context and resolve it to an absolute path.
2. Report the exact target path before editing.
3. Verify the expected core sheet names and header rows.
4. Distinguish the canonical workbook from backups, exports, temporary files, downloaded copies, and files under generic `output` or `outputs` folders.

Do not infer that a workbook is canonical because its filename matches, it is the newest copy, or it is inside the current workspace. If more than one plausible workbook exists, stop and ask Tian which one is authoritative. Never update a derived copy while reporting that the CRM itself was updated.

## CRM Dimensions

The workbook has four core tabs: `Prospects`, `Clients`, `Projects`, and `Contracts`.

### Prospects

`Prospects` is the outreach pool for companies and contacts that have not yet become active CRM clients. It may contain hundreds of contacts across multiple countries.

Recommended fields:

- `Name`
- `Country`
- `Address`
- `Zip`
- `City`
- `State`
- `Tel`
- `Fax`
- `Activities`
- `Emails`
- `状态`

Use only these prospect statuses:

- `待联系`: no confirmed outreach has been sent.
- `已发送待回复`: outreach was confirmed sent and there is no meaningful human reply yet. Automatic replies remain here when the contact may still be active.
- `有效回复待处理`: a human reply shows real business interest or provides actionable acquisition context, but promotion has not yet been completed.
- `已转Clients`: the company has been created in or matched to `Clients`.
- `暂无需求`: a real person replied that there is no current need, but the relationship may be revisited.
- `退信/邮箱无效`: delivery failed or the address is invalid.
- `不再联系`: the contact opted out or Tian decided to stop outreach.

Promotion rules:

- Do not create a `Clients` row merely because an outreach email was sent.
- Automatic replies, out-of-office messages, delivery notices, and silence are not meaningful replies.
- A meaningful human reply includes interest, a catalogue or title offer, a screener or link offer, a request for more information, or another concrete business response.
- On a meaningful human reply, create or update the matching `Clients` row, then keep the original prospect row and set `状态 = 已转Clients`.
- Before promotion, search `Clients` by company, domain, and contact email. Update an existing client instead of creating a duplicate.
- Do not create a `Projects` row until a specific title or package discussion begins.
- Record bounces and corrected addresses in `Prospects`; do not promote an invalid address to `Clients`.

### Clients

Client-level records describe companies and contacts.

Use `Clients` for:

- Company and contact information.
- Overall relationship context.
- Current open topics with that contact.
- Relationship-maintenance and lineup-request follow-up.

Client records can be created from:

- User-provided company, contact, and communication details.
- A newsletter or email from a new contact when the user asks to add that contact.
- A `Prospects` row after a meaningful human reply, following the promotion rules above.

When creating a new client from email, search available inbox and sent-mail context for the same company or domain if the runtime supports it. Add any other active acquisition, project, or contract matters found for that client so related work is not split across hidden threads.

If one company has multiple contacts, keep the client record compact:

- Put only the main business contact in `业务对接人`.
- Put only that person's email in `业务对接人邮箱`.
- Put legal, finance, material, and notification-only contacts in `备注`.
- In `Projects`, record the main business contact.
- In `Contracts`, record the contacts responsible for the current contract workflow.

Recommended fields:

- `客户ID`
- `公司名`
- `业务对接人`
- `业务对接人邮箱`
- `地区/国家`
- `上次收到片单日期`
- `片单状态`
- `上次联系日期`
- `在谈事宜`
- `下次跟进日期`
- `备注`

Use `上次收到片单日期` and `片单状态` to track lineup health. Suggested `片单状态` values:

- `有片单`
- `待要片单`
- `片单过期`
- `不适用`

### Projects

Project-level records describe acquisition opportunities.

A project may be:

- A single film.
- A package of films.
- A potential acquisition lead before the right customer is known.

Project records can be created from:

- User instructions.
- Email context where a specific title or package is mentioned and the user asks to track it.
- A project-first workflow where Tian wants to identify or contact the right seller later.

Most projects should have a `客户ID`. In rare project-first cases, allow a temporary value such as `待匹配客户` until a client is identified.

Recommended fields:

- `项目ID`
- `客户ID`
- `公司名`
- `业务对接人`
- `片名或片包名`
- `项目状态`
- `上次跟进日期`
- `上次跟进内容`
- `下一步动作`
- `下次跟进日期`
- `备注`

### Contracts

Contract-level records begin after the project has crossed from opportunity into deal execution.

Create a contract record when both sides have confirmed the core commercial terms, such as price, rights, territory, and license term. A contract record must always have both a `客户ID` and a `项目ID`.

Use `Contracts` for:

- Contract-detail negotiation.
- Drafting and signature follow-up.
- Payment follow-up.
- Copyright documents.
- Delivery materials.
- Any post-deal execution matter.

Recommended fields:

- `合同ID`
- `客户ID`
- `项目ID`
- `公司名`
- `业务对接人`
- `合同/项目名称`
- `合同状态`
- `合同文件`
- `付款进度`
- `版权文件`
- `介质/物料`
- `上次跟进日期`
- `上次跟进内容`
- `下一步动作`
- `下次跟进日期`
- `备注`

Use four execution overview fields rather than one column per micro-step:

- `合同文件`: contract terms, signing, and fully signed agreement status.
- `付款进度`: invoice, first payment, censorship-dependent balance payment, and payment status.
- `版权文件`: draft, signed version, notarized/legalized version, and related copyright document status.
- `介质/物料`: master, subtitles, poster, stills, trailer, and other delivery materials.

Suggested compact values:

- `合同文件`: `条款确认中`, `待对方签署`, `已收双方签字版`.
- `付款进度`: `待发票`, `首款待付`, `首款已付`, `尾款待过审`, `尾款已付`.
- `版权文件`: `待起草`, `待对方签署`, `已收签字版`, `待公证版`, `已收公证版`.
- `介质/物料`: `未开始`, `待对方提供`, `已收`, `缺字幕`, `缺海报`.

When a project becomes a contract:

- Add a new row in `Contracts`.
- Keep the original row in `Projects`.
- Mark the project status as `已转合同`.
- Clear the project's `下次跟进日期`.
- Move future execution follow-up to `Contracts`.

After conversion, `Contracts` controls execution follow-up dates. `Clients` controls only relationship-level follow-up, such as seasonal check-ins or broader catalogue conversations, and does not need to mirror the contract follow-up date.

## Status Vocabulary

Use fixed status terms plus concise notes. Prefer stable statuses for filtering and reminders, then put details in `备注`.

Prospect statuses are defined in the `Prospects` section and must not be replaced with free-form synonyms.

Project statuses:

- `待我方回复`
- `等待对方回复`
- `内部评估`
- `待看片`
- `待报价`
- `等待对方确认`
- `已转合同`
- `暂缓`
- `关闭`

Contract statuses:

- `合同条款谈判`
- `等待签署`
- `等待发票`
- `等待首款`
- `等待版权文件`
- `等待过审`
- `等待尾款`
- `等待公证版权文件`
- `等待介质/物料`
- `完成`
- `暂缓`
- `关闭`

If none of these fits, use the closest status and explain the nuance in `备注`.

## Update Rules

When the user provides a new email, reply, or instruction:

1. Identify the relevant prospect, client, project, or contract.
2. Update an existing record when possible; do not create duplicates.
3. Create a new prospect, client, project, or contract only when the user asks or the workflow clearly requires it.
4. Keep the workbook lightweight. Put complex terms, rights, pricing, tax, payment structure, delivery details, risks, censorship concerns, and replacement mechanisms into `备注` instead of creating many new columns.
5. Store date fields as plain text in `YYYY-MM-DD` format, such as `2026-06-16`. Do not write spreadsheet date objects or datetime objects, because they may render as serial numbers, include unwanted time suffixes, or display as `######`.
6. Before appending rows, ignore or compact fully empty rows so new records are added directly after the last row with real data, not after preformatted blank rows.
7. Do not record external contract numbers as CRM identifiers. Use the CRM's own `合同ID` as the unique contract identifier. External contract numbers can remain in filenames or contract folders unless Tian explicitly asks to record them.

### Evidence-First Transaction Rules

Cross-system work must follow the actual evidence order:

1. Read and identify the source email or user instruction.
2. If an attachment matters, confirm its name and type, save it to the explicitly approved destination, and verify the saved path and size.
3. Draft the outbound email and obtain the mailbox tool's required confirmation.
4. Send once, then verify Sent or obtain Tian's explicit confirmation of delivery.
5. Only then write the resulting facts and dates to the canonical CRM workbook.

Do not pre-commit future states. Use precise intermediate states when a workflow is incomplete:

- Attachment listed in email but not saved: `邮件已收到附件，待归档`.
- Attachment saved and verified: `已归档` plus the relevant document description.
- Draft prepared but not sent: `待我方发送`.
- Send confirmation requested but not completed: `待确认发送`.
- SMTP result uncertain: `发送状态待核验`; do not advance the last-contact date.
- Sent verified: record the actual sending date and next action.

For contract documents, do not write `已归档双方签字版` merely because the email says a signed copy is attached. Require the attachment to be saved and verified first. If a step is blocked, report the last verified state and leave later CRM states unchanged unless Tian explicitly asks to record that partial state.

Before every CRM write, run this preflight checklist:

1. If the matter has a contract, write the follow-up date only in `Contracts`.
2. If a project has moved to `Contracts`, `Projects.项目状态` must be `已转合同` and `Projects.下次跟进日期` must be blank.
3. `Clients.下次跟进日期` is only for independent client-level matters, such as relationship maintenance, lineup requests, or other client-level topics. Do not copy project or contract follow-up dates into `Clients`.

These checks are by matter, not by client. One client may have separate active follow-up dates for different matters across `Contracts`, `Projects`, and `Clients`, as long as the same matter is not duplicated across sheets.

If the user only asks for an email draft:

- Do not mark anything as sent.
- If updating is appropriate, mark the next action as `待我方发送` or equivalent.

If the user says the email was sent:

- First verify the message in Sent mail or receive an explicit confirmation from Tian that it was sent.
- Only after confirmation, update the correct owner row as one atomic CRM action: set `上次联系日期` or `上次跟进日期` to the actual sending date, summarize what was sent, set the next action, and set a reasonable `下次跟进日期`.
- If sending is uncertain, do not mark the CRM as sent or advance the last-contact date.
- For bulk prospect outreach, update each confirmed recipient to `已发送待回复`; do not create individual `Clients` or `Projects` rows until the promotion rules are met.

## Spreadsheet Presentation Rules

Keep the workbook visually usable after edits.

- Use plain text `YYYY-MM-DD` for all follow-up and contact dates.
- Remove or ignore fully empty rows before appending new data.
- Adjust obvious narrow columns when the runtime supports it, especially date and long-note fields.

## Header-Driven Spreadsheet Write Rules

Before modifying Excel, read and confirm the header row for the target sheet. Write cells by mapping header names to columns. Do not hand-count columns, and do not overwrite a full row with an unchecked positional array.

Expected 0-based header order for validation:

- `Prospects`: `0=Name`, `1=Country`, `2=Address`, `3=Zip`, `4=City`, `5=State`, `6=Tel`, `7=Fax`, `8=Activities`, `9=Emails`, `10=状态`.
- `Clients`: `0=客户ID`, `1=公司名`, `2=业务对接人`, `3=业务对接人邮箱`, `4=地区/国家`, `5=上次收到片单日期`, `6=片单状态`, `7=上次联系日期`, `8=在谈事宜`, `9=下次跟进日期`, `10=备注`.
- `Projects`: `0=项目ID`, `1=客户ID`, `2=公司名`, `3=业务对接人`, `4=片名或片包名`, `5=项目状态`, `6=上次跟进日期`, `7=上次跟进内容`, `8=下一步动作`, `9=下次跟进日期`, `10=备注`.
- `Contracts`: `0=合同ID`, `1=客户ID`, `2=项目ID`, `3=公司名`, `4=业务对接人`, `5=合同/项目名称`, `6=合同状态`, `7=合同文件`, `8=付款进度`, `9=版权文件`, `10=介质/物料`, `11=上次跟进日期`, `12=上次跟进内容`, `13=下一步动作`, `14=下次跟进日期`, `15=备注`.

After every spreadsheet write, scan the edited rows and any related project or contract rows:

- `Prospects.状态` must use the fixed vocabulary above. A row marked `已转Clients` must have a matching `Clients` record.
- Date fields must be plain text in `YYYY-MM-DD` format.
- `上次跟进内容` must not contain only a date.
- `下一步动作` must contain an action or owner, not the previous follow-up summary.
- `下次跟进日期` must contain only a date or be blank, never action text.
- Follow-up ownership must satisfy the CRM write preflight checklist above.

Reference column widths:

- `Prospects`: `Name` 32, `Country` 14, `Address` 45, `Activities` 46, `Emails` 38, `状态` 18.
- `Clients`: `客户ID` 12, `公司名` 24, `业务对接人` 22, `业务对接人邮箱` 32, `上次收到片单日期` 16, `片单状态` 14, `在谈事宜` 55, `下次跟进日期` 16.
- `Projects`: `片名或片包名` 35, `上次跟进内容` 55, `备注` 50.
- `Contracts`: `合同/项目名称` 35, `合同文件` 24, `付款进度` 24, `版权文件` 24, `介质/物料` 24, `上次跟进内容` 40, `备注` 40.

## Follow-Up Date Rules

The three action dimensions revolve around `下次跟进日期`, but the date means different things by dimension:

- `Clients`: relationship maintenance, lineup requests, and other independent client-level matters.
- `Projects`: active acquisition opportunity follow-up before core commercial terms are confirmed.
- `Contracts`: execution follow-up after core commercial terms are confirmed.

Follow-up ownership is exclusive. One matter must have only one follow-up date source:

1. `Contracts` overrides `Projects` and `Clients`.
2. `Projects` overrides `Clients`.
3. `Clients` follow-up dates are only for independent client-level matters, not copies of project or contract follow-up.

When updating a contract matter, update only the `Contracts` follow-up date. Do not copy the same date into the related `Projects` or `Clients` rows.

When updating an active project matter that has not moved to `Contracts`, update only the `Projects` follow-up date. Do not copy the same date into `Clients`.

When updating client relationship maintenance, lineup requests, or another client-level topic, update `Clients` even if the same client has an active project or contract, provided it is genuinely a different matter. Never duplicate a project or contract reminder in `Clients`.

If duplicate follow-up dates for the same matter are found across sheets, keep the highest-priority owner and clear the lower-priority duplicates:

- Keep `Contracts`; clear related `Projects` and `Clients`.
- If no `Contracts`, keep `Projects`; clear related `Clients`.
- Keep `Clients` only for standalone relationship maintenance or lineup requests.

Default timing:

- Formal offer: follow up in 3-5 business days.
- Gentle reminder: follow up in 2-3 business days.
- Counterparty needs internal confirmation: follow up in 3-5 business days.
- Contract, payment, materials, or copyright documents: follow up in 2-5 business days depending on urgency.

If the next action is internal review, set the date for when Tian should make that decision, not when the counterparty should reply.

If the next action is waiting for the counterparty, set the date for when Tian should consider sending a reminder.

After any CRM write, verify that follow-up ownership is not violated. If a project has moved to `Contracts`, the original `Projects` row must remain for history, have `项目状态 = 已转合同`, and have a blank `下次跟进日期`.

## Daily Follow-Up Check

When Tian asks what needs follow-up today:

1. Check `Prospects`, `Clients`, `Projects`, and `Contracts`.
2. Include all `Prospects` rows with `状态 = 有效回复待处理`, plus dated records where `下次跟进日期 <= today`.
3. Group results into:
   - `需要主动发邮件`
   - `需要内部判断`
   - `等待对方但到期可催`
   - `暂不用跟进`
4. Explain each item with the contact, title or contract, why it is due, and the suggested next step.

Keep client relationship-maintenance reminders separate from active project or contract follow-ups. Keep the answer concise. Mention future non-due items only if they are important context.

## Email Drafting Rules

English acquisition emails should be:

- Short.
- Clear.
- Natural.
- Commercially specific.
- Polite but not weak.

Common patterns:

- `Thanks for your reply.`
- `Just wanted to follow up on...`
- `Would it be possible for you to check...`
- `Please kindly let me know...`
- `If this structure works for you, we can then discuss the remaining contract details.`

When Tian asks for text that can be copied and sent:

- Do not include a subject line.
- Do not include Chinese explanation.
- Output only the email body.
- End with:

```text
Best,
Tian
```

When an approved mailbox tool can send email:

- Draft first.
- Ask for or require user confirmation before sending.
- Extract recipient addresses from the original email's `From` or `Reply-To` fields when replying. Do not infer domains or addresses from memory.
- If one user instruction involves multiple recipients or multiple separate emails, show each draft separately and require confirmation for each one before sending.
- After confirmed and verified sending, update the canonical CRM workbook with the actual send date and next follow-up date.

### Email Sending Safety Rules

When sending through any approved script-driven mail tool:

- Use a send timeout of at least 60 seconds.
- If the command times out, the terminal disconnects, or there is no explicit SMTP failure message, do not retry immediately.
- Before any retry, check Sent mail for a message with the same recipient, same subject, close send time, and substantially similar body.
- If the message may have been sent but cannot be confirmed, report the uncertainty to Tian and ask before sending again.

### Reply Thread Rules

Replies to existing matters must stay in the original email thread by default. Do not create a new thread unless Tian explicitly asks or no thread context is available and Tian confirms sending anyway.

When sending a reply:

1. Prefer the current original email's `Message-ID`, `References`, subject, sender, date, and body.
2. If the current original email is not available, search the inbox for the latest relevant email from the counterparty and use its `Message-ID`.
3. If the inbox has no relevant counterparty email, search Sent mail for Tian's latest relevant message to that counterparty and use its `Message-ID`.
4. Set `In-Reply-To` to the found `Message-ID`.
5. Set `References` to the prior `References` plus the found `Message-ID` when available; otherwise use the found `Message-ID`.
6. Preserve the original subject, including `Re:` when appropriate.
7. Include quoted context at the bottom of the body:

```text
On [date], [name] <[email]> wrote:
> [quoted original text]
```

If no usable `Message-ID` or original text can be found, tell Tian that the reply cannot be safely threaded and ask whether to send a new-thread email.

When Codex cannot send email:

- Draft the message for Tian to copy.
- Update the CRM as sent only after Tian confirms it has been sent.

## Decision Defaults

- Prefer updating existing records over creating new ones.
- Prefer concise summaries over full email copies.
- Prefer notes in `备注` over new columns.
- Keep unresponsive outreach leads in `Prospects`; promote only meaningful human replies to `Clients`.
- Keep signed or executing deals in `Contracts`, not active acquisition `Projects`.
- Keep `Clients` focused on business contacts, lineup status, relationship maintenance, and overall open topics.
- Keep `Projects` focused on active acquisition opportunities before confirmed core commercial terms.
- Keep `Contracts` focused on contract files, payment, copyright documents, and materials.
- Treat the latest email and the CRM workbook as source of truth.
- Ask Tian when a decision would materially change whether a record is created, moved to contracts, or closed.
