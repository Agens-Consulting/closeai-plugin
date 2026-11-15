---
name: instalar
description: Configura o CloseAI pela primeira vez, ou volta a configurá-lo. Verifica a licença, conduz o perfil do negócio, regista de onde vêm as transcrições, descobre e confirma o mapa do CRM (onde vive cada coisa e que campos e etapas há), explica a tarefa agendada de hora a hora no Claude Desktop e propõe uma primeira análise de teste.
when_to_use: Quando o utilizador acabou de instalar o plugin CloseAI, diz «configurar o CloseAI», «instalar o CloseAI», «mudar o CRM», «mudei de ferramenta de transcrição», «atualizar o perfil do negócio», ou quando outra skill do CloseAI diz que falta o perfil, a fonte ou o mapa do CRM.
argument-hint: "[perfil | fonte | crm | agendar]"
---

# Instalar o CloseAI

Vais pôr o CloseAI a funcionar para este utilizador, por esta ordem. Se ele pediu só uma parte
(`$ARGUMENTS` = `perfil`, `fonte`, `crm` ou `agendar`), salta para essa parte depois do passo 1.

As ferramentas do CloseAI chamam-se `mcp__plugin_closeai_closeai__<nome>`; aqui escrevem-se pelo nome
curto. Para escolhas, usa a pergunta com opções (AskUserQuestion); se não estiver disponível, uma lista
numerada. **Nada se grava sem o utilizador confirmar o resumo.** Fala em português de Portugal.

## Antes de tudo: limites para reuniões longas

Uma reunião longa não cabe nos limites por omissão do Claude Code (a resposta do modelo e o resultado
de uma ferramenta MCP, por omissão 25 000 tokens). **Não edites o ficheiro tu**: explica e pede ao
utilizador que o faça.

1. Pede-lhe que abra `~/.claude/settings.json` (se não existir, cria-o com `{}`) e acrescente, ao
   nível de cima:

   ```json
   "env": {
     "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "64000",
     "MAX_MCP_OUTPUT_TOKENS": "100000"
   }
   ```

   Se já houver um bloco `"env"`, junta as duas linhas lá dentro. O ficheiro é JSON estrito: sem
   comentários e sem vírgula a mais.
2. Depois, que feche e volte a abrir o Claude Desktop (ou o Claude Code no terminal). O separador Code
   do Desktop e o terminal leem o mesmo ficheiro; as conversas normais do Claude Desktop (fora do
   separador Code) não o leem. Se a variável já estiver definida no ambiente com que o Desktop arranca,
   essa ganha ao ficheiro.
3. Se ele não quiser agora, segue: só as reuniões longas é que podem ficar cortadas.

## 1. Licença e ligação

1. Chama `estado_da_licenca` com `versao_do_plugin: "0.1.0"`.
   - Se a ferramenta não existir ou pedir autenticação: diz ao utilizador para abrir `/mcp`, escolher
     `plugin:closeai:closeai` e «Authenticate» (entra com a conta Google da organização). Depois volta
     a correr `/closeai:instalar`. Para aqui.
   - Se `ativa` vier `false`: mostra a `mensagem` do servidor tal como vem e para aqui.
   - Se `versao_minima_do_plugin` for maior do que `0.1.0`: diz para atualizar o plugin
     (`/plugin` › Installed › closeai › Update) e para aqui.
2. Diz em uma frase a que organização ficou ligado (`organizacao`).

## 2. O perfil do negócio

1. Chama `ler_perfil`. Se já houver perfil, mostra um resumo curto e pergunta se quer revê-lo ou
   seguir em frente.
2. Chama `perguntas_de_onboarding` e faz as perguntas que vierem, **uma ou duas de cada vez**, pela
   ordem do servidor. Usa as `opcoes` quando vierem; aceita respostas livres.
3. O perfil é sobre o negócio (o que vendem, a quem, as etapas, os tipos de reunião, as objeções
   frequentes, a duração mínima de reunião a analisar). **Não aceites listas de contactos nem dados
   pessoais de clientes**: se o utilizador os colar, diz que não entram no perfil e não os guardes.
4. Mostra o resumo do perfil e pergunta: «Guardo assim?» (Guardar / Corrigir).
5. Só com «Guardar», chama `guardar_perfil` com `perfil` (só os campos de topo que mudaram; o
   servidor junta-os ao perfil atual) e `confirmadoPeloUtilizador: true`. Diz a versão que ficou.

## 3. De onde vêm as transcrições (a fonte)

1. Pergunta **como transcrevem** as reuniões, com opções: Google Meet (Gemini), Fireflies, Otter,
   tl;dv, Teams, outra.
2. Pergunta **onde caem** as transcrições (por exemplo, a pasta «Meet Recordings» do Google Drive,
   o espaço do Fireflies, uma pasta partilhada).
3. Confirma que **o conector dessa ferramenta está ligado e responde**: entre as ferramentas que tens,
   procura as desse conector e faz uma leitura pequena (listar os 3 ficheiros ou reuniões mais
   recentes desse sítio, só nomes e datas). Mostra o que encontraste.
   - Se não houver conector: explica que tem de o ligar no Claude (Definições › Conectores, ou `/mcp`)
     e volta a este passo depois.
   - No Meet, o que serve é o documento da **transcrição**, não as «Notas» do Gemini: um resumo não é
     uma transcrição e o CloseAI recusa-o.
