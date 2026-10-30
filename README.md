# Agente de Cobrança

Régua automática de cobrança que lê uma planilha de controle, identifica clientes pendentes e atrasados, e envia lembretes por WhatsApp e/ou e-mail em pontos específicos do ciclo, sem intervenção manual.

## O problema

O acompanhamento de pagamentos pendentes era manual: alguém precisava revisar a planilha todo dia e lembrar de mandar mensagem pro cliente certo, na hora certa. Isso gerava inconsistência (às vezes o lembrete atrasava ou não saía) e consumia tempo do time comercial.

## A solução

Um workflow que roda uma vez por dia, lê a planilha inteira e segue duas réguas independentes e paralelas:

1. **Lembrete (antes do vencimento):** dispara em D-5, D-3, D-1 e no dia do vencimento
2. **Cobrança (depois do vencimento):** escala em D+3, D+4, D+5, e em D+7 manda um alerta interno pro responsável, sinalizando que precisa de atenção humana

O canal de envio (WhatsApp, e-mail ou os dois) é decidido por uma coluna na própria planilha.

## Stack

- **n8n**: orquestração do workflow, agendado para rodar 1x por dia
- **Google Sheets**: fonte de dados e registro de controle (status, datas, histórico de envio)
- **WhatsApp (Evolution API)**: canal de envio via mensagem de texto
- **Gmail**: canal de envio por e-mail, com corpo HTML formatado

## Meu papel

- Desenhei a lógica completa das duas réguas (lembrete e cobrança) e a transição automática de status (PENDENTE → ATRASADO)
- Implementei o mecanismo de trava que impede reenvio duplicado da mesma mensagem no mesmo estágio
- Configurei o roteamento por canal (WhatsApp, e-mail ou ambos) a partir de uma coluna da planilha
- Criei o alerta interno de escalada (D+7) para dar visibilidade a cobranças que passaram de uma semana sem resposta

## Estrutura do repositório

```
agente-cobranca-n8n/
├── README.md            → este arquivo
├── SECURITY.md           → o que foi sanitizado e onde ficam as credenciais reais
├── CHANGELOG.md          → histórico de versões
├── .env.example          → variáveis de ambiente (sem valores reais)
├── .gitignore
├── docs/
│   └── architecture.md   → diagrama do fluxo e decisões de design
├── config/
│   └── README.md         → checklist de onde cadastrar cada credencial
└── workflow/
    └── agente-cobranca.json   → export sanitizado do n8n
```

## Como rodar

1. Importe `workflow/agente-cobranca.json` no seu n8n
2. Copie `.env.example` para `.env` e preencha com seus dados
3. Configure as credenciais de WhatsApp (Evolution API) e Gmail na interface do n8n. Ver `config/README.md` para o checklist
4. Aponte o node de leitura da planilha para sua própria planilha de controle, com as colunas descritas em `docs/architecture.md`
5. Ative o workflow

## Segurança

Este repositório contém uma versão **sanitizada** do workflow em produção: nenhuma chave, token, telefone ou e-mail real está aqui. Detalhes completos do que foi removido e por quê estão em [`SECURITY.md`](./SECURITY.md).
