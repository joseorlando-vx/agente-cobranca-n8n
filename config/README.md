# Configuração de Credenciais

Esta pasta não contém nenhuma credencial real, só instruções de onde cadastrar cada uma. Ver também `SECURITY.md` na raiz do projeto.

## Checklist de configuração (na interface do n8n, não em arquivo)

- [ ] Credencial **Evolution API / WhatsApp**: URL da instância + API key (via credencial `httpHeaderAuth`, nunca colada direto no node)
- [ ] Credencial **Gmail**: OAuth2
- [ ] Credencial **Google Sheets**: OAuth2, com acesso restrito só à planilha de controle

## Variáveis de ambiente (`.env`, fora do Git)

Ver `.env.example` na raiz. Copie para `.env` e preencha localmente.
