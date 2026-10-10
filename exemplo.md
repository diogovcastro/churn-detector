
<h1 align="center">📰 COMPASS UOL - PROJETO 2 | SQUAD 3</h1>


<p align="center">
  <img src="assets/compass-header.jpeg" alt="Banner do projeto" width="600px">
</p>

# Projeto: Pipeline de Análise de Dados de ELA na AWS - Scholarship Compass UOL (IA para AWS)

## 🎯 Objetivo Principal
Este projeto, desenvolvido ao longo das **sprints 4, 5 e 6** do Scholarship Compass UOL, implementa um **pipeline de dados completo na nuvem AWS** para o pré-processamento e análise de um conjunto de dados sobre pacientes com **Esclerose Lateral Amiotrófica (ELA)**.

O foco principal é na **preparação dos dados** e na realização de uma **Análise Exploratória de Dados (EDA)** para identificar tendências, padrões e compreender as relações entre as variáveis clínicas e demográficas, com ênfase no impacto sobre os **meses de sobrevivência** dos pacientes.

## 🏗️ Arquitetura e Fluxo de Dados na AWS
A solução utiliza uma arquitetura **serverless e escalável** na AWS, seguindo os passos:

### 📤 Upload e Armazenamento (S3)
O dataset inicial (`trilha_ela_dataset.csv`) foi carregado via **boto3** para um **bucket no Amazon S3**, organizado em camadas (Raw).

### 🏷️ Catalogação (Glue Data Catalog)
Um **AWS Glue Crawler** foi executado sobre a camada Raw para **inferir o esquema** e **criar/atualizar uma tabela** no catálogo de dados.

### 🧾 Consulta Inicial (Athena)
O **AWS Athena** foi usado para validar o **esquema** e a **acessibilidade** dos dados brutos via SQL.

### ⚙️ Processamento ETL (Glue Job com PySpark)
Um **script Python com PySpark**, executado como **AWS Glue Job**, realizou:
- Leitura dos dados do S3.
- Limpeza e padronização.  
  **Nota:** Valores ausentes em `Gene` e outliers em `AgeOfOnset` e `SurvivalMonths` foram mantidos intencionalmente por serem considerados relevantes para a análise clínica.
- Salvamento do dataset processado em **formato Parquet** na camada Trusted do S3.

### 📊 Análise Exploratória e Visualização (QuickSight)
- O dataset Parquet (Trusted) foi a fonte para a EDA, analisando **distribuições** (idade de início), **frequências** (local de início) e **sobrevivência**.
- O **AWS QuickSight** foi conectado aos dados processados para criar **dashboards interativos**, visualizando a jornada dos pacientes e fatores associados à sobrevivência.

### 📜 Logs (CloudWatch)
Os logs de execução dos processos (especialmente do Glue Job) foram monitorados e armazenados no **AWS CloudWatch**.

## 🔬 Análises Adicionais
Utilizando os dados processados da camada Trusted, foram realizadas análises complementares (detalhadas nos notebooks `eda_notebook.ipynb` e `Regressao_Linear.ipynb`):

- **Regressão Linear Múltipla:** Um modelo de **machine learning (scikit-learn)** foi treinado para prever os **meses de sobrevivência (`SurvivalMonths`)** com base em múltiplas variáveis. O modelo (**R² ≈ 0.24**) ajudou a quantificar a associação de cada fator com a sobrevida. As previsões foram adicionadas ao dataset (`ela_data_with_predictions.parquet`).

## Tecnologias utilizadas

### EDA

- Pandas: manipulação de DataFrames

- PySpark: para manipulação de grandes volumes

- Statsmodels: análise de variância, regressões

- Integração com QuickSight (saídas compatíveis).

- Matplotlib/Seaborn : visualizações

- Scikit-learn: regressão linear e métricas

### Infraestrutura

- Amazon athena: para consultas SQL

- Boto3 (SDK da AWS para Python).

- Botocore: tratamento de exceções AWS.

- dotenv: carregamento de variáveis de ambiente (.env).

### ETL

- AWS Glue (awsglue library): gerenciamento do job, contexto e parâmetros.

- PySpark (SparkContext, DataFrame, SQL functions): transformação e limpeza de dados.

- S3 + Glue Catalog: leitura e escrita de dados parquet particionados.

- Python (datetime, json, sys, traceback): logging, controle de fluxo e métricas.

## Dificuldades
- Tivemos dificuldade em fazer o tratamento dos dados e formatar as colunas para terem o tipo correto de dados

## ⚙️ Como Utilizar o Sistema (Fluxo de Operação)

Este projeto implementa um pipeline de dados na AWS para pré-processamento, análise e visualização de dados sobre Esclerose Lateral Amiotrófica (ELA). Siga os passos abaixo para configurar e executar o pipeline:

