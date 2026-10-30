# Minecraft Server

Servidor **Filhos do Velho Barreiro**: Minecraft Java Edition 26.3,
Fabric, Docker Compose e Playit.gg para acesso externo.

O mundo fica em `./data`. Mudar dificuldade, whitelist, OP, nome ou ícone
não apaga o mapa. O mundo só some se a pasta `data/world` for apagada.

## Requisitos

- Docker e Docker Compose
- Cliente Minecraft **26.3** (a versão do launcher tem que bater com a do servidor)
- Conta no [Playit.gg](https://playit.gg) (para acesso pela internet)

## Configuração

As opções do servidor ficam no `.env`. Na primeira vez:

```bash
cp .env.example .env
```

Edite o `.env`, salve e aplique:

```bash
docker compose -f composer.yaml up -d
```

O `composer.yaml` lê esse arquivo. Sem `.env`, vale o que está no exemplo. O `.env` fica fora do Git; o `.env.example` é o modelo versionado.

## Opções

Tudo abaixo é o `.env`, salvo quando dito o contrário. Depois de mudar qualquer item, rode `docker compose -f composer.yaml up -d`.

### Tipo e versão

| Opção | Valor atual | O que faz |
| --- | --- | --- |
| `EULA` | `TRUE` | Aceita o EULA da Mojang. Sem isso o servidor não sobe. |
| `TYPE` | `FABRIC` | Motor do servidor. `FABRIC` usa mods. `PAPER` usa plugins. Os dois não misturam: plugin do Paper não carrega no Fabric, e mod do Fabric não carrega no Paper. Trocar o tipo não apaga o mundo, mas o mapa do Paper precisa de um ajuste de arquivos para o Fabric (já feito neste servidor). |
| `VERSION` | `26.3` | Versão do Minecraft. Tem que ser a mesma do launcher de quem entra. |

### Desempenho e capacidade

| Opção | Valor atual | O que faz |
| --- | --- | --- |
| `MEMORY` | `4G` | RAM do Java. Ex.: `2G`, `6G`, `8G`. |
| `MAX_PLAYERS` | `10` | Quantos jogadores o servidor aceita ao mesmo tempo. |
| `VIEW_DISTANCE` | `10` | Chunks que o cliente vê ao redor. Número maior = mapa mais longe, mais CPU e RAM. |
| `SIMULATION_DISTANCE` | `6` | Chunks em que o jogo de fato roda (mobs, fazenda, redstone). Pode ser menor que a visão. |

### Jogo

| Opção | Valor atual | O que faz |
| --- | --- | --- |
| `MODE` | `survival` | Modo padrão de quem entra: `survival`, `creative` ou `adventure`. |
| `DIFFICULTY` | `normal` | `peaceful`, `easy`, `normal` ou `hard`. Dá para mudar na hora com `rcon-cli difficulty ...`; para ficar permanente, altere aqui. |
| `SPAWN_PROTECTION` | `0` | Raio, em blocos, em volta do spawn onde só OP quebra e coloca bloco. `0` desliga. O padrão do Minecraft é `16`. |

### Quem entra

| Opção | Valor atual | O que faz |
| --- | --- | --- |
| `ONLINE_MODE` | `TRUE` | `TRUE` = só conta original (Mojang/Microsoft). `FALSE` = conta original e conta sem original no mesmo endereço. Ver [Conta original e conta sem original](#conta-original-e-conta-sem-original). |
| `ENABLE_WHITELIST` | `TRUE` | `TRUE` = só nick na whitelist entra. Continua valendo com `ONLINE_MODE=FALSE`. |

### Aparência

| Opção | Valor atual | O que faz |
| --- | --- | --- |
| `SERVER_NAME` | `Filhos do Velho Barreiro` | Nome interno do servidor. |
| `MOTD` | texto com `§` | Texto da lista de servidores. `§6` dourado, `§l` negrito, `§r` limpa o formato, `§8` cinza, `\n` quebra a linha. |
| `ICON` | `/icon.png` | Caminho da logo **dentro** do container. Não precisa mudar. O arquivo real é `icon.png` na raiz do projeto (64×64). |
| `OVERRIDE_ICON` | `TRUE` | Recopia o `icon.png` a cada subida. Sem isso, um ícone antigo em `data/` pode ficar. |

### Mods (Fabric)

| Opção | Valor atual | O que faz |
| --- | --- | --- |
| `MODRINTH_PROJECTS` | `fabric-api` | Lista do [Modrinth](https://modrinth.com), separada por vírgula. O `fabric-api` é o básico que quase todo mod precisa. Ex.: `fabric-api,lithium,sodium`. |

JARs soltos vão em `./mods`. A imagem copia essa pasta para dentro do servidor na subida. Cliente e servidor têm que usar os **mesmos** mods (e a mesma versão).

### Playit (internet)

Não fica no `.env`. A chave do agente está em `playit.secret` (fora do Git). Sem esse arquivo o container `playit` não autentica o túnel. Em outra máquina, copie o arquivo ou gere um agente novo no [Playit.gg](https://playit.gg).

A porta `25565` no `composer.yaml` está em `127.0.0.1`, então o Minecraft não fica aberto no roteador; quem entra de fora usa o endereço `*.tun.ply.gg` / `*.craft.playit.gg`.

## Subir, parar e ver logs

```bash
docker compose -f composer.yaml up -d    # sobe (cria na primeira vez)
docker compose -f composer.yaml start    # liga de novo depois de um stop
docker compose -f composer.yaml stop     # para, o mundo continua em ./data
docker compose -f composer.yaml logs -f  # acompanha os logs
```

Quem está online:

```bash
docker exec minecraft-server rcon-cli list
```

## Acesso

- Nesta máquina: `localhost` (ou `127.0.0.1`). Não use o endereço do
  Playit para jogar no mesmo PC — o túnel não “volta” para a própria
  casa, e a lista fica em *Can't connect* / ping infinito.
- Internet: `ambulance-bulgarian.tun.ply.gg:53890`. Sem a porta, o
  cliente tenta `25565` e o ping não responde. A 53890 é a porta que o
  Playit publicou neste túnel (também existe um registro SRV).

Cliente **26.3** (Fabric). O servidor só escuta em `127.0.0.1:25565`;
o túnel sai pelo container `playit`, sem abrir o roteador.

Na primeira vez em outra máquina, copie o `playit.secret` (fica fora do
Git) ou gere um agente novo em [Playit.gg](https://playit.gg) e coloque
a chave nesse arquivo.

A whitelist está ligada. Antes de alguém entrar:

```bash
docker exec minecraft-server rcon-cli whitelist add NICK
docker exec minecraft-server rcon-cli whitelist remove NICK
docker exec minecraft-server rcon-cli whitelist list
```

## Conta original e conta sem original

Quem pode entrar é o `ONLINE_MODE` no `.env`.

- `ONLINE_MODE=TRUE` — só conta original (Mojang/Microsoft). É o valor atual.
- `ONLINE_MODE=FALSE` — conta original e conta sem original entram no mesmo endereço. O cliente original não muda nada.

A whitelist continua obrigatória nos dois casos.

Para trocar, edite o `.env` e suba de novo:

```bash
docker compose -f composer.yaml up -d
```

Com `FALSE`, o servidor para de confirmar a conta na Mojang. Duas consequências:

- O personagem de quem já jogou com conta original fica no UUID antigo. A mesma pessoa entra como jogador novo: inventário, baú e conquistas não acompanham.
- O nick passa a ser a identidade. Quem souber um nick da whitelist entra como essa pessoa.

## Operador

Tirar OP de alguém (o comando continua disponível):

```bash
docker exec minecraft-server rcon-cli deop NICK
```

Devolver:

```bash
docker exec minecraft-server rcon-cli op NICK
```

## Dificuldade

Na hora, sem reiniciar e sem perder o mundo. Valores: `peaceful`, `easy`, `normal`, `hard`.

```bash
docker exec minecraft-server rcon-cli difficulty normal
```

Para a mudança sobreviver a um `up -d`, altere `DIFFICULTY` no `.env` e suba de novo:

```bash
docker compose -f composer.yaml up -d
```

Outros comandos pelo mesmo caminho:

```bash
docker exec minecraft-server rcon-cli time set day
docker exec minecraft-server rcon-cli weather clear
```

## Proteção da área inicial

`SPAWN_PROTECTION` no `.env` é o raio, em blocos, em volta do
spawn onde só OP quebra e coloca bloco. Está em `0`, então todo mundo
constrói na área inicial.

Para ligar de novo, mude o valor (o padrão do Minecraft é `16`) e suba:

```bash
docker compose -f composer.yaml up -d
```

## Nome e ícone

No `.env`:

- `SERVER_NAME` — nome do servidor
- `MOTD` — texto da lista de servidores. `§6` é dourado, `§l` é negrito, `§r` limpa o formato, `§8` é cinza, `\n` quebra a linha
- `icon.png` na raiz do projeto — logo 64×64 mostrada na lista

Depois de trocar nome ou ícone:

```bash
docker compose -f composer.yaml up -d
```

## Backup do mapa

Os arquivos vão para `backups/`. O Git guarda só o `.gitkeep`; os `.tar.gz` ficam fora do repositório.

Com o servidor no ar, grave o mundo e compacte:

```bash
docker exec minecraft-server rcon-cli save-off
docker exec minecraft-server rcon-cli save-all flush
stamp=$(date +%Y%m%d-%H%M%S)
tar -C data -czf "backups/world-${stamp}.tar.gz" world
docker exec minecraft-server rcon-cli save-on
```

O autosave volta no último comando. O arquivo inclui overworld, Nether, End e os dados dos jogadores.

Para restaurar, pare o servidor, troque a pasta e suba de novo:

```bash
docker compose -f composer.yaml stop
mv data/world "data/world-antes-$(date +%Y%m%d-%H%M%S)"
tar -C data -xzf backups/world-AAAAMMDD-HHMMSS.tar.gz
docker compose -f composer.yaml start
```

Troque `world-AAAAMMDD-HHMMSS.tar.gz` pelo arquivo que quiser voltar.
