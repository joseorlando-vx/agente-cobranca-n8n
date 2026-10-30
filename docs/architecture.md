# Architecture

## Flow

```
[Daily schedule, 8 AM]
        │
        ▼
[Google Sheets: read control sheet]
        │
        ├──► [Filter STATUS = PENDENTE] ──► [Compute reminder dates] ──► [Matches D-5/D-3/D-1/D0?]
        │                                                                          │
        │                                                                          ├─ yes ─► [Route by channel] ─► [WhatsApp and/or Email] ─► [Update sheet]
        │                                                                          └─ no ─► [Past due?] ─► [Set STATUS = ATRASADO]
        │
        └──► [Filter STATUS = ATRASADO] ──► [Compute days overdue]
                                                      │
                                    ┌─────────────────┼─────────────────┬──────────────────┐
                                    ▼                 ▼                 ▼                  ▼
                              [D+3: locked?]     [D+4: locked?]     [D+5: locked?]     [D+7: locked?]
                                    │                 │                 │                  │
                                    ▼                 ▼                 ▼                  ▼
                          [Send + lock]  [Send + lock]  [Send + lock]  [Internal alert]
```

## Components

| Component | Role |
|---|---|
| n8n (Schedule Trigger) | Triggers the run once a day |
| Google Sheets | Single source of truth: status, due date, channel, contact, send history |
| Evolution API | Sends WhatsApp messages |
| Gmail | Sends HTML-formatted emails |

## Expected spreadsheet columns

| Column | Use |
|---|---|
| `PS` | Unique charge identifier (used to locate the row on updates) |
| `STATUS` | `PENDENTE`, `ATRASADO` or any other value (e.g. `PAGO`), which drops out of the filters |
| `VENCIMENTO` | Due date, format `DD/MM/YYYY` |
| `CANAL` | `WHATSAPP`, `E-MAIL` or `AMBOS` (both) |
| `CLIENTE`, `WHATSAPP`, `E-MAIL`, `VALOR (R$)`, `TIPO`, `PARCELA` | Data used in the message body |
| `AVISO ENVIADO`, `MSG_D3`, `MSG_D4`, `MSG_D5`, `MSG_RESPONSAVEL` | Control columns, filled automatically to block re-sends |

## Design decisions

- **Two parallel, independent schedules** (reminders and collections): both run from the same spreadsheet read but do not depend on each other, which keeps the logic of each simple.
- **Lock by empty column, not by counter**: each stage (D+3, D+4, D+5) fires only if its spreadsheet field is still empty. After sending, the field is filled with a timestamp, which naturally prevents re-sends.
- **Automatic status transition**: a PENDENTE row that passes its due date becomes ATRASADO by itself, with no manual step. On the next run it enters the collections schedule.
- **Escalation to a human at D+7**: the system does not try to resolve long-overdue charges on its own; it alerts an internal owner to step in manually.
