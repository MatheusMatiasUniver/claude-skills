# Minhas Skills do Claude Code

Skills pessoais para uso no Claude Code CLI.

## Instalar

```
/plugin marketplace add seu-usuario/skills
/plugin install learning-copilot@skills
```

## learning-copilot

Tutor de programação **opt-in por projeto**: ele fica dormente até você ativá-lo. Em projetos onde você nunca o chamou, ele nunca interrompe.

### Ativar (uma vez por projeto)

```
/learning-copilot           # ativa no nível médio (padrão)
/learning-copilot alto      # ativa já no nível alto
```

Isso grava o estado em `.claude/learning-copilot/state.json`. A partir daí ele passa a ajudar nos pedidos de código **daquele projeto** — e um chat novo no mesmo projeto já lembra o nível.

### Níveis (trocados por frase natural, e persistidos)

| Nível | Comportamento | Como trocar |
|---|---|---|
| **Baixo** | Explica bastante as decisões técnicas, interrompe pouco. | "modo baixo", "explica mais e pergunta menos" |
| **Médio** *(padrão)* | Explicações mais profundas + perguntas frequentes pra fixar. | "modo médio", "volta pro padrão" |
| **Alto** | Sempre cobra entendimento e incentiva você a escrever o código no VSCode. | "modo alto", "me cobra", "quero escrever eu mesmo" |
| **Desativado** | Sem tutoria nem interrupções — código completo direto (mas o `/estudo` continua valendo). | "desativa o copilot", "modo livre" |

> "Desativado" ≠ desinstalar: a skill segue ativada no projeto (o material de estudo continua disponível). Para remover de vez do projeto, apague `.claude/learning-copilot/`.

### Material de estudo em HTML

```
/learning-copilot:estudo    # gera o HTML de revisão
```

Durante as sessões nos níveis de aprendizado, o copilot vai anotando as lições em `.claude/learning-copilot/study-log.md`. O comando `/learning-copilot:estudo` consolida esse log + o que foi feito na conversa num HTML autocontido (via `md-to-visual-html`), explicando **o que foi feito, por quê e os conceitos da linguagem**. O HTML só é gerado quando você pede.