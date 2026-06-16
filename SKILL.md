---
name: film-acquisition-crm
description: "Personal workflow for maintaining Tian's film acquisition CRM across Clients, Projects, and Contracts. Use when updating film-buying contacts, projects, contract follow-ups, next follow-up dates, or drafting acquisition emails from user instructions or email context. This skill defines workflow rules only; it does not contain live CRM data, fixed file paths, email transport code, or spreadsheet scripts."
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

## Operating Boundaries

- Do not hardcode the CRM workbook path. The user or runtime environment must provide it.
- Do not include or assume a specific email access method.
- Hermes may read and send email through IMAP/SMTP when available and only after user confirmation.
- Codex should draft email text for the user to copy and send unless the user has provided another safe sending tool.
- Do not add scripts, automations, or transport-specific logic as part of this v1 workflow.
- Do not expose internal tool details in user-facing responses unless the user asks.

## CRM Dimensions

The workbook has three core tabs: `Clients`, `Projects`, and `Contracts`.

### Clients

Client-level records describe companies and contacts.

Use `Clients` for:

- Company and contact information.
- Overall relationship context.
- Current open topics with that contact.
- Contact-level next follow-up date.

Client records can be created from:

- User-provided company, contact, and communication details.
- A newsletter or email from a new contact when the user asks to add that contact.

Recommended fields:

- `客户ID`
- `公司名`
- `联系人`
- `职位`
- `邮箱`
- `地区/国家`
- `主要片库/类型`
- `优先级`
- `上次联系日期`
- `在谈事宜`
- `下次跟进日期`
- `备注`

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
- `联系人`
- `片名或片包名`
- `当前阶段`
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
- `联系人`
- `合同/项目名称`
- `当前状态`
- `上次跟进日期`
- `上次跟进内容`
- `下一步动作`
- `下次跟进日期`
- `备注`

When a project becomes a contract:

- Add a new row in `Contracts`.
- Keep the original row in `Projects`.
- Mark the project status as `已转合同`.
- Move future execution follow-up to `Contracts`.

## Status Vocabulary

Use fixed status terms plus concise notes. Prefer stable statuses for filtering and reminders, then put details in `备注`.

Client and project statuses:

- `待我方回复`
- `等待对方回复`
- `内部评估`
- `待看片`
- `待报价`
- `等待对方确认`
- `已转合同`
- `暂缓`

Contract statuses:

- `合同条款谈判`
- `等待合同草稿`
- `等待签署`
- `等待付款`
- `等待版权文件`
- `等待物料`
- `交付跟进`
- `已完成`
- `暂缓`

If none of these fits, use the closest status and explain the nuance in `备注`.

## Update Rules

When the user provides a new email, reply, or instruction:

1. Identify the relevant client, project, or contract.
2. Update an existing record when possible; do not create duplicates.
3. Create a new client, project, or contract only when the user asks or the workflow clearly requires it.
4. Keep the workbook lightweight. Put complex terms, rights, pricing, tax, payment structure, delivery details, risks, censorship concerns, and replacement mechanisms into `备注` instead of creating many new columns.
5. Use real date values or clear ISO-style dates such as `2026-06-16`. Avoid spreadsheet serial-number displays.

If the user only asks for an email draft:

- Do not mark anything as sent.
- If updating is appropriate, mark the next action as `待我方发送` or equivalent.

If the user says the email was sent:

- Update `上次联系日期` or `上次跟进日期` to the actual sending date.
- Summarize the sent email in `上次跟进内容`.
- Set `下一步动作` to the expected next action.
- Set a reasonable `下次跟进日期`.

## Follow-Up Date Rules

All three dimensions revolve around `下次跟进日期`.

Default timing:

- Formal offer: follow up in 3-5 business days.
- Gentle reminder: follow up in 2-3 business days.
- Counterparty needs internal confirmation: follow up in 3-5 business days.
- Contract, payment, materials, or copyright documents: follow up in 2-5 business days depending on urgency.

If the next action is internal review, set the date for when Tian should make that decision, not when the counterparty should reply.

If the next action is waiting for the counterparty, set the date for when Tian should consider sending a reminder.

## Daily Follow-Up Check

When Tian asks what needs follow-up today:

1. Check `Clients`, `Projects`, and `Contracts`.
2. Select records where `下次跟进日期 <= today`.
3. Group results into:
   - `需要主动发邮件`
   - `需要内部判断`
   - `等待对方但到期可催`
   - `暂不用跟进`
4. Explain each item with the contact, title or contract, why it is due, and the suggested next step.

Keep the answer concise. Mention future non-due items only if they are important context.

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

When Hermes can send email:

- Draft first.
- Ask for or require user confirmation before sending.
- After confirmed sending, update the CRM with the actual send date and next follow-up date.

When Codex cannot send email:

- Draft the message for Tian to copy.
- Update the CRM as sent only after Tian confirms it has been sent.

## Decision Defaults

- Prefer updating existing records over creating new ones.
- Prefer concise summaries over full email copies.
- Prefer notes in `备注` over new columns.
- Keep signed/executing deals in `Contracts`, not active acquisition `Projects`.
- Treat the latest email and the CRM workbook as source of truth.
- Ask Tian when a decision would materially change whether a record is created, moved to contracts, or closed.
