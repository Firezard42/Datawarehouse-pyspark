# 📊 Mini Data Warehouse com PySpark e Arquitetura Medalhão

Este projeto tem como objetivo construir um mini **Data Warehouse** utilizando **PySpark**, baseado na **Arquitetura Medalhão (Medallion Architecture)**, com dados públicos da Receita Federal.

O fluxo segue as camadas tradicionais da arquitetura:

- **Landing (LND)**: Armazenamento dos dados brutos diretamente dos arquivos CSV extraídos.
- **Raw (RAW)**: Estruturação dos dados com schemas definidos em PySpark.
- **Trusted (TRS)**: Aplicação de regras de consistência, enriquecimento (enrichment), e junções entre fatos e dimensões.
- **Análise (Notebook Final)**: Consultas e visualizações com o Delta Lake.

---

## 🗂️ Dados utilizados

Os dados foram obtidos do portal oficial da Receita Federal na seção de [Dados Abertos CNPJ](https://dadosabertos.rfb.gov.br/CNPJ). Os arquivos utilizados incluem:

- **Empresas**
- **Estabelecimentos**
- **Natureza Jurídica**
- **CNAE (Classificação Nacional de Atividades Econômicas)**
- **Simples Nacional**
- **Municípios (da Receita Federal, não do IBGE)**

> ⚠️ **Observação**: Os dados complementares necessários para este projeto — como Natureza Jurídica, CNAE, Simples Nacional, Municípios da Receita, entre outros — **não estão disponíveis via API pública**. No entanto, podem ser baixados diretamente como arquivos `.CSV` no portal da Receita Federal.

---

## ⚙️ Estratégia de Coleta

- **Sem uso de paginação**:\
  O processo de ingestão **não utilizou paginação via API**.\
  Isso ocorreu pois **os dados foram obtidos em arquivos **``** contendo arquivos **``** separados por partes**, como `Empresas0.zip`, `Empresas1.zip` etc.

- **Processamento arquivo por arquivo**:\
  Cada um desses arquivos foi baixado, extraído, convertido e carregado manualmente em formato `.csv`.\
  Por isso, **não há um único notebook de extração genérica com paginação ou cursor**.\
  Cada CSV foi processado separadamente na camada de **Landing (LND)** e convertido para **Raw** com schema definido no PySpark.

---

## 🧱 Camadas da Arquitetura

### 🔹 Landing (LND)

Armazenamento inicial dos arquivos CSV convertidos de `.zip`, sem transformação.\
Exemplo: `empresas_2025-07-17.csv`, `simples_2025-07-17.csv`.

### 🔹 Raw (RAW)

Leitura dos CSVs com `pyspark.read.csv()` aplicando `schema` e gravando no formato Delta Lake.

### 🔹 Trusted (TRS)

Transformações aplicadas:

- Regras de consistência: remoção de `null`, `""`, ou registros incompletos
- Joins entre empresas e:
  - Natureza Jurídica
  - CNAE
  - Simples Nacional
  - Município (com enriquecimento da `UF`)
- Renomeação e limpeza de colunas

### 🔹 Análise

Consulta e visualização de dados agregados:

- Quantidade de empresas por UF
- Empresas optantes pelo Simples Nacional
- Distribuição por natureza jurídica
- Versionamento via `DESCRIBE HISTORY`

---

## 🛆 Formato e Versionamento

- Todos os dados das camadas **RAW** e **TRUSTED** foram salvos no formato **Delta Lake**.
- Foram feitas **duas versões** das tabelas Delta para demonstrar o versionamento e uso de `overwrite`.

Exemplo de comando usado:

```python
spark.sql("""
    DESCRIBE HISTORY delta.`C:/caminho/para/tabela`
""").show(truncate=False)
```

---

## ✅ Status do Projeto

| Etapa                            | Status |
| -------------------------------- | ------ |
| Coleta dos dados (manual)        | ✅      |
| Conversão e salvamento em LND    | ✅      |
| Preparação com schema na RAW     | ✅      |
| Regras de qualidade (Trusted)    | ✅      |
| Enriquecimento de dados          | ✅      |
| Joins entre fato e dimensões     | ✅      |
| Escrita em Delta + versionamento | ✅      |
| Análise e visualizações          | ✅      |

---

## 🛠️ Tecnologias e Bibliotecas

- **PySpark** 4.00
- **Delta Lake**
- **Jupyter Notebooks**
- **Requests**
- **findspark**

---

## 🧠 Conceitos Abordados

- Arquitetura de dados (Medallion)
- Tabelas Fato e Dimensão
- Operações de Join
- Qualidade de dados (regras de consistência)
- Versionamento com Delta Lake

---

## 📁 Organização de Pastas

```
IBGE_PROJETO/
🗂️ data/
   ├️ LND/         # Dados brutos (.csv)
   ├️ RAW/         # Dados estruturados em Delta
   └️ TRS/         # Dados confiáveis com joins e enrichments
🗋 notebooks/
   ├️ 01_coleta_dados.ipynb
   ├️ 02_preparacao_raw.ipynb
   ├️ 03_transformacao_trusted.ipynb
   └️ 04_analise_resultados.ipynb
```

---

## ✍️ Autor

Este projeto foi desenvolvido por Ivaldo Januário dos Santos Neto (Firezard42) como parte do treinamento prático da Residência em TIC da UFAL/EASY em parceria com o SENAI/AL, com foco em Engenharia de Dados.

O objetivo desta atividade é nivelar o conhecimento técnico dos residentes, preparando-os para o desenvolvimento do projeto real que será realizado nas etapas seguintes da residência.

---