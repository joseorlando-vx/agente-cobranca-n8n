# Segurança e Sanitização

Este repositório é uma **versão pública e sanitizada** de uma automação em produção em uma empresa de engenharia predial de médio porte em Brasília. Este documento existe pra deixar explícito o que foi removido e onde as credenciais reais devem ficar.

## O que NÃO está neste repositório (por design)

| Removido | Onde fica de verdade |
|---|---|
| Chave de API da Evolution API | Gerenciador de credenciais do n8n (nunca em arquivo versionado) |
| ID real da planilha de controle financeiro | Substituído por placeholder |
| Nome real da instância WhatsApp | Substituído por placeholder |
| Telefone pessoal usado no alerta interno (D+7) | Substituído por placeholder |
| Nome da empresa e da marca | Substituídos por termos genéricos |
| Textos de mensagem com dados de cliente real | Simplificados para exemplos sem dado sensível |

## Achado corrigido durante a sanitização

A versão original deste workflow tinha um problema real de segurança: a chave da Evolution API estava configurada corretamente como credencial do n8n (`authentication: genericCredentialType`), mas **também** estava colada em texto puro no cabeçalho HTTP de 7 nodes diferentes, como um parâmetro manual redundante. Isso significa que, mesmo com a credencial configurada certa, a chave real continuava exposta em qualquer export do workflow.

Na sanitização, esse parâmetro manual foi removido por completo. A versão publicada aqui depende exclusivamente da credencial do n8n, que é o padrão correto.

**Se você reconhece esse padrão em algum workflow seu**: procure por parâmetros de cabeçalho HTTP com nome tipo `apikey`, `Authorization` ou `token` com o valor colado direto, mesmo em nodes que já têm uma credencial configurada. A presença de uma credencial não garante que o valor solto foi removido.

## Onde ficam as credenciais reais

1. Copie `.env.example` para `.env` (o `.env` já está no `.gitignore`, nunca vai pro Git)
2. No n8n, crie uma **credencial** pra cada serviço: WhatsApp/Evolution API, Gmail, Google Sheets
3. Os nós do workflow referenciam a credencial pelo nome, nunca pelo valor

## Superfície de risco caso este workflow seja reaproveitado

- A planilha de controle contém dados financeiros de clientes (valores, vencimentos, contatos). O acesso a ela deve ser restrito só à conta de serviço usada pelo n8n, nunca compartilhada com link público
- Cada bot/instância de WhatsApp deve ser isolado: não reaproveitar a mesma instância Evolution API pra mais de uma automação, para não cruzar webhooks
- O alerta interno (D+7) hoje usa um número de telefone fixo no código. Recomenda-se mover isso para uma variável de ambiente ou tabela de configuração, para facilitar troca sem editar o workflow

## Reportar um problema

Este é um repositório de portfólio pessoal. Se encontrar algo sensível que passou pela sanitização, abra uma issue ou entre em contato diretamente.
