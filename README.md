# 📉 Churn Detector — Previsão de Cancelamento de Clientes

Projeto de mineração de dados e aprendizado de máquina para **classificar clientes com risco de cancelar um serviço**. A base utilizada é o arquivo `WA_Fn-UseC_-Telco-Customer-Churn.csv`, que contém informações sobre clientes de telecomunicações, serviços contratados, cobrança e cancelamento.

O projeto será desenvolvido e executado **localmente**, com Python, notebooks e arquivos armazenados nas camadas `raw` e `processed`.

> **Status atual:** estrutura de pastas, dados brutos e documentação inicial preparados. O processamento, os notebooks e os modelos ainda serão implementados. Não há métricas de modelos treinados nesta etapa.

## 🎯 Objetivo principal

Construir um modelo de **classificação binária** que utilize as características dos clientes para estimar a probabilidade de churn e apoiar a priorização de ações de retenção.

A variável-alvo é `Churn`:

- **`Yes` → 1:** cliente identificado na base como tendo cancelado o serviço.
- **`No` → 0:** cliente identificado na base como não tendo cancelado o serviço.

O trabalho abrange a preparação dos dados, a análise exploratória, o treinamento de modelos, a avaliação dos resultados e a geração de previsões para novos registros com as mesmas características de entrada.

**Escopo da previsão:** esta base registra churn, mas não contém uma série temporal de uso nem datas suficientes para definir uma janela futura de cancelamento. O modelo aprenderá a relação entre as características e o rótulo disponível. Prever abandono por inatividade ou cancelamento nos próximos 30 dias exigiria dados e rótulos adicionais.

## 🗂️ Base de dados

Arquivo original: [`data/raw/WA_Fn-UseC_-Telco-Customer-Churn.csv`](data/raw/WA_Fn-UseC_-Telco-Customer-Churn.csv).

Os números abaixo foram conferidos diretamente no CSV fornecido:

| Característica | Valor |
| --- | --- |
| Clientes | 7.043 |
| Colunas | 21 |
| Identificador | `customerID` |
| Variável-alvo | `Churn` |
| Clientes com churn (`Yes`) | 1.869 — 26,54% |
| Clientes sem churn (`No`) | 5.174 — 73,46% |
| Identificadores únicos | 7.043 |
| Linhas duplicadas | 0 |
| Valores em branco | 11 em `TotalCharges` |

Depois de separar o identificador e a variável-alvo, ficam **19 variáveis candidatas** a entradas do modelo. A seleção final será avaliada durante a modelagem.

### Dicionário das variáveis

| Coluna | Descrição | Papel no projeto |
| --- | --- | --- |
| `customerID` | Identificador do cliente | Identificação; excluído das entradas do modelo |
| `gender` | Gênero registrado | Categórica |
| `SeniorCitizen` | Indicador de cliente idoso (`0` ou `1`) | Binária |
| `Partner` | Indicador de parceiro(a) | Categórica binária |
| `Dependents` | Indicador de dependentes | Categórica binária |
| `tenure` | Tempo de permanência do cliente, em meses | Numérica |
| `PhoneService` | Contratação de serviço telefônico | Categórica binária |
| `MultipleLines` | Contratação de múltiplas linhas ou ausência de telefonia | Categórica |
| `InternetService` | Tipo de internet: DSL, fibra óptica ou sem internet | Categórica |
| `OnlineSecurity` | Contratação de segurança online ou ausência de internet | Categórica |
| `OnlineBackup` | Contratação de backup online ou ausência de internet | Categórica |
| `DeviceProtection` | Contratação de proteção de dispositivos ou ausência de internet | Categórica |
| `TechSupport` | Contratação de suporte técnico ou ausência de internet | Categórica |
| `StreamingTV` | Contratação de streaming de TV ou ausência de internet | Categórica |
| `StreamingMovies` | Contratação de streaming de filmes ou ausência de internet | Categórica |
| `Contract` | Tipo de contrato: mensal, um ano ou dois anos | Categórica |
| `PaperlessBilling` | Uso de faturamento eletrônico | Categórica binária |
| `PaymentMethod` | Forma de pagamento | Categórica |
| `MonthlyCharges` | Valor cobrado mensalmente | Numérica |
| `TotalCharges` | Valor total cobrado | Numérica após conversão |
| `Churn` | Indicador de cancelamento (`Yes` ou `No`) | Variável-alvo |

