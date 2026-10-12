---
name: auditar-ferramenta
description: Audita uma ferramenta HTTP ou um servidor MCP cadastrado na TwoLabs (configuração visível, agentes que a usam e, só para ferramenta HTTP, taxa de erro) só com ferramentas de leitura do MCP.
when_to_use: Quando perguntarem por que uma ferramenta falha, quem usa uma integração, ou pedirem revisão de ferramentas e servidores MCP antes de mudar algo.
allowed-tools:
  - mcp__plugin_twolabs_twolabs__listar_ferramentas
  - mcp__plugin_twolabs_twolabs__ler_ferramenta
  - mcp__plugin_twolabs_twolabs__execucoes_de_ferramenta
  - mcp__plugin_twolabs_twolabs__listar_servidores_mcp
  - mcp__plugin_twolabs_twolabs__ler_servidor_mcp
  - mcp__plugin_twolabs_twolabs__listar_agentes
---

# Auditar ferramenta

Em TODA chamada, preencha `contexto` com o objetivo numa frase, sem dados pessoais.

## Chave

Usa `tools.read` (ferramentas, servidores MCP e execuções) e `agents.read` (`listar_agentes`), ambas no modelo
**Investigar com IA**.

## Passos

1. Localizar: `listar_ferramentas` (filtre por `agent_id` se souber o agente) ou `listar_servidores_mcp`.
2. Ler: `ler_ferramenta` ou `ler_servidor_mcp`. Você verá host, método, nomes dos parâmetros e de onde
   vem o valor (`origem`), nomes dos cabeçalhos, se usa conexão de autenticação, e os agentes vinculados.
   Valores de cabeçalho, corpo, URL completa, parâmetros fixos e segredos NUNCA vêm na configuração; não
   peça nem tente deduzir. Nas execuções (passo 3) a requisição resumida vem com a URL sem query nem
   credenciais (`?<omitida>`).
3. Comportamento, só para ferramenta HTTP: `execucoes_de_ferramenta` com o `tool_id` e uma janela
   (`desde`, `ate`). Servidor MCP não tem esse passo: as execuções dele são gravadas por `mcp_server_id`
   e a ferramenta só aceita `tool_id`, então para servidor MCP há apenas configuração e agentes
   vinculados, sem taxa de erro. Siga `proximo_inicio` (passando-o como `inicio`) até vir `null` antes de
   agregar; contar só a primeira página subestima os erros. Conte por
   `status` e `erro_classe` (`timeout`, `http_4xx`, `http_5xx`, `outro`). Em agentes com retenção zero
   ou anonimização (ou ligação com alguma perna assim), resumos e texto de erro não vêm: use só as classes.
4. Cruzar: se a taxa de erro se concentra num agente, cite o agente; se em `http_4xx`, suspeite de
   parâmetro ou credencial; se em `timeout`, de latência do destino.

## Relatório

- Veredito: a ferramenta está saudável? (Servidor MCP: só configuração e agentes vinculados.)
- Tabela: status, erro_classe, quantidade, primeira e última ocorrência.
- Agentes afetados.
- O que não dá para afirmar: valores e segredos não são visíveis pelo MCP; mudanças exigem o editor.
