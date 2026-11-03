# 12-Factor App — análise aplicada ao ToggleMaster

Estado: análise inicial do código e da execução local em 28/10/2026. Não certifica comportamento ainda não testado na AWS.

Referência conceitual: [The Twelve-Factor App, tradução em português](https://12factor.net/pt_br/). As classificações e observações abaixo são nossa análise do repositório, e não uma avaliação publicada pelos autores da metodologia.

“Atende no código” significa que há evidência da prática na implementação. “Parcial” indica limites ou ausência de validação operacional. Os ajustes citados são pontos de discussão para produção mais robusta; não representam mudanças aprovadas no escopo da fase.

| Fator | Avaliação | Evidência no projeto | Ajuste ou validação para produção |
| --- | --- | --- | --- |
| I. Base de código | Atende no repositório | Código versionado em Git; a mesma aplicação pode ser executada em ambientes distintos. | Registrar o commit publicado na EC2 para relacionar execução e versão. |
| II. Dependências | Parcial | `requirements.txt` declara Flask, Werkzeug, psycopg2 e Gunicorn; o contêiner isola o ambiente Python. | Dependências transitivas não estão todas fixadas; pacotes do sistema e imagem base podem variar. Na EC2, usar ambiente virtual e registrar versões. |
| III. Configurações | Parcial | `app.py` lê host, banco, usuário e senha de variáveis de ambiente. | O Compose contém credenciais de exemplo versionadas, apenas para uso local. As credenciais reais devem ser fornecidas fora do Git. `DB_PORT` é usado no script de espera, mas não é passado explicitamente à conexão Python, que usa a porta padrão. |
| IV. Serviços de apoio | Atende no código para PostgreSQL | Banco acessado por conexão externa; host e credenciais são configuráveis. | Validar a mudança do banco local para RDS PostgreSQL por configuração. Isso não significa que MySQL possa ser usado sem adaptar o código. |
| V. Construir, lançar, executar | Parcial | O Dockerfile constrói a imagem e o entrypoint inicia o processo. | O Compose monta o código do host sobre `/app`, e o início também cria a tabela. Falta um registro explícito de release imutável, configuração associada e versão implantada. |
| VI. Processos | Atende no código | As flags ficam no PostgreSQL; as rotas não usam arquivos locais ou sessões em memória para persistir dados de negócio. | Confirmar o comportamento ao reiniciar a aplicação e evitar introduzir estado local ao evoluir o projeto. |
| VII. Vínculo de porta | Atende no ambiente local | Gunicorn publica HTTP em `0.0.0.0:5000`; Compose mapeia para `127.0.0.1:5001` no Mac. | Definir o acesso HTTP/HTTPS da EC2 conforme a arquitetura e validar suas portas. A porta do Gunicorn está fixa no comando atual. |
| VIII. Concorrência | Parcial | A execução usa Gunicorn, mas o comando fornecido não configura vários workers. | Dimensionar e testar processos conforme a carga e a capacidade do banco; não houve teste de concorrência. |
| IX. Descartabilidade | Parcial | `exec gunicorn` faz o servidor receber os sinais enviados ao processo principal. | O script espera o banco indefinidamente e ignora falha da inicialização com `flask init-db || true`; `init_db` também captura exceções sem sinalizar falha ao chamador. Não foram medidos tempo de início nem término com requisições em andamento. |
| X. Semelhança dev/prod | Parcial | Desenvolvimento usa PostgreSQL e a proposta AWS também usará PostgreSQL. | Local usa Docker e a implantação exigida será manual na EC2. Registrar diferenças de versões, dependências e configuração; não presumir equivalência. |
| XI. Logs | Parcial | Mensagens de inicialização e logs do Gunicorn são observáveis por `docker compose logs`; Python usa saída sem buffer na imagem. | O comando atual não habilita log de acesso do Gunicorn; erros das rotas são retornados ao cliente, sem registro operacional explícito nessas rotas. Definir coleta e conteúdo dos logs para produção. |
| XII. Processos administrativos | Parcial | Existe comando Flask `init-db` que utiliza a configuração e o código da aplicação. | A inicialização é chamada automaticamente no entrypoint e não há histórico de migrações. No deploy manual, executá-la explicitamente com o mesmo ambiente da aplicação e validar o resultado. |

## Conclusão da análise

A aplicação já separa o banco do processo HTTP, usa variáveis de ambiente e possui dependências declaradas. As principais limitações se concentram em reprodutibilidade da release, inicialização do banco, registros operacionais e validação de processos sob falha e carga. A análise identifica melhorias possíveis, preservando o código fornecido para o trabalho.
