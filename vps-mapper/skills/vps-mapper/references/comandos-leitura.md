# Catálogo de comandos somente-leitura

Todos os comandos abaixo apenas leem estado. Nenhum escreve, instala, remove ou reinicia nada.

Convenções:
- `[sudo]` marca comandos que dão resultado bem mais completo com privilégio, mas rodam (parcialmente) sem ele. Use `sudo` apenas se o usuário autorizou.
- Se um comando não existir na máquina (`command not found`), registre como lacuna e siga — não instale nada.
- Prefira sempre a variante que limita o output. Uma VPS pode ter milhares de arquivos; despejar tudo é inútil e estoura contexto.

## Índice

1. [Sistema base](#1-sistema-base)
2. [Serviços](#2-serviços)
3. [Containers](#3-containers)
4. [Rede e portas](#4-rede-e-portas)
5. [Storage e dados](#5-storage-e-dados)
6. [Agendamentos](#6-agendamentos)
7. [Aplicações](#7-aplicações)
8. [Bancos de dados](#8-bancos-de-dados)

---

## 1. Sistema base

```bash
hostnamectl                      # hostname, SO, kernel, virtualização
uname -a                         # kernel
cat /etc/os-release              # distribuição e versão
uptime                           # tempo ligado e carga
nproc && free -h                 # CPU e memória
timedatectl                      # timezone e sincronização de hora
who -a                           # sessões ativas
cut -d: -f1,3,6,7 /etc/passwd    # usuários do sistema e seus shells
getent group sudo docker         # quem tem privilégio elevado
[sudo] ls -la /root              # o que existe no home do root
```

O `hostnamectl` já revela se a máquina é KVM, container LXC ou bare metal, o que muda o que faz sentido procurar nas camadas seguintes.

## 2. Serviços

```bash
systemctl list-units --type=service --state=running --no-pager
systemctl list-unit-files --state=enabled --no-pager   # habilitados no boot
systemctl list-units --state=failed --no-pager         # units quebradas
systemctl cat <unit>                                   # definição completa de uma unit
systemctl show <unit> -p FragmentPath,ExecStart,WorkingDirectory,User
ps aux --sort=-%mem | head -30                         # maiores consumidores
[sudo] journalctl -u <unit> -n 30 --no-pager           # últimas linhas de log
```

`systemctl cat` é o comando mais valioso desta camada: revela `ExecStart` e `WorkingDirectory`, que ligam o serviço ao diretório da aplicação. Rode nas units que não forem do sistema base.

Para separar o que é do sistema do que é do usuário, ignore units que começam com `systemd-`, `dbus`, `cron`, `ssh`, `polkit`, `apparmor` — a menos que estejam falhando.

## 3. Containers

```bash
docker ps -a --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
docker images --format 'table {{.Repository}}\t{{.Tag}}\t{{.Size}}'
docker volume ls
docker network ls
docker inspect <container>          # env, mounts, restart policy, rede
docker compose ls                   # stacks compose ativas
docker stats --no-stream            # uso de recursos, snapshot único
```

Ao inspecionar containers, extraia especialmente: **mounts** (onde os dados persistem no host), **portas publicadas** (`0.0.0.0:` = exposto; `127.0.0.1:` = local) e **restart policy**. Cuidado ao registrar variáveis de ambiente: podem conter senhas e chaves — documente os *nomes* das variáveis, nunca os valores.

Localize os `docker-compose.yml` para entender a intenção original das stacks:

```bash
find / -name 'docker-compose*.y*ml' -not -path '*/node_modules/*' 2>/dev/null | head -40
```

## 4. Rede e portas

```bash
ss -tulpn                           # portas escutando + processo dono
[sudo] ss -tulpn                    # com sudo mostra o processo de todas as portas
ip -brief addr                      # interfaces e IPs
ip route                            # rotas
[sudo] ufw status verbose           # firewall ufw
[sudo] iptables -L -n -v            # regras iptables
[sudo] nft list ruleset             # regras nftables
[sudo] nginx -T                     # TODA a config nginx resolvida (inclui includes)
ls -la /etc/nginx/sites-enabled/    # virtual hosts ativos
[sudo] apachectl -S                 # virtual hosts apache
[sudo] certbot certificates         # certificados TLS e validade
cat /etc/hosts                      # mapeamentos locais
```

`nginx -T` é o comando central desta camada: resolve todos os `include` e mostra a configuração efetiva, incluindo cada `server_name` e `proxy_pass`. É o que permite ligar domínio → porta interna → aplicação.

Ao listar portas, classifique cada uma como **exposta** (`0.0.0.0` ou IP público) ou **local** (`127.0.0.1` / `::1`). Cruze com as regras de firewall: uma porta em `0.0.0.0` bloqueada pelo ufw não está de fato acessível, e essa nuance precisa aparecer na documentação.

## 5. Storage e dados

```bash
df -h                                        # uso por filesystem
du -sh /var/* /opt/* /srv/* 2>/dev/null      # maiores diretórios de sistema
du -sh /home/*/ 2>/dev/null
lsblk                                        # discos e partições
[sudo] du -sh /var/lib/docker                # peso do docker
ls -la /opt /srv /var/www                    # onde apps costumam morar
[sudo] find /var/backups /backup -maxdepth 2 -type f 2>/dev/null | head -20
```

Use `du -sh` (sumarizado) e não `du -h`, que lista recursivamente e gera output gigante. Quando um diretório aparecer grande, aprofunde um nível de cada vez, em vez de varrer tudo.

## 6. Agendamentos

```bash
crontab -l                                   # cron do usuário atual
[sudo] crontab -l                            # cron do root
[sudo] ls -la /etc/cron.d /etc/cron.daily /etc/cron.hourly
[sudo] cat /etc/crontab
for u in $(cut -f1 -d: /etc/passwd); do [sudo] crontab -u $u -l 2>/dev/null; done
systemctl list-timers --all --no-pager       # timers systemd
[sudo] ls -la /etc/systemd/system/*.timer
```

Timers systemd são frequentemente esquecidos porque a maioria das pessoas só verifica cron. Sempre cheque os dois. Para cada agendamento, verifique se o script apontado ainda existe — apontar para arquivo inexistente é o sintoma clássico de automação órfã.

## 7. Aplicações

```bash
ls -la ~/projetos /opt /srv /var/www 2>/dev/null
find /opt /srv /home /var/www -maxdepth 3 -name '.git' -type d 2>/dev/null | head -30
# para cada repositório encontrado:
git -C <caminho> remote -v
git -C <caminho> log -1 --format='%h %ad %s' --date=short
git -C <caminho> status --short --branch
find /opt /srv /home -maxdepth 3 -name 'package.json' -not -path '*/node_modules/*' 2>/dev/null | head -20
pm2 list                                     # se usar pm2
```

Para cada aplicação, o conjunto útil é: caminho, repositório remoto, branch atual, data do último commit, se há alterações não commitadas, e qual serviço/container a executa. Aplicação com último commit muito antigo e alterações locais não commitadas é um risco a registrar nas observações.

**Nunca leia nem transcreva o conteúdo de arquivos `.env`, `secrets`, chaves privadas ou `credentials`.** Registre apenas que o arquivo existe e onde está. Se precisar documentar configuração, use os nomes das variáveis, jamais os valores.

## 8. Bancos de dados

```bash
sudo -u postgres psql -c '\l'                          # listar bancos postgres
sudo -u postgres psql -c '\du'                         # roles
sudo -u postgres psql -d <db> -c '\dt'                 # tabelas de um banco
mysql -e 'SHOW DATABASES;'                             # mysql/mariadb
redis-cli INFO server                                  # redis
docker exec <container> psql -U <user> -c '\l'         # postgres em container
```

Todas as consultas acima são de metadados e não alteram dados. Não execute `SELECT` em tabelas de negócio — o objetivo é mapear a estrutura, não inspecionar conteúdo, e ler dados de produção é tanto desnecessário quanto potencialmente sensível.

Para cada banco, registre: engine e versão, nome dos bancos, onde os dados persistem no disco (`SHOW data_directory;` no postgres, ou o volume do container), e qual aplicação o consome.
