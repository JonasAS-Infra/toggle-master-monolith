# Guia de execução — implantação e apresentação

Planejamento informado pelo usuário em 30/10/2026: implantar e validar em 30/10; gravar em 31/10. Conta pessoal AWS; região de referência `us-east-2`. São metas, não etapas concluídas.

O provisionamento será manual pelo console, como pede o enunciado. Resultados efetivos e evidências serão registrados em `IMPLANTACAO_AWS.md`. A aplicação fornecida será preservada.

## Hoje — 30/10

1. Conferir região e preparar a estimativa na Calculadora de Preços da AWS para a configuração de `ARQUITETURA_AWS.md`. Salvar link e print em `CUSTOS_AWS.md` e `evidencias/custos/`. Considerar EC2, EBS, IPv4 público e RDS/armazenamento; não presumir gratuidade.
2. Criar VPC, sub-redes, gateway, tabelas de rotas e Security Groups conforme abaixo.
3. Criar o grupo de sub-redes RDS e o RDS PostgreSQL privado, Single-AZ.
4. Criar a EC2 pública e conectar por SSH.
5. Instalar o código e as dependências em ambiente virtual; fornecer as variáveis de conexão sem gravar senhas na documentação.
6. Executar a inicialização do banco e verificar a existência da tabela. O código original captura erros de inicialização; a mensagem/saída do comando sozinha não substitui testar a consulta ao banco.
7. Iniciar Gunicorn e configurar o Nginx para acesso HTTP. Fechar a configuração de certificado/listener HTTPS antes de declarar essa parte concluída.
8. Validar criação, consulta e atualização pela URL pública, comprovando a persistência no RDS. Guardar capturas sem segredos.
9. Atualizar o diagrama para a configuração real, os documentos, o checklist e o relatório. Registrar a forma de manter ou reiniciar a aplicação para a gravação.

## Rede — passo a passo no console

### VPC

No serviço VPC, criar **somente a VPC** para configurar cada componente explicitamente:

- Name: `togglemaster-vpc`.
- IPv4 CIDR: `10.0.0.0/16`.
- IPv6: não necessário neste desenho.
- Tenancy: padrão.
- Após criar, habilitar DNS resolution e DNS hostnames nas configurações.

### Sub-redes

Selecionar duas zonas disponíveis da região. Os nomes abaixo usam A/B como identificação lógica; registrar depois os nomes reais escolhidos no console.

| Nome | Zona | CIDR |
| --- | --- | --- |
| `togglemaster-public-a` | A | `10.0.1.0/24` |
| `togglemaster-private-a` | A | `10.0.11.0/24` |
| `togglemaster-private-b` | B | `10.0.12.0/24` |

Todas pertencem à mesma VPC. A EC2 receberá IPv4 público na configuração de lançamento; não habilitar IP público nas sub-redes privadas.

### Internet Gateway e rotas

1. Criar `togglemaster-igw` e associar à VPC.
2. Criar `togglemaster-public-rt`, associar à sub-rede pública e adicionar `0.0.0.0/0 → togglemaster-igw`, preservando a rota local.
3. Criar `togglemaster-private-rt` e associar às duas privadas. Preservar apenas a rota local `10.0.0.0/16 → local`.
4. Conferir as associações explícitas das três sub-redes.

### Security Groups

Criar os dois grupos dentro de `togglemaster-vpc`:

- `togglemaster-ec2-sg`: entrada HTTP/TCP 80 de `0.0.0.0/0`; SSH/TCP 22 de **Meu IP**, com máscara `/32`. A regra TCP 443 será habilitada junto da configuração HTTPS. Manter saída padrão inicialmente.
- `togglemaster-rds-sg`: entrada PostgreSQL/TCP 5432 com origem no **ID do SG da EC2**. Nenhuma outra entrada.

Não abrir 5000 nem 5432 para a internet. O Gunicorn ficará no loopback da EC2; o RDS terá acesso público desativado.

## RDS — parâmetros para conferir antes de criar

- Standard create, mecanismo PostgreSQL, versão 16 disponível na conta.
- Uma instância Single-AZ; classe proposta `db.t3.micro`, 20 GiB gp3. Conferir valores e opções da conta antes de confirmar.
- Identificador sugerido `togglemaster-db`; nome do banco inicial **`togglemaster`**. O identificador da instância não cria um banco com esse mesmo nome.
- VPC do projeto e DB subnet group contendo as **duas sub-redes privadas**.
- Public access: **No**; SG: somente `togglemaster-rds-sg`.
- Definir credenciais e fornecer à aplicação por ambiente, sem incluí-las no Git ou nas evidências.
- Conferir armazenamento, backups e opções de monitoramento na tela de resumo; registrar a configuração efetiva na estimativa.
- Aguardar o estado disponível e copiar o endpoint DNS, sem presumir IP fixo para o banco.

O grupo de duas zonas não significa que criamos um banco Multi-AZ. No PostgreSQL 15 ou posterior, o RDS exige SSL por padrão; manter essa proteção e validar a conexão do cliente fornecido. Referência: [SSL no RDS PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/PostgreSQL.Concepts.General.SSL.html).

## EC2 — parâmetros para conferir antes de criar

- Amazon Linux 2023, arquitetura x86_64, classe proposta `t3.micro`.
- EBS gp3, criptografado; 8 GiB propostos, respeitando o mínimo da AMI.
- VPC do projeto, sub-rede pública e atribuição de IPv4 público habilitada.
- Somente `togglemaster-ec2-sg` como SG de acesso da aplicação.
- Criar ou selecionar um par de chaves e guardar a chave privada fora do repositório.
- Conectar pelo SSH do Mac; a chave privada não precisa ser enviada ao assistente.

Os comandos de instalação e configuração serão registrados conforme forem executados e verificados. Antes de testar a API publicamente, confirmar que ela está usando o endpoint do RDS.

## Amanhã — 31/10

1. Conferir a disponibilidade da aplicação antes da gravação; repetir os testes e atualizar o IP/DNS se houver mudança.
2. Gravar seguindo `ROTEIRO_VIDEO.md`, em até 15 minutos.
3. Publicar vídeo e documentação, testar os links e preencher `ENTREGA.txt` com nomes e links finais.
4. Incluir o print da estimativa e conferir o checklist com o PDF.
5. Definir quando os recursos podem ser removidos após a avaliação; registrar o estado final. Parar instâncias não elimina necessariamente cobranças de armazenamento.

## Evidências para guardar durante o provisionamento

- VPC, sub-redes e associações de rotas.
- SG da EC2: portas e restrição do SSH; SG do RDS: origem referenciando o SG da EC2.
- RDS sem acesso público, mecanismo e endpoint.
- SSH e aplicação na EC2, sem mostrar senha ou chave privada.
- Operações da API via endereço público e confirmação no banco.
- Estimativa mensal na calculadora.

## Referências

- [Criação e conexão ao RDS PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html).
- [RDS em uma VPC e grupo de sub-redes](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html).
- [Internet Gateway e acesso público](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html).
