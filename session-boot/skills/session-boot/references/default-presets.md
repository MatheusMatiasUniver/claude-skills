# Presets padrão do Session Boot

Estes são os presets embutidos no plugin. O usuário pode sobrescrevê-los ou criar novos em
`<projeto>/.claude/session-boot/presets.json` (mesmo formato JSON, mesma estrutura de chaves).
Presets per-project têm precedência sobre estes quando os nomes colidem.

## Formato

Cada preset é uma chave no objeto JSON com:
- `description` — frase curta descrevendo o preset
- `skills` — array de objetos `{ "skill": "nome", "args": "argumentos opcionais" }`

## Presets

```json
{
  "produtividade": {
    "description": "Foco em produtividade: delegação ao Codex + brainstorming estruturado",
    "skills": [
      { "skill": "codex-delegator" },
      { "skill": "superpowers:brainstorming" }
    ]
  },
  "aprendizado": {
    "description": "Foco em aprender: tutoria + verificação antes de completar",
    "skills": [
      { "skill": "learning-copilot", "args": "medio" },
      { "skill": "superpowers:verification-before-completion" }
    ]
  },
  "full-stack": {
    "description": "Stack completa: delegação + tutoria + brainstorming",
    "skills": [
      { "skill": "codex-delegator" },
      { "skill": "learning-copilot" },
      { "skill": "superpowers:brainstorming" }
    ]
  }
}
```
