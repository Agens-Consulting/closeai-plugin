---
name: analisar
description: Analisa uma reunião comercial com o CloseAI a partir da transcrição completa (ficheiro, conector da ferramenta de transcrição, ou texto colado), passo a passo, com subagentes de modelo fixo, e mostra o resumo e a ligação para a análise na app. Também retoma uma análise que ficou a meio.
when_to_use: Quando o utilizador diz «analisa esta reunião», «analisa a reunião com o cliente X», «o que se sabe deste negócio depois da call», cola uma transcrição, aponta um ficheiro de transcrição, ou pede para continuar uma análise do CloseAI que ficou a meio.
argument-hint: "[reunião, ficheiro ou identificador para retomar]"
---

# Analisar uma reunião com o CloseAI

O CloseAI analisa no servidor dele o que tu lhe publicares, com as instruções dele. Tu orquestras:
obténs a transcrição, abres a reunião, lanças um subagente por passo e fechas. **Tu não avalias a
reunião**: quem segue as instruções de análise são os subagentes.

As ferramentas do CloseAI chamam-se `mcp__plugin_closeai_closeai__<nome>`; aqui escrevem-se pelo nome
curto. Fala em português de Portugal.

O que o utilizador pediu: $ARGUMENTS

## 1. Obter a transcrição

- **Retomar:** se te deram o identificador de uma reunião aberta (ou se o utilizador pede para
  continuar a última), salta para o passo 3 com `abrir_reuniao` só com `reuniao`.
- **Ficheiro:** lê-o.
- **Conector** (Google Drive, Fireflies, Otter…): vai buscar o documento da **transcrição**. No Meet é
  o documento da transcrição e não as «Notas» do Gemini; no Fireflies e no Otter, a transcrição e não
  o resumo.
  - Uma reunião longa vem num resultado grande, e o Claude Code guarda-o num **ficheiro** (a resposta
    diz o caminho). **Não o leias com `Read`**: corta as linhas com mais de 2000 caracteres, e uma
    transcrição tem linhas assim. Tira o texto com Bash: `jq -r` sobre o ficheiro (por exemplo
    `jq -r '.[].text' <ficheiro> > <pasta temporária>/transcricao.txt`, ou o campo onde o conector
    põe o texto), e conta com `wc -c` e `wc -l`.
- **Colada:** usa o texto tal como veio.

Não alteres o texto: nem resumir, nem corrigir, nem tirar partes. A normalização é um passo da análise.

**Confere que chegou inteira** antes de enviar: olha para as últimas falas (`tail -n 15` sobre o
texto). Têm de soar ao fim de uma reunião (despedidas, próximos passos, «Reunião encerrada após…»
no Meet), e não acabar a meio de uma frase. Se não der para confirmar, ou o texto parecer cortado,
diz ao utilizador o que viste («a transcrição parece acabar a meio, aos 34 minutos») e pergunta se
segue assim ou se vai buscar outra vez.

## 2. Filtros baratos (antes de gastar o que for)

Para sem chamar subagentes, e diz porquê, se:
- **é um resumo, notas ou pontos de ação** e não uma transcrição (não há falas com o nome de quem
  fala, ou é muito curto para uma reunião);
- a reunião é **mais curta do que a duração mínima** do perfil (se a souberes por `ler_perfil`);
- **não é uma reunião comercial** com um cliente (por exemplo, uma reunião interna).

Na dúvida, segue: o servidor tem a palavra final.

## 3. Abrir a reunião

**Até 20 000 caracteres, vai inteira.** Acima disso (qualquer reunião de mais de 15 minutos), vai
**por partes**, para nenhum argumento passar do limite: corta o texto em partes de até 20 000
caracteres, **sempre em fim de linha** e sem mudar nada. Com Bash (funciona no macOS; o `split -C`
não existe lá):

```bash
awk -v d="<pasta temporária>" 'BEGIN{n=1;t=0} {l=length($0)+1; if(t>0 && t+l>20000){close(f);n++;t=0} t+=l; f=sprintf("%s/parte-%02d",d,n); print > f}' <ficheiro>
```

