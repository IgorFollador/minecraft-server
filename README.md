# Minecraft Server

Servidor **Filhos do Velho Barreiro**: Minecraft Java Edition 26.3,
Paper, Docker Compose e Playit.gg para acesso externo.

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

Edite versão, memória, dificuldade, whitelist, `ONLINE_MODE`, proteção de spawn, nome e MOTD ali. O `composer.yaml` lê esse arquivo. Sem `.env`, vale o que está no exemplo.

Depois de salvar:

```bash
docker compose -f composer.yaml up -d
```

O `.env` fica fora do Git. O `.env.example` é o modelo versionado.

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

- Nesta máquina: `localhost:25565`
- Internet: na primeira subida, o plugin do Playit imprime um link
  `https://playit.gg/claim/...` nos logs. Abra o link, conclua o claim
  e use o endereço que o site entregar (algo como `*.craft.playit.gg`
  ou `*.tun.ply.gg`).

A porta do Minecraft fica só em `127.0.0.1`. O túnel sai de dentro do
container, sem abrir o roteador.

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
