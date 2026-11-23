---
name: automacao
description: Corrida agendada do CloseAI, feita para a tarefa local do Claude Desktop de hora a hora. Procura transcrições novas na fonte registada, analisa as que o CloseAI ainda não tem (até ao teto por corrida), corre a reunião de calibração quando o servidor pedir, deixa as propostas ao CRM pendentes e regista sempre um sinal de vida. Não faz perguntas.
when_to_use: Só quando a tarefa agendada «closeai-automacao» corre, ou quando o utilizador pede expressamente «corre a automação do CloseAI agora».
disable-model-invocation: true
disallowed-tools: AskUserQuestion
---

# Corrida da automação do CloseAI

Corres sem ninguém a ver. **Não fazes perguntas** (não há quem responda), **não escreves no CRM**, e
**acabas sempre com `sinal_de_vida`**, corra bem ou mal. Se houver um pedido de permissão, a corrida
fica parada até alguém o aprovar; por isso usa só as ferramentas do CloseAI e as de leitura do conector
da fonte.

As ferramentas do CloseAI chamam-se `mcp__plugin_closeai_closeai__<nome>`; aqui escrevem-se pelo nome
curto.

Vai contando, para o resumo final: `vistas`, `novas`, `analisadas`, `recusadas`, `falhadas`,
`calibracao` (sim ou não) e `erro` (se houver).

## 1. Licença

`estado_da_licenca` com `versao_do_plugin: "0.1.1"`.
- `ativa: false` → **para**: chama `sinal_de_vida` com `erro: "licença suspensa"` e termina.
- `versao_minima_do_plugin` acima de `0.1.1` → para da mesma forma, com `erro: "plugin desatualizado"`.
- Sem `fontes` → não há nada a procurar; termina sem `sinal_de_vida` (não há fonte a que o dar) e
  diz numa linha que falta `/closeai:instalar fonte`.

## 2. Para cada fonte

1. **Metadados desde a última corrida:** pelo conector da fonte (`conector`, em `onde`), lista só os
   ficheiros ou reuniões criados desde `ultima_corrida` (se não houver, os últimos 7 dias). **Só
   metadados**: identificador na fonte, nome, data, tamanho e, se o conector o der, um hash. Não abras
   ainda nenhum documento. No Meet, conta só os documentos da transcrição, não as «Notas».
2. **`por_analisar`** com `fonte` e `candidatos` (`id_na_fonte`, `nome`, `data`, `tamanho`, `hash`).
   Vem `novos`, `teto`, `abertas` e, às vezes, `calibracao`.
3. **Primeiro as `abertas`** (análises desta fonte que ficaram a meio numa corrida anterior), dentro
   do `teto`: para cada uma, `abrir_reuniao` só com `reuniao`. Se ainda faltarem partes da
   transcrição, abre o documento outra vez pelo conector (o `titulo` ajuda a encontrá-lo) e envia as
   que faltam com `anexar_transcricao`; depois corre os passos que voltarem por fazer e
   `fechar_reuniao`. Cada aberta retomada conta para o teto e, se fechar, em `analisadas`.
4. **Depois, para cada um dos `novos`, até ao que sobrar do `teto`** (3 se não vier): abre o documento e faz o que
   `/closeai:analisar` faz — ler o texto do resultado do conector com Bash/`jq` se vier num ficheiro,
   conferir que o fim da reunião chegou, filtros baratos, `abrir_reuniao` (com `origem: { fonte,
   id_na_fonte, hash }`, `versao_do_plugin: "0.1.1"`, `plataforma: "claude-desktop"`) e, acima de
   20 000 caracteres, por partes com `anexar_transcricao`, os passos do plano pelos subagentes
   `closeai:*` (os blocos da normalização e os grupos de KPIs em paralelo), e `fechar_reuniao`.
   - Uma transcrição que parece cortada no fim → não a analises: conta em `falhadas`, com o `erro`
     «transcrição incompleta», e segue (a próxima corrida não a devolve outra vez: diz no fim que
     há uma a analisar à mão com `/closeai:analisar`).
   - Um resumo, uma reunião curta demais, ou uma recusa do servidor → conta em `recusadas` e segue.
   - Um passo que falha duas vezes → conta em `falhadas`, deixa a reunião aberta (a próxima corrida
     retoma-a) e segue para a seguinte.
   - Não mostres o resumo de cada análise: fica na app.
5. **Calibração:** se `por_analisar` trouxer `calibracao`, a reunião de calibração já vem aberta pelo
   servidor (`reuniao` e `plano`). Corre os passos do plano como em qualquer outra e `fechar_reuniao`.
   Não abras outra reunião para ela. Fica fora das análises do cliente; marca `calibracao: sim`.

## 3. CRM: fica pendente

Não chames `proposta_para_crm` nem escrevas no CRM. As propostas ficam pendentes na app, e o
utilizador trata-as numa sessão com `/closeai:crm`.

## 4. Sinal de vida (sempre)

Para cada fonte, `sinal_de_vida` com `fonte`, `resumo` (`vistas`, `novas`, `analisadas`,
`recusadas`, `falhadas`, `calibracao`, `erro`) e `versao_do_plugin: "0.1.1"`. Também quando algo
correu mal a meio: nesse caso, com o `erro` numa frase.

Termina com uma linha: quantas reuniões novas, quantas analisadas, e se há alguma coisa a tratar.
Sem números sobre pessoas.
