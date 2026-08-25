---
name: vps-mapper
description: Mapeia e documenta um servidor Linux/VPS inteiro em modo somente-leitura, gerando documentação markdown organizada por camada (serviços, rede, containers, storage, agendamentos, configs). Use SEMPRE que o usuário pedir para "mapear o servidor", "documentar a VPS", "entender o que está rodando na máquina", "descobrir quais serviços/portas/containers existem", "me perdi na minha própria VPS", "fazer um inventário da infra", ou pedir para atualizar/re-rodar a documentação de infraestrutura existente. Use também quando ele estiver debugando algo e não souber onde um serviço mora, qual porta usa ou onde os dados persistem. NÃO use para auditoria de segurança/CVEs nem para infraestrutura como código (Terraform/Ansible) — esta skill inspeciona a máquina viva.
---

# VPS Mapper

Mapear um servidor que cresceu organicamente é um problema de **inventário**, não de execução. O objetivo aqui é produzir um mapa confiável e re-gerável de tudo que existe na máquina, sem nunca alterar o estado dela.

## Regra fundamental: somente leitura

Esta skill opera em modo estritamente somente-leitura. Isso não é preciosismo — é o que torna a varredura segura de rodar num servidor de produção sem janela de manutenção.

**Nunca execute**, em nenhuma hipótese, durante o mapeamento:

- Qualquer coisa que altere estado de serviço: `systemctl start|stop|restart|reload|enable|disable`, `service ... restart`, `docker start|stop|restart|rm|prune`
- Instalação ou remoção de pacotes: `apt install|remove|upgrade|autoremove`, `snap install`, `npm i -g`, `pip install`
- Escrita ou remoção de arquivos fora do diretório de saída da documentação: `rm`, `mv`, `chmod`, `chown`, `truncate`, `>` e `>>` em qualquer caminho do sistema
- Alterações de rede/firewall: `ufw allow|deny|enable`, `iptables -A|-D`, `nft add|delete`
- Operações em banco de dados que não sejam consultas de metadados
- `apt update` — parece inofensivo, mas escreve em `/var/lib/apt` e pode disparar hooks

Se durante a varredura ficar claro que alguma informação só é obtível com um comando que altera estado, **pare e pergunte ao usuário** em vez de executar. Descrever o que você precisaria rodar e por quê é sempre melhor do que rodar.

A única escrita permitida é a criação dos arquivos de documentação no diretório de saída acordado com o usuário.

## Antes de começar

Alinhe três coisas com o usuário, porque cada uma muda o resultado:

1. **Diretório de saída** da documentação. Sugestão padrão: `./infra-docs/` no diretório atual, ou `~/projetos/infra-docs/`. Nunca escreva em `/etc`, `/opt` ou `/srv`.
2. **Escopo de `sudo`**. Boa parte do mapeamento útil (configs em `/etc`, processos de outros usuários, regras de firewall) exige leitura privilegiada. Pergunte se ele quer rodar com `sudo` nos comandos de leitura ou se prefere documentar apenas o que é visível como usuário comum, marcando as lacunas. Não assuma.
3. **Profundidade**. Varredura completa (todas as camadas) ou só camadas específicas. Numa máquina grande, a varredura completa gera bastante output — vale confirmar.

## Fluxo de trabalho

Colete e documente **uma camada por vez**, escrevendo o arquivo de cada camada antes de passar para a próxima. Isso importa por dois motivos: o output de cada camada é grande e some do contexto se você acumular tudo antes de escrever; e depois, quando o usuário mudar só uma coisa no servidor, ele consegue re-rodar apenas a camada afetada em vez da varredura inteira.

As camadas, na ordem recomendada (cada uma dá contexto para a seguinte):

| # | Camada | Arquivo de saída | Responde à pergunta |
|---|--------|------------------|---------------------|
| 1 | Sistema base | `01-sistema.md` | Que máquina é esta? |
| 2 | Serviços | `02-servicos.md` | O que está rodando? |
| 3 | Containers | `03-containers.md` | O que roda em Docker? |
| 4 | Rede e portas | `04-rede.md` | O que está exposto e como? |
| 5 | Storage e dados | `05-storage.md` | Onde os dados moram? |
| 6 | Agendamentos | `06-agendamentos.md` | O que roda sozinho? |
| 7 | Aplicações | `07-aplicacoes.md` | Que projetos vivem aqui? |

