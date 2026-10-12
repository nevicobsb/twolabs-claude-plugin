# Plugin TwoLabs para o Claude Code

Conecta o Claude Code ao servidor MCP da [TwoLabs](https://twolabs.app) e traz quatro skills de leitura para investigar ligações, a saúde de um agente, ferramentas e versões publicadas.

## Instalação

Numa sessão interativa do Claude Code:

```text
/plugin marketplace add nevicobsb/twolabs-claude-plugin
/plugin install twolabs@twolabs
```

Na instalação o Claude Code pede:

- **Endereço do MCP:** aceite o padrão `https://api-twolabs.twolabs.app/mcp` (ou use `https://SEU_HOST/mcp` numa instalação própria).
- **API Key com canal MCP:** crie em **Developers › API Keys** com o modelo **Investigar com IA** (a chave começa com `twr_`). Ela fica no cofre de credenciais do sistema, nunca neste repositório.

Depois, `/mcp` deve listar `plugin:twolabs:twolabs` conectado.

## Skills

| Skill | Para quê |
|---|---|
| `investigar-ligacao` | Veredito, trava e causa de uma ligação. |
| `saude-do-agente` | Volume, sucesso, latência, assuntos, custo e QA num período. |
| `auditar-ferramenta` | Configuração visível, agentes que usam e taxa de erro de uma ferramenta. |
| `comparar-versoes-do-agente` | O que mudou entre duas versões publicadas e o efeito nos indicadores. |

As skills só usam ferramentas de leitura e respeitam a política de privacidade de cada agente (retenção zero e anonimização): quando o texto não é liberado, trabalham só com metadados.