### Pontos de atenção encontrados

- `TotalCharges` contém 11 campos preenchidos apenas com espaços. Todos esses registros apresentam `tenure = 0`. A conversão para número deverá preservar esses casos como valores ausentes até a aplicação da estratégia de tratamento.
- Categorias como `No internet service` e `No phone service` representam ausência do serviço correspondente; não são valores ausentes.
- As classes são desbalanceadas. Prever sempre `No` produziria 73,46% de acurácia e recall zero para churn, por isso a acurácia isolada não será suficiente para avaliar os modelos.

## 🏗️ Fluxo de dados local

```text
data/raw/ — CSV original
    ↓
Validação e limpeza determinística
    ↓
data/processed/ — dados limpos
    ↓
Análise exploratória e definição das variáveis
    ↓
Separação estratificada entre treino e teste
    ↓
Pré-processamento e treinamento com validação cruzada no treino
    ↓
Avaliação final no teste
    ↓
models/ — pipeline salvo
reports/ — gráficos e métricas
data/processed/ — previsões geradas
```

### 1. Dados brutos — `raw`

Armazenar o CSV original em `data/raw/`, mantendo seu conteúdo intacto. Essa camada será a fonte para reproduzir o processamento.

### 2. Preparação dos dados — `processed`

O processamento deverá:

- Validar as colunas esperadas, os identificadores, os rótulos e os tipos.
- Verificar duplicidades e valores ausentes a cada execução.
- Remover espaços desnecessários e converter `TotalCharges` para número.
- Mapear `Churn` para `0` e `1`, mantendo `customerID` para rastrear os registros.
- Salvar uma versão limpa em `data/processed/telco_churn_clean.csv`.

O CSV limpo poderá conservar valores ausentes. Imputação, codificação e escalonamento serão ajustados durante o treinamento, dentro do pipeline, para evitar vazamento de dados.

### 3. Análise exploratória — EDA

Os notebooks deverão investigar:

- A distribuição de churn e o equilíbrio entre as classes.
- A relação de churn com tempo de permanência, tipo de contrato e cobranças.
- A distribuição de churn por serviços contratados e formas de pagamento.
- Valores ausentes, possíveis outliers e relações entre variáveis.

Os gráficos serão armazenados em `reports/figures/`. As análises descreverão associações observadas na base; não demonstram, por si só, causas de cancelamento.

### 4. Pré-processamento e modelagem

A estratégia inicial será reservar **20% dos dados para teste**, com separação estratificada por `Churn` e `random_state=42`. Os outros 80% serão usados para treinamento e validação cruzada estratificada.

O pré-processamento utilizará `Pipeline` e `ColumnTransformer` do scikit-learn:

- Imputação de valores numéricos ausentes, inicialmente pela mediana do treino.
- Codificação das categorias com one-hot encoding e tratamento de categorias desconhecidas.
- Escalonamento das variáveis numéricas para modelos que se beneficiem dele, como a Regressão Logística.
- Exclusão de `customerID` e `Churn` das entradas do modelo.

Essas transformações serão ajustadas **somente no treino de cada divisão ou fold** e depois aplicadas à validação ou ao teste. O conjunto de teste ficará reservado para a avaliação final.

Modelos previstos para comparação:

| Modelo | Objetivo |
| --- | --- |
| `DummyClassifier` | Estabelecer uma referência mínima de desempenho |
| Regressão Logística | Criar uma referência interpretável para classificação |
| Random Forest | Avaliar relações não lineares e interações entre variáveis |

O uso de pesos de classe será avaliado na validação. O limiar de decisão também poderá ser ajustado com base nos resultados de validação e no equilíbrio desejado entre identificar churn e gerar falsos positivos.

### 5. Avaliação e previsões

As métricas previstas são:

