# Resumo dos Encontros 01–03 — Foco na parte da Fabi

> **Como ler este resumo**
>
> - Tudo que está fora de caixas de destaque vem **direto dos slides**.
> - Blocos marcados com **➕ Complemento** são informações que **não estão (ou estão pouco explicadas) nos slides**, mas que considero importantes para a prova.
>
> **Conteúdos prováveis da prova (segundo a professora):**
> 1. EDA — análise **univariada, bivariada e multivariada** (histogramas, boxplots, heatmaps, gráficos de barras)
> 2. **Coeficientes de correlação** (Pearson e Spearman)
> 3. **Variáveis categóricas** (principalmente *quando usar cada técnica*)
> 4. **Scaling**

---

# 1. Onde a EDA se encaixa

&emsp;O **ciclo de vida de ML** apresentado no Encontro 01 tem 7 etapas:

```text
1. Problema de negócio
2. Coleta & preparação
3. EDA   ← foco do Encontro 01
4. Feature engineering   ← Encontro 02
5. Modelagem   ← Encontro 03
6. Avaliação
7. Deploy & monitoramento
```

&emsp;A **EDA (Exploratory Data Analysis)** é a etapa em que exploramos os dados a fundo, **antes** de criar features ou treinar qualquer modelo, para entender **qualidade, padrões e riscos escondidos**.

&emsp;Todo projeto começa com uma **pergunta de negócio** (não com um algoritmo). Exemplo do slide: *"prever quais clientes vão cancelar a assinatura nos próximos 30 dias, para priorizar ações de retenção"*.

> **Exemplo do slide de EDA:** descobrir, ainda na EDA, que 80% do churn acontece nos primeiros 3 meses de contrato — um padrão que muda a estratégia do projeto antes mesmo de treinar um modelo.

---

# 2. EDA — Análise Exploratória de Dados

## 2.1. Primeiro olhar: estatística descritiva

&emsp;É o primeiro olhar **quantitativo**: média, mediana, desvio padrão, mínimo, máximo e quartis.

&emsp;**Dica dos slides:** a diferença entre **média e mediana** já é um sinal de **assimetria**.

```python
df.describe()   # contagem, média, desvio, min, quartis (25/50/75%), max
df.info()       # tamanho, tipos e nulos
df.dtypes       # tipo de cada coluna
```

> **➕ Complemento — checklist do que procurar na EDA**
>
> - **Valores nulos** (`df.isna().sum()`) e como tratá-los
> - **Duplicatas** (`df.duplicated().sum()`)
> - **Tipos errados** (data lida como texto, número lido como string)
> - **Outliers** (erro de digitação ou valor legítimo?)
> - **Desbalanceamento** da variável alvo (`df['alvo'].value_counts(normalize=True)`)
> - **Escalas muito diferentes** entre variáveis (importa para o scaling)
> - **Variáveis com cardinalidade alta** (importa para o encoding)

## 2.2. Os três níveis de análise

| Nível | Quantas variáveis? | O que revela | Técnicas (slides) |
| :--- | :--- | :--- | :--- |
| **Univariada** | **1** por vez | Distribuição, tendência central e dispersão | Histogramas · Boxplots · Gráficos de barras |
| **Bivariada** | **2** | **Força, direção e padrão** da associação | Correlação (Pearson/Spearman) · Scatter plots |
| **Multivariada** | **3 ou mais** | Estrutura e **redundância** | Heatmaps · PCA |

---

## 2.3. Análise univariada

### Histograma

&emsp;**O que é:** mostra a **distribuição de frequência** de uma variável **numérica**, dividindo os valores em intervalos (**bins**). Revela a **forma** da distribuição: simétrica, assimétrica, concentrada ou dispersa.

&emsp;**Quando usar:** para entender como **uma única variável contínua** se comporta, antes de qualquer comparação.

&emsp;**Exemplo do slide:** idade dos clientes de um e-commerce concentrada entre 25 e 45 anos, com média de 34 — útil para direcionar campanhas por faixa etária.

