---
name: estado
description: Mostra o estado do CloseAI para este utilizador — licença, ligação ao servidor, fontes de transcrições, última corrida da automação, última análise, perfil e mapa do CRM — e diz o que fazer quando alguma coisa falha.
when_to_use: Quando o utilizador pergunta «o CloseAI está a funcionar?», «a automação correu?», «porque é que esta reunião não foi analisada?», «estado do CloseAI», ou quando outra skill do CloseAI falha sem razão clara.
---

# Estado do CloseAI

Junta o que se sabe, mostra-o numa lista curta e, para cada coisa que falhe, diz o que fazer. Só leitura:
esta skill não grava nada, não analisa nada e não escreve no CRM.

As ferramentas do CloseAI chamam-se `mcp__plugin_closeai_closeai__<nome>`; aqui escrevem-se pelo nome
curto. Fala em português de Portugal.

## O que verificar

1. **Ligação e licença:** `estado_da_licenca` com `versao_do_plugin: "0.1.0"`.
   - Ferramenta inexistente ou a pedir autenticação → «O CloseAI não está ligado. Abre `/mcp`,
     escolhe `plugin:closeai:closeai` e Authenticate (entra com o Google).»
   - `ativa: false` → mostra a `mensagem` do servidor tal como vem. Diz que o que já existe continua
     visível na app.
   - `versao_minima_do_plugin` acima de `0.1.0` → «Atualiza o plugin: `/plugin` › Installed ›
     closeai › Update.»
2. **Fontes e automação:** das `fontes` que vierem em `estado_da_licenca`, para cada uma: a
   ferramenta, onde caem as transcrições, a `ultima_corrida` e a `ultima_analise`.
   - Sem fontes → «Falta registar de onde vêm as transcrições: `/closeai:instalar fonte`.»
   - Última corrida há mais de 2 horas num dia útil (ou nenhuma) → a automação não está a correr.
     Diz para confirmar: o Claude Desktop está aberto; o computador não está a dormir («Keep
     computer awake»); a tarefa `closeai-automacao` existe em Code › Scheduled e está Active; o
     histórico da tarefa não mostra uma corrida parada à espera de uma permissão (se mostrar, abrir
     essa sessão e aprovar com «always allow»). Para a criar: `/closeai:instalar agendar`.
3. **Análises a meio e a última:** das `reunioes_abertas` de `estado_da_licenca`, o total e, para
   cada uma das primeiras, o título e o que falta (partes da transcrição por enviar, ou os passos por
   fazer). Para cada uma, sugere retomá-la com `/closeai:analisar` e o identificador da reunião
   (ficam abertas 24 horas; depois é preciso analisar de novo). A seguir, `listar_analises` com
   `limite: 1`: o título, a data e o estado da última análise fechada.
4. **Perfil e mapa do CRM:** `ler_perfil`: a versão do perfil e se há `mapaDoCrm` (o CRM, o conector
   e quem o confirmou e quando).
   - Sem perfil → `/closeai:instalar perfil`.
   - Sem mapa → `/closeai:instalar crm`.
   - Confirma que o conector do mapa está ligado (só a lista de ferramentas, sem ler dados).

## Como mostrar

Uma linha por ponto, com «certo» ou o problema, e no fim só as ações que faltam, por ordem. Sem
números sobre pessoas: nada de contagens de reuniões ou análises por comercial.
