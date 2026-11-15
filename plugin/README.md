# CloseAI — o plugin

O plugin é a forma de usar o CloseAI: corre no Claude Code e nas sessões do separador Code do
Claude Desktop, e traz a ligação ao servidor CloseAI, os subagentes de cada passo (com o modelo
fixado e isolados uns dos outros) e a automação de hora a hora.

## O que é

O CloseAI analisa reuniões comerciais. O teu Claude vai buscar a transcrição à ferramenta que já usas
(Google Meet, Fireflies, Otter…), o CloseAI dá as instruções de cada passo e confere o resultado, e no
fim tens o que se sabe do negócio depois da conversa, o que há a melhorar, as objeções, as tarefas e o
que escrever no CRM. O CRM só é escrito com o teu sim.

O resultado de uma reunião é um **patamar** que descreve o negócio, não a pessoa: o CloseAI não dá
notas, percentagens nem rankings sobre quem conduziu a reunião.

## Antes de começar

- Uma organização no CloseAI (a Agens cria-a e manda o convite para o email do dono).
- Claude Code no terminal, ou o **Claude Desktop** (precisas dele para a automação de hora a hora).
- Um plano Claude **Team, Enterprise ou API** (trazem acordo de tratamento de dados e não treinam
  com as tuas conversas).
- O conector da tua ferramenta de transcrição e o do teu CRM ligados no Claude.

## Instalar

Numa sessão do Claude Code (ou do separador Code do Claude Desktop):

```
/plugin marketplace add Agens-Consulting/closeai-plugin
/plugin install closeai@closeai
```

Escolhe o âmbito **utilizador**, para o plugin ficar disponível em todas as pastas e no Desktop.

Na shell, o mesmo:

```bash
claude plugin marketplace add Agens-Consulting/closeai-plugin
claude plugin install closeai@closeai --scope user
```

O repositório é privado durante o piloto: o `marketplace add` usa as credenciais git que já tens
(`gh auth login`, Keychain ou SSH). Sem acesso ao repositório, pede-o à Agens.

As atualizações de um marketplace de terceiros vêm desligadas por omissão. Liga-as em `/plugin` ›
Marketplaces › closeai, ou atualiza à mão com `claude plugin update closeai@closeai`.

## Reuniões longas: subir os limites

Por omissão, o Claude Code limita o tamanho da resposta do modelo e o do resultado de uma ferramenta
MCP (25 000 tokens). Uma reunião longa passa disso. Acrescenta ao `~/.claude/settings.json` (cria-o
com `{}` se não existir):

```json
{
  "env": {
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "64000",
    "MAX_MCP_OUTPUT_TOKENS": "100000"
  }
}
```

Se o ficheiro já tiver um bloco `"env"`, junta as duas linhas lá dentro. Depois fecha e volta a abrir
o Claude Desktop (ou o Claude Code no terminal). Serve para o terminal e para o separador Code do
Claude Desktop, que leem o mesmo ficheiro; uma variável já definida no ambiente com que o Desktop
arranca ganha ao ficheiro. O Claude Code baixa sozinho o `CLAUDE_CODE_MAX_OUTPUT_TOKENS` ao máximo do
modelo, se este for menor. O `/closeai:instalar` explica isto mas não mexe no ficheiro.

## Ligar ao servidor

1. Abre `/mcp`, escolhe `plugin:closeai:closeai` e **Authenticate**.
2. O browser abre a página do CloseAI: entra com o Google, com a conta convidada.
3. Corre `/closeai:instalar`.

O plugin liga-se a `https://app.closeai.pt/mcp`. Para testar contra outro servidor (por exemplo, o
local da Agens), junte-o à parte como conector personalizado (Customize › Connectors › Add custom
connector) ou, no Claude Code, com `claude mcp add`.

### Alternativa: token de acesso, sem OAuth

Para quem não pode usar o OAuth (por exemplo, uma máquina sem browser), cria um token na app do
CloseAI («Ligar ao Claude») e acrescenta um servidor de utilizador com o cabeçalho de autorização:

```bash
claude mcp add --transport http --scope user closeai-token https://app.closeai.pt/mcp \
  --header "Authorization: Bearer closeai_..."
```

Num `.mcp.json` escreve-se `"headers": { "Authorization": "Bearer ${CLOSEAI_TOKEN}" }`, com um nome de
variável próprio: o Claude Code deixa vazias as variáveis com nomes de credenciais conhecidas. O token
fica guardado nas definições do Claude Code; revoga-o na app quando deixar de ser preciso. Nota: as skills e os subagentes deste plugin chamam as ferramentas do
servidor do plugin (`mcp__plugin_closeai_closeai__…`); um servidor acrescentado à mão tem outro nome e
serve só para usar as ferramentas diretamente.

## Comandos

| Comando | Para quê |
|---|---|
| `/closeai:instalar` | Licença, perfil do negócio, fonte das transcrições, mapa do CRM, automação e primeira análise |
| `/closeai:analisar` | Analisar uma reunião (ficheiro, conector ou texto colado), ou retomar uma a meio |
| `/closeai:crm` | Escrever no CRM o que a análise propõe, grupo a grupo, só com o teu sim |
| `/closeai:estado` | Ver se está tudo ligado e o que fazer quando algo falha |
| `/closeai:automacao` | A corrida de hora a hora; é a tarefa agendada que a chama |

## A automação

É uma tarefa local agendada do Claude Desktop que corre `/closeai:automacao` de hora a hora. Só corre
com o Desktop aberto e o computador acordado; liga «Keep computer awake» nas definições. O
`/closeai:instalar` dá os passos exatos. A automação não faz perguntas nem escreve no CRM: as
propostas ficam pendentes até correres `/closeai:crm`.

## Permissões

- O plugin **não corre comandos de shell** e **não escreve ficheiros**. Não tem `bin/` nem hooks.
- Os subagentes (`closeai:normalizar`, `closeai:papeis-e-tipo`, `closeai:kpis`, `closeai:sintese`) têm
  **só duas ferramentas**: `instrucoes_do_passo` e `publicar_passo`. Não chegam ao CRM, à shell nem a
  ficheiros, e o que estiver escrito numa transcrição não os consegue mandar fazer outra coisa.
- As skills usam as ferramentas do CloseAI e as do conector da fonte e do CRM que já tens ligados.
  No CRM, só leem sem perguntar; escrever pede sempre o teu sim.

## Ferramentas MCP que o plugin usa

Do servidor CloseAI (com o nome completo `mcp__plugin_closeai_closeai__<ferramenta>`):

`estado_da_licenca`, `perguntas_de_onboarding`, `guardar_perfil`, `ler_perfil`, `registar_fonte`,
`por_analisar`, `sinal_de_vida`, `abrir_reuniao`, `instrucoes_do_passo`, `publicar_passo`,
`fechar_reuniao`, `proposta_para_crm`, `listar_analises`.

## Privacidade

- A transcrição vai para o servidor CloseAI (na UE) durante a análise, cifrada, e é apagada quando a
  análise fecha. Fica o que a análise precisa para se mostrar.
- A análise corre no **teu** Claude, com a tua subscrição.
- **O Claude Code guarda uma cópia local das conversas** no teu computador, em texto simples, em
  `~/.claude/projects/` — incluindo as transcrições e o que os subagentes leram. Por omissão
  apagam-se ao fim de 30 dias; as sessões do Claude Desktop ficam sem limite. Para encurtar, define
  `cleanupPeriodDays` e `desktopSessionCleanupPeriodDays` nas definições do Claude Code.
- O plugin não guarda nada fora do que o Claude Code já guarda.