> **➕ Complemento — como ler a forma do histograma**
>
> | Forma | Relação média × mediana | Observação |
> | :--- | :--- | :--- |
> | **Simétrica** (tipo normal) | média ≈ mediana | Distribuição "equilibrada" |
> | **Assimétrica à direita** (cauda longa para valores altos) | **média > mediana** | Típica de renda, preço, gasto |
> | **Assimétrica à esquerda** | **média < mediana** | Menos comum |
> | **Bimodal** (dois picos) | — | Pode indicar **dois grupos misturados** |
>
> - O número de **bins** muda a aparência: poucos bins escondem detalhes, muitos geram ruído.
> - Variáveis muito assimétricas costumam se beneficiar de uma **transformação log**.

### Boxplot (diagrama de caixa)

&emsp;**O que é:** resume a distribuição por **quartis (Q1, mediana, Q3)**, valores mínimo/máximo e **outliers**. Permite **comparar dispersão e simetria entre vários grupos lado a lado**.

&emsp;**Quando usar:** para **identificar outliers** e **comparar a variabilidade** de uma variável entre categorias.

&emsp;**Exemplo do slide:** gasto mensal por categoria de produto — Eletrônicos tem o maior gasto médio e mais outliers (compras de alto valor).

> **➕ Complemento — anatomia do boxplot**
>
> ```text
> IQR = Q3 − Q1
> Limite inferior = Q1 − 1,5 × IQR
> Limite superior = Q3 + 1,5 × IQR
> ```
>
> - A **caixa** vai de Q1 a Q3 (contém 50% dos dados centrais); a linha dentro é a **mediana**.
> - Os **"bigodes"** (whiskers) chegam até o último valor dentro dos limites acima.
> - Pontos **fora** dos limites são plotados como **outliers**.
> - Outlier **não é automaticamente erro**: pode ser um cliente de alto valor legítimo. Investigue antes de remover.

### Gráfico de barras

&emsp;**O que é:** compara a **frequência ou o total** de uma variável **categórica** entre categorias. A altura (ou comprimento) da barra é proporcional ao valor.

&emsp;**Quando usar:** para comparar contagens ou totais entre categorias distintas de forma direta.

&emsp;**Exemplo do slide:** categoria de produto preferida — Eletrônicos lidera em número de clientes, seguida por Moda e Casa.

> **➕ Complemento — histograma × gráfico de barras**
>
> | | Histograma | Gráfico de barras |
> | :--- | :--- | :--- |
> | Variável | **Numérica contínua** | **Categórica** |
> | Eixo X | Intervalos (bins) contínuos | Categorias separadas |
> | Barras | **Coladas** (sem espaço) | **Separadas** |
> | Ordem | Ordem natural dos valores | Pode-se ordenar por frequência |

---

## 2.4. Análise bivariada

### Scatter plot (gráfico de dispersão)

&emsp;**O que é:** cada observação é um ponto no plano formado por **duas variáveis numéricas**. Revela **padrões, tendências, clusters e outliers** que os coeficientes sozinhos não mostram.

&emsp;**Quando usar:** para **inspecionar visualmente** a relação entre duas variáveis, **complementando** os coeficientes de correlação.

&emsp;**Exemplo do slide:** idade × gasto mensal, segmentado por valor do cliente — clientes de alto valor ficam acima do gasto mediano, sem padrão claro de idade.

&emsp;A correlação em si é detalhada na **seção 3**.

---

## 2.5. Análise multivariada

### Heatmap (mapa de calor)

&emsp;**O que é:** exibe uma **matriz de valores** (tipicamente a **matriz de correlação**) usando **cores para indicar a magnitude**. Permite identificar rapidamente **pares de variáveis fortemente relacionadas** em datasets com muitas colunas.

&emsp;**Quando usar:** para ter uma **visão geral das relações entre várias variáveis numéricas** ao mesmo tempo.

&emsp;**Exemplo do slide:** matriz de correlação de 5 variáveis do cliente — gasto mensal e nº de compras têm a correlação mais forte (0,9+); avaliação tem pouca relação com idade.

```python
import seaborn as sns
sns.heatmap(df.corr(numeric_only=True), annot=True, cmap="coolwarm", vmin=-1, vmax=1)
```

> **➕ Complemento — como ler um heatmap de correlação**
>
> - A **diagonal** é sempre 1 (cada variável com ela mesma).
> - A matriz é **simétrica** (a metade de cima espelha a de baixo).
> - Cores fortes = correlação forte (positiva ou negativa, dependendo da escala).
> - **Uso prático:** achar variáveis **redundantes** (correlação muito alta entre *features*, ex.: > 0,9) e variáveis com **relação forte com o alvo**.
> - Cuidado: `df.corr()` por padrão é **Pearson** — use `method="spearman"` quando fizer sentido.

