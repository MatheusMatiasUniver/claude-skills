---
name: codex-delegator
description: >
  Orquestra tarefas de desenvolvimento no Claude Code delegando AUTOMATICAMENTE ao Codex (plugin
  `codex-plugin-cc`, subagente `codex:codex-rescue`) os pedaços que o Codex faz melhor — trabalho
  mecânico, bem-delimitado e verificável — para poupar o orçamento de tokens/raciocínio do Claude
  para as partes difíceis. Use SEMPRE que estiver tocando uma tarefa de implementação de mais de um
  passo: planejar, quebrar em pedaços, decidir o que delega e o que o Claude faz, escrever a spec do
  que vai pro Codex, disparar a delegação e revisar/integrar o resultado. NÃO delegue o que exige
  julgamento: arquitetura, design/UX, ambiguidade, trade-offs, integração entre arquivos, debugging
  de causa desconhecida — é onde o token do Claude vale a pena.
---

# Codex Delegator

Skill de **orquestração com delegação automática**. Numa tarefa de implementação, o Claude age como
engenheiro-líder: pensa o todo, mas **empurra pro Codex os pedaços mecânicos e bem-especificados** em
vez de gastar o próprio contexto neles. O objetivo é econômico — preservar o orçamento de
tokens/raciocínio do Claude pras partes que exigem pensamento profundo, e deixar o braçal (ler/escrever
muitos arquivos, loops de implementar-e-testar) rodar na sessão do Codex.

## Por que isso economiza (o racional)

Quando o Claude implementa um pedaço sozinho, ele paga token por tudo: carregar arquivos, gerar o diff,
rodar testes, corrigir, repetir. Se o pedaço é mecânico e bem-definido, isso é orçamento caro gasto em
trabalho barato. Delegando ao Codex:

- o braçal acontece na **sessão do Codex**, não no contexto do Claude;
- o Claude só paga pelo que é insubstituível — **planejar, especificar e revisar**;
- o consumo conta no teu limite da OpenAI/ChatGPT, não no da Anthropic (o `rescue` foi feito pra isso);
- com `--background`, o Codex grinda enquanto o Claude segue raciocinando em paralelo.

Regra de bolso: **se o pedaço é caro em token e pobre em julgamento, delega.**

## Loop de orquestração

```
1. PLANEJAR     -> Claude quebra a tarefa em pedaços (esse raciocínio fica com o Claude)
2. TRIAR        -> cada pedaço: Codex-shaped (mecânico+verificável) ou Claude-shaped (julgamento)?
3. ESPECIFICAR  -> pros Codex-shaped, Claude escreve a spec exata (o proxy é fino: só vai o que escrever)
4. DELEGAR      -> Claude dispara o codex-rescue sozinho (automático; --background pra paralelizar)
5. REVISAR      -> Claude confere diff + critério de pronto, integra
6. repete; o pensamento profundo e os pedaços Claude-shaped o Claude faz ele mesmo
```

O Claude **anuncia** cada delegação em uma linha ("delegando ao Codex: schema Zod de X") pra você ter
visibilidade e poder vetar — mas não pede permissão a cada passo; a delegação é automática.

> Mecânica: a skill NÃO intercepta um `/codex:rescue` que você digita. Ela faz o contrário — o próprio
> Claude roteia a tarefa pro subagente `codex:codex-rescue` (delegação em linguagem natural, o mesmo
> destino do `/codex:rescue`). É isso que torna a delegação automática, sem você digitar o comando.

---

## 1. Triagem (o que vai pro Codex)

Delega quando as DUAS coisas valem:

**(a) É Codex-shaped** — mecânico, bem-delimitado, com done-condition verificável:
- CRUD/endpoints com contrato definido, DTOs, mappers, schemas (Zod/Prisma), migrations
- boilerplate repetitivo, scaffolding e testes pra comportamento já definido
- refactor com alvo explícito ("renomeia X em todos os usos", "extrai Y de Z")
- conversões/portes mecânicos

**(b) Vale o token** — caro pro Claude fazer (muitos arquivos, diff longo, loop de teste) mas pobre em
julgamento. Ajuste de uma linha não compensa o overhead da delegação; faz direto.

