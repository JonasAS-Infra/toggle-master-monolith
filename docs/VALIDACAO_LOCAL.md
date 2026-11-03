# Execução e validação local

Estado: validada em 28/10/2026. Capturas para apresentação ainda pendentes.

## Ambiente

Mac Apple Silicon, Docker Desktop e Docker Compose. O Compose executa a aplicação Flask com Gunicorn em um contêiner e PostgreSQL 13 em outro. O banco usa o volume `postgres_data`.

Na raiz do repositório:

```sh
docker compose up -d --build
docker compose ps
docker compose logs --tail 20 app
```

A API fica disponível em `http://localhost:5001`. Não há interface gráfica na aplicação fornecida.

## Resultados observados

| Requisição | Resultado |
| --- | --- |
| `GET /health` | HTTP 200, `{"status":"ok"}` |
| `POST /flags` com `demo-fiap` e `is_enabled: true` | HTTP 201 |
| `GET /flags/demo-fiap` | HTTP 200, estado `true` |
| `PUT /flags/demo-fiap` com `is_enabled: false` | HTTP 200 |
| `GET /flags/demo-fiap` após atualização | HTTP 200, estado `false` |
| `GET /flags` | HTTP 200, lista contendo `demo-fiap` desativada |

O contêiner do PostgreSQL ficou saudável. Os logs da aplicação confirmaram a criação da tabela e a inicialização do Gunicorn. As operações acima validaram acesso ao banco, além da resposta HTTP do servidor.

## Exemplos para a apresentação

Usar um nome ainda não cadastrado no POST; repetir o mesmo nome retorna conflito.

```sh
curl http://localhost:5001/health
curl -X POST http://localhost:5001/flags -H 'Content-Type: application/json' -d '{"name":"exemplo-video","is_enabled":true}'
curl http://localhost:5001/flags/exemplo-video
curl -X PUT http://localhost:5001/flags/exemplo-video -H 'Content-Type: application/json' -d '{"is_enabled":false}'
curl http://localhost:5001/flags
```

## Ajustes necessários neste Mac

1. A porta 5000 estava ocupada por um serviço da Apple. O Compose passou a mapear `127.0.0.1:5001:5000`, mantendo 5000 dentro do contêiner.
2. O download de pacotes falhou no endereço Debian original. O Dockerfile passou a usar `https://ftp.debian.org/debian` e `https://security.debian.org/debian-security`, preservando a verificação de assinatura dos pacotes.
3. Os exemplos locais do README foram atualizados para a porta 5001 e os comandos de início/fim para `docker compose`.

A lógica de `app.py` permanece original.

## Encerramento

```sh
docker compose down
```

O comando remove os contêineres e a rede do Compose, preservando o volume do banco. Capturas de evidência, quando produzidas, serão armazenadas em `evidencias/local/`.
