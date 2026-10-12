---
name: saude-do-agente
description: Resume a saúde de um agente da TwoLabs num período (volume, sucesso, latência, assuntos, custo de LLM e QA) só com ferramentas de leitura do MCP.
when_to_use: Quando perguntarem como um agente está indo, pedirem um panorama semanal ou mensal, ou quiserem saber onde o agente perde dinheiro ou qualidade.
allowed-tools:
  - mcp__plugin_twolabs_twolabs__listar_agentes
  - mcp__plugin_twolabs_twolabs__metricas_do_agente
  - mcp__plugin_twolabs_twolabs__topicos_do_agente
  - mcp__plugin_twolabs_twolabs__custo_de_llm_do_agente
  - mcp__plugin_twolabs_twolabs__resultados_de_qa
  - mcp__plugin_twolabs_twolabs__listar_sessoes
---

# Saúde do agente

Em TODA chamada, preencha `contexto` com o objetivo numa frase, sem dados pessoais.

## Passos

1. Sem `agent_id`: `listar_agentes` e confirme com quem pediu qual agente.
2. Janela: padrão dos últimos 7 dias (máximo 90), datas ISO 8601.
3. `metricas_do_agente` com `granularidade` `1d` (ou `1sem` para 90 dias): conversas, sucesso,
   duração, custo e tempo de resposta, com `cobertura` (quantas sessões tinham cada dado).
4. `topicos_do_agente`: assuntos mais frequentes com sentimento e taxa de sucesso. Agente com retenção
   zero ou anonimização devolve lista vazia com `aviso`: diga isso e não infira assuntos por outro meio.
5. `custo_de_llm_do_agente`: custo do período, por conversa e por minuto, e a estimativa pela tabela do
   provedor. A `nota` da resposta explica que os dois são estimativas.
6. `resultados_de_qa`: últimas tentativas de teste e veredito. `habilitado: false` quer dizer que a
   organização não usa QA v2; não é falha.
7. Para exemplos concretos de falha: `listar_sessoes` com `desfecho: "falha"` (só agentes com texto liberado).

## Relatório

- Uma frase de veredito (melhorou, piorou, estável) com o número que sustenta.
- Tabela: indicador, valor, cobertura.
- Até três pontos de atenção, cada um com a ferramenta e o número de onde saiu.
- O que não dá para afirmar: cobertura baixa, período curto, texto bloqueado por retenção zero ou anonimização.
