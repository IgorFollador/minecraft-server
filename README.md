# Minecraft Server

Servidor Minecraft Java Edition utilizando Paper,
Docker Compose e Playit.gg para acesso externo.

## Requisitos

- Docker Desktop
- Docker Compose
- Conta no [Playit.gg](https://playit.gg) (para acesso pela internet)

## Iniciar servidor

```bash
docker compose -f composer.yaml up -d
```

## Visualizar logs

```bash
docker compose -f composer.yaml logs -f
```

## Parar servidor

```bash
docker compose -f composer.yaml stop
```

## Administração

Adicionar jogador à whitelist:

```bash
docker exec minecraft-server rcon-cli whitelist add NICK
```

Conceder permissão de operador:

```bash
docker exec minecraft-server rcon-cli op NICK
```

## Acesso

- Local: `localhost:25565`
- Internet: na primeira subida, o plugin do Playit imprime um link
  `https://playit.gg/claim/...` nos logs. Abra o link, conclua o claim
  e use o endereço `*.craft.playit.gg` que o site entregar.

## Persistência

Os dados ficam armazenados em `./data`,
fora do controle de versão.
