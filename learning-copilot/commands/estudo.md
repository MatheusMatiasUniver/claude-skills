---
description: Gera um material de estudo em HTML (o que foi feito, por quê e os conceitos da linguagem) a partir do log do Learning Copilot e da conversa atual.
argument-hint: "[tema opcional]"
---

# /learning-copilot:estudo — material de estudo em HTML

Você foi chamado explicitamente para gerar o material de estudo. Este é o **único** caminho que produz HTML — fora daqui, nunca gere automaticamente.

## Passos

1. **Reúna o conteúdo** de duas fontes:
   - `.claude/learning-copilot/study-log.md` do projeto, se existir (a memória acumulada entre chats).
   - O que foi feito e ensinado **nesta conversa** (decisões técnicas, conceitos, bugs e suas causas).
   Se o usuário passou um tema em `$ARGUMENTS`, foque o material nesse tema.

   Se não houver log nem conteúdo relevante na conversa, diga isso ao usuário e sugira ativar a skill (`/learning-copilot`) para acumular lições — não invente conteúdo.

2. **Monte o markdown de estudo** em `.claude/learning-copilot/estudo-AAAA-MM-DD.md` (crie a pasta se preciso), com esta estrutura:

```markdown
# Estudo — <tema / projeto> (<AAAA-MM-DD>)

## O que foi feito
Resumo objetivo das mudanças e decisões.

## Por que foi feito assim
As decisões técnicas e o raciocínio por trás de cada uma, incluindo
alternativas descartadas e o motivo.

## Conceitos técnicos da linguagem
Cada conceito relevante (sintaxe, padrão, recurso da linguagem/framework)
explicado de forma reusável em outro contexto. Exemplos curtos quando ajudar.

## Para revisar depois
Perguntas e pontos que valem revisitar para fixar.
```

3. **Gere o HTML** deixando a skill `md-to-visual-html` converter o markdown recém-criado — ela produz um HTML autocontido (CSS inline, diagramas Mermaid) em `.claude/learning-copilot/.visual/estudo-AAAA-MM-DD.html`. Se ela não disparar sozinha, invoque-a explicitamente. **Não escreva HTML à mão.**

4. **Avise** o caminho do HTML e ofereça abrir. No WSL, lembre que dá pra abrir com:
   `explorer.exe "$(wslpath -w <caminho-do-html>)"`

## Observações

- Funciona em qualquer nível, **inclusive desativado** — é a ponte entre "codar rápido agora" e "estudar com calma depois".
- O conteúdo deve explicar o **porquê** das decisões, não só o **como** — é material de estudo, não changelog.
