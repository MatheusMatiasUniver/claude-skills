# Persistência do Learning Copilot

Tudo fica dentro de `.claude/learning-copilot/` no **projeto atual** (a pasta do código em que o usuário está, não o repositório da skill). Crie a pasta se não existir.

```
<projeto>/.claude/learning-copilot/
├── state.json        # configuração: se está ativa e em que nível
├── study-log.md      # memória das lições, alimentada automaticamente
└── .visual/          # HTML gerado pelo /estudo (via md-to-visual-html)
```

---

## `state.json`

Guarda o estado da skill para o projeto. É lido no Passo 0 (guard) de toda invocação.

```json
{
  "active": true,
  "level": "medio",
  "activated_at": "2026-05-23",
  "updated_at": "2026-05-23"
}
```

- `active`: sempre `true` enquanto a skill estiver ativada no projeto. A existência do arquivo já é o sinal de "ativada"; remover o arquivo é o que realmente "desinstala" do projeto.
- `level`: um de `"baixo"`, `"medio"`, `"alto"`, `"desativado"`.
  - `desativado` ≠ remover o arquivo. Em `desativado` a skill continua ativada (o `/estudo` funciona), mas não tutora.
- `activated_at`: data da primeira ativação (não sobrescreva em trocas de nível).
- `updated_at`: data da última mudança de nível.

Ao trocar de nível por frase natural ("modo alto", "desativa o copilot"), reescreva só `level` e `updated_at`.

---

## `study-log.md`

Memória append-only das lições. Alimentada **automaticamente em todos os níveis** (inclusive `desativado`). É a fonte que o `/estudo` consolida em HTML — por isso precisa sobreviver entre chats.

Cabeçalho ao criar o arquivo:

```markdown
# Log de estudo — <nome do projeto>

Registro automático do Learning Copilot. Cada entrada é uma lição.
Gere o HTML de revisão com `/learning-copilot:estudo`.
```

Formato de cada entrada (append no fim):

```markdown
## <AAAA-MM-DD> — <tema curto>

- **O que foi feito:** <1–2 linhas objetivas>
- **Por quê:** <a decisão técnica e o raciocínio; alternativa descartada, se houve>
- **Conceito(s):** <nome do conceito + explicação curta reusável>
- **Revisar:** <pergunta ou ponto pra fixar depois> (opcional)
```

Regras:
- Append leve e silencioso — não anuncie cada gravação nem peça confirmação.
- Uma entrada por lição/decisão relevante, não por mensagem. Agrupe quando fizer sentido.
- No nível `desativado`, registre só **decisões arquiteturais/técnicas e conceitos da tecnologia** (preencha "O que foi feito", "Por quê" e "Conceito(s)"; o campo "Revisar" é opcional e normalmente fica de fora, já que não há tutoria).
- Se o `study-log.md` não existir quando for anotar, crie-o com o cabeçalho primeiro.
