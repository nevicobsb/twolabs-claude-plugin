---
name: investigar-ligacao
description: Investiga uma ligação de voz da TwoLabs só com ferramentas de leitura do MCP e entrega veredito, trava, causa e o que não dá para afirmar.
when_to_use: Quando pedirem para analisar ou revisar uma chamada, passarem um session_id, ou perguntarem o que houve numa ligação. Também quando só houver um trecho do que o cidadão disse.
allowed-tools:
  - mcp__plugin_twolabs_twolabs__buscar_em_transcricoes
  - mcp__plugin_twolabs_twolabs__listar_sessoes
  - mcp__plugin_twolabs_twolabs__ler_sessao
  - mcp__plugin_twolabs_twolabs__ler_rastro_de_decisao
  - mcp__plugin_twolabs_twolabs__ler_agente
  - mcp__plugin_twolabs_twolabs__ler_fluxo
  - mcp__plugin_twolabs_twolabs__listar_ferramentas
  - mcp__plugin_twolabs_twolabs__execucoes_de_ferramenta
---

# Investigar uma ligação

Tudo aqui é leitura. Em TODA chamada de ferramenta, preencha `contexto` com o objetivo numa frase,
sem dados pessoais (por exemplo: "Entender por que a ligação não transferiu"). Ele aparece no
registro de atividade da organização.

## 1. Achar a ligação

- Com `session_id`: vá direto ao passo 2.
- Com um trecho de fala: `buscar_em_transcricoes` (mínimo 3 caracteres). Ela só encontra ligações de
  agentes sem retenção zero e sem anonimização; não achar não prova que a ligação não existe.
- Com agente e período: `listar_sessoes` com `agent_id`, `desde`, `ate`; siga `proximo_cursor`.

## 2. Ler o retrato

1. `ler_sessao`: metadados, `visited_nodes`, `voice_quality`, `politica`, `transcricao` em fatias
   (repita com `inicio = transcricao.proximo_inicio` até vir `null`) e o resumo pós-chamada.
2. `ler_rastro_de_decisao`: turnos com `decisao` (critério, evidência e resultado por aresta), 20 por
   página; siga `proximo_inicio`.
3. `ler_agente` e `ler_fluxo` do `agent_id` da sessão: prompt, modelos e as arestas do fluxo. `ler_fluxo`
   vem paginado (nós e arestas em páginas): siga `proximo_inicio` (passando-o como `inicio`) até vir `null`
   antes de concluir que uma aresta ou um nó não existe; a primeira página não é o fluxo inteiro.
4. Se houve ferramenta: `listar_ferramentas` com o `agent_id` para achar o `tool_id`, depois
   `execucoes_de_ferramenta` com `tool_id`, `agent_id` e a janela da ligação.

## 3. Privacidade (não contornar)

Se `politica.texto_liberado` for falso (retenção zero ou anonimização), não há transcrição, rastro nem
resumo. Trabalhe só com metadados, diga isso no relatório e NÃO tente reconstruir o texto por outras
ferramentas (busca, tópicos, execuções). O mesmo vale para perna de transferência: cada perna segue a
política do próprio agente.

## 4. A leitura que importa

- Conte os turnos em que nenhuma aresta casou e veja em que nó se concentram: muitos no mesmo nó
  indicam arestas que não cobrem o que o cidadão diz.
- Leia o que o cidadão disse imediatamente antes de cada um: costuma ser a correção mais barata.
- Ligação longa com percurso curto = travou. `ttfb_llm_ms` acima de ~1500 ms soa como demora.
- Nó com várias arestas que dependem todas da mesma pré-condição é beco sem saída.
- O resumo pós-chamada é gerado por LLM: descreve, não diagnostica.

## 5. Relatório

1. Veredito em uma frase: a ligação atingiu o objetivo?
2. A trava: onde e por quê, com horário e a frase (se o texto estiver liberado).
3. Causa: desenho do fluxo, motor ou provedor. Diga qual.
4. O resto, em tabela, com evidência.
5. O que não dá para afirmar: áudio não ouvido, texto bloqueado pela política, uma ligação só.