### 1. 📋 Pré-requisitos e Configuração Inicial

- **Credenciais AWS**: Certifique-se de que suas credenciais AWS estejam configuradas corretamente no ambiente onde os scripts serão executados (via `aws configure` ou variáveis de ambiente).
- **Arquivo `.env`**: Crie um arquivo `.env` na raiz do projeto (`sprints-4-5-6-agosto/`) para armazenar as variáveis de ambiente necessárias para os scripts na pasta `infra/`. As variáveis incluem:
    * `BUCKET_NAME` (ou `BUCKET_NAME_1`): Nome do bucket S3 para armazenar os dados (ex: `datalake-ela-project-raw-squad-3`).
    * `AWS_DEFAULT_REGION` (ou `REGION`): Região AWS (ex: `us-east-1`).
    * `ROLE_ARN`: ARN do IAM Role que o AWS Glue usará para acessar outros serviços.
    * `CRAWLER_NAME_1`: Nome para o crawler do Glue (ex: `crawler-ela-raw`).
    * `DATABASE_NAME_1`: Nome para o banco de dados no Glue Data Catalog (ex: `db_ela_raw`).
    * `PREFIX_1`: Prefixo no S3 onde os dados estão (ex: `raw/` ou `trusted/`, dependendo do crawler).
    * *(Nota: Pode ser necessário duplicar/ajustar essas variáveis para crawlers diferentes, como um para `raw` e outro para `trusted`)*.
- **Dados Brutos**: Coloque o arquivo `trilha_ela_dataset.csv` na raiz do projeto (`sprints-4-5-6-agosto/`).
- **Dependências Python**: Instale as bibliotecas necessárias listadas em `requirements.txt` (boto3, botocore, python-dotenv, pyspark, pandas, statsmodels, scikit-learn, etc.).

### 2. 🏗️ Configuração da Infraestrutura AWS (Pasta `infra/`)

1.  **Criar Bucket e Upload Inicial**: Execute o script para criar o bucket S3 e fazer o upload dos dados brutos:
    ```bash
    python infra/create_bucket.py
    ```
- Este script utiliza `boto3` para criar o bucket definido em `.env` (se não existir) e carrega `trilha_ela_dataset.csv` para a pasta `raw/` dentro do bucket.

2.  **Catalogar Dados Brutos (Glue Crawler - Camada Raw)**:
- Configure as variáveis no `.env` para o crawler da camada `raw` (ex: `CRAWLER_NAME_1`, `DATABASE_NAME_1`, `BUCKET_NAME_1`, `PREFIX_1=raw/`).
- Execute o script `infra/glue_crawler_trusted.py` (adaptado para a camada `raw` ou use um script similar específico para `raw`).
        ```bash
        python infra/glue_crawler_raw.py # (Nome exemplo, ajuste conforme seu arquivo)
        ```
- Isso criará o banco de dados e o crawler no Glue, e o iniciará para catalogar o `trilha_ela_dataset.csv` no S3.

3.  **Validar Dados Brutos (Athena)**:
- Acesse o AWS Athena no console.
- Execute as consultas SQL presentes em `infra/athena-queries.sql` para verificar se a tabela da camada `raw` foi criada corretamente e se os dados estão acessíveis.

### 3. ⚙️ Processamento ETL (Pasta `etl/`)

1.  **Executar o Glue Job**:
    * Faça o upload do script `etl/glue-job-p2.py` para um bucket S3 acessível pelo Glue.
    * Crie um AWS Glue Job no console AWS, configurando:
        * O script Python (`glue-job-p2.py`).
        * O IAM Role (`ROLE_ARN` do `.env`).
        * Os parâmetros do Job (passados como argumentos para o script), como `JOB_NAME`, `BUCKET_NAME` (para leitura e escrita), `RAW_PREFIX`, `TRUSTED_PREFIX`, `DATABASE_NAME` (onde a tabela processada será registrada), `TABLE_NAME` (nome da tabela processada), `OUTPUT_PATH` (caminho S3 para os Parquets processados).
    * Execute o Glue Job. Ele lerá o CSV da camada `raw`, aplicará as transformações, salvará os dados processados em formato Parquet na camada `trusted` do S3 (particionados por `DiagnosisYear`), **e registrará automaticamente a tabela final no Catálogo de Dados do Glue** (através do comando `.saveAsTable`). 


### 4. 🔬 Análise Exploratória e Modelagem (extras)

- Os notebooks podem ser executados localmente (com Spark e dependências instaladas) ou em um ambiente como AWS SageMaker ou EMR que tenha acesso aos arquivos Parquet no S3 (ou copie os arquivos Parquet para o ambiente de execução).*

