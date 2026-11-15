---
name: treino
description: Treino de vendas por roleplay com o CloseAI — o Claude faz de cliente (ou de comercial) de uma reunião R1, R2, R3 ou one call close, com um cliente inventado a partir do perfil do negócio e dos padrões da equipa, e no fim dá sugestões de melhoria, sem notas nem números sobre a pessoa.
when_to_use: Quando o utilizador diz «quero treinar», «vamos fazer um roleplay», «faz de cliente», «treina comigo a reunião de diagnóstico», «quero praticar a apresentação da proposta», ou pede `/closeai:treino`. Para treinar só objeções, uma de cada vez, é a skill `objecoes`.
---

# Treino de uma reunião

As ferramentas do CloseAI chamam-se `mcp__plugin_closeai_closeai__<nome>`; aqui escrevem-se pelo nome
curto. Fala em português de Portugal.

## 1. Perguntar, numa mensagem só

Pergunta ao utilizador, numa só mensagem curta, o que ele ainda não tiver dito:

- **Queres que eu seja o cliente ou tu?** (`modo`: `claude_cliente` se o Claude é o cliente;
  `claude_comercial` se o Claude é o comercial e o utilizador faz de cliente.)
- Que reunião quer treinar: R1 (diagnóstico), R2 (apresentação da proposta), R3 (feedback e
  decisão), OCC (fecho numa só reunião), ou o nome que a empresa lhe dá.
- A dificuldade: fácil, normal ou difícil (por omissão, normal).
- Se quer treinar alguma objeção em especial (preço, timing, decisor, fornecedor, confiança,
  necessidade).

Não perguntes mais nada.

## 2. Preparar

Chama `preparar_roleplay` com `modo`, `tipo_de_reuniao`, `dificuldade`, `objecao` (se houver),
`foco: "reuniao"` e `canal: "texto"`. Se recusar um valor, diz ao utilizador porquê e pergunta
outra vez. Se a resposta trouxer `sobreOPerfil`, diz-lho numa frase (o perfil melhora o treino;
preenche-se com `/closeai:instalar perfil`).

Diz o `aviso` tal como vem. Depois diz que está pronto e que pode começar; se ele quiser falar,
lembra que o `/voice` do Claude Code transcreve o que ele disser.

## 3. O roleplay

Segue as `regrasDoPapel` e a `persona`. Durante o roleplay não chames mais nenhuma ferramenta do
CloseAI, não saias do papel (salvo «pausa») e não dês dicas. Acaba quando o utilizador disser
«acabou», «para» ou «feedback», quando a persona chegar a um fim natural, ou ao fim de uns 20
minutos.

## 4. O feedback

Segue o `guiaoDoFeedback`:

- Primeiro o que correu bem, citando o que foi dito.
- Depois duas ou três sugestões, cada uma ligada a um momento da conversa e com uma alternativa
  concreta; a resposta aprovada do perfil para a objeção que custou mais; o que ficou por saber do
  negócio do cliente.
- No fim, uma proposta de próximo treino.

**Nunca:** notas, percentagens, contagens, pontuações, patamar, «melhor que da última vez»,
comparações com colegas ou com a equipa, juízos sobre a pessoa («és bom a…», «tens tendência
para…»). **Nada inferido da forma de falar** (tom, nervosismo, hesitação, emoções): fala só do que
foi dito. No modo `claude_comercial` não avalies o utilizador: diz em duas ou três frases o que
fizeste e porquê.

O feedback fica nesta conversa. Não o guardes em ficheiros, não o mandes para o CRM nem para
ninguém.

## 5. Registar os temas (opcional)

Pergunta se quer registar os temas treinados para o agregado da equipa, sem quem treinou. Se
disser que sim, chama `fechar_roleplay` com estes argumentos, soltos (não passes o `pedido` inteiro):

- `versao`: a `versao` que `preparar_roleplay` devolveu;
- `modo`: `pedido.modo`;
- `tipo_de_reuniao`: `pedido.tipo`;
- `dificuldade`: `pedido.dificuldade`;
- `foco`: `pedido.foco`;
- `temas`: os temas da lista `temasParaFechar` que se treinaram (ex. `objecao:preco`,
  `qualificacao:decisor`, `proximo_passo`). Nunca texto livre nos temas.
