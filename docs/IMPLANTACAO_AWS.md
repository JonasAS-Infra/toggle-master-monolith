# Registro da implantação manual na AWS

Estado em 03/11/2026: API implantada e validada por HTTP. HTTPS e capturas locais das configurações de rede ainda pendentes.

## Ambiente

- Região: `us-east-2` (Ohio).
- EC2 Amazon Linux 2023, `t3.micro`: `i-0c8f51662ac454703`, IP público `18.220.243.88`.
- RDS PostgreSQL: instância `togglemaster-db`, banco `togglemaster`, endpoint `togglemaster-db.cpg0ewkq8uhl.us-east-2.rds.amazonaws.com`.
- Código clonado de `https://github.com/dougls/toggle-master-monolith.git` em `/home/ec2-user/toggle-master-monolith`; dependências instaladas em `venv`.
- Commit exato implantado: a confirmar na EC2.

## Acesso e segurança

- A conexão SSH falhou por timeout quando o IP do Mac mudou; a regra SSH do Security Group foi atualizada para o novo IP e a conexão funcionou.
- O SG `togglemaster-ec2-sg` mostra entrada TCP 80 e 443 de `0.0.0.0/0`, SSH TCP 22 de `45.229.54.221/32` e saída liberada para `0.0.0.0/0` (captura enviada pelo usuário em 03/11/2026).
- A porta 80 da EC2 respondeu publicamente. Gunicorn escuta apenas em `127.0.0.1:5000`; Nginx encaminha requisições da porta 80 para ele.
- O RDS aceitou a conexão originada pela EC2. O console mostra o SG `togglemaster-rds-sg` (`sg-067c5ea22b75d6046`) com entrada originada no SG `sg-0204376560b94f163`; o usuário confirmou a porta PostgreSQL padrão `5432`. Rotas e sub-redes ainda precisam ser conferidas.
- O endpoint HTTP ainda não usa TLS. HTTPS permanece pendente.

## Aplicação e banco

- `flask --app app init-db` retornou `Tabela 'flags' inicializada com sucesso.`
- Variáveis `DB_HOST`, `DB_NAME`, `DB_USER` e `DB_PASSWORD` foram fornecidas à aplicação sem gravar a senha no repositório. O serviço lê `/etc/togglemaster.env`, criado com permissão `0600`.
- Serviço `togglemaster.service` criado com `User=ec2-user`, início automático e `Restart=always`. `systemctl is-active togglemaster` retornou `active`.
- Nginx habilitado com `systemctl enable --now nginx`; `nginx -t` passou.

## Validação pública

| Verificação | Resultado observado |
| --- | --- |
| `GET /health` | `200 OK`, `{"status":"ok"}` |
| `POST /flags` com `teste-aws=true` | `201 CREATED` |
| `GET /flags/teste-aws` | `200 OK`, `is_enabled:true` |
| `PUT /flags/teste-aws` com `is_enabled=false` | `200 OK` |
| `GET /flags/teste-aws` após atualização e início pelo systemd | `200 OK`, `{"is_enabled":false,"name":"teste-aws"}` |

O último resultado comprova que a API respondeu pelo IP público após a troca do Gunicorn manual pelo serviço e leu o estado atualizado do RDS. O usuário enviou uma captura desta resposta na conversa em 03/11/2026. Esse print da API é opcional; o enunciado exige apenas o print da estimativa de custos. Registro textual: [evidência da API](evidencias/aws/VALIDACAO_API_2026-11-03.md).

## Pendências

- Confirmar rotas, sub-redes e configuração de acesso público do RDS.
- Definir e implantar HTTPS, se exigido para a entrega.
- Confirmar o commit exato implantado e atualizar a estimativa de custos.