1.  **Análise Inicial (Opcional)**: Execute `eda/analise_da_base.ipynb` usando o CSV original para uma exploração rápida com Pandas.
2.  **Regressão Linear**: Execute `eda/Regressão_Linear.ipynb`. Este notebook:
    - Lê os arquivos Parquet da camada `trusted` (pode precisar ajustar o caminho `parquet_directory_path`).
    - Treina um modelo de Regressão Linear com `scikit-learn` para prever `SurvivalMonths`.
    - Salva um novo arquivo Parquet (`ela_data_with_predictions.parquet`) contendo os dados originais processados mais as colunas de previsão (`SurvivalMonths_Predicted`) e erro (`Prediction_Error`). Coloque este arquivo na pasta `parquets/` ou ajuste os caminhos nos próximos passos.
3.  **Análises Adicionais (ANOVA e Agregações)**: Execute `eda/eda_notebook.ipynb`. Este notebook:
    - Lê o arquivo `ela_data_with_predictions.parquet` usando PySpark.
    - Utiliza funções do `eda/analise.py` para:
        * Realizar um teste ANOVA (Idade de Início vs. Gene * Ano).
        * Gerar agregações (frequência de local de início, idade média por gene/ano, média de sobrevida por local/progressão, média de sintomas motores por gene/sexo).
    - Salva múltiplos arquivos Parquet com esses dados agregados na pasta `./resultados_para_quicksight/` (ou caminho especificado em `output_path` no notebook).

### 5. 📊 Visualização (AWS QuickSight)

1.  **Conectar Fonte de Dados**: No AWS QuickSight, crie uma nova análise e um novo conjunto de dados.
2.  **Escolher Dados**: Como fonte, você pode usar:
    **Athena**: Conecte-se à tabela do Glue Data Catalog referente à camada `trusted`. Esta é a abordagem mais direta para usar os dados processados completos.
    **S3**: Faça upload direto do arquivo csv gerado pelo `Regressão_linear` para o SPICE (motor de memória do QuickSight) ou conecte-se diretamente ao arquivo no S3.
3.  **Criar Visualizações**: Use os dados carregados no QuickSight para criar os dashboards e gráficos interativos, como os exemplos mostrados na seção "Resultados — AWS QuickSight" deste Readme.

### 6. 📜 Monitoramento (AWS CloudWatch)

- Monitore os logs de execução do Glue Job e dos Crawlers no AWS CloudWatch para verificar o status e depurar possíveis erros. O Glue Job (`glue-job-p2.py`) também salva métricas de execução (linhas lidas/escritas, status) em formato JSON na camada `trusted` do S3.

## Organização

- Dailys de squad via discord
- Trello

<p align="center">
  <img src="assets/trello.jpeg" alt="Resultados" width="600px">
</p>

## Estrutura do projeto

```bash
sprints-4-5-6-agosto/  
│  
├── README.md  
├── requirements.txt  
├── .gitignore  
│  
├── assets/               \# imagens   
│  
├── data/  
│   ├── raw/              \# Dados originais  
│   └── parquets/         \# Arquivos parquet gerados  
│  
├── eda/  
│   ├── Regressao\_linear.ipynb  
│   ├── analise\_da\_base.ipynb  
│   ├── eda\_notebook.ipynb  
│   └── analise.py  
│  
├── etl/  
│   └── glue-job-p2.py  
│  
├── infra/  
│   ├── athena-consultations.sql  
│   ├── create\_bucket.py  
│   └── glue\_crawler.py
```

## 📊 Resultados — AWS QuickSight

### 👶 1. Distribuição da idade de início da doença
<p align="center">
  <img src="assets/res-1.png" alt="Resultados" width="600px">
</p>

📅 **Descrição**:
O histograma exibe a quantidade de pacientes por faixa etária de início da doença (ageofonset).

📉 **Análise**:
A maior concentração ocorre entre 50 e 60 anos, o que está de acordo com o perfil clínico típico da ELA.
Casos abaixo dos 40 ou acima dos 70 anos são menos frequentes, reforçando a incidência predominante em adultos de meia-idade.

### 💪 2. Pontuação funcional ALSFRS-R por ano de diagnóstico
<p align="center">
  <img src="assets/res-2.png" alt="Resultados" width="600px">
</p>

📊 **Descrição**:
O gráfico compara a pontuação ALSFRS-R (indicador da capacidade motora) com o ano de diagnóstico.

🧩 **Análise**:
Pontuações mais baixas indicam maior gravidade.
Pacientes diagnosticados em anos recentes tendem a apresentar valores funcionais mais altos, o que pode refletir diagnóstico precoce e melhor acompanhamento clínico.

### 🧠 3. Locais de início da doença
<p align="center">
  <img src="assets/res-3.png" alt="Resultados" width="600px">
</p>

