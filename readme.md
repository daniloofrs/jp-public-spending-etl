# 📊 Pipeline ETL: Raio-X dos Gastos Públicos de João Pessoa (PB)

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Cleaning-150458?logo=pandas&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Container-2496ED?logo=docker&logoColor=white)
![Status](https://img.shields.io/badge/Status-Em%20Desenvolvimento-yellow)

## 📌 Sobre o Projeto

Este projeto consiste em um pipeline completo de **Engenharia e Análise de Dados (ETL)** focado nas despesas públicas do município de **João Pessoa - PB**. 

Dados governamentais brutos frequentemente apresentam problemas de padronização, codificação de caracteres (*encoding*), formatação monetária incompatível com bancos de dados e alta redundância. O objetivo deste repositório é coletar os dados abertos de execução orçamentária, aplicar regras rigorosas de limpeza com **Python (Pandas)** e carregá-los em um Data Warehouse local modelado no formato **Star Schema** em **PostgreSQL** (via **Docker**), permitindo análises rápidas e confiáveis via SQL.

---

## 🎯 Problema de Negócio e Perguntas Respondidas

A partir dos dados estruturados no banco relacional, o projeto busca responder às seguintes questões analíticas:

1. **Maiores Fornecedores:** Quais são as 10 empresas ou pessoas físicas que mais receberam recursos da prefeitura no período analisado?
2. **Distribuição por Pasta:** Quais secretarias municipais (Saúde, Educação, Infraestrutura, etc.) concentram o maior volume de pagamentos efetivados?
3. **Sazonalidade Orçamentária:** Qual é a evolução mensal dos gastos ao longo do ano? Há picos de execução no fechamento do exercício fiscal?
4. **Eficiência de Execução:** Qual a proporção entre o valor empenhado (prometido/reservado) e o valor efetivamente pago por secretaria?

---

## 🏗️ Arquitetura do Pipeline e Fluxo de Dados

```text
[Portal da Transparência JP / Sagres TCE-PB] 
       │
       ▼ (Download CSV Bruto)
📂 data/raw/
       │
       ▼ (Extract & Transform: Python + Pandas)
⚙️ Limpeza de Moeda, Datas, CNPJ/CPF e Encoding
       │
       ▼ (Load: SQLAlchemy + psycopg2)
🐳 PostgreSQL (Docker) ──► Modelagem Star Schema (Fato + Dimensões)
       │
       ▼ (Analytics)
📊 Consultas SQL Analíticas / Dashboards
```

---

## 🗄️ Modelagem de Dados (Star Schema)

Para evitar a redundância de uma tabela única (*flat table*) e demonstrar boas práticas de **Data Warehousing**, os dados foram divididos em duas tabelas de dimensão e uma tabela fato:

### `dim_orgao`
Armazena as unidades gestoras e secretarias municipais sem duplicidade.
* `id_orgao` (SERIAL PRIMARY KEY)
* `codigo_orgao` (VARCHAR)
* `nome_secretaria` (VARCHAR)

### `dim_fornecedor`
Cadastro único de credores da prefeitura.
* `id_fornecedor` (SERIAL PRIMARY KEY)
* `documento` (VARCHAR - CPF/CNPJ higienizado apenas com dígitos)
* `razao_social` (VARCHAR - Nome padronizado em caixa alta)
* `tipo_pessoa` (CHAR(2) - `'PF'` ou `'PJ'`)

### `fato_despesas`
Registra os eventos financeiros de empenho e pagamento.
* `id_despesa` (SERIAL PRIMARY KEY)
* `numero_empenho` (VARCHAR)
* `id_orgao` (INT FOREIGN KEY $\rightarrow$ `dim_orgao`)
* `id_fornecedor` (INT FOREIGN KEY $\rightarrow$ `dim_fornecedor`)
* `data_pagamento` (DATE)
* `valor_empenhado` (NUMERIC(15,2))
* `valor_pago` (NUMERIC(15,2))
* `descricao_despesa` (TEXT)

---

## 🛠️ Tecnologias Utilizadas

* **Linguagem:** Python 3.10+
* **Manipulação e Limpeza de Dados:** `pandas`, `numpy`
* **Banco de Dados:** PostgreSQL 15
* **ORM / Conector:** `SQLAlchemy`, `psycopg2-binary`
* **Infraestrutura e Ambiente:** Docker e Docker Compose

---

## 📂 Estrutura de Diretórios

```text
jp-public-spending-etl/
├── data/
│   ├── raw/                  # CSVs brutos baixados do portal (ignorados no Git)
│   └── processed/            # Dados limpos exportados para validação
├── sql/
│   ├── init_schema.sql       # DDL para criação automática do Star Schema no container
│   └── analytics_queries.sql # Consultas SQL respondendo às perguntas de negócio
├── src/
│   ├── extract.py            # Leitura e validação de schema dos CSVs brutos
│   ├── transform.py          # Tratamento de strings, datas, valores e deduplicação
│   └── load.py               # Carga nas tabelas dimensão e fato no PostgreSQL
├── docker-compose.yml        # Orquestração do banco de dados PostgreSQL
├── requirements.txt          # Dependências do ambiente Python
└── README.md                 # Documentação do projeto
```

---

## 🧹 Etapas de Tratamento de Dados (`transform.py`)

Os dados públicos brutos exigiram as seguintes transformações antes da carga no banco:

1. **Correção de Encoding e Delimitadores:** Tratamento de arquivos CSV exportados em `ISO-8859-1` (`latin1`) separados por ponto e vírgula (`;`).
2. **Conversão Monetária:** Remoção de símbolos (`R$`), pontos de separação de milhar e substituição de vírgula decimal por ponto (`"1.250,50"` $\rightarrow$ `1250.50`).
3. **Higienização de Documentos:** Aplicação de expressões regulares (Regex) para remover pontos, barras e traços de CPFs e CNPJs, classificando automaticamente entre Pessoa Física (`PF`) e Jurídica (`PJ`).
4. **Padronização de Strings:** Remoção de espaços extras (`.str.strip()`), conversão para letras maiúsculas e remoção de acentuação nos nomes de secretarias e fornecedores para evitar cadastros duplicados.
5. **Padronização Temporal:** Conversão de datas do formato brasileiro (`DD/MM/YYYY`) para o padrão ISO do banco de dados (`YYYY-MM-DD`).

---

## 🚀 Como Executar o Projeto Localmente

### Pré-requisitos
* [Docker](https://www.docker.com/) e Docker Compose instalados.
* [Python 3.10+](https://www.python.org/) instalado.

### Passo a Passo

1. **Clone o repositório:**
   ```bash
   git clone https://github.com/seu-usuario/jp-public-spending-etl.git
   cd jp-public-spending-etl
   ```

2. **Suba o banco de dados PostgreSQL com Docker:**
   ```bash
   docker-compose up -d
   ```
   *O script `sql/init_schema.sql` será executado automaticamente na criação do container, gerando as tabelas do Star Schema.*

3. **Crie o ambiente virtual e instale as dependências:**
   ```bash
   python -m venv venv
   # No Windows:
   venv\Scripts\activate
   # No Linux/Mac:
   source venv/bin/activate

   pip install -r requirements.txt
   ```

4. **Adicione os dados brutos:**
   Baixe o arquivo CSV de despesas no [Portal da Transparência de João Pessoa](https://transparencia.joaopessoa.pb.gov.br/) ou no portal [Sagres (TCE-PB)](https://sagresonline.tce.pb.gov.br/) e salve em `data/raw/despesas.csv`.

5. **Execute o pipeline ETL:**
   ```bash
   python src/extract.py
   python src/transform.py
   python src/load.py
   ```

---

## 📈 Exemplos de Consultas Analíticas (SQL)

No arquivo `sql/analytics_queries.sql`, você encontrará consultas prontas para extrair insights do banco, como o ranking de gastos por secretaria:

```sql
SELECT 
    o.nome_secretaria,
    COUNT(f.id_despesa) AS total_pagamentos,
    SUM(f.valor_pago) AS montante_total_pago,
    ROUND(AVG(f.valor_pago), 2) AS ticket_medio_pagamento
FROM fato_despesas f
JOIN dim_orgao o ON f.id_orgao = o.id_orgao
GROUP BY o.nome_secretaria
ORDER BY montante_total_pago DESC
LIMIT 10;
```

---

## ✒️ Autor

Desenvolvido por **Danilo** – Projeto de portfólio focado em Engenharia de Dados e Business Intelligence.