Fica com o **Claude** (não delega):
- arquitetura, design de API, escolha de stack/abordagem
- frontend/UX e qualquer coisa que peça taste
- ambiguidade, exploração, entender um fluxo desconhecido
- debugging de causa desconhecida (delega só depois que causa+fix viram spec)
- integração que cruza muitos arquivos com decisão em cada junção
- trade-offs de segurança/produto

> Camada não é o critério — clareza + custo são. Backend mecânico é o caso típico; backend exploratório
> fica com o Claude; um front trivial e fechado pode ir pro Codex.

Pedaço Codex-shaped mas ainda ambíguo: o Claude **primeiro** fecha a ambiguidade (essa parte vale o
token dele) e só então especifica e delega a execução.

---

## 2. Spec (o que o Codex recebe é só isto)

O subagente `codex-rescue` é um **proxy fino**: não lê o repo, não preenche lacuna — só repassa teu
prompt ao Codex. A spec é o handoff inteiro. Formato:

```markdown
## Objetivo
[1 frase: estado final]
## Contexto
- Stack: ...
- Arquivos relevantes: caminho — [o que tem / por que importa]
- Padrão a seguir: [ex: "igual a src/services/userService.ts"]
## Requisitos (exatos)
1. ...
## Restrições
- NÃO alterar: ...
- Só libs já no projeto; sem dependência nova sem necessidade
## Critério de pronto (DoD)
- [ ] [verificável]
- [ ] testes passam: `npm test`
```

Ouro: caminhos concretos; contrato/tipos definidos (não deixa o Codex escolher); DoD = teste mental
checável. Se você não consegue checar o DoD objetivamente, o pedaço ainda não está pronto pra delegar.

---

## 3. Delegação (automática, via plugin)

O Claude dispara a delegação ele mesmo, roteando pro subagente `codex:codex-rescue` (mesmo destino do
`/codex:rescue`, sem você digitar). Passa a spec inteira como prompt.

- `--background` — **default pra pedaço grande**: Codex grinda enquanto o Claude segue. Acompanha com
  `/codex:status`, colhe com `/codex:result`, aborta com `/codex:cancel`.
- `--wait` — quando o próximo passo do Claude depende do resultado.
- `--resume` — continua a thread anterior do Codex.
- `--model <m>` / `--effort <nível>` — modelo e esforço conforme a dificuldade.

Só-leitura/revisão (não altera código): `/codex:review` ou `/codex:adversarial-review`.

**Paralelize:** pedaços Codex-shaped independentes vão em `--background` ao mesmo tempo, enquanto o
Claude trabalha um pedaço Claude-shaped. É aí que a economia aparece de verdade.

---

## 4. Revisão e integração (não confia cego)

O proxy é fino e o Codex erra com confiança. Pra cada resultado (background: pega com `/codex:result`):

1. `git diff` — só mexeu no que devia? (checa as Restrições)
2. Bate o DoD item a item.
3. Roda os testes da spec.
4. Integra. Desvio pequeno -> Claude ajusta. Desvio grande -> re-delega com a spec mais apertada
   (faltou restrição ou caminho), não repete a mesma spec.

Revisar custa muito menos token que implementar — é por isso que o saldo fecha a favor do Claude.
Nunca commitar resultado do Codex sem essa passada.

---

## Exemplos

**Delega (Codex-shaped + caro):** "implementa o CRUD de /agendamentos (rotas, service, validação Zod,
testes) no padrão de /clientes" -> muitos arquivos, padrão definido, DoD claro. Claude planeja, escreve
a spec, delega em `--background`, segue noutro pedaço, depois revisa o diff.

**Não delega (Claude-shaped):** "o agendamento tá com bug intermitente" -> causa desconhecida. Claude
investiga (vale o token dele); só quando causa+fix viram spec é que delega a aplicação do fix.

**Híbrido (o caso comum):** "monta o módulo de notificações" -> Claude decide arquitetura e contrato
(Claude-shaped) e delega ao Codex os adaptadores/DTOs/testes mecânicos (Codex-shaped), em paralelo.

---

## Anti-padrões

- Delegar pedaço ainda ambíguo — o proxy não resolve; volta pior. Fecha a ambiguidade antes.
- Delegar micro-tarefa cujo overhead supera o que economiza.
- Spec sem caminhos concretos, ou deixando o Codex escolher lib/padrão/contrato.
- Delegar julgamento (arquitetura/design/segurança) só porque "é código".
- Aceitar resultado sem `git diff` + DoD.
