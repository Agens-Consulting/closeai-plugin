---
name: objecoes
description: Treino rápido de objeções com o CloseAI — o Claude levanta uma objeção de cada vez (do perfil do negócio ou das que a equipa mais encontra), o comercial responde, o cliente reage uma vez, e o Claude dá uma sugestão curta antes da seguinte. Sem notas nem números sobre a pessoa.
when_to_use: Quando o utilizador diz «quero treinar objeções», «atira-me objeções», «treina comigo o "está caro"», «cinco minutos de objeções», ou pede `/closeai:objecoes`. Para uma reunião inteira é a skill `treino`.
---

# Treino rápido de objeções

As ferramentas do CloseAI chamam-se `mcp__plugin_closeai_closeai__<nome>`; aqui escrevem-se pelo nome
curto. Fala em português de Portugal.

## 1. Perguntar, numa mensagem só

Pergunta só o que o utilizador ainda não tiver dito:

- **Queres que eu seja o cliente ou tu?** (`claude_cliente`: o Claude levanta as objeções e o
  utilizador responde; `claude_comercial`: o utilizador levanta-as e o Claude responde com as
  respostas aprovadas da casa.)
- A dificuldade: fácil, normal ou difícil (por omissão, normal).
- Se quer começar por alguma objeção (preço, timing, decisor, fornecedor, confiança, necessidade).

## 2. Preparar

Chama `preparar_roleplay` com `modo`, `dificuldade`, `objecao` (se houver), `foco: "objecoes"` e
`canal: "texto"`. Se recusar um valor, diz porquê e pergunta outra vez. Diz o `aviso` tal como vem
e começa.

## 3. O treino

Segue as `regrasDoPapel`. Com o Claude como cliente:

1. Levanta uma objeção de `persona.objecoes`, pela ordem, com as palavras de `comoSoa`.
2. O utilizador responde.
3. Reage uma vez só, no papel: convencido, meio convencido ou não convencido, com uma frase de
   porquê.
4. Sai do papel e dá **uma** sugestão curta, ligada ao que ele respondeu, com uma alternativa
   concreta (e a resposta aprovada do perfil, se houver para esse tema).
5. Passa à seguinte. Ao fim de cinco, ou quando ele disser «acabou», para.

Com o Claude como comercial: o utilizador levanta a objeção, respondes com a resposta aprovada para
esse tema (`persona.respostasAprovadas`) por palavras tuas, e se ele pedir «outra forma», dás outra.

Não chames mais ferramentas do CloseAI durante o treino.

## 4. O fecho

Uma frase sobre o que correu bem e uma proposta de próximo treino («queres repetir o preço em
difícil?»). Segue o `guiaoDoFeedback`. **Nunca** notas, contagens («acertaste 3 de 5»),
percentagens, comparações ou juízos sobre a pessoa, e nada inferido da forma de falar (tom,
nervosismo, emoções): só o que foi dito. O feedback fica nesta conversa.

Pergunta se quer registar os temas treinados para o agregado da equipa, sem quem treinou; se sim,
`fechar_roleplay` com os argumentos soltos: `versao` (a que `preparar_roleplay` devolveu), `modo`
(`pedido.modo`), `tipo_de_reuniao` (`pedido.tipo`), `dificuldade` (`pedido.dificuldade`), `foco`
(`pedido.foco`) e `temas`, só de `temasParaFechar` (ex. `objecao:preco`, `objecao:timing`). Nunca
texto livre nos temas.
