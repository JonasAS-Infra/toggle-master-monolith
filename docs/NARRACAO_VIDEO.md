# Narração do vídeo — ToggleMaster

Texto para voz de IA, dividido em partes. Cada parte indica o que mostrar enquanto a fala é reproduzida. Duração total máxima: 15 minutos.

## Parte 1 — Abertura (aproximadamente 40 segundos)

**Na tela:** abra a [documentação do projeto](README.md). Mostre o título e, em seguida, o índice com a arquitetura, a implantação e os custos. Não é necessário abrir o terminal nesta parte.

**Fala:**

> Olá. Neste vídeo vou apresentar o ToggleMaster, projeto da primeira fase do Tech Challenge da FIAP. Ele é uma API para gerenciar feature flags: registros que permitem ativar ou desativar funcionalidades de uma aplicação. O desafio foi entender o código monolítico fornecido, executá-lo localmente e implantar a mesma API na AWS, conectada a um banco PostgreSQL no RDS. Vou mostrar o funcionamento local, o desenho da arquitetura, a aplicação na EC2 e as configurações de segurança usadas na implantação.

**Transição para a próxima parte:** abra o terminal ou o cliente de API com o ambiente Docker local.

## Parte 2 — Demonstração local (aproximadamente 1 minuto)

**Na tela:** mostre primeiro `~/Documents/docker.mov` com os contêineres `app-1` e `db-1` ativos. Em seguida, mostre `~/Documents/terminaltodos.mov` com `/health`, criação da flag `video-fiap-02`, consulta ativada, atualização, consulta desativada e listagem. As capturas já estão prontas.

**Fala:**

> Agora a aplicação está rodando localmente em dois contêineres Docker. Um contém a API Flask e o outro, o PostgreSQL. O Docker Compose inicia os dois e espera o banco ficar pronto. A API pode ser acessada em localhost, na porta cinco mil e um. Primeiro, a rota de saúde confirma que o servidor está respondendo. Em seguida, vou criar uma feature flag ativada. A consulta mostra que ela foi salva no banco. Agora atualizo o estado para desativado e consulto novamente para confirmar a mudança. A aplicação fornecida é uma API, sem interface gráfica. Suas rotas, validações e acesso ao banco ficam juntos no arquivo app.py, por isso ela é um monólito.

**Transição para a próxima parte:** abra o diagrama da arquitetura AWS.

## Parte 3 — Arquitetura AWS (aproximadamente 50 segundos)

**Na tela:** mostre o diagrama [ARQUITETURA_AWS.drawio](ARQUITETURA_AWS.drawio) inteiro. Deixe visíveis a VPC, a sub-rede pública com a EC2, as sub-redes privadas e o RDS. Uma imagem fixa legível é suficiente.

**Fala:**

> Este é o desenho da implantação na região de Ohio. A VPC separa a rede do projeto. A EC2 fica na sub-rede pública e recebe as chamadas da API pela porta oitenta. Na mesma máquina, o Nginx encaminha as requisições ao Gunicorn, que executa a aplicação Flask. O banco PostgreSQL está no RDS, em uma sub-rede privada. A comunicação da aplicação com o banco usa a porta cinco mil quatrocentos e trinta e dois, permitida apenas a partir do grupo de segurança da EC2. A segunda sub-rede privada compõe o grupo de sub-redes exigido pelo RDS; não há uma segunda instância do banco. O grupo de segurança da EC2 permite as portas oitenta e quatrocentos e quarenta e três, enquanto o acesso SSH fica restrito ao endereço autorizado do Mac.

**Transição para a próxima parte:** mostre a API pública na EC2 e seus resultados usando o RDS.

## Parte 4 — API pública na AWS (aproximadamente 50 segundos)

**Na tela:** grave o Terminal do Mac em uma única tomada. Execute os comandos abaixo, deixando cada resposta visível por alguns segundos. O endereço público chega ao Nginx na EC2; as rotas de flags consultam o PostgreSQL no RDS. Não mostre a senha do banco.

