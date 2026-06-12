---
description: Lista todos os presets disponíveis (padrão + per-project), mostrando nome, descrição e skills de cada um.
---

# /session-boot:presets — listar presets

Liste todos os presets disponíveis para o usuário, mesclando defaults com per-project.

## Passos

1. **Carregue os presets padrão** de `references/default-presets.md` (o bloco JSON dentro dele).

2. **Carregue os presets per-project** de `<projeto>/.claude/session-boot/presets.json`, se o arquivo existir. Se não existir ou estiver malformado, avise e use só os defaults.

3. **Mescle** — presets per-project sobrescrevem defaults com o mesmo nome.

4. **Exiba** uma tabela ou lista com:
   - **Nome** do preset
   - **Descrição**
   - **Skills** (nomes, com args se houver)
   - **Origem** (padrão / projeto)

Exemplo de saída:

| Preset | Descrição | Skills | Origem |
|--------|-----------|--------|--------|
| produtividade | Delegação + brainstorming | codex-delegator, superpowers:brainstorming | padrão |
| aprendizado | Tutoria + verificação | learning-copilot (medio), superpowers:verification-before-completion | padrão |
| meu-preset | Meu workflow | codex-delegator, learning-copilot (alto) | projeto |

5. **Dica final:** "Para ativar: `/session-boot <nome>`. Para criar um preset: `/session-boot:save <nome>` após um boot."