4. Mostra o resumo («ferramenta, conector, onde, a partir de que duração») e pede confirmação.
5. Com o sim, chama `registar_fonte` com `ferramenta`, `conector` (o nome do conector como o vês),
   `onde` (o identificador da pasta ou do espaço, não um caminho com dados pessoais), `descricao` e,
   se o perfil tiver, `duracao_minima_minutos`.

## 4. O mapa do CRM

O CloseAI não escreve em CRM nenhum: diz o que escrever, e tu escreves pelo conector do CRM do
utilizador. Para isso precisas de saber **onde vive cada coisa** nesse CRM. Isso é o `mapaDoCrm`.

1. Pergunta qual é o CRM (ClickUp, HubSpot, Pipedrive, outro) e confirma que o conector está ligado.
2. **Descobre a estrutura lendo o CRM pelo conector, só com ferramentas de leitura.** Nesta fase
   não crias, não alteras e não apagas nada no CRM.
   - Exemplo no ClickUp: lê a hierarquia do espaço de trabalho (espaços, pastas, listas); encontra
     a lista onde vivem os negócios (muitas vezes uma pasta «Pipeline» ou «CRM» com listas como
     Oportunidades, Empresas, Pessoas); lê os **campos personalizados** dessa lista (nome, id, tipo,
     opções) e os **estados** (que costumam ser as etapas da pipeline); vê onde ficam as tarefas de
     seguimento e se as notas da reunião vão como comentário da tarefa do negócio ou como documento.
   - Noutro CRM, o mesmo: onde estão os negócios, as empresas, as pessoas, as tarefas e as notas; que
     campos há; que etapas tem a pipeline.
3. Propõe o mapa, em linguagem simples e numa tabela curta:
   - **entidades**: para `negocio`, `empresa`, `pessoa`, `tarefa` e `nota`, `onde` vivem (em palavras,
     por exemplo «lista Oportunidades, na pasta Pipeline»), os `identificadores` opacos que o conector
     usa (por exemplo o id da lista) e a `ligacao` às outras (por exemplo «comentário na tarefa do
     negócio», «tarefa ligada ao negócio pelo campo Empresa»);
   - **campos**: que campo do CRM recebe cada coisa da proposta do CloseAI — `etapa`, `proximoPasso`,
     `data`, `valor`, `notas`, `tarefas` —, cada um com `nome`, `id` e `tipo`;
   - **etapas**: para cada etapa do perfil, `etapaDoPerfil`, `etapaNoCrm` (o estado ou etapa no CRM) e o
     `id`.
   Usa só nomes de estrutura e identificadores. Nada de nomes de clientes, emails ou valores de negócios.
4. Pergunta: «Este mapa está certo?» (Guardar / Corrigir uma parte / Não guardar).
5. Com «Guardar», chama `guardar_perfil` com `perfil: { mapaDoCrm: { crm, conector, entidades, campos,
   etapas } }` e `confirmadoPeloUtilizador: true`. Quem confirmou e quando põe-no o servidor.

## 5. A automação de hora a hora (Claude Desktop)

A automação é uma **tarefa local agendada do Claude Desktop**, que corre `/closeai:automacao` (uma skill
deste plugin): só corre com a app aberta e o computador acordado. As tarefas na nuvem não carregam plugins, por isso não servem. Dá estes passos, exatamente:

1. Abre o **Claude Desktop** e vai ao separador **Code**.
2. Na barra lateral, clica em **Scheduled** (ou em **More** › **Scheduled**; em versões antigas chama-se
   **Routines**).
3. Clica em **New task**. Se aparecer um menu, escolhe **Set up manually** (e, se perguntar, **Local**).
4. Em **Name**, escreve `closeai-automacao`. Se pedir **Description**: «Procura reuniões novas e
   analisa-as no CloseAI».
5. Na caixa das instruções escreve só: `/closeai:automacao`
6. Em **Advanced settings** › **General**, liga **Only on this computer** e escolhe uma pasta em
   **Local folder** (uma pasta vazia, por exemplo `~/CloseAI`; a automação não escreve lá nada).
7. Em **Trigger**, escolhe **Hourly**.
8. Na barra da caixa das instruções, escolhe o modo de permissões por omissão e o modelo
   **Sonnet** (os passos da análise têm o modelo fixado à parte).
9. Clica em **Create** e, na página da tarefa, em **Run now**. Na sessão que abre, aprova as
   ferramentas do CloseAI e do conector da fonte com **always allow**, para as corridas seguintes não
   pararem à espera.
10. Em **Settings** › **Desktop app** (ou **This computer** › **System**), liga **Keep computer awake**.
    Fechar a tampa do portátil adormece-o na mesma.

Se o utilizador preferir, podes também criar a tarefa por ele, se tiveres a ferramenta de tarefas
agendadas do Desktop: «corre `/closeai:automacao` de hora a hora, só neste computador». Confirma
antes de a criar.

A app do CloseAI mostra «ligado» quando receber o primeiro sinal de vida, e avisa se ficar 48 horas
sem sinal em dias úteis.

## 6. A primeira análise

Propõe analisar já uma reunião, para ver tudo a funcionar: a mais recente da fonte, ou uma que o
utilizador escolha. Se aceitar, segue `/closeai:analisar` com essa reunião. No fim, oferece
`/closeai:crm`.

## Para fechar

Resume o que ficou feito (licença, perfil e versão, fonte, mapa do CRM, automação) e o que falta.
Lembra que `/closeai:estado` mostra isto a qualquer momento.
