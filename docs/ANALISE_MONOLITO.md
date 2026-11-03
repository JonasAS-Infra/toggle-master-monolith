# Análise da aplicação monolítica

Estado: análise do código fornecido, registrada em 28/10/2026.

## O que a aplicação faz

O ToggleMaster fornece uma API HTTP para gerenciar feature flags: identificadores com um estado ligado ou desligado. Outro sistema pode consultar esse estado para decidir se disponibiliza uma funcionalidade. O repositório não contém interface gráfica nem um sistema cliente que aplique as flags.

| Operação | Rota | Implementação |
| --- | --- | --- |
| Verificar se o servidor responde | `GET /health` | Retorna `status: ok`, sem consultar o banco |
| Criar flag | `POST /flags` | Insere nome e estado no PostgreSQL |
| Listar flags | `GET /flags` | Consulta os registros ordenados por nome |
| Consultar uma flag | `GET /flags/<name>` | Consulta pelo nome e retorna 404 se ausente |
| Atualizar flag | `PUT /flags/<name>` | Altera o estado e retorna 404 se ausente |

## Por que é um monólito

Todas as operações pertencem à mesma aplicação Flask, definida em `app.py`. Nesse arquivo estão as rotas HTTP, as validações e o acesso ao PostgreSQL. O Gunicorn carrega `app:app` e publica essas funcionalidades juntas. Não existem serviços independentes para criação, consulta ou atualização de flags.

A unidade de implantação é a aplicação inteira. Uma mudança em uma rota exige publicar novamente essa unidade, mesmo quando as demais rotas permanecem iguais. O banco ser outro processo ou ficar no RDS não altera essa classificação: persistência externa não significa que a aplicação foi dividida em microserviços.

## Papel dos arquivos

- `app.py`: aplicação Flask, rotas, conexão e criação da tabela.
- `requirements.txt`: dependências Python declaradas com versões diretas fixadas.
- `Dockerfile`: construção da imagem da aplicação e instalação do cliente PostgreSQL.
- `docker-compose.yaml`: execução local da aplicação e do PostgreSQL, com volume de dados.
- `entrypoint.sh`: espera o banco, chama `flask init-db` e inicia o Gunicorn.

## Vantagens para este MVP

A estrutura reduz a quantidade de componentes para desenvolver e operar. As rotas compartilham configuração e acesso aos dados, o teste local pode usar um único ambiente Compose e a publicação exige uma aplicação e um banco. Para o conjunto pequeno de operações deste MVP, isso facilita compreender o fluxo e validar a proposta.

## Desvantagens e limites

As funcionalidades compartilham o mesmo ciclo de publicação e recursos do processo. Uma falha no processo pode afetar toda a API. Caso apenas as consultas cresçam muito, aumentar a quantidade de processos da aplicação também replica as demais rotas. Concentrar regras, SQL e HTTP em um arquivo tende a dificultar manutenção à medida que a aplicação cresce; separar módulos internamente seria possível sem abandonar o monólito.

## Diferenças e observações do código original

O texto introdutório do enunciado menciona exclusão de flags, mas o código fornecido não possui rota DELETE. A diferença fica registrada; a decisão do projeto é preservar as funcionalidades fornecidas.

O `/health` comprova que o servidor HTTP responde, mas não comprova conexão com o banco. A validação da persistência foi feita criando, consultando e atualizando uma flag.

O roteiro manual do README inicia o Gunicorn, mas não inclui explicitamente `flask --app app init-db`. A tabela precisa existir antes de testar as operações. Na execução Docker, o entrypoint já chama essa inicialização. Isso será considerado na implantação manual.

O campo `details` de algumas respostas de erro expõe a mensagem da exceção. A aplicação também não implementa autenticação. São limitações observadas para discussão; nenhuma alteração de funcionalidade foi realizada.
