# Collections Agent (n8n)

An automated payment-reminder and collections workflow. It reads a control spreadsheet, finds pending and overdue clients, and sends reminders by WhatsApp and/or email at specific points of the billing cycle, with no manual work.

## The problem

Following up on pending payments was manual: someone had to review the spreadsheet every day and remember to message the right client at the right time. Reminders were sometimes late or never sent, and the commercial team lost time on it.

## The solution

A workflow that runs once a day, reads the whole spreadsheet and follows two independent, parallel schedules:

1. **Reminders (before the due date):** sent at D-5, D-3, D-1 and on the due date
2. **Collections (after the due date):** escalating messages at D+3, D+4 and D+5, and at D+7 an internal alert to the person in charge, flagging that a human needs to step in

The delivery channel (WhatsApp, email or both) is chosen by a column in the spreadsheet itself.

## Stack

- **n8n**: workflow orchestration, scheduled once a day
- **Google Sheets**: data source and control record (status, dates, send history)
- **WhatsApp (Evolution API)**: text message delivery
- **Gmail**: email delivery with formatted HTML body

## My role

- Designed the full logic of both schedules (reminders and collections) and the automatic status transition (PENDENTE to ATRASADO)
- Implemented the lock that prevents the same message from being sent twice at the same stage
- Set up channel routing (WhatsApp, email or both) driven by a spreadsheet column
- Built the internal escalation alert (D+7) so collections older than a week get human attention

## Repository structure

```
agente-cobranca-n8n/
|-- README.md            this file
|-- SECURITY.md          what was sanitized and where real credentials live
|-- CHANGELOG.md         version history
|-- .env.example         environment variables (no real values)
|-- .gitignore
|-- docs/
|   `-- architecture.md  flow diagram and design decisions
|-- config/
|   `-- README.md        checklist of where to register each credential
`-- workflow/
    `-- agente-cobranca.json   sanitized n8n export
```

## How to run

1. Import `workflow/agente-cobranca.json` into your n8n
2. Copy `.env.example` to `.env` and fill in your own values
3. Configure the WhatsApp (Evolution API) and Gmail credentials in the n8n interface. See `config/README.md` for the checklist
4. Point the spreadsheet-reading node to your own control spreadsheet, with the columns described in `docs/architecture.md`
5. Activate the workflow

## Security

This repository contains a **sanitized** version of the production workflow: no real keys, tokens, phone numbers or emails are included. Details of what was removed and why are in [`SECURITY.md`](./SECURITY.md).

Note: column values such as `PENDENTE` and `ATRASADO` are kept in Portuguese because the workflow filters on them.
