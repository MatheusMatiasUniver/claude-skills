---
description: Salva a última sessão ativada como um preset nomeado no projeto atual.
argument-hint: "<nome-do-preset>"
---

# /session-boot:save — salvar preset

Salva a lista de skills da última ativação como um preset reutilizável no projeto atual.

## Passos

1. **Verifique `$ARGUMENTS`** — o nome do preset é obrigatório. Se vazio, peça o nome e pare.

2. **Leia `<projeto>/.claude/session-boot/last-session.json`** para obter a lista de skills da última ativação.
   - Se o arquivo não existir, avise que nenhuma sessão foi iniciada ainda e sugira rodar `/session-boot` primeiro.

3. **Monte o preset** a partir de `last-session.json`:
   - Use apenas as skills com `status: "ok"` (ignore as que falharam).
   - Peça uma descrição curta ao usuário (uma frase) ou sugira uma baseada nas skills.

4. **Leia (ou crie) `<projeto>/.claude/session-boot/presets.json`:**
   - Se o arquivo já existir, leia-o e adicione/sobrescreva a chave com o nome dado.
   - Se não existir, crie-o com o preset como única entrada.
   - Se o nome colidir com um preset padrão, avise que o per-project vai sobrescrevê-lo e pergunte se quer continuar.

5. **Grave** o arquivo atualizado.

6. **Confirme:** "Preset **nome** salvo com N skills. Para usar: `/session-boot nome`."
