# Changelog

Histórico legível do projeto: o que mudou e **por quê**. Mais recente primeiro.
Regras de escrita em `AGENTS.md`.

## 2026-08-24

### fix(vps-mapper): reestrutura plugin para o padrão do marketplace

O plugin `vps-mapper` tinha `skills/vps-mapper/SKILL.md` e `references/` prontos
mas sem `.claude-plugin/plugin.json`, ficando fora do padrão usado por
`codex-delegator`, `learning-copilot` e `session-boot`. Criado o `plugin.json`
(schema idêntico aos demais: `name`, `description`, `version: 0.1.0`, `author`)
e confirmada a consistência do `name` entre o frontmatter do `SKILL.md`, o
`plugin.json` e a entrada já existente em `.claude-plugin/marketplace.json`.

_Por quê:_ um plugin sem `plugin.json` não é instalável/validável pelo
`claude plugin`, mesmo já registrado no marketplace — a entrada no
`marketplace.json` aponta pro diretório, mas é o manifesto do plugin que o
CLI valida e usa para carregar as skills.