### PCA (Análise de Componentes Principais)

&emsp;**O que é:** técnica de **redução de dimensionalidade**. Transforma variáveis **correlacionadas** em **componentes principais não correlacionados**. Os primeiros componentes concentram a maior parte da **variância**, permitindo visualizar dados multivariados em 2D/3D.

&emsp;**Quando usar:** quando há **muitas variáveis numéricas correlacionadas** e é preciso simplificar, visualizar ou reduzir redundância.

&emsp;**Exemplo do slide:** com 5 variáveis de comportamento, CP1 e CP2 explicam ~71% da variância; clientes de alto valor se separam ao longo do CP1.

> **➕ Complemento:** o PCA é **sensível à escala** (por isso aparece na lista de algoritmos que precisam de **scaling**, seção 5). Normalmente aplica-se **padronização** antes do PCA.

---

## 2.6. Qual gráfico usar? (resumo rápido)

| Pergunta | Tipos de variável | Gráfico |
| :--- | :--- | :--- |
| Como uma variável numérica se distribui? | 1 numérica | **Histograma** |
| Há outliers? Como comparar grupos? | 1 numérica (por categoria) | **Boxplot** |
| Quantos itens em cada categoria? | 1 categórica | **Gráfico de barras** |
| Duas numéricas se relacionam? | 2 numéricas | **Scatter plot** + correlação |
| Quais variáveis se relacionam entre si? | várias numéricas | **Heatmap** de correlação |
| Posso resumir muitas variáveis em poucas? | várias numéricas | **PCA** |

---

# 3. Coeficientes de correlação (Pearson e Spearman)

&emsp;**Objetivo:** quantificar o **grau de associação** entre duas variáveis **antes de assumir causalidade**. Ambos variam de **−1 a +1**.

| | **Pearson (r)** | **Spearman (ρ)** |
| :--- | :--- | :--- |
| **Mede** | Relação **LINEAR** | Relação **MONOTÔNICA** |
| **Baseado em** | Valores originais | **Postos (ranks)** dos valores |
| **Sensível a outliers?** | **Sim** | **Menos** (mais robusto) |
| **Detecta relação não linear?** | Não (só linear) | Sim, **se for monotônica** |
| **Tipo de dado** | Numéricas contínuas | Numéricas **ou ordinais** |

&emsp;**Exemplo do slide:** tempo no site × gasto mensal → **Pearson r = 0,72** e **Spearman ρ = 0,69** → relação **positiva forte** (mais tempo no site, mais gasto).

## 3.1. Como interpretar

```text
+1  → relação positiva perfeita (uma sobe, a outra sobe)
 0  → ausência de relação (linear, no caso do Pearson)
−1  → relação negativa perfeita (uma sobe, a outra desce)
```

> **➕ Complemento — regra prática (convenções variam)**
>
> | \|valor\| | Interpretação usual |
> | :--- | :--- |
> | 0,00 – 0,30 | Fraca |
> | 0,30 – 0,70 | Moderada |
> | 0,70 – 1,00 | Forte |
>
> O **sinal** dá a **direção**; o **módulo** dá a **força**.

## 3.2. Quando usar qual?

- **Pearson:** relação aproximadamente **linear**, dados **sem outliers fortes**, variáveis numéricas contínuas (idealmente sem grande assimetria).
- **Spearman:** presença de **outliers**, relação **monotônica mas curva** (ex.: sempre cresce, só que cada vez mais devagar), dados **ordinais** (ex.: nota de 1 a 5) ou distribuição bem assimétrica.
- **Dica de EDA:** calcule **os dois**. Se Pearson ≪ Spearman, provavelmente há **não linearidade** ou **outliers** distorcendo o Pearson.

```python
df["tempo_site"].corr(df["gasto"], method="pearson")
df["tempo_site"].corr(df["gasto"], method="spearman")
```

