# Arquitetura

## Fluxo

```
[Agendamento diário, 8h]
        │
        ▼
[Google Sheets: ler planilha de controle]
        │
        ├──► [Filtrar STATUS = PENDENTE] ──► [Calcular datas de lembrete] ──► [Bate com D-5/D-3/D-1/D0?]
        │                                                                          │
        │                                                                          ├─ sim ─► [Rotear por canal] ─► [WhatsApp e/ou E-mail] ─► [Atualizar planilha]
        │                                                                          └─ não ─► [Já venceu?] ─► [Marcar STATUS = ATRASADO]
        │
        └──► [Filtrar STATUS = ATRASADO] ──► [Calcular dias de atraso]
                                                      │
                                    ┌─────────────────┼─────────────────┬──────────────────┐
                                    ▼                 ▼                 ▼                  ▼
                              [D+3: trava?]     [D+4: trava?]     [D+5: trava?]     [D+7: trava?]
                                    │                 │                 │                  │
                                    ▼                 ▼                 ▼                  ▼
                          [Enviar + travar]  [Enviar + travar]  [Enviar + travar]  [Alerta interno]
```

## Componentes

| Componente | Papel |
|---|---|
| n8n (Schedule Trigger) | Dispara a execução 1x por dia |
| Google Sheets | Fonte de dados única: status, vencimento, canal, contato, histórico de envio |
| Evolution API | Envio de mensagens de WhatsApp |
| Gmail | Envio de e-mails formatados em HTML |

## Colunas esperadas na planilha

| Coluna | Uso |
|---|---|
| `PS` | Identificador único da cobrança (usado para localizar a linha em updates) |
| `STATUS` | `PENDENTE`, `ATRASADO` ou qualquer outro valor (ex: `PAGO`), que some dos filtros |
| `VENCIMENTO` | Data de vencimento, formato `DD/MM/AAAA` |
| `CANAL` | `WHATSAPP`, `E-MAIL` ou `AMBOS` |
| `CLIENTE`, `WHATSAPP`, `E-MAIL`, `VALOR (R$)`, `TIPO`, `PARCELA` | Dados usados no corpo da mensagem |
| `AVISO ENVIADO`, `MSG_D3`, `MSG_D4`, `MSG_D5`, `MSG_RESPONSAVEL` | Colunas de controle, preenchidas automaticamente para travar reenvio |

## Decisões de design

- **Duas réguas paralelas e independentes** (lembrete e cobrança): rodam a partir da mesma leitura da planilha, mas não dependem uma da outra. Isso simplifica a lógica de cada uma.
- **Trava por coluna vazia, não por contador**: cada estágio (D+3, D+4, D+5) só dispara se o campo correspondente na planilha ainda estiver vazio. Depois do envio, o campo é preenchido com data/hora, o que naturalmente impede reenvio.
- **Transição de status automática**: uma linha PENDENTE que passa da data vira ATRASADO sozinha, sem precisar de intervenção manual. Na próxima execução, ela já entra na régua de cobrança.
- **Escalada para atenção humana em D+7**: o sistema não tenta resolver cobranças muito atrasadas sozinho, ele avisa um responsável interno pra intervir manualmente.
