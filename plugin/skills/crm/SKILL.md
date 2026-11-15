---
name: crm
description: Escreve no CRM do utilizador o que o CloseAI propõe depois de uma reunião analisada (nota da reunião, tarefas combinadas e os campos que o mapa cobrir), pelo conector do CRM e usando o mapa do CRM guardado no perfil. Mostra primeiro o que vai escrever e onde, e só escreve o que o utilizador aceitar nesta sessão. Trata também as propostas que a automação deixou pendentes.
when_to_use: Quando o utilizador diz «atualiza o CRM», «mete isto no ClickUp», «escreve a reunião no CRM», «o que ficou pendente para o CRM», depois de /closeai:analisar, ou quando abre uma sessão e há análises da automação com proposta ao CRM por tratar.
argument-hint: "[análise]"
---

# Escrever no CRM o que o CloseAI propõe

O CloseAI não escreve em CRM nenhum: diz **o que** escrever. Tu traduzes isso para o CRM do
utilizador, pelas ferramentas do conector dele, usando o **mapa do CRM** guardado no perfil. **Nunca
escreves nada sem um sim explícito do utilizador nesta sessão** (um sim de outra sessão, ou da
automação, não conta).

As ferramentas do CloseAI chamam-se `mcp__plugin_closeai_closeai__<nome>`; aqui escrevem-se pelo nome
curto. Fala em português de Portugal.

Que análise: $ARGUMENTS

## 1. Que análise

- Se te deram uma análise (ou acabaste de correr `/closeai:analisar`), usa essa.
- Se não, chama `listar_analises` e mostra as mais recentes, com o título e a data, e pergunta qual.
  Se a lista marcar análises com a proposta ao CRM pendente (as da automação), propõe essas primeiro.

## 2. A proposta e o mapa

1. Chama `proposta_para_crm` com `id` (o identificador da análise). Não nomeia nenhum CRM e traz:
   - `nota`: `titulo` (o título da reunião) e `corpo` (o resumo, o que se ficou a saber, o combinado,
     as objeções e o que falta saber), com `chave_do_mapa: "notas"`;
   - `campos_do_mapa`: a lista plana do que escrever, cada um com `chave_do_mapa` (`etapa`,
     `proximoPasso`, `data`, `valor`, `decisor`, `prazo`, `dor`, `solucaoAtual`, `patamar`,
     `tipoDeReuniao`, `notas`), `rotulo`, `valor` (texto, ou `null`), `estado` e `origem`;
   - `proximo_passo` (`estado` `combinado` ou `nao_se_falou`, `o_que`, `quem` como papel, `quando`,
     `data`) e `data_do_proximo_contacto` (`data` em AAAA-MM-DD ou `null`, `como_foi_dita`);
   - `qualificacao` (orçamento, decisor, prazo, dor, solução atual, cada um com `estado`:
     `sabe_se`, `parcial`, `nao_se_falou` ou `nao_avaliado`) e `objecoes`: servem para mostrar ao
     utilizador antes de confirmar;
   - `etapa_sugerida`: `null` ou uma **sugestão** de etapa (`etapa`, `etapa_no_crm`, `razao`);
   - `falta_saber`, `campos` (`tipo_de_reuniao`, `patamar_do_negocio`, `analisada_em`) e `tarefas`
     (o que ficou combinado, cada uma com `titulo`, `dataLimite` ou vazia, e `contexto`);
   - `comoUsar`: as regras do próprio CloseAI para esta proposta. Segue-as.

   Um `valor` a `null` não se escreve, e não o deduzes do resumo nem da conversa. Empresa, pessoa e
   responsável não vêm na proposta: não os proponhas.
2. Chama `ler_perfil` e lê o `mapaDoCrm`.
   - **Sem `mapaDoCrm`:** não adivinhes onde escrever. Diz ao utilizador para correr
     `/closeai:instalar crm`, que descobre e confirma o mapa, e para aqui.
   - Confirma que o conector indicado no mapa (`conector`) está ligado e responde (uma leitura
     pequena). Se não estiver, diz qual falta e para.
3. **Encontra o negócio**, só com leituras: procura pelo **título da reunião** (`nota.titulo`, ou
   palavras dele, como o nome da empresa se lá estiver) no sítio que `entidades.negocio` indica (o
   `onde` e os `identificadores`).
   - No ClickUp: procura tarefas na lista do negócio (`entidades.negocio.identificadores`) com
     palavras do título; não procures no espaço de trabalho inteiro.
   - Mostra os candidatos (nome e etapa atual, com ligação se o conector a der) e **pergunta qual
     é**. Mesmo que haja só um, pergunta: nunca escolhes sozinho.
   - Se não houver nenhum, pergunta se é para **criar** o negócio (com o título da reunião como
     nome, e nada mais que não venha da proposta) ou para **saltar** as escritas que precisam dele.
     Sem resposta, não crias.

## 3. Traduzir cada operação

