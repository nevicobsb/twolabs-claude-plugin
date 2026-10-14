---
name: comparar-versoes-do-agente
description: Compara duas versões publicadas de um agente da TwoLabs (prompt, modelos, voz, vínculos) e o efeito nos indicadores antes e depois, só com ferramentas de leitura do MCP.
when_to_use: Quando perguntarem o que mudou entre versões, se a última publicação piorou o agente, ou pedirem para entender uma regressão depois de publicar.
allowed-tools:
  - mcp__plugin_twolabs_twolabs__listar_versoes_do_agente
  - mcp__plugin_twolabs_twolabs__ler_versao_do_agente
  - mcp__plugin_twolabs_twolabs__ler_agente
  - mcp__plugin_twolabs_twolabs__ler_fluxo
  - mcp__plugin_twolabs_twolabs__metricas_do_agente
---

# Comparar versões do agente

Em TODA chamada, preencha `contexto` com o objetivo numa frase, sem dados pessoais.

## Passos

1. `listar_versoes_do_agente`: versões mais novas primeiro; `no_ar` marca a publicada. Siga
   `proximo_cursor` se precisar de versões antigas. `parcial: true` é snapshot antigo do coach (só prompt).
2. `ler_versao_do_agente` das duas versões: compare `campos` chave a chave e os `vinculos`
   (ids de ferramentas, servidores MCP e documentos). O texto da base de conhecimento não vem; compare
   `tamanho_da_base_de_conhecimento`.
3. Fluxo: `flow_version_id` diz qual Workflow foi congelado; `ler_fluxo` mostra o fluxo atual. Ele
   vem paginado: para o fluxo inteiro, chame com `inicio` 0, siga `proximo_inicio` (passando-o como `inicio`) até vir `null` e
   junte as páginas na ordem; uma página só não permite afirmar que um nó ou aresta saiu.
4. Efeito: `metricas_do_agente` numa janela igual antes e depois de `first_published_at` da versão nova.

## Privacidade

As métricas valem para qualquer agente. Se o agente tem retenção zero ou anonimização, não há
transcrição para exemplificar a mudança: diga isso em vez de buscar texto por outro caminho.

## Relatório

- O que mudou (tabela campo, antes, depois).
- Efeito medido (tabela indicador, antes, depois, cobertura).
- O que não dá para afirmar: correlação de período não é causa; outras mudanças (fluxo, provedor) no
  mesmo intervalo.
