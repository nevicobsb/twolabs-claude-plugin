# TwoLabs: plugin para Claude Code, Codex e Cursor

Conecta o assistente de código ao MCP da TwoLabs (`https://api-twolabs.twolabs.app/mcp`) e traz dez skills. A versão atual é a 0.3.0.

Leitura (chave do modelo **Investigar com IA**): `investigar-ligacao`, `saude-do-agente`, `auditar-ferramenta`
e `comparar-versoes-do-agente`.

Escrita (chave separada, sempre com confirmação antes de publicar): `editar-agente`, `editar-fluxo`,
`manter-base-de-conhecimento`, `manter-ferramenta-http`, `conectar-servidor-mcp` e `rodar-testes-de-qa`.

O Claude Code pede a sua permissão a cada uso de `publicar_agente`, de `rodar_testes_de_qa` (gasta LLM e pode
acionar integrações reais) e de `criar_servidor_mcp` (conecta num servidor externo): as skills não as
pré-aprovam, e o texto delas manda confirmar com você antes.

## Antes de começar

1. Seja owner ou admin da organização (só eles criam API Keys) e deixe o MCP ligado em
   **Developers › MCP da TwoLabs › Controles da organização**.
2. Em **Developers › API Keys**, crie uma chave pelo modelo **Investigar com IA** (restrita, canal MCP,
   leitura: `agents.read`, `calls.read`, `tools.read`, `knowledge.read`) e copie a chave `twr_...`. Ela
   aparece uma única vez.
   As skills de escrita pedem uma chave separada, só com o que usam: `agents.write` (editar-agente,
   editar-fluxo, criação de testes), `tools.manage` (manter-ferramenta-http, conectar-servidor-mcp),
   `knowledge.manage` (manter-base-de-conhecimento) e `calls.manage` (rodar-testes-de-qa).
3. A chave nunca fica em arquivo do plugin: cada cliente a lê de uma configuração segura dele.

## Claude Code

```text
/plugin marketplace add nevicobsb/twolabs-claude-plugin
/plugin install twolabs@twolabs
```

O Claude Code pede só a chave (guardada no cofre de credenciais do sistema). Rode `/reload-plugins` se pedido
e confira com `/mcp` (`plugin:twolabs:twolabs`). Trocar a chave: `/plugin configure twolabs@twolabs`.

Alternativa sem plugin (sem skills; serve também para instalação própria, trocando a URL):

```bash
claude mcp add --transport http twolabs https://api-twolabs.twolabs.app/mcp --header "X-API-Key: SUA_CHAVE"
```

## Codex

O Codex lê a chave da variável de ambiente `TWOLABS_API_KEY` (cabeçalho `X-API-Key` via `env_http_headers`).
Defina a variável no ambiente em que o Codex roda e instale:

```bash
export TWOLABS_API_KEY=twr_...
codex plugin marketplace add nevicobsb/twolabs-claude-plugin
codex plugin add twolabs@twolabs
```

O Codex usa o mesmo `marketplace.json` do Claude Code e carrega `.codex-plugin/plugin.json`
(skills em `skills/`, servidor em `codex.mcp.json`).

## Cursor

O manifesto é `.cursor-plugin/plugin.json`: skills em `skills/`, servidor em `cursor.mcp.json` e a chave
declarada como variável obrigatória `TWOLABS_API_KEY`, que o Cursor pede em **Plugins › Configure** e
injeta no cabeçalho `X-API-Key`. Instale o plugin a partir deste repositório (marketplace de equipe ou
plugin local, conforme a doc do Cursor).

Alternativa sem plugin: em `~/.cursor/mcp.json`,

```json
{ "mcpServers": { "twolabs": { "url": "https://api-twolabs.twolabs.app/mcp",
  "headers": { "X-API-Key": "${env:TWOLABS_API_KEY}" } } } }
```

## Validação

- Claude Code: `claude plugin validate . --strict` passa.
- Codex: o plugin foi instalado com o Codex CLI 0.155.1 (`codex plugin marketplace add` e
  `codex plugin add`) e `codex mcp get twolabs` mostrou `env_http_headers: X-API-Key=TWOLABS_API_KEY`.
- Cursor: manifesto conferido contra a documentação (cursor.com/docs/reference/plugins); não foi
  instalado no Cursor real.

## Problemas comuns

`401`: sem chave ou com a credencial do editor (`twls_`). `403`: chave inválida, vencida, revogada, sem
canal MCP, IP fora da lista, ou MCP desligado na organização. "Esta credencial não tem a permissão ...":
falta a permissão da ferramenta na chave; o texto diz qual, e um administrador a inclui na chave pela tela
API Keys (no OAuth, a permissão vem dos grupos da pessoa). Abra uma sessão nova depois de editar a chave.

`413`: o corpo da chamada passou do limite; mande menos dados por chamada (texto ou fluxo em partes
menores). `429`: limite de chamadas ou de escritas por minuto; a mensagem traz em quantos segundos tentar
de novo, então espere esse tempo em vez de repetir de imediato. "Parâmetro desconhecido": o nome do
argumento não existe na ferramenta; a mensagem lista os aceitos (por exemplo `limite`, não `limit`), e
valor fora do limite ou da lista de opções é recusado com o campo e o motivo. `403` com "Origem não
permitida": a chamada veio de um navegador com um cabeçalho `Origin` que o servidor não aceita; clientes
de linha de comando não enviam esse cabeçalho.
