---
name: session-boot
description: >
  Bootstrap de sessão: ativa múltiplas skills de uma vez por preset nomeado ou lista explícita.
  Use SEMPRE que o usuário digitar `/session-boot` seguido de um nome de preset OU de uma lista
  de skills separadas por vírgula. Também use quando o usuário pedir pra "iniciar sessão",
  "carregar skills", "ativar o modo X" referindo-se a um conjunto de skills. Esta skill NÃO age
  sozinha — só é invocada explicitamente.
---

# Session Boot — Bootstrap de sessão

Ponto de entrada único para preparar uma sessão do Claude Code. Em vez de invocar cada skill
manualmente (`/skill-a`, `/skill-b args`, `/skill-c`), o usuário roda um comando e todas as
skills necessárias são carregadas no contexto da conversa.

---

## Como usar

- `/session-boot <preset>` — ativa um preset (grupo nomeado de skills)
- `/session-boot skill1, skill2 args, skill3` — ativa skills avulsas com parâmetros opcionais
- `/session-boot` (sem args) — lista os presets disponíveis

---

## Interpretação dos argumentos

Ao receber `$ARGUMENTS`:

1. **Vazio** → liste os presets disponíveis (mesmo comportamento de `/session-boot:presets`) e pare.

2. **Tente resolver como preset:**
   - Leia `<projeto>/.claude/session-boot/presets.json` (se existir). Se a chave `$ARGUMENTS` existir nele, use esse preset.
   - Senão, consulte os presets padrão em `references/default-presets.md`. Se a chave existir, use-o.
   - Se encontrou o preset, expanda para a lista de skills dele.

3. **Se não é preset, trate como lista de skills:**
   - Separe por vírgula: `"codex-delegator, learning-copilot alto"` → `["codex-delegator", "learning-copilot alto"]`.
   - Para cada item, trim espaços e separe pelo primeiro espaço: o que vem antes é o nome da skill, o que vem depois são os args.
   - Ex: `"learning-copilot alto"` → skill: `learning-copilot`, args: `alto`.

4. **Não encontrou nada (nem preset, nem parece lista):**
   - Sugira os presets disponíveis e pergunte se quis dizer algum deles.

---

## Protocolo de ativação

Para cada skill na lista resolvida, **na ordem**, faça:

1. **Verifique duplicatas** — se essa skill já foi ativada nesta sequência, pule.
2. **Verifique auto-invocação** — nunca inclua `session-boot` na lista. Ignore silenciosamente.
3. **Invoque a skill** chamando a ferramenta `Skill` com:
   - `skill`: o nome completo da skill (ex: `"codex-delegator"`, `"superpowers:brainstorming"`)
   - `args`: os argumentos da skill, ou string vazia se não houver
4. **Se carregou com sucesso**, anote `status: "ok"`.
5. **Se falhou** (skill não encontrada, erro de carregamento), anote `status: "falhou"`, avise o usuário em uma linha curta (ex: "⚠ skill 'xyz' não encontrada, pulando") e **continue** com as próximas. Não aborte a sequência.

### Após todas as skills

1. **Grave o estado** em `<projeto>/.claude/session-boot/last-session.json` (crie a pasta `.claude/session-boot/` se necessário):

```json
{
  "preset": "nome-do-preset ou null se foi lista avulsa",
  "skills_activated": [
    { "skill": "codex-delegator", "args": null, "status": "ok" },
    { "skill": "learning-copilot", "args": "alto", "status": "ok" }
  ],
  "activated_at": "AAAA-MM-DDTHH:MM:SSZ"
}
```

2. **Resuma** o que foi ativado — uma tabela ou lista curta com o nome de cada skill e seu status. Seja breve: o objetivo é confirmar, não explicar o que cada skill faz.

Exemplo de resumo:
> **Sessão iniciada** (preset: full-stack)
> - ✅ codex-delegator
> - ✅ learning-copilot (medio)
> - ⚠ skill-inexistente — não encontrada, pulada

---

## Comunicação

- Português brasileiro, tom direto.
- Uma linha de confirmação por skill carregada — não explique o que cada skill faz.
- Resumo final curto.
- Se houve falhas, mencione quais e sugira verificar se estão instaladas.
