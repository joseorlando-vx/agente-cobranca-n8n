# Credentials Setup

This folder contains no real credentials, only instructions on where to register each one. See also `SECURITY.md` at the project root.

## Setup checklist (in the n8n interface, not in a file)

- [ ] **Evolution API / WhatsApp** credential: instance URL + API key (via an `httpHeaderAuth` credential, never pasted directly into a node)
- [ ] **Gmail** credential: OAuth2
- [ ] **Google Sheets** credential: OAuth2, with access restricted to the control spreadsheet only

## Environment variables (`.env`, outside Git)

See `.env.example` at the root. Copy it to `.env` and fill it in locally.