> **➕ Complemento — armadilhas clássicas**
>
> - **Correlação ≠ causalidade.** Alta correlação não prova que X causa Y (pode haver uma **terceira variável** ou coincidência).
> - **Pearson ≈ 0 não significa independência:** pode haver relação **não linear** forte (ex.: formato de "U").
> - **Sempre olhe o scatter plot.** Conjuntos de dados bem diferentes podem ter o mesmo coeficiente (**Quarteto de Anscombe**) — por isso o slide diz que o scatter **complementa** os coeficientes.
> - **Outliers** podem inflar ou derrubar o Pearson.
> - Existe também o **Kendall (τ)**, parecido com o Spearman (baseado em postos), mais usado em amostras pequenas — menos provável de cair.
> - Para **duas variáveis categóricas**, Pearson/Spearman não se aplicam (usa-se, por exemplo, qui-quadrado/Cramér's V).

---

# 4. Variáveis e técnicas para categóricas

## 4.1. Tipos de variáveis

&emsp;**Variável** é qualquer atributo mensurável de um dataset. **Feature** é a variável **efetivamente usada como input do modelo**. *Toda feature é uma variável, mas nem toda variável vira feature* (ex.: um **ID de linha** não deve virar feature).

| Tipo | Subtipo | Definição | Exemplos |
| :--- | :--- | :--- | :--- |
| **Numérica** | **Contínua** | Infinitos valores dentro de um intervalo | altura, peso, temperatura, valor monetário |
| **Numérica** | **Discreta** | Valores contáveis, geralmente inteiros | nº de filhos, nº de compras, cliques |
| **Categórica** | **Nominal** | Categorias **sem ordem** | cor, cidade, código |
| **Categórica** | **Ordinal** | Categorias **com ordem** | nível baixo/médio/alto |

&emsp;Quanto à origem, uma feature pode ser:

- **Independente (raw):** vem direto da fonte, sem transformação (ex.: data de nascimento, valor bruto da compra).
- **Derivada (engineered):** criada a partir de outras via cálculo/transformação (ex.: **idade** a partir da data de nascimento; **ticket médio** = valor total ÷ nº de compras).

## 4.2. As três técnicas dos slides

| Técnica | Como funciona | **Quando usar** |
| :--- | :--- | :--- |
| **One-Hot Encoding** | Cria **uma coluna binária (0/1) por categoria** | Variável **nominal** com **baixa cardinalidade** (poucas categorias distintas) |
| **Label / Ordinal Encoding** | Atribui **números às categorias respeitando uma ordem** | Variável **ordinal** (existe hierarquia: baixo < médio < alto) |
| **Frequency / Count Encoding** | Substitui a categoria pela **frequência** com que ocorre | **Muitas categorias** (alta cardinalidade); simples e eficaz |

&emsp;**Exemplos práticos:**

```text
One-Hot  (cor: vermelho/azul/verde)
  cor_vermelho  cor_azul  cor_verde
        1          0          0

Ordinal  (nível: baixo/médio/alto)
  baixo → 0,  médio → 1,  alto → 2

Frequency  (cidade)
  São Paulo (aparece 5000x) → 5000
  Cuiabá    (aparece 12x)   → 12
```

## 4.3. Como decidir (regra de bolso para a prova)

```text
A categoria tem ORDEM natural?
├── SIM → Ordinal / Label Encoding
└── NÃO (nominal)
     ├── Poucas categorias  → One-Hot Encoding
     └── Muitas categorias  → Frequency/Count Encoding
                              (ou target encoding, ver abaixo)
```

> **➕ Complemento — por que a escolha importa**
>
> - **Label encoding em variável nominal é um erro comum:** transformar cidades em 0, 1, 2… cria uma **ordem falsa** (o modelo "entende" que 2 > 1). Isso prejudica principalmente **modelos lineares, redes neurais e baseados em distância**. Modelos de **árvore** toleram melhor.
> - **One-Hot com alta cardinalidade** (ex.: 5.000 cidades) gera **5.000 colunas** → dados **esparsos**, mais memória e risco de **overfitting** ("maldição da dimensionalidade").
> - **Armadilha da variável dummy:** em modelos lineares, k categorias geram k colunas **perfeitamente colineares**; usa-se `drop_first=True` (ou `drop="first"`) para ficar com k−1.
> - **Frequency encoding — limitação:** duas categorias diferentes com a **mesma frequência** ficam com o **mesmo valor** (colisão).
> - **Target Encoding** (não está nos slides): substitui a categoria pela **média do alvo** naquela categoria. Poderoso para alta cardinalidade, mas **alto risco de data leakage** — deve ser calculado **só com o treino** (e com validação cruzada/suavização).
> - **Categorias novas na produção:** use `handle_unknown="ignore"` no `OneHotEncoder` para não quebrar o modelo.
> - O encoder, como o scaler, deve ser **ajustado (`fit`) apenas no treino**.

```python
from sklearn.preprocessing import OneHotEncoder, OrdinalEncoder

# One-hot (nominal, poucas categorias)
ohe = OneHotEncoder(handle_unknown="ignore", sparse_output=False)

# Ordinal (ordem definida explicitamente!)
oe = OrdinalEncoder(categories=[["baixo", "médio", "alto"]])

# Frequency encoding (manual com pandas)
freq = df["cidade"].value_counts()
df["cidade_freq"] = df["cidade"].map(freq)
```

## 4.4. Texto e embeddings (também no Encontro 02)

&emsp;Para **texto**, as categorias viram vetores numéricos por **embeddings**. Os slides citam três famílias:

| Família | Ideia | Exemplos |
| :--- | :--- | :--- |
| **Baseados em frequência** | Importância da palavra relacionada à frequência com que aparece | **Bag of Words**, **TF-IDF** (score maior para palavras repetidas em um texto e menor para palavras comuns a todos os documentos) |
| **Baseados em predição** | Capturam **relações semânticas** entre palavras | **Word2Vec**, **GloVe** |
| **Baseados em contexto** | Usam o **contexto** do documento (mesma palavra, significados diferentes) | Ex.: "banco" (instituição financeira) × "banco" (de sentar) |

&emsp;**Ideia central do slide de embeddings:** um embedding é uma **lista de números (vetor)** que representa o significado de algo. **"Mais perto = mais parecido"**: palavras/itens semelhantes (gato e cachorro) ficam próximos no espaço; itens diferentes (carro, pizza) ficam longe. Usos: **busca, recomendações, chatbots e correspondência de imagens**.

---

# 5. Scaling

&emsp;**Definição:** ajustar a **escala/faixa de valores das features numéricas** para um intervalo **comparável**, **sem distorcer as diferenças relativas** entre elas.

## 5.1. Quem precisa e quem não precisa

| Precisa de scaling (**sensíveis à escala**) | **Não** precisa (**invariantes à escala**) |
| :--- | :--- |
| Algoritmos **baseados em distância**: **KNN, K-means, SVM** | Modelos **baseados em árvores**: **Decision Tree, Random Forest, Gradient Boosting / XGBoost** |
| **Gradient descent**: **redes neurais**, regressão linear/logística (com regularização) | Motivo: trabalham com **splits (cortes)**, não com distâncias |
| **PCA** | |

&emsp;**Exemplo do slide:** num dataset com **idade (0–100)** e **renda (0–50.000)**, sem scaling a **renda domina o cálculo de distância** em um KNN, **mascarando o efeito da idade**.

## 5.2. Principais técnicas

> **➕ Complemento:** os slides explicam o **conceito**, mas não detalham as técnicas. Elas costumam ser cobradas.

| Técnica | Fórmula | Resultado | Quando usar |
| :--- | :--- | :--- | :--- |
| **Padronização** (`StandardScaler`, z-score) | `z = (x − média) / desvio` | Média 0 e desvio 1 (sem limite fixo) | **Padrão na maioria dos casos**; dados ~normais; SVM, regressão, redes neurais, PCA |
| **Normalização Min-Max** (`MinMaxScaler`) | `x' = (x − min) / (max − min)` | Valores entre **0 e 1** | Quando se quer **faixa limitada** (ex.: redes neurais, imagens) e **sem outliers fortes** |
| **Robust Scaler** (`RobustScaler`) | `x' = (x − mediana) / IQR` | Centrado na mediana | Dados **com outliers** (usa mediana e IQR, que são robustos) |
| **Transformação log** (`np.log1p`) | `x' = log(1 + x)` | Reduz assimetria | Variáveis **muito assimétricas à direita** (renda, preço) — **não é scaling propriamente dito**, mas costuma ser combinada |

> **Atenção à terminologia:** "normalizar" às vezes significa **Min-Max** e às vezes é usado de forma genérica para qualquer scaling. Na dúvida, descreva a fórmula.

## 5.3. Boas práticas (liga com o Encontro 03: data leakage)

&emsp;O Encontro 03 lista como **erro clássico de vazamento** *"normalizar ou imputar antes do split"*: a média e o desvio do `StandardScaler` carregam informação do teste.

```text
✅ Certo:
  1. Separar treino e teste
  2. scaler.fit(X_treino)          ← aprende média/desvio SÓ do treino
  3. X_treino = scaler.transform(X_treino)
  4. X_teste  = scaler.transform(X_teste)   ← aplica os valores do treino

❌ Errado:
  scaler.fit_transform(X_inteiro) ANTES de dividir em treino/teste
```

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("clf", KNeighborsClassifier()),
])
pipe.fit(X_train, y_train)     # o scaler só vê o treino
```

> **➕ Complemento**
>
> - Dentro da **validação cruzada**, o scaler deve estar **dentro do `Pipeline`** para ser reajustado em cada fold (como mostra o slide do SMOTE).
> - **Não** é necessário escalar colunas **one-hot** (já são 0/1).
> - Normalmente **não** se escala a variável **alvo** (em classificação nunca; em regressão é opcional).
> - Árvores **não precisam** de scaling, mas escalar **não faz mal** a elas.
> - Scaling **não resolve outliers**: o Min-Max, por exemplo, é bastante afetado por eles (use Robust).

---

# 6. Demais conteúdos dos slides (visão rápida)

## 6.1. Encontro 01 — Contexto da área de dados

- **Papéis:** Data Engineer (pipelines e infraestrutura), Analytics Engineer (modelagem para consumo analítico, dbt), Data Analyst (SQL e dashboards), Data Scientist (estatística e experimentação), ML Engineer (modelos em produção), AI Engineer (LLMs e IA generativa), Data Architect (arquitetura e padrões), Data Product Manager (roadmap).
- **Data Lake:** estruturados, semiestruturados e não estruturados; grandes volumes; custo menor (S3, GCS).
- **Data Warehouse:** dados estruturados; apoia análises de negócio; custo maior (Redshift, BigQuery, Snowflake).
- **Data Lakehouse:** une flexibilidade do Lake com governança, SQL e performance do Warehouse (Databricks, Microsoft Fabric).
- **Arquitetura Medalhão:** organização do dado em camadas progressivas de qualidade (**Bronze → Silver → Gold**, termos usuais da arquitetura).
- **Boas práticas de engenharia de dados:** arquitetura e organização, qualidade, schema e contrato de dados, **idempotência**, orquestração, observabilidade, segurança e governança, performance e custos.
- **Pipeline eficiente = processa só o necessário:** **particionamento** (por data, região), **processamento incremental** (só o novo/alterado) e **formato/consulta** (Parquet colunar + `SELECT` específico). Ex.: `SELECT *` lê tudo; `WHERE data_venda >= '2026-01-01'` lê só a partição necessária.
- **Três pilares de habilidades:** Estatística & Matemática, Programação, Visão de negócio.

## 6.2. Boas práticas com Pandas

```python
# ❌ Evite: loops linha a linha
for i in range(len(df)):
    df.loc[i, 'total'] = df.loc[i, 'preco'] * df.loc[i, 'quantidade']

