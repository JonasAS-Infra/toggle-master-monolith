# Estimativa de custos AWS

**Revisão concluída em 31/10/2026:** os três serviços foram alterados para Ohio (`us-east-2`). Total mantido em USD 27,32/mês; novo link e nova captura salvos.

Consulta atualizada: 31/10/2026. Valores observados no resumo da Calculadora de Preços da AWS compartilhado pelo usuário. Estado: estimativa concluída, link registrado e captura do resumo salva.

## Configuração considerada

| Campo | Valor |
| --- | --- |
| Região | Leste dos EUA (Ohio), `us-east-2` |
| Moeda | USD |
| Utilização | 100% do mês, sob demanda; referência mensal de 730 horas da calculadora |
| EC2 | Uma `t3.micro`, Linux, instâncias compartilhadas, On-Demand |
| EBS | Um volume gp3 de 8 GB, 3000 IOPS e 125 MBps informados; sem snapshots |
| RDS | PostgreSQL, uma `db.t3.micro`, Single-AZ, On-Demand; armazenamento gp3 de 20 GB solicitado durante o preenchimento |
| Versão principal do PostgreSQL | 16 planejada para implantação; sem Suporte Estendido na estimativa |
| Opcionais RDS | Sem RDS Proxy e sem Database Insights na estimativa |
| IPv4 público | Um endereço em uso; zero inativos |
| Créditos e benefícios | Nenhum desconto ou crédito presumido |
| Transferência de dados | Sem volume adicional informado; estimativa voltada à demonstração de baixo tráfego. Custos de tráfego efetivo não estão dimensionados aqui. |
| Backups adicionais | Não dimensionados; conferir a configuração real e eventuais cobranças fora das franquias antes da implantação |

## Resultado mostrado pela calculadora

| Item | USD/mês |
| --- | ---: |
| EC2 e EBS | 8,23 |
| RDS PostgreSQL | 15,44 |
| IPv4 público (item VPC) | 3,65 |
| **Total mensal** | **27,32** |

Custo inicial: **USD 0,00**. Projeção exibida de 12 meses: **USD 327,84**.

Os valores representam os itens configurados na calculadora, não uma fatura nem um compromisso anual. O custo efetivo depende do uso e das configurações criadas; não inclui impostos aplicáveis. A data e os valores foram registrados a partir das telas enviadas, com conferência adicional do resumo e das configurações pela janela da calculadora aberta no Chrome.

## Evidências para entrega

- Nome da estimativa: `ToggleMaster - Tech Challenge Fase 1`.
- [Link compartilhável da Calculadora de Preços da AWS](https://calculator.aws/#/estimate?id=cbebee3def5152831130a070766d96939ff16e9f), fornecido pelo usuário.
- [Print do resumo](evidencias/custos/estimativa-aws-ohio-2026-10-31.jpg), capturado da janela da calculadora em 31/10/2026.
- Exportação da calculadora: PENDENTE, opcional para apoio.

Link e print registrados no relatório de entrega e item de estimativa concluído no checklist. Conferir os detalhes do RDS na estimativa exportada ou compartilhada para manter a implantação alinhada aos parâmetros orçados.

## Histórico

A estimativa anterior em Norte da Virgínia foi substituída pela revisão de Ohio. A captura antiga de 30/10 fica apenas no workspace local; usar na entrega o link e o print atuais acima.
