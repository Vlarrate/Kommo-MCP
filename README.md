# Kommo MCP Server

Servidor [MCP](https://modelcontextprotocol.io) (Model Context Protocol) para integração com o [Kommo CRM](https://pt-developers.kommo.com/docs/kommo-para-desenvolvedores). Expõe tools, resources e prompts para clientes MCP (Cursor, Claude, etc.).

<a href="https://glama.ai/mcp/servers/@Miguelgbastos/Kommo-MCP">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/@Miguelgbastos/Kommo-MCP/badge" alt="Kommo CRM Server MCP server" />
</a>

## Funcionalidades

- **Protocolo MCP v2**: Lifecycle, transporte Streamable HTTP, validação de headers (`MCP-Protocol-Version`, `MCP-Session-Id`)
- **23 Tools**: Leads, contatos, empresas, tarefas, pipelines, notas, relatórios, dashboard, Salesbot, motivos de perda
- **5 Resources**: Relatório de vendas, pipelines, motivos de perda, dashboard, conta
- **4 Prompts**: Templates para análise de vendas, leads, pipelines e motivos de perda
- **ask_kommo**: Interface conversacional em linguagem natural
- **Arquitetura modular**: Código organizado em módulos (`kommo-api`, `mcp/`, `ask-kommo`)
- **Segurança**: Validação de Origin, bind em localhost por default, autenticação opcional

## Pré-requisitos

- Node.js 20+
- Docker (opcional)
- Token de acesso do Kommo (integração privada ou OAuth2)

## Configuração

1. Copie o arquivo de exemplo:
```bash
cp env.example .env
```

2. Configure no `.env`:
```
KOMMO_BASE_URL=https://seu-dominio.kommo.com
KOMMO_ACCESS_TOKEN=seu-token-aqui
```

**Variáveis opcionais:**

| Variável | Descrição | Default |
|----------|-----------|---------|
| `MCP_HOST` | Host de binding | `127.0.0.1` |
| `MCP_ALLOWED_ORIGINS` | Origens permitidas (separadas por vírgula) | — |
| `MCP_AUTH_TOKEN` | Se definido, exige `Authorization: Bearer` ou `X-API-Key` no `/mcp` | — |

## Execução

**Desenvolvimento:**
```bash
npm install
npm run build
npm start
```

**Docker:**
```bash
docker build -t kommo-mcp-server .
docker run -d -p 3001:3001 --name kommo-mcp-server kommo-mcp-server
```

O servidor sobe em `http://127.0.0.1:3001` (ou `MCP_HOST:PORT`).

## Endpoints

- **MCP**: `POST http://localhost:3001/mcp` — JSON-RPC (initialize, tools/list, tools/call, resources/list, resources/read, prompts/list, prompts/get)
- **Health**: `GET http://localhost:3001/health`

## Ferramentas MCP

### Conta e Dashboard
| Tool | Descrição |
|------|-----------|
| `get_account` | Informações da conta Kommo |
| `get_dashboard` | Dados do dashboard |

### Leads
| Tool | Descrição |
|------|-----------|
| `get_leads` | Listar leads (limit, page, query) |
| `get_lead` | Obter lead por ID |
| `create_lead` | Criar lead (name, price, status_id, pipeline_id) |
| `update_lead` | Atualizar lead existente |
| `move_lead` | Mover lead para outro status/pipeline |

### Pipelines e Relatórios
| Tool | Descrição |
|------|-----------|
| `get_pipelines` | Listar pipelines (com status opcional por pipeline_id) |
| `get_sales_report` | Relatório de vendas (dateFrom, dateTo) |

### Contatos, Empresas e Tarefas
| Tool | Descrição |
|------|-----------|
| `get_contacts` | Listar contatos |
| `get_companies` | Listar empresas |
| `get_tasks` | Listar tarefas |
| `create_task` | Criar tarefa vinculada a entidade |
| `get_users` | Listar usuários da conta |

### Notas
| Tool | Descrição |
|------|-----------|
| `get_notes` | Listar notas de lead/contato/empresa |
| `add_note` | Adicionar nota de texto |
| `pin_note` | Fixar nota |
| `unpin_note` | Desafixar nota |

### Motivos de Perda e Salesbot
| Tool | Descrição |
|------|-----------|
| `get_loss_reasons` | Listar motivos da perda de leads |
| `get_loss_reason` | Obter motivo de perda por ID |
| `run_salesbot` | Iniciar Salesbot |
| `stop_salesbot` | Parar Salesbot |

### IA Conversacional
| Tool | Descrição |
|------|-----------|
| `ask_kommo` | Perguntas em linguagem natural sobre o CRM |

## Resources

| URI | Descrição |
|-----|-----------|
| `kommo://reports/sales` | Relatório de vendas (último mês) |
| `kommo://pipelines` | Lista de pipelines |
| `kommo://loss_reasons` | Motivos da perda de leads |
| `kommo://dashboard` | Dados do dashboard |
| `kommo://account` | Informações da conta |

## Prompts

| Nome | Descrição |
|------|-----------|
| `analisar_vendas_mes` | Analisar vendas do mês |
| `resumo_leads_status` | Resumo de leads por status |
| `analise_pipeline` | Analisar performance de pipeline |
| `motivos_perda` | Analisar motivos de perda |

## Estrutura do Projeto

```
src/
├── kommo-api.ts          # Cliente da API Kommo
├── ask-kommo.ts          # Lógica conversacional ask_kommo
├── http-streamable.ts    # Servidor MCP HTTP
└── mcp/
    ├── types.ts          # Tipos MCP
    ├── tool-definitions.ts  # Schemas das tools
    ├── tool-handlers.ts  # Execução das tools
    ├── resources.ts      # Resources MCP
    └── prompts.ts        # Prompts MCP
```

## Exemplos de uso

**1. Inicializar sessão MCP:**
```bash
curl -X POST http://localhost:3001/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"cli","version":"1.0.0"}}}'
```

**2. Listar ferramentas:**
```bash
curl -X POST http://localhost:3001/mcp \
  -H "Content-Type: application/json" \
  -H "MCP-Protocol-Version: 2025-06-18" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'
```

**3. Mover lead para outro status:**
```bash
curl -X POST http://localhost:3001/mcp \
  -H "Content-Type: application/json" \
  -H "MCP-Protocol-Version: 2025-06-18" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"move_lead","arguments":{"lead_id":12345,"status_id":142}}}'
```

**4. Adicionar nota a um lead:**
```bash
curl -X POST http://localhost:3001/mcp \
  -H "Content-Type: application/json" \
  -H "MCP-Protocol-Version: 2025-06-18" \
  -d '{"jsonrpc":"2.0","id":4,"method":"tools/call","params":{"name":"add_note","arguments":{"entity_type":"leads","entity_id":12345,"text":"Cliente interessado no plano premium"}}}'
```

## Documentação

- [docs/MCP_EVOLUCAO.md](docs/MCP_EVOLUCAO.md) — Plano de evolução e conformidade MCP
- [docs/KOMMO_API_EVOLUCAO.md](docs/KOMMO_API_EVOLUCAO.md) — Evoluções da API Kommo
- [Kommo para desenvolvedores](https://pt-developers.kommo.com/docs/kommo-para-desenvolvedores)
- [Changelog Kommo](https://pt-developers.kommo.com/changelog)

## Changelog

### v2.0.0
- Arquitetura modular (separação em `mcp/`, `ask-kommo.ts`)
- 11 novas tools: `get_account`, `get_lead`, `update_lead`, `move_lead`, `get_pipelines`, `get_dashboard`, `create_task`, `get_users`, `get_loss_reason`, `get_notes`, `add_note`
- 2 novos resources: `kommo://dashboard`, `kommo://account`
- 2 novos prompts: `analise_pipeline`, `motivos_perda`
- API de notas: listar e criar notas
- Remoção de valores hardcoded e código duplicado
- `env.example` sanitizado (sem tokens reais)

## Licença

MIT