🥧 **Descrição**:
Gráfico de pizza que mostra a frequência dos locais de início da ELA (onsetsite).

🔍 **Análise**:

- Upper Limb: 36%
- Lower Limb: 31%
- Bulbar: 22%
- Axial: 11%

A maior parte dos casos se inicia nos membros superiores, reforçando a tendência observada em estudos clínicos.
Essa visão auxilia na análise comparativa da progressão entre grupos.

### 🧬 4. Taxa de sobrevivência ao longo do tempo
<p align="center">
  <img src="assets/res-4.png" alt="Resultados" width="600px">
</p>

📊 **Descrição**:
Este gráfico apresenta a soma dos meses de sobrevivência (survivalmonths) ao longo dos anos de diagnóstico, segmentada pelos principais fatores genéticos (Gene).

🔍 **Análise**:
O grupo sem mutação conhecida (None) apresenta os maiores valores de sobrevivência, enquanto genes como Sod1 e Tardbp mostram menores taxas em determinados períodos.

## 📊 Resultados — Regressão Linear

As previsões de sobrevida foram exploradas visualmente através de gráficos no Amazon QuickSight, com o objetivo de entender as relações entre a idade de início da doença, a pontuação de sintomas motores e o tempo de vida previsto pelo modelo de regressão linear.

### 1. Relação entre idade de início e meses de sobrevida previstos
<p align="center">
  <img src="assets/rl-01.png" alt="Resultados" width="600px">
</p>

📊 **Descrição**:
Exibe a soma de SurvivalMonths_Predicted em função da idade de início (AgeOfOnset). O gráfico mostra como o tempo de sobrevida previsto varia conforme a idade em que os sintomas surgem.

🔍 **Análise**:
Observa-se uma tendência decrescente — quanto menor a idade de início, maior o tempo de sobrevida previsto. Isso indica uma correlação negativa entre idade e expectativa de vida estimada.

### 2. Relação entre sintomas motores e sobrevida prevista
<p align="center">
  <img src="assets/rl-02.png" alt="Resultados" width="600px">
</p>

📊 **Descrição**:
Apresenta a relação entre MotorSymptomsScore (pontuação de sintomas motores) e SurvivalMonths_Predicted, com os pontos coloridos segundo a idade de início (AgeOfOnset). Cada ponto representa um conjunto de pacientes com diferentes combinações dessas variáveis.

🔍 **Análise**:
Mostra uma tendência positiva entre pontuação de sintomas motores e tempo de sobrevida previsto: quanto mais leve o quadro motor, maior o tempo de sobrevida. As cores reforçam que a idade de início também influencia indiretamente a sobrevida, pois pacientes mais jovens (tons frios) geralmente apresentam melhores projeções.Em conjunto, os gráficos sugerem que a idade e a gravidade dos sintomas motores são fatores-chave para estimar o tempo de vida projetado pelo modelo.


## Desenvolvedores

<table style="width:80%; border-collapse: collapse; margin: auto;">
  <tr>
    <td style="text-align:center; vertical-align: middle; padding: 20px; border: 1px solid #ddd;">
      <img src="https://avatars.githubusercontent.com/u/72031716?v=4" alt="Luana Ataide" width="120" height="120" style="border-radius: 50%;">
      <br>
      <strong>Luana Ataide</strong>
      <br>
      <a href="https://github.com/ataidekaroline" target="_blank">
        <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white" alt="GitHub">
      </a>
      <a href="https://www.linkedin.com/in/ataidekaroline/" target="_blank">
        <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn">
      </a>
    </td>
    <td style="text-align:center; vertical-align: middle; padding: 20px; border: 1px solid #ddd;">
      <img src="https://avatars.githubusercontent.com/u/87333686?v=4" alt="Juan Melo" width="120" height="120" style="border-radius: 50%;">
      <br>
      <strong>Juan Melo</strong>
      <br>
      <a href="https://github.com/MeloJu" target="_blank">
        <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white" alt="GitHub">
      </a>
      <a href="https://www.linkedin.com/in/juan-melo-705626199/" target="_blank">
        <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn">
      </a>
    </td>
    <td style="text-align:center; vertical-align: middle; padding: 20px; border: 1px solid #ddd;">
      <img src="https://avatars.githubusercontent.com/u/193746509?s=400&u=80fdd679c009121f9d414ea12482298d1365dc76&v=4" alt="Diogo Vieira" width="120" height="120" style="border-radius: 50%;">
      <br>
      <strong>Diogo Vieira</strong>
      <br>
      <a href="https://github.com/WizardNoob25" target="_blank">
        <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white" alt="GitHub">
      </a>
      <a href="https://www.linkedin.com/in/diogo-vieira-b12787219/" target="_blank">
        <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn">
      </a>
    </td>
  </tr>
</table>