```bash
curl -i http://18.220.243.88/health
curl -i -X POST http://18.220.243.88/flags -H 'Content-Type: application/json' -d '{"name":"video-fiap-aws-03","is_enabled":true}'
curl -i http://18.220.243.88/flags/video-fiap-aws-03
curl -i -X PUT http://18.220.243.88/flags/video-fiap-aws-03 -H 'Content-Type: application/json' -d '{"is_enabled":false}'
curl -i http://18.220.243.88/flags/video-fiap-aws-03
```

**Fala:**

> Agora estou acessando a API pelo endereço público da instância EC2 na AWS. A rota de saúde responde com status duzentos, confirmando que a aplicação está disponível. Em seguida, crio uma feature flag ativada; a API confirma a criação com status duzentos e um. Consulto essa flag e recebo o estado ativado. Depois atualizo o valor para desativado e faço uma nova consulta. O resultado confirma que a alteração foi persistida. Na implantação, o Nginx recebe as chamadas HTTP e as encaminha à aplicação Flask. As operações com flags usam o PostgreSQL hospedado no RDS.

**Transição para a próxima parte:** mostre os grupos de segurança da EC2 e do RDS no console AWS.

## Parte 5 — Segurança e credenciais (aproximadamente 40 segundos)

**Na tela:** no console AWS, mostre as regras de entrada do grupo `togglemaster-ec2-sg` (22, 80 e 443) e depois do `togglemaster-rds-sg` (5432 com origem no grupo da EC2). Por fim, na EC2, mostre apenas os comandos abaixo e suas saídas. Não abra `/etc/togglemaster.env`, pois ele contém a senha.

```bash
sudo systemctl cat togglemaster | grep -E 'EnvironmentFile|User|ExecStart'
sudo stat -c '%a %U %n' /etc/togglemaster.env
```

**Fala:**

> Aqui estão as regras de entrada da EC2. As portas oitenta e quatrocentos e quarenta e três aceitam conexões da internet. O SSH, na porta vinte e dois, fica limitado ao endereço autorizado do Mac. No grupo de segurança do RDS, a porta cinco mil quatrocentos e trinta e dois aceita conexões somente do grupo de segurança da EC2. Assim, o banco não recebe conexões diretas da internet. A aplicação recebe os dados de conexão por variáveis de ambiente carregadas pelo serviço. O arquivo fica fora do repositório e tem permissão restrita; a senha não aparece na gravação.

**Transição para a próxima parte:** apresente as decisões, os desafios e a documentação de custos.

## Parte 6 — Decisões, desafios e custos (aproximadamente 45 segundos)

**Na tela:** comece em [ENTREGA.txt](ENTREGA.txt), na seção “DESAFIOS ENCONTRADOS E DECISÕES TOMADAS”. Depois abra [CUSTOS_AWS.md](CUSTOS_AWS.md) no resumo mensal e mostre o [print da estimativa](evidencias/custos/estimativa-aws-ohio-2026-10-31.jpg). Termine no [índice da documentação](README.md). Não mostre credenciais.

**Fala:**

> Para fechar, mantivemos a aplicação monolítica fornecida e usamos PostgreSQL, compatível com o código original. No ambiente local, a porta cinco mil do Mac estava ocupada; por isso, publicamos a API na porta cinco mil e um. Na AWS, a mudança do IP do Mac exigiu atualizar a regra de SSH. Essas decisões e os limites da aplicação estão registrados na documentação. A estimativa da Calculadora AWS para Ohio considera uma EC2, um RDS PostgreSQL e um IPv4 público, totalizando vinte e sete dólares e trinta e dois centavos por mês. Esse valor é uma estimativa, não uma fatura. O link e o print estão na documentação. Assim, demonstramos a execução local, a arquitetura e a API funcionando na AWS.
