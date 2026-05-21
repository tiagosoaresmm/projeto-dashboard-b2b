# projeto-dashboard-b2b
Projeto de Dashboard de pedidos B2B

# Sistema de Visibilidade de Pedidos e Alertas B2B (MVP)

## 1. Visão Geral do Projeto
[cite_start]Este projeto consiste no desenvolvimento de um sistema automatizado de monitoramento de status de pedidos B2B[cite: 193]. [cite_start]O objetivo principal é consolidar dados de múltiplas tabelas e Explores do Looker em um único dashboard intuitivo focado no time de vendas[cite: 194, 196, 197]. [cite_start]Além da visualização, a solução automatiza o envio de alertas por e-mail para casos críticos que exigem intervenção humana proativa[cite: 194].

### Objetivos Principais:
* [cite_start]**Centralização:** Unificar informações de diferentes visões do Looker em um único ecossistema[cite: 196].
* [cite_start]**Visibilidade Operacional:** Criar um dashboard dinâmico focado em pedidos com atrasos, avarias ou problemas logísticos[cite: 197].
* [cite_start]**Proatividade:** Implementar disparos automáticos de notificações por e-mail diretamente para os vendedores responsáveis[cite: 194, 198, 202].
* [cite_start]**Agilidade de Entrega:** Desenvolver um MVP (Mínimo Produto Viável) funcional sem dependência direta do time de TI/Tecnologia[cite: 191, 199].

---

## 2. Arquitetura da Solução e Fluxo de Dados
[cite_start]A solução utiliza o conceito de *Modern Data Stack* simplificada dentro do ecossistema Google[cite: 201]:

| Etapa | Ferramenta | Descrição |
| :--- | :--- | :--- |
| **Extração (Ingestão)** | Looker (Schedules) | [cite_start]Agendamento de envios automáticos de relatórios em formato CSV para uma pasta restrita no Google Drive[cite: 202, 231]. |
| **Processamento (ETL)** | Python (Pandas) | [cite_start]Script hospedado no Google Colab que lê os CSVs, realiza a limpeza das chaves, executa o JOIN das tabelas e aplica as regras de negócio[cite: 202, 208]. |
| **Armazenamento (Saída)** | Google Sheets | [cite_start]O script Python atualiza uma planilha mestre consolidada, que serve como banco de dados para o painel[cite: 202]. |
| **Visualização** | Looker Studio | [cite_start]Dashboard visual conectado diretamente à planilha mestre do Google Sheets[cite: 202]. |
| **Alertas** | Python (Gmail API/SMTP) | [cite_start]Lógica que varre a base processada e dispara e-mails automáticos para o vendedor ao detectar criticidade[cite: 202]. |

---

## 3. Fontes de Dados e Regras de Cruzamento (Join)
[cite_start]O sistema é alimentado pelo cruzamento de duas grandes visões operacionais[cite: 124]:

1. [cite_start]**Pedidos Supply (Ciclo do Pedido):** Base desenvolvida para gerenciar e controlar detalhadamente os prazos planejados e cumpridos de cada etapa logística dos pedidos (Faturamento, First Mile, Middle Mile, Last Mile, etc.)[cite: 2, 3, 4].
2. [cite_start]**Cubo de Vendas:** Base comercial que fornece a inteligência de negócios, identificando o código do vendedor (`TDV ID Vendedor Tdv`), o nome do produto (`Produto Nome Produto`) e o modelo da operação (`Vendas Tipo Venda` - 1P ou 3P)[cite: 123, 127, 135, 141].

### Estratégia de Higienização de Chaves
[cite_start]Como o campo de identificação do pedido na tabela de Supply possui prefixos de texto (ex: `Z64985914`), e na tabela comercial o identificador é puramente numérico (ex: `64985914`), o script executa um tratamento via Expressão Regular ($\text{Regex}$) antes do cruzamento de dados[cite: 146]:

```python
# Trecho do script Python para higienização e unificação das bases [cite: 149]
import pandas as pd

df_supply = pd.read_csv('pedido_supply_raw.csv')
df_cubo = pd.read_csv('cubo_de_vendas_raw.csv')

# Remove qualquer caractere que NÃO seja número e garante tipo string [cite: 155, 156]
df_supply['pedido_venda_limpo'] = df_supply['Pedidos Supply Pedido Venda'].astype(str).str.replace(r'\D', '', regex=True) [cite: 156]
df_cubo['vendas_id_str'] = df_cubo['Vendas ID Pedido'].astype(str) [cite: 157]

# Executa o JOIN perfeito das tabelas utilizando as chaves tratadas [cite: 158]
df_consolidado = pd.merge(df_cubo, df_supply, left_on='vendas_id_str', right_on='pedido_venda_limpo', how='inner') [cite: 159]