Isto cria `parte-01`, `parte-02`… pela ordem. Lê cada parte com Bash (`cat`) no momento de a enviar.

Chama `abrir_reuniao` com:
- `transcricao`: o texto inteiro, ou só a primeira parte;
- `partes`: o número total de partes (omite se vai inteira);
- `titulo`: como o utilizador chama a reunião (ou o nome do ficheiro);
- `acontecida_em`: a data em ISO 8601, se a souberes;
- `origem`: `{ fonte, id_na_fonte, hash }` se veio de uma fonte registada (omite se foi colada);
- `versao_do_plugin`: `"0.1.0"`;
- `plataforma`: `claude-code` ou `claude-desktop`.

Se vier `recusada: true`, mostra o `porque` do servidor tal como vem (por exemplo, «isto é um resumo,
não uma transcrição», ou a mensagem de licença suspensa com o que fazer) e para aqui.

Por partes, a resposta traz `faltam` em vez do `plano`: envia cada parte seguinte com
`anexar_transcricao` (`reuniao`, `parte` de 2 ao total, `texto`), pela ordem. Com a última vem o
`plano` (ou `recusada: true`, que mostras tal como vem). Se a ligação cair a meio, `abrir_reuniao`
só com `reuniao` diz que partes faltam.

Guarda `reuniao` e o `plano`.

## 4. Correr os passos do plano

O `plano` é uma lista de passos, cada um com `passo`, `subagente`, `modelo`, `ordem` e `estado`.

- Salta os passos com `estado: publicado` (a análise está a ser retomada).
- Corre os passos **por ordem de `ordem`**. Os que têm a **mesma `ordem`** lança-os **em paralelo,
  todos na mesma mensagem**: os blocos `normalizar:<bloco>` (uma reunião longa tem um por cada ~8000
  caracteres, uma hora dá perto de onze) e, depois, os grupos `kpis:<grupo>`. Se forem mais de dez,
  lança-os em levas de dez. Só passas à `ordem` seguinte quando todos os da anterior responderem.
- Para cada passo, lança o subagente indicado em `subagente` (`closeai:normalizar`,
  `closeai:papeis-e-tipo`, `closeai:kpis`, `closeai:sintese`) com uma mensagem que contém **só**:

  ```
  reuniao: <reuniao>
  passo: <passo>
  ```

  Não lhe passes a transcrição, nem instruções tuas, nem o modelo: ele vai buscar tudo ao servidor e
  publica lá o resultado.
- Cada subagente responde `publicado` ou um erro. Se um passo der erro, lança-o outra vez uma só
  vez. Se voltar a falhar, para, diz ao utilizador qual passo falhou e que pode retomar mais tarde
  com `/closeai:analisar` e o identificador da reunião. Não feches a reunião.

Enquanto correm, diz ao utilizador em que passo vais, numa linha. Não mostres resultados parciais.

## 5. Fechar e mostrar

Chama `fechar_reuniao` com `reuniao`. O servidor confere tudo, calcula o patamar e devolve `analise`,
`resumo`, `patamar` e `ligacao`.

Se o fecho for recusado e a mensagem disser que passos voltaram a por fazer (KPIs por avaliar
demais), corre esses passos outra vez, da mesma forma (os grupos de KPIs em paralelo, depois a
síntese), e fecha de novo. Só uma vez: se voltar a ser recusado, diz ao utilizador que a análise
ficou aberta e que pode retomá-la com `/closeai:analisar` e o identificador.

Mostra ao utilizador:
- o `resumo` **tal como o servidor o devolve** (é o mesmo texto que a app mostra);
- o patamar, que descreve o que se sabe do negócio depois da conversa;
- a `ligacao` para a análise na app.

**Nunca acrescentes notas, percentagens, contagens nem rankings sobre quem conduziu a reunião**, nem
cálculos teus a partir do resumo. Não infiras emoções de ninguém. Se o utilizador pedir uma nota à
pessoa, explica que o CloseAI descreve o negócio e a conversa, não avalia pessoas.

## 6. A seguir

Oferece escrever no CRM: «Queres que prepare o que escrever no CRM? (`/closeai:crm`)». Não escrevas
nada no CRM nesta skill.

