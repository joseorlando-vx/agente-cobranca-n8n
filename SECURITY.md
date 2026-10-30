# Security and Sanitization

This repository is a **public, sanitized version** of an automation that runs in production at a mid-sized building-engineering company in Brasilia, Brazil. This document states what was removed and where real credentials should live.

## What is NOT in this repository (by design)

| Removed | Where it really lives |
|---|---|
| Evolution API key | n8n credential manager (never in a versioned file) |
| Real ID of the finance control spreadsheet | Replaced by a placeholder |
| Real WhatsApp instance name | Replaced by a placeholder |
| Personal phone number used by the internal alert (D+7) | Replaced by a placeholder |
| Company and brand names | Replaced by generic terms |
| Message texts containing real client data | Simplified to examples with no sensitive data |

## Finding fixed during sanitization

The original version of this workflow had a real security problem: the Evolution API key was correctly set up as an n8n credential (`authentication: genericCredentialType`), but it was **also** pasted in plain text into the HTTP header of 7 different nodes as a redundant manual parameter. That means that even with the credential configured correctly, the real key stayed exposed in any export of the workflow.

During sanitization that manual parameter was removed entirely. The version published here depends only on the n8n credential, which is the correct pattern.

**If you recognize this pattern in one of your own workflows**: look for HTTP header parameters named like `apikey`, `Authorization` or `token` with the value pasted directly, even in nodes that already have a credential configured. Having a credential does not guarantee that the loose value was removed.

## Where real credentials go

1. Copy `.env.example` to `.env` (`.env` is already in `.gitignore` and never goes to Git)
2. In n8n, create a **credential** for each service: WhatsApp/Evolution API, Gmail, Google Sheets
3. Workflow nodes reference the credential by name, never by value

## Risk surface if this workflow is reused

- The control spreadsheet holds clients' financial data (amounts, due dates, contacts). Access should be restricted to the service account used by n8n and never shared by public link
- Each WhatsApp bot/instance should be isolated: do not reuse the same Evolution API instance for more than one automation, to avoid crossing webhooks
- The internal alert (D+7) currently uses a fixed phone number in the code. Moving it to an environment variable or a configuration table is recommended, so it can be changed without editing the workflow

## Reporting a problem

This is a personal portfolio repository. If you find something sensitive that slipped through sanitization, please open an issue or contact me directly.