Depois de todas as camadas, escreva o índice `INFRA.md` (ver "Índice geral" abaixo).

O catálogo completo de comandos de leitura por camada está em `references/comandos-leitura.md`. Leia esse arquivo antes de iniciar a coleta e use-o como fonte dos comandos — ele foi montado para conter apenas comandos que não alteram estado.

## Como interpretar, não só transcrever

Despejar a saída bruta de `systemctl list-units` num arquivo markdown não resolve o problema do usuário — ele já podia rodar esse comando. O valor da documentação está na **correlação entre camadas**.

Ao escrever cada arquivo, faça o trabalho que só é possível vendo o conjunto:

- Amarre a porta ao processo, o processo à unit systemd, a unit ao diretório da aplicação, e o diretório ao repositório git (se houver). Uma porta 3000 aberta é ruído; "porta 3000 → `node server.js` → unit `api-x.service` → `/opt/api-x` → repo `github.com/user/api-x`" é o mapa que ele precisa.
- Marque explicitamente o que está **exposto para a internet** versus escutando só em `127.0.0.1`. Essa distinção é o dado mais frequentemente esquecido e o mais consequente.
- Sinalize o que parece **órfão ou esquecido**: units habilitadas que falham, containers parados há meses, cron jobs apontando para scripts inexistentes, diretórios grandes sem dono aparente. Não conserte nada — apenas registre como observação.
- Onde houver ambiguidade genuína (um processo que você não conseguiu atribuir a nada), escreva isso no documento em vez de inventar uma explicação. Um "não identificado" honesto vale mais que um palpite plausível.

## Estrutura de cada arquivo de camada

Use este esqueleto para manter os arquivos comparáveis entre si e re-geráveis:

```markdown
# [Nome da camada]

> Gerado em: AAAA-MM-DD · Host: [hostname] · Coletado [com|sem] sudo

## Resumo
[2-4 linhas: o que existe nesta camada, em linguagem direta]

## Inventário
[Tabelas ou seções por item, com as correlações entre camadas]

## Observações
[Coisas que chamaram atenção: órfãos, exposições, inconsistências]

## Lacunas
[O que não foi possível determinar e por quê — permissão, comando indisponível, ambiguidade]

## Comandos usados
[Lista dos comandos de leitura executados, para o usuário reproduzir a coleta]
```

A seção "Comandos usados" existe para que o usuário possa auditar o que foi rodado e re-executar a coleta sozinho depois. A seção "Lacunas" evita que uma ausência silenciosa vire uma falsa sensação de completude.

## Índice geral

Ao final, escreva `INFRA.md` no diretório de saída com:

- Uma descrição de 3-5 linhas do que essa máquina é e para que serve, escrita a partir do que foi encontrado
- Um mapa em texto ou tabela ligando **domínio/porta → serviço → diretório → dados**, que é a visão que resolve o "me perdi na minha VPS"
- Links para cada arquivo de camada com uma linha explicando o que cada um cobre
- Uma seção "Pontos de atenção" consolidando as observações mais relevantes de todas as camadas
- A data da varredura e como re-rodá-la

## Re-execução

Quando o usuário pedir para atualizar a documentação, pergunte o que mudou e re-rode só as camadas afetadas, preservando as demais. Atualize a data no arquivo tocado e no `INFRA.md`. Se ele não souber o que mudou, uma varredura completa é aceitável — mas mencione que rodar por camada é mais rápido e mais fácil de revisar no diff.

## Sobre permissões da sessão

Esta skill foi desenhada para rodar com o sistema de permissões do agente **ativo** (modo padrão). Os comandos de leitura vão gerar prompts de aprovação, e isso é a segunda camada de proteção: se algo fora do catálogo somente-leitura for proposto, o usuário vê antes de rodar.

Se o usuário mencionar que está rodando em modo de bypass de permissões, avise que a rede de segurança externa não está ativa e que as regras deste documento passam a ser a única proteção — e ofereça configurar um hook `PreToolUse` que bloqueie comandos de escrita.