# ✅ Prefira operações vetorizadas
df['total'] = df['preco'] * df['quantidade']
```

```python
# Juntar vários arquivos: lista + concat uma única vez
dfs = []
for arquivo in arquivos:
    dfs.append(pd.read_csv(arquivo))
df_final = pd.concat(dfs, ignore_index=True)
```

&emsp;**Exercício do Encontro 01 (Sales Dataset):** 12 meses de vendas de uma loja de eletrônicos. Perguntas iniciais: *qual o melhor mês em faturamento? qual cidade concentra mais compras? quais produtos são vendidos juntos?* Entrega: notebook com análises e comentários.

## 6.3. Encontro 02 — Feature Store

&emsp;Os slides de Feature Store são basicamente figuras/links (documentação do **Feast**: `docs.feast.dev/getting-started/quickstart`).

> **➕ Complemento:** uma **Feature Store** centraliza e reutiliza features, garantindo que sejam calculadas **da mesma forma no treino e na inferência** (evita *training-serving skew*). Tem uma parte **offline** (treino/histórico) e uma **online** (baixa latência, produção).

## 6.4. Encontro 03 — Experimentação e Modelagem

- **Ciclo do experimento:** hipótese → desenho → execução (registrar tudo: params, métricas, artefatos) → análise (a diferença é maior que o ruído entre folds?) → decisão. O experimento precisa ser **reprodutível**.
- **Dados desbalanceados** (caso: fraude em cartão — 284.807 transações, 492 fraudes = 0,172%; desbalanceamento é a *natureza do problema*). Estratégias: **Undersampling** (remove da majoritária; perde informação), **Oversampling/SMOTE** (sintetiza exemplos da minoria; cuidado com ruído), **Pesos de classe** (`class_weight='balanced'`, `scale_pos_weight`).
- **Pergunta do slide:** *SMOTE antes ou depois do split?* → **Depois**, e só no treino (dentro do pipeline/fold).
- **Data leakage — erros clássicos:** normalizar/imputar antes do split; reamostrar (SMOTE) antes do split; selecionar features com o dataset inteiro; usar feature que só existe depois do evento (ex.: `valor_estornado`); duplicatas ou mesmo cliente nos dois lados (use **GroupKFold**).
- **Cross-validation (K-Fold):** divide o treino em K folds, treina em K−1 e valida no restante, repete K vezes; **reporta média e desvio-padrão**. Variantes: **KFold** (classes equilibradas), **StratifiedKFold** (desbalanceado), **GroupKFold** (mesmo cliente/cartão), **TimeSeriesSplit** (dados temporais), **RepeatedStratifiedKFold** (base pequena/métrica ruidosa).
- **Hyperparameter tuning:** **GridSearchCV** (todas as combinações; exaustivo, mas custoso), **RandomizedSearchCV** (amostra combinações; cobre mais espaço com menos treinos), **Bayesiana/Optuna** (usa resultados anteriores para decidir a próxima tentativa). Bergstra & Bengio (2012): busca aleatória acha boas soluções com uma fração do orçamento do grid.
- **Também na agenda:** Feature importance (permutation e SHAP) e MLflow (demonstração + exercício).

## 6.5. Cronograma das aulas

| Data | Conteúdo |
| :--- | :--- |
| 06/08 | Contexto da área de dados + EDA |
| 11/08 | Feature Engineering + Feature Store |
| 13/08 | Modelagem e Métricas |
| 18/08 | Modelagem e Métricas (continuação) |
| 25/08 | Ponderada em sala |
| 05/10 | Revisão da prova + conceitos de MLOps |

---

# 7. Checklist de revisão para a prova

## EDA

- [ ] O que é EDA e em que etapa do ciclo de ML ela ocorre?
- [ ] Qual a diferença entre análise **univariada, bivariada e multivariada**? Quais gráficos de cada uma?
- [ ] Para que serve um **histograma**? Que informação a forma dele dá?
- [ ] Como ler um **boxplot** (Q1, mediana, Q3, IQR, outliers)?
- [ ] Quando usar **gráfico de barras** (e por que difere do histograma)?
- [ ] O que um **heatmap** de correlação mostra?
- [ ] Para que serve o **PCA**?
- [ ] O que `df.describe()` e `df.info()` entregam?

## Correlação

- [ ] O que **Pearson** mede? E **Spearman**?
- [ ] Qual é mais robusto a outliers? Por quê?
- [ ] Quando usar Spearman em vez de Pearson?
- [ ] Por que correlação não implica causalidade?
- [ ] Por que olhar o scatter plot além do coeficiente?

## Variáveis categóricas

- [ ] Qual a diferença entre variável **nominal** e **ordinal**?
- [ ] Quando usar **One-Hot**? **Ordinal/Label**? **Frequency/Count**?
- [ ] Por que **Label Encoding em variável nominal** é problemático?
- [ ] O que é **cardinalidade** e por que importa?

## Scaling

- [ ] O que é scaling e por que fazer?
- [ ] Quais algoritmos **precisam** e quais **não precisam**? Por quê?
- [ ] Diferença entre **padronização**, **Min-Max** e **Robust Scaler**?
- [ ] Por que ajustar o scaler **apenas no treino** (data leakage)?