| Métrica | O que ajuda a avaliar |
| --- | --- |
| Recall da classe churn | Quantos clientes que cancelam são identificados |
| Precisão da classe churn | Quantas previsões de cancelamento estão corretas |
| F1-score da classe churn | Equilíbrio entre precisão e recall |
| Average precision (AP), com curva precisão-recall | Qualidade da ordenação por risco na classe de interesse |
| ROC-AUC | Capacidade de distinguir as classes ao variar o limiar |
| Matriz de confusão | Quantidade de acertos, falsos positivos e falsos negativos |
| Acurácia | Desempenho geral, como medida complementar |

O recall será uma prioridade inicial para encontrar clientes em risco, acompanhado da precisão para avaliar o volume de contatos desnecessários. A seleção do modelo e do limiar acontecerá na validação; o teste medirá o desempenho da solução escolhida.

A saída de previsão planejada será `data/processed/churn_predictions.csv`, contendo `customerID`, `churn_probability` e `churn_prediction`. O pipeline completo de pré-processamento e classificação será salvo em `models/churn_pipeline.joblib`.

## 🛠️ Tecnologias previstas

| Tecnologia | Uso |
| --- | --- |
| Python | Processamento, treinamento e inferência local |
| Pandas e NumPy | Manipulação e análise dos dados |
| Matplotlib e Seaborn | Gráficos exploratórios e de avaliação |
| Scikit-learn | Pré-processamento, classificação, validação e métricas |
| JupyterLab | Desenvolvimento dos notebooks |
| Joblib | Persistência do pipeline treinado |

As dependências iniciais estão em [`requirements.txt`](requirements.txt). As versões serão fixadas quando a implementação e o ambiente de execução forem validados.

## 📁 Estrutura do projeto

```text
churn-detector/
├── README.md
├── exemplo.md                   # Referência de documentação fornecida
├── requirements.txt             # Dependências iniciais
├── .gitignore
├── data/
│   ├── raw/
│   │   └── WA_Fn-UseC_-Telco-Customer-Churn.csv
│   └── processed/               # Dados limpos e previsões gerados localmente
├── notebooks/                   # Análise exploratória e experimentos
├── src/
│   ├── data/                    # Leitura, validação e limpeza dos dados
│   ├── features/                # Transformações e construção de variáveis
│   ├── models/                  # Treinamento, avaliação e inferência
│   └── visualization/           # Funções para gerar gráficos
├── models/                      # Pipelines treinados e serializados
└── reports/
    ├── figures/                 # Gráficos de EDA e avaliação
    └── metrics/                 # Métricas e comparação dos modelos
```

As pastas sem conteúdo possuem `.gitkeep` para preservar a estrutura no versionamento. `src/models/` guardará código; `models/` guardará os artefatos produzidos por esse código.

## ⚙️ Preparação do ambiente local

Pré-requisitos: Python 3, pip e suporte a ambientes virtuais. Execute os comandos na raiz do projeto.

### Linux / macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

### Windows — PowerShell

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Para abrir o ambiente de notebooks após instalar as dependências:

```bash
jupyter lab
```

O CSV já está em `data/raw/`. Os notebooks e scripts serão adicionados nas próximas etapas; ainda não há um comando de execução do pipeline completo.

## 📋 Etapas de desenvolvimento

- [x] Conferir a estrutura e as características iniciais da base.
- [x] Organizar as camadas `raw` e `processed`.
- [x] Criar a estrutura de pastas e a documentação inicial.
- [ ] Implementar a validação e a limpeza do CSV.
- [ ] Criar o notebook de análise exploratória.
- [ ] Implementar o pipeline de pré-processamento e classificação.
- [ ] Comparar os modelos e ajustar o limiar na validação.
- [ ] Avaliar a solução escolhida no conjunto de teste.
- [ ] Salvar o pipeline e gerar previsões para novos registros.
- [ ] Documentar os resultados, as limitações e as conclusões.

## 📊 Resultados esperados

Ao concluir o projeto, serão entregues o conjunto de dados limpo, os notebooks de análise, a comparação dos modelos, os gráficos e métricas de avaliação, o pipeline salvo e um arquivo com estimativas de risco de churn.

Os resultados medidos e as conclusões serão registrados aqui depois da execução dos experimentos.