Para cada parte da proposta, decide com o mapa **que ferramenta do conector** a escreve e **onde**:

- **Nota da reunião** (`nota`) → `entidades.nota` do mapa. No ClickUp, normalmente um comentário na
  tarefa do negócio, com o `titulo` na primeira linha e o `corpo` a seguir. Se o mapa tiver também
  `campos.notas`, pergunta qual dos dois sítios o utilizador quer; não escrevas nos dois.
- **Tarefas** (`tarefas`) → `entidades.tarefa` do mapa (no ClickUp, subtarefas da tarefa do negócio,
  ou tarefas numa lista de seguimento, como a `ligacao` disser), com o `titulo`, a `dataLimite` como
  data de entrega quando vier, e o `contexto` na descrição. Se o mapa não tiver `entidades.tarefa` mas
  tiver `campos.tarefas`, escreve-as nesse campo, uma por linha. Sem responsável, a não ser que o
  utilizador o diga agora.
- **Campos** (`campos_do_mapa`), um a um, só os que têm `valor` e só se o mapa tiver
  `mapaDoCrm.campos[chave_do_mapa]`:
  - Com sítio, usa o `id` e o `tipo` que o mapa guarda (no ClickUp, um campo personalizado da lista do
    negócio). Num campo de **data**, usa `data_do_proximo_contacto.data` (ou a `data` do
    `proximo_passo`) em milissegundos; se for `null`, não escrevas a data e deixa o `como_foi_dita`
    na nota. Num campo de **opções**, procura a opção pelo texto e mostra-a; sem opção certa, não
    escrevas. Num campo de **moeda ou número**, o `valor` é texto («até oito mil euros este ano»):
    propõe o número ao utilizador e só escreves o que ele confirmar.
  - Sem sítio no mapa, o valor fica só na nota (que já o traz): mostra-o como «sem sítio no mapa».
  - `chave_do_mapa: "notas"` segue a regra da nota, acima; `etapa` segue a regra da etapa, abaixo.
- **Etapa** → só a partir de `etapa_sugerida`, que é sempre uma **sugestão**, ou da etapa que o
  utilizador disser nesta sessão. Usa `etapa_sugerida.etapa_no_crm.id` ou procura a `etapa` em
  `etapas` do mapa (no ClickUp, o estado da tarefa do negócio, pelo `etapaNoCrm`). Mostra a etapa
  atual, a sugerida e a `razao`, e pergunta. Sem correspondência no mapa, diz que não sabes para onde
  mudar e não mudes. Nunca mudas para ganho ou perdido por tua conta.

Uma parte que o mapa não cobre não se escreve: mostra-a como «sem sítio no mapa» e sugere
`/closeai:instalar crm`. Não inventes campos, operações nem dados que não venham da proposta.

## 4. Mostrar o que vais escrever e onde

Antes de qualquer escrita, mostra uma lista curta, por grupos (exemplo no ClickUp):

```
Negócio          → tarefa «<nome>» na lista «<lista do negócio>»
Nota da reunião  → comentário na tarefa «<nome>»: «<titulo>» + resumo
Tarefas          → <n.º> subtarefas de «<nome>»: <titulo — dataLimite>
Campos           → <campo do mapa>: <valor atual> → <valor da proposta>
Etapa (sugestão) → <etapa atual> → <etapa sugerida> (<razao>)
Sem sítio no mapa→ <rotulo> (<valor>), …
Para confirmar   → qualificação (o que se sabe e o que falta) e objeções, só para leres
```

Nada de notas, percentagens ou rankings sobre quem conduziu a reunião: a proposta não os traz, e tu
também não.

## 5. Perguntar, grupo a grupo

Usa a pergunta com opções (AskUserQuestion), uma pergunta por grupo (nota da reunião, tarefas,
campos e etapa), cada uma com as opções (a etapa só se houver `etapa_sugerida` ou o utilizador a pedir):
- **Escrever**
- **Editar** (o utilizador diz o que muda; mostras o grupo editado e voltas a perguntar)
- **Não escrever**

Se a pergunta com opções não estiver disponível, faz uma lista numerada e espera pela resposta.
Se não houver ninguém a responder (corrida agendada), não escrevas nada: a proposta fica pendente.

## 6. Escrever só o que foi aceite

- Escreve, pelo conector, **só os grupos com «Escrever»** (com as edições feitas).
- Uma operação de cada vez; se uma falhar, para esse grupo, diz qual falhou e segue com os outros.
- Não apagues nada no CRM.
- No fim, quando o utilizador tiver respondido a todos os grupos (escrever, editar ou não
  escrever), chama `marcar_crm_tratado` com o id da `analise`, para ela sair da lista das pendentes.
  Se ficou algum grupo por responder, não a marques.

## 7. Resumo

No fim, diz o que ficou escrito e onde (com ligações, se o conector as der), o que ficou por
escrever por escolha do utilizador, e o que falhou.
