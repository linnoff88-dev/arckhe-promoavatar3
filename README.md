# promoavatar3

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

TrÃªs vÃ­deos por pÃºblico, em vez de um. O bot escreve os textos e PARA;
`/aprovar C#N` libera avatar, download e reel.

**Este projeto Ã© AUTÃ”NOMO** (desde 2026-08-06). Ele jÃ¡ foi descrito como "igual
ao promoavatar": nÃ£o Ã© mais. O motor do reel (`scripts/`), os layouts
(`templates/`) e a skill de ediÃ§Ã£o vivem AQUI. Mexer nos alvos, prompts ou
templates de lÃ¡ **nÃ£o afeta nada aqui**.

O `promoavatar` esteve congelado de 2026-08-06 a 2026-08-09 e **voltou a evoluir**.
Isso nÃ£o junta os dois: continuam sistemas separados, cada um com seu motor e seus
alvos. A diferenÃ§a de propÃ³sito Ã© a de sempre â€” lÃ¡ Ã© **um** vÃ­deo por pÃºblico, aqui
sÃ£o **trÃªs** (alcance, autoridade, promocional).

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/promoavatar3/guia/**

ReferÃªncia: `C#7`. O `promoavatar` Ã© `A#`, o `promoclub` Ã© `P#` â€” o bot recusa
referÃªncia com prefixo trocado.

Uso, opÃ§Ãµes e a tabela dos trÃªs tipos: `HELP.md` (ou `/promoavatar3 help` no
chat). As decisÃµes de engenharia e o porquÃª de cada uma: `CLAUDE.md`.

## O que Ã© deste repo, e o que Ã© do bot

A divisÃ£o vale para os dois lados e evita a documentaÃ§Ã£o em dobro que envelhece
desencontrada:

| aqui (domÃ­nio) | no [`inemaccbot`](https://github.com/inematds/inemaccbot) (motor) |
|---|---|
| **quem** Ã© o pÃºblico (`alvos` no `flow.json`), o gatilho e o fecho de cada um | como uma fila funciona, lease, retomada, portÃ£o |
| para **qual canal** vai (`lives2`, `lives22`â€¦) â€” pelo NOME, nunca o caminho | onde esse nome mora no disco |
| o **prompt** da fase de texto (`prompts/`) | como um prompt de fase Ã© executado |
| o **template** de reel (`templates/`) e o motor (`scripts/`) | como a fase `reel.montar` dispara o motor |
| o `TEMPLATE-AVATAR` e o motor de voz (`engine`, `voice_id`) | as rotas `\| api`, `\| creditos`, `\| estudio` e de que bolso cada uma sai â†’ [`docs/rotas-de-avatar.md`](https://github.com/inematds/inemaccbot/blob/master/docs/rotas-de-avatar.md) |
| o CTA (`cta/cta-9x16.mp4`) | instalaÃ§Ã£o, `.env`, systemd, comandos do chat |

Regra curta: **se muda com o pÃºblico, Ã© daqui; se muda com a mÃ¡quina, Ã© de lÃ¡.**

## Onde mudar o quÃª

| quero mudar | arquivo |
|---|---|
| pÃºblico, gatilho, fecho, canal | `flow.json` â†’ `alvos` |
| **acrescentar** um pÃºblico novo | `docs/adicionar-publico.md` (5 arquivos, sem restart) |
| como o texto Ã© escrito (as 3 versÃµes, as `## IMAGENS`) | `prompts/fase1-3versoes.md` |
| avatar, voz, motor, template do estÃºdio | `flow.json` â†’ `avatar_id`, `voice_id`, `engine`, `template` |
| o layout do reel | `templates/` (e `template` na raiz do `flow.json`) |
| o motor do reel | `scripts/montar-reel.py` e vizinhos |
| a ajuda que o chat responde | `HELP.md` |
| o vÃ­deo de CTA | `cta/cta-9x16.mp4` (versionado desde 2026-08-08) |

**Nada aqui tem caminho de mÃ¡quina.** O que dependia (`localhost:8000` do
inemaimg, a chave do Groq) virou variÃ¡vel de ambiente com o mesmo default de
antes â€” `INEMAIMG_HOST`, `INEMAIMG_MODEL`, `GROQ_ENV_PATH`, documentadas no
`.env.example` do bot. O teste que pega regressÃ£o Ã©
`git grep /home/ -- .` dar zero em arquivo versionado.

## As imagens: aqui GPU, fora API

`scripts/gen-imagem.py` fala com mais de um provedor. **O default nÃ£o mudou** â€”
quem roda em casa continua na GPU local, sem configurar nada:

| `IMG_PROVEDOR` | quem gera | custo | seed |
|---|---|---|---|
| `inemaimg` *(default)* | a GPU local, `flux2-klein` | zero | **respeitado** |
| `agnes` | API da Agnes AI, `agnes-image-2.1-flash` | **US$ 0**, ~10 s/imagem | **nÃ£o existe** |
| `kie`, `fal` | â€” | â€” | **nÃ£o implementados**: o script recusa em vez de fingir |

Na VPS: `IMG_PROVEDOR=agnes` e a chave em `IMG_ENV_PATH` (arquivo com
`AGNES_API_KEY=`, `chmod 600`) â€” ou `AGNES_API_KEY` direto no ambiente.

**Duas coisas mudam ao sair da GPU, e nenhuma tem conserto aqui:**

1. **O determinismo cai.** A Agnes nÃ£o aceita seed, entÃ£o a mesma `--seed-key`
   gera imagem diferente a cada render. SÃ³ o inemaimg cumpre "mesmo reel, mesma
   imagem" â€” inclusive por tÃºnel (`ssh -R 8000:localhost:8000 <vps>`), que Ã© a
   opÃ§Ã£o a considerar se a reprodutibilidade importar mais que a independÃªncia.
2. **O tamanho pedido vira sugestÃ£o.** Medido: pedimos 1088x736 e voltou
   1248x832. O adaptador **normaliza** cortando pelo centro (nunca esticando,
   que deformaria rosto) â€” sem isso o `preparar.py` regeraria a imagem toda vez,
   porque ele compara dimensÃ£o para decidir reaproveitar.

Detalhe por provedor, com o que foi medido em cada um:
[`inemaimg/docs/prompt-por-provedor.md`](https://github.com/inematds/inemaimg/blob/main/docs/prompt-por-provedor.md).

## Os trÃªs tipos, e por que nÃ£o sÃ£o trÃªs variaÃ§Ãµes

| sufixo | tipo | duraÃ§Ã£o | o que o pÃºblico faz depois |
|---|---|---|---|
| `-alc` | alcance | 25â€“40s | compartilha, comenta |
| `-aut` | autoridade | 35â€“60s | salva, segue |
| `-pro` | promocional | 30â€“45s | clica |

NÃ£o Ã© o mesmo roteiro com trÃªs ganchos. SÃ£o trÃªs FUNÃ‡Ã•ES diferentes, e o que as
separa Ã© o fecho: o `-alc` nÃ£o tem CTA comercial nenhum (nem falado, nem no
reel), o `-aut` toca a marca de leve, o `-pro` converte. Um vÃ­deo que tenta ser
os trÃªs ao mesmo tempo Ã© promocional com abertura simpÃ¡tica â€” e Ã© o de menor
alcance dos trÃªs.

Ordem de publicaÃ§Ã£o recomendada: alcance â†’ autoridade â†’ promocional.

## O prompt do texto (`prompts/fase1-3versoes.md`)

Nesta ordem: os alvos e o que cada sufixo significa Â· CONTEXTO FIXO (Nei e Tiza
como gestores) Â· NÃƒO MEXA NA MÃQUINA Â· PASSO ZERO (tese central, motivo para
assistir, elemento demonstrÃ¡vel) Â· **assunto que Ã© debate** Â· as 14 regras de
escrita Â· o que muda em cada tipo Â· o contrato de saÃ­da.

VariÃ¡veis que o bot injeta: `{{input}}` (o assunto), `{{publicos}}` (os alvos
REAIS do fluxo, jÃ¡ filtrados por `| alvos=`), `{{pasta}}` (onde gravar,
absoluto), `{{ref}}`, `{{saida}}`.

### Assunto que Ã© DEBATE: o prompt crava uma posiÃ§Ã£o

Assunto que chega como pergunta em aberto ("isso Ã© bom ou ruim?", "o que vocÃª
acha?") tinha um resultado previsÃ­vel: o agente explicava os dois lados e
fechava em "o importante Ã© se preparar". Correto e morno â€” ninguÃ©m comenta com
equilibrista.

A causa nÃ£o era falta de talento: sÃ£o as regras 9 e 10 (nÃ£o invente dado, nÃ£o
invente urgÃªncia) fazendo o agente recuar atÃ© o meio-termo, que Ã© o Ãºnico lugar
onde ele tem certeza de nÃ£o estar afirmando nada.

Aqui isso custava mais caro que no `promoavatar`: **sem uma posiÃ§Ã£o na mesa, os
trÃªs tipos colapsam num sÃ³.** Os trÃªs viram o mesmo resumo equilibrado com trÃªs
embalagens, e o `-alc` fica impossÃ­vel â€” "opiniÃ£o contrÃ¡ria", o formato que mais
engaja em tema polÃªmico, nÃ£o existe sem alguÃ©m tomando um lado.

EntÃ£o o prompt manda tomar um lado e **escrever no `resumo-estrategico.md` qual
posiÃ§Ã£o cravou e por quÃª**. Com a posiÃ§Ã£o definida os trÃªs se separam de
verdade: o `-alc` defende, o `-aut` explica a mecÃ¢nica que sustenta, o `-pro`
transforma em consequÃªncia prÃ¡tica. Isso nÃ£o afrouxa as regras 9 e 10: opiniÃ£o Ã©
permitida, fato inventado nÃ£o.

**A posiÃ§Ã£o que vocÃª mandar vence a dele.** Se vocÃª escreve a sua no assunto, ele
usa a sua; o bloco sÃ³ existe para quando vocÃª nÃ£o escreveu. Escrever a sua
continua sendo o melhor caminho â€” junto com um fato concreto (para a linha PROVA
nÃ£o ficar vazia) e a pergunta que vocÃª quer nos comentÃ¡rios.

Como o resumo diz a posiÃ§Ã£o escolhida, vocÃª discorda dela **no portÃ£o**, antes de
gerar avatar nenhum. Aqui isso vale trÃªs renders manuais por pÃºblico, nÃ£o um.

## Legenda: a do reel Ã© nossa, a do estÃºdio Ã© herdada

Desde 2026-08-13 **o reel legenda por default** (palavra a palavra, Ã¢mbar na
palavra-chave â€” `docs/legenda.md`), com o mesmo motor do promoavatar. A do
estÃºdio Ã© outra coisa, e vem de fora:

A fase `baixar` prefere o `video_url_caption` (o MP4 com a legenda **queimada**)
quando a HeyGen o devolve preenchido, e cai no `video_url` limpo quando nÃ£o vem
(`escolherUrl`, `inemaccbot/src/fila/tarefas/heygen.ts`). Gravou com legenda no
estÃºdio, o reel sai com ela; gravou sem, sai sem â€” o bot nÃ£o escolhe.

Duas consequÃªncias que nenhum cÃ³digo desfaz: legenda queimada vem enquadrada
para 16:9 e no reel 9:16 pode ser cortada ou colidir com a base; e ela soma com
a legenda do REEL, que desde 2026-08-13 Ã© **ligada por default** aqui (o motor
passou a legendar, igual ao promoavatar â€” ver `docs/legenda.md`). Gravou com
legenda no estÃºdio e deixou a do reel ligada? Saem **duas**. Escolher uma Ã©
desligar a outra, e a do estÃºdio se desliga no `TEMPLATE-AVATAR`, nÃ£o aqui.
Neste repo isso vale por trÃªs vÃ­deos por pÃºblico, nÃ£o um.

## O que roda sem modelo

Das quatro fases, **sÃ³ a de texto usa LLM**. As outras trÃªs sÃ£o funÃ§Ã£o:

| fase | como roda |
|---|---|
| texto | agente â€” escreve os 3 roteiros por pÃºblico |
| avatar (`\| estudio`) | **script** Playwright no estÃºdio do HeyGen (~50s/pÃºblico) |
| baixar | funÃ§Ã£o â€” acha pelo TÃTULO e baixa |
| reel | **funÃ§Ã£o** â€” `scripts/montar-reel.py`, e entrega no canal |

Medido no promoavatar antes do porte: o custo por vÃ­deo caiu de **US$ 3,09 para
US$ 0,18** quando avatar e reel deixaram de ser agente. O detalhamento estÃ¡ em
`inemaccbot/docs/custo-por-fase-a19-a29.md`.

## O motor do reel mora aqui

`scripts/montar-reel.py` encadeia: preparar â†’ portÃ£o 1 (lint + ritmo) â†’ render â†’
revisor â†’ CTA â†’ QC. Nomes fixos (`motion/corpo.mp4`, `final/reel.mp4`,
`qc/mosaico.png`), exit 3 quando um portÃ£o reprova.

**O portÃ£o de entrada Ã© a seÃ§Ã£o `## IMAGENS` do texto** (regra 11b do prompt da
fase 1): uma linha por SEGMENTO da fala, com `headline:` e `hook:`. Sem ela o
`preparar.py` sai com exit 3 e o reel nÃ£o Ã© montado â€” nÃ£o Ã© preferÃªncia de
estilo, Ã© o que impede o reel de sair com painel vazio ou headline inventada.

Os layouts estÃ£o em `templates/` e o padrÃ£o Ã© o `template` da raiz do
`flow.json`; um alvo pode cravar o seu.

## AtenÃ§Ã£o: tudo aqui Ã© congelado na criaÃ§Ã£o

`flow.json` e o prompt sÃ£o copiados para dentro do fluxo no momento em que ele
nasce. Editar aqui sÃ³ afeta fluxos NOVOS â€” um fluxo em andamento nÃ£o muda de
regra no meio. Para valer no que jÃ¡ existe, crie outro.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe