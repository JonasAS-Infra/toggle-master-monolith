# ToggleMaster — checklist da fase 1

Atualizado em 03/11/2026.

Fonte: enunciado da fase 1 da FIAP, páginas 3 a 5 (PDF mantido apenas localmente). Os itens abaixo acompanham os requisitos técnicos e os entregáveis do PDF. As observações registram evidências e pendências; não acrescentam requisitos ao trabalho.

## 1. Cultura DevOps e arquitetura de aplicações

- [x] Executar a aplicação monolítica `toggle-master-monolith` localmente para entender seu funcionamento.
  - Aplicação e PostgreSQL executados com Docker Compose no Mac.
  - Validados: `/health`, criação, listagem, consulta individual e atualização de uma flag.
  - Endereço local: `http://localhost:5001`. O banco está saudável e as operações da API acessaram o PostgreSQL com sucesso.
- [x] Analisar o código e identificar por que a aplicação é considerada um monólito.
  - Análise registrada em [ANALISE_MONOLITO.md](ANALISE_MONOLITO.md).
- [x] Discutir as vantagens e desvantagens do monólito para um MVP.
  - Discussão registrada em [ANALISE_MONOLITO.md](ANALISE_MONOLITO.md).
- [x] Ler e compreender os 12 Fatores (12-Factor App), identificando quais a aplicação atende e quais exigem ajustes para produção mais robusta.

## 2. Arquitetura cloud

- [x] Finalizar o arquivo local editável do diagrama da implantação em [ARQUITETURA_AWS.drawio](ARQUITETURA_AWS.drawio), incluindo:
  - [x] VPC com sub-redes públicas e privadas.
  - [x] Instância EC2 na sub-rede pública para executar a aplicação.
  - [x] RDS PostgreSQL na sub-rede privada para o banco de dados.
  - [x] Security Group da EC2 permitindo HTTP/HTTPS e SSH de um IP específico.
  - [x] Security Group do RDS permitindo tráfego somente do Security Group da EC2.
  - HTTPS permanece pendente como serviço; a regra 443 já existe no SG. Diagrama publicado no fork pessoal.
- [x] Fazer a estimativa de custo mensal da arquitetura na Calculadora de Preços da AWS.

Observação sobre o banco: a aplicação fornecida usa PostgreSQL (`psycopg2`). PostgreSQL é a opção compatível com o código original, embora o enunciado também mencione MySQL.

Estimativa revisada para Ohio (`us-east-2`) em 31/10/2026: USD 27,32/mês. Link e print atualizados em `CUSTOS_AWS.md` e `ENTREGA.txt`.

## 3. AWS — prática

- [x] Provisionar manualmente VPC, sub-redes, Security Groups, EC2 e RDS pelo console da AWS.
- [x] Conectar-se à instância EC2 via SSH.
- [x] Instalar as dependências da aplicação e o código-fonte na EC2.
- [x] Configurar a conexão da aplicação com o RDS, usando variáveis de ambiente para as credenciais.
- [x] Executar a aplicação na EC2 e validar o acesso público pelo IP da instância.
  - `GET /health`, criação, consulta e atualização de flag confirmados; serviço `togglemaster` ativo. Detalhes em [IMPLANTACAO_AWS.md](IMPLANTACAO_AWS.md).

## 4. Vídeo de demonstração — até 15 minutos

- [x] Apresentar a aplicação funcionando localmente.
- [x] Explicar o diagrama da arquitetura proposta.
- [x] Demonstrar a aplicação funcionando na EC2 e acessando o RDS.
- [x] Mostrar as configurações dos Security Groups e como as credenciais do banco foram gerenciadas.
- [x] Concluir o vídeo com duração máxima de 15 minutos. Arquivo final: `~/Desktop/ToggleMaster_video_final.mp4` (4 min 33 s).
- [x] Publicar o vídeo e verificar o acesso pelo link.

## 5. Documentação

- [x] Disponibilizar o link do diagrama de arquitetura, renderizado no GitHub e com arquivo editável `.drawio`.
- [x] Documentar a discussão dos 12-Factor App aplicada ao projeto.

## 6. Relatório de entrega — PDF ou TXT

- [x] Informar o nome e o RM do participante.
- [x] Incluir o link da documentação.
- [x] Incluir o link do vídeo.
- [x] Resumir os desafios encontrados e as decisões tomadas.
- [x] Incluir o print da estimativa de custos da AWS.
  - Evidência salva em `evidencias/custos/estimativa-aws-ohio-2026-10-31.jpg`, incorporada ao [PDF de entrega](ENTREGA.pdf) e referenciada em `ENTREGA.txt`.

## Estado atual e decisões já tomadas

A execução local e a implantação HTTP na AWS estão validadas; as análises do monólito e dos 12 fatores estão documentadas. O SG da EC2 permite 80/443 e restringe SSH ao IP do Mac; o SG do RDS permite PostgreSQL da origem SG da EC2. O diagrama e a documentação estão publicados no fork pessoal. O vídeo foi publicado e os links e dados do participante estão no relatório. HTTPS segue sem configuração. Consulte o [índice da documentação](README.md).

O código da aplicação (`app.py`) permanece original. Os ajustes locais foram:

- Publicar a aplicação somente em `127.0.0.1:5001`, pois a porta 5000 do Mac estava ocupada por um serviço da Apple. A porta interna do contêiner continua sendo 5000.
- Trocar os endereços de download dos pacotes Debian no Dockerfile para espelhos oficiais acessíveis por HTTPS, após falhas de rede com o endereço original.
- Atualizar os exemplos locais do README para a porta 5001 e os comandos de inicialização/encerramento para `docker compose`.

O texto introdutório do PDF menciona exclusão de flags, mas o código fornecido não implementa uma rota DELETE. Essa diferença será registrada na análise, sem acrescentar funcionalidades por iniciativa própria, conforme a decisão de usar a aplicação fornecida.

## Ordem sugerida de trabalho

1. Registrar a análise do monólito e dos 12 fatores.
2. Desenhar a arquitetura exigida e estimar os custos.
3. Provisionar manualmente e validar EC2 + RDS.
4. Finalizar a documentação, gravar o vídeo e preparar o relatório de entrega.
