# Engenharia de Machine Learning e MLOps --- Guia de Estudo

> Material baseado no arquivo **Engenharia de Machine Learning e MLOps:
> Da Teoria Rigorosa à Implementação de Produção com Containers e
> Docker**.
>
> O foco deste guia é entender **o problema, o conceito, por que usar
> cada tecnologia, seus benefícios e como tudo se conecta**. O código
> aparece apenas quando ajuda a compreender uma ideia.

------------------------------------------------------------------------

# 1. Machine Learning não termina no treinamento

É comum imaginar ML assim:

``` text
Dados → Treinamento → Modelo → Predição
```

Isso funciona para um notebook. Em produção, o ciclo é maior:

``` text
Dados
 ↓
Preparação
 ↓
Treinamento
 ↓
Validação
 ↓
Registro
 ↓
Deploy
 ↓
Inferência
 ↓
Monitoramento
 ↓
Detecção de problemas
 ↓
Retreinamento
 ↓
Novo modelo
```

A ideia central de **MLOps** é justamente transformar esse ciclo em um
processo confiável, rastreável e automatizado.

> **Treinar um modelo é uma etapa. Manter esse modelo funcionando
> corretamente em produção é um problema de engenharia.**

------------------------------------------------------------------------

# 2. Por que existe o problema "funciona na minha máquina"?

Imagine um modelo treinado com:

-   Python 3.11;
-   determinada versão do scikit-learn;
-   bibliotecas específicas;
-   determinado pipeline de pré-processamento;
-   arquivo de modelo treinado;
-   configurações locais.

Ao enviar apenas o código e o `.pkl` para outro servidor, podem existir:

-   outra versão do Python;
-   bibliotecas incompatíveis;
-   bibliotecas do sistema ausentes;
-   caminhos de arquivos diferentes;
-   configurações de GPU diferentes;
-   variáveis de ambiente diferentes.

Em Machine Learning isso é ainda mais crítico porque não basta
reproduzir o código. É necessário reproduzir o **ambiente, os dados, o
pré-processamento e o modelo**.

------------------------------------------------------------------------

# 3. Um sistema de ML é maior que o algoritmo

Podemos pensar em:

``` text
Sistema de ML
│
├── Código
├── Dados
├── Hiperparâmetros
├── Features
├── Pré-processamento
├── Modelo treinado
├── Infraestrutura
├── API
├── Monitoramento
└── Processo de atualização
```

O algoritmo matemático é apenas uma parte.

Por isso, um modelo pode apresentar excelentes resultados no laboratório
e ainda falhar quando integrado a um sistema real.

------------------------------------------------------------------------

# 4. Código, dados e hiperparâmetros

O material apresenta a ideia:

``` text
Sistema Inteligente = f(Código, Dados, Hiperparâmetros)
```

## Código

Define como os dados são tratados, quais transformações são feitas e
como o modelo é utilizado.

## Dados

Determinam aquilo que o modelo aprende. Alterar o conjunto de
treinamento pode produzir um modelo diferente mesmo com o mesmo código.

## Hiperparâmetros

São configurações do treinamento, como taxa de aprendizado, profundidade
de árvores, regularização e número de estimadores.

### Por que isso importa?

Para reproduzir um modelo precisamos saber **o que foi usado para
produzi-lo**, não apenas possuir o arquivo final.

------------------------------------------------------------------------

# 5. Bare-metal, máquinas virtuais e containers

## Bare-metal

A aplicação executa diretamente no sistema operacional da máquina
física:

``` text
Aplicação
 ↓
Sistema Operacional
 ↓
Hardware
```

### Benefício

Pouco overhead.

### Problema

As aplicações compartilham o mesmo ambiente. Projetos que precisam de
versões incompatíveis de Python, bibliotecas ou CUDA podem entrar em
conflito.

------------------------------------------------------------------------

## Máquina virtual

Uma VM virtualiza o hardware:

``` text
Hardware
 ↓
Hypervisor
 ↓
VM
 ↓
Sistema Operacional convidado
 ↓
Aplicação
```

Cada VM possui seu próprio sistema operacional.

### Benefícios

-   isolamento forte;
-   possibilidade de sistemas operacionais diferentes;
-   separação maior entre ambientes.

### Desvantagens

-   mais memória;
-   mais disco;
-   inicialização mais lenta;
-   maior overhead.

------------------------------------------------------------------------

## Container

Containers utilizam o kernel do host e isolam processos:

``` text
Sistema Operacional
 ↓
Kernel
 ↓
Docker
 ↓
Container
 ↓
Aplicação
```

> **VM virtualiza uma máquina; container isola processos dentro de uma
> máquina.**

------------------------------------------------------------------------

# 6. VM x Container

  Característica        VM                  Container
  --------------------- ------------------- --------------------
  Virtualização         Hardware            Ambiente/processos
  Sistema operacional   Cada VM possui um   Compartilhado
  Kernel                Próprio             Compartilhado
  Consumo               Maior               Menor
  Inicialização         Mais lenta          Mais rápida
  Isolamento            Mais forte          Mais leve

Containers são especialmente úteis quando queremos empacotar aplicações
e suas dependências de maneira reproduzível.

------------------------------------------------------------------------

# 7. A ideia da "cesta"

Imagine uma cesta de piquenique. Em vez de depender do que existe no
parque, você leva tudo o que precisa.

Para ML, essa cesta pode conter:

``` text
Python
+ bibliotecas
+ código
+ modelo treinado
+ pesos
+ pré-processamento
+ configurações
```

Assim, o ambiente de execução deixa de depender tanto da máquina onde o
software foi instalado.

------------------------------------------------------------------------

# 8. Namespaces

Namespaces controlam **o que um processo consegue enxergar**.

Um container pode possuir uma visão isolada de:

-   processos;
-   rede;
-   pontos de montagem;
-   usuários;
-   hostname.

### PID namespace

Isola a árvore de processos. O processo principal pode ser visto como
PID 1 dentro do container.

### Network namespace

Isola interfaces, IPs, portas e rotas.

### Mount namespace

Isola a visão dos pontos de montagem e do sistema de arquivos.

### User namespace

Permite separar identificadores de usuários entre container e host.

### Regra para lembrar

> **Namespaces = visão do sistema.**

------------------------------------------------------------------------

# 9. cgroups

Cgroups, ou Control Groups, controlam **quanto de recurso um processo
pode consumir**.

Podemos estabelecer limites de:

-   CPU;
-   memória;
-   número de processos;
-   I/O.

Por exemplo:

``` text
Container
├── CPU: limite definido
└── Memória: limite definido
```

### Por que usar?

Evita que um serviço consuma recursos indefinidamente e prejudique os
demais.

Isso melhora:

-   isolamento;
-   previsibilidade;
-   estabilidade;
-   utilização da infraestrutura.

------------------------------------------------------------------------

# 10. CPU x memória

Os limites possuem comportamentos diferentes.

### CPU

Um processo pode sofrer **throttling** e ficar mais lento.

### Memória

Quando o limite é ultrapassado e não há memória suficiente para
recuperar, pode ocorrer **OOM kill**.

Isso pode aparecer associado ao:

``` text
exit code 137
```

Portanto:

``` text
CPU → pode gerar lentidão

Memória → pode provocar encerramento
```

------------------------------------------------------------------------

# 11. Imagem, container e registry

## Imagem

É o modelo utilizado para criar containers.

``` text
Imagem → modelo
```

## Container

É uma execução de uma imagem.

``` text
Container → instância em execução
```

Uma analogia:

``` text
Imagem = classe
Container = objeto
```

## Registry

É um repositório de imagens.

O fluxo pode ser:

``` text
Dockerfile
 ↓
Build
 ↓
Imagem
 ↓
Registry
 ↓
Pull
 ↓
Container
```

------------------------------------------------------------------------

# 12. Camadas de uma imagem

Imagens Docker são formadas por camadas:

``` text
Camada base
 ↓
Dependências
 ↓
Código
 ↓
Configuração
```

### Por que isso é útil?

Camadas que não mudaram podem ser reutilizadas.

Isso:

-   acelera builds;
-   reduz downloads;
-   economiza espaço;
-   evita trabalho repetido.

------------------------------------------------------------------------

# 13. Dockerfile

O Dockerfile descreve **como construir uma imagem**.

É uma receita automatizada do ambiente.

Principais instruções:

## `FROM`

Define a imagem base.

``` dockerfile
FROM python:3.11-slim
```

Evita começar o ambiente do zero.

## `WORKDIR`

Define o diretório de trabalho.

## `COPY`

Copia arquivos para a imagem.

## `RUN`

Executa comandos durante o build.

``` dockerfile
RUN pip install -r requirements.txt
```

## `CMD`

Define o processo principal executado quando o container inicia.

``` dockerfile
CMD ["python", "app.py"]
```

## `EXPOSE`

Documenta a porta usada pela aplicação.

> `EXPOSE` não publica a porta no computador.

------------------------------------------------------------------------

# 14. `RUN` x `CMD`

Essa diferença precisa estar muito clara:

``` text
RUN
→ acontece durante o build

CMD
→ acontece quando o container inicia
```

Confundir os dois é um erro comum.

------------------------------------------------------------------------

# 15. Por que a ordem do Dockerfile importa?

Uma organização comum é:

``` dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

As dependências geralmente mudam menos que o código.

Se apenas o código mudar:

``` text
requirements.txt não mudou
        ↓
camada de instalação pode ser reutilizada
        ↓
somente o código é reconstruído
```

Isso aproveita melhor o cache do Docker.

------------------------------------------------------------------------

# 16. Multi-stage build

Multi-stage build usa diferentes etapas para construir a aplicação e
gerar uma imagem final menor.

A lógica é:

``` text
Etapa de build
 ↓
Compilar / instalar / preparar
 ↓
Artefatos necessários
 ↓
Imagem final
```

Ferramentas usadas somente para construir a aplicação não precisam
permanecer na imagem final.

### Benefícios

-   imagem menor;
-   download mais rápido;
-   menor armazenamento;
-   menor superfície de ataque;
-   deploy mais rápido.

------------------------------------------------------------------------

# 17. Usuário não-root

Aplicações não precisam necessariamente executar como `root`.

Utilizar um usuário com menos privilégios reduz o impacto potencial de
uma vulnerabilidade.

Esse princípio é chamado de **least privilege**.

``` text
Aplicação
 ↓
Privilégios mínimos
```

é preferível a:

``` text
Aplicação
 ↓
root
```

quando o root não é necessário.

------------------------------------------------------------------------

# 18. Docker aplicado a Machine Learning

Uma aplicação de ML pode precisar carregar:

``` text
Python
+ bibliotecas
+ código
+ modelo
+ pré-processamento
+ configurações
```

Isso é importante porque o modelo precisa receber os dados na mesma
representação esperada durante o treinamento.

Exemplo:

``` text
JSON
 ↓
Validação
 ↓
Normalização
 ↓
Feature engineering
 ↓
Modelo
 ↓
Predição
```

Se o pré-processamento for diferente entre treino e produção, o modelo
pode receber dados em um formato diferente daquele que aprendeu.

------------------------------------------------------------------------

# 19. Model Serving

Model serving é disponibilizar o modelo para receber entradas e retornar
previsões.

``` text
Cliente
 ↓
POST /predict
 ↓
API
 ↓
Validação
 ↓
Pré-processamento
 ↓
Modelo
 ↓
Predição
 ↓
JSON
```

Isso transforma o modelo em um serviço que pode ser consumido por outros
sistemas.

------------------------------------------------------------------------

# 20. FastAPI

FastAPI pode ser utilizada para criar a API de inferência.

Ela funciona como uma camada entre:

``` text
Sistema consumidor
        ↓
      API
        ↓
     Modelo
```

O consumidor não precisa conhecer os detalhes matemáticos do modelo.

------------------------------------------------------------------------

# 21. Pydantic e contratos

Uma API precisa verificar se os dados recebidos são válidos.

Por exemplo:

``` text
customer_id
age
annual_income
credit_score
loan_amount
```

Pydantic permite definir e validar esse contrato.

### Benefícios

-   tipos explícitos;
-   validação automática;
-   erros mais claros;
-   contrato de entrada bem definido.

Isso evita que dados inválidos cheguem ao modelo.

------------------------------------------------------------------------

# 22. Por que validar antes da inferência?

Sem validação:

``` text
Entrada inválida
 ↓
Modelo
 ↓
Erro inesperado
```

Com validação:

``` text
Entrada
 ↓
Validação
 ├── inválida → erro controlado
 └── válida → modelo
```

Isso melhora a confiabilidade da API.

------------------------------------------------------------------------

# 23. O problema industrial de ML

No laboratório:

``` text
Notebook
 ↓
Treina
 ↓
Avalia
 ↓
Modelo
```

Na empresa:

``` text
Dados
 ↓
Ingestão
 ↓
Transformação
 ↓
Features
 ↓
Treinamento
 ↓
Validação
 ↓
Registro
 ↓
Deploy
 ↓
Inferência
 ↓
Logs
 ↓
Monitoramento
 ↓
Drift
 ↓
Retreinamento
```

MLOps existe para organizar esse segundo cenário.

------------------------------------------------------------------------

# 24. Por que ML é diferente de software tradicional?

Em software tradicional, frequentemente pensamos:

``` text
Código + Entrada → Saída
```

Em ML:

``` text
Código
+
Dados
+
Configurações
 ↓
Modelo
 ↓
Comportamento
```

Mesmo sem alterar o código, mudanças nos dados podem alterar o
comportamento do sistema.

------------------------------------------------------------------------

# 25. Dívida técnica em Machine Learning

O material utiliza o trabalho de Sculley et al. para mostrar que o
algoritmo é apenas uma pequena parte de um sistema de ML.

O sistema pode envolver:

-   coleta de dados;
-   pipelines;
-   features;
-   infraestrutura;
-   integração;
-   monitoramento;
-   configuração;
-   segurança.

Portanto:

> **Um modelo funcionando em um notebook não significa que temos um
> sistema de ML pronto para produção.**

------------------------------------------------------------------------

# 26. Boundary Erosion

**Boundary Erosion** é a erosão dos limites entre responsabilidades.

Um exemplo ruim seria um único script responsável por:

``` text
ler banco
 ↓
tratar dados
 ↓
treinar
 ↓
gerar gráficos
 ↓
salvar modelo
 ↓
iniciar API
```

### Problema

Fica difícil:

-   testar;
-   modificar;
-   reutilizar;
-   entender.

Separar responsabilidades torna o sistema mais sustentável.

------------------------------------------------------------------------

# 27. CACE --- Changing Anything Changes Everything

Em ML, uma pequena alteração pode gerar efeitos em várias partes do
sistema.

Exemplo:

``` text
Nova feature
 ↓
Novo espaço de entrada
 ↓
Novo treinamento
 ↓
Novos pesos
 ↓
Novas métricas
 ↓
Novo comportamento
```

Por isso mudanças precisam ser rastreáveis.

------------------------------------------------------------------------

# 28. Pipeline Jungles

Uma pipeline jungle acontece quando o fluxo de ML cresce sem
organização.

Exemplo:

``` text
script.py
cron
SQL manual
notebook_final.ipynb
modelo_v2.pkl
dados_final.csv
dados_final_v2.csv
```

Depois fica difícil responder:

> Qual processo realmente gerou o modelo em produção?

MLOps tenta substituir esse conjunto de scripts por pipelines
rastreáveis e automatizados.

------------------------------------------------------------------------

# 29. Glue Code

Glue code é código improvisado para conectar sistemas.

Por exemplo:

``` text
Banco
 ↓
Script
 ↓
CSV
 ↓
Python
 ↓
JSON
 ↓
API
```

Quanto mais conversões improvisadas existirem, maior a fragilidade do
sistema.

Uma arquitetura bem definida reduz esse acoplamento.

------------------------------------------------------------------------

# 30. Data Testing Debt

Testar apenas o código não basta.

Podemos ter:

``` text
Código → 100% dos testes passando
```

e:

``` text
Dataset → coluna inteira nula
```

Por isso também precisamos testar:

-   presença de colunas;
-   tipos;
-   valores ausentes;
-   intervalos;
-   distribuição;
-   qualidade dos dados.

------------------------------------------------------------------------

# 31. O que é MLOps?

MLOps significa **Machine Learning Operations**.

Não é uma única ferramenta.

É a combinação de:

-   práticas;
-   processos;
-   automação;
-   arquitetura;
-   ferramentas.

O objetivo é permitir que modelos sejam:

-   entregues;
-   reproduzidos;
-   monitorados;
-   auditados;
-   atualizados;
-   mantidos em produção.

> **MLOps aplica princípios de engenharia e operações ao ciclo de vida
> de Machine Learning.**

------------------------------------------------------------------------

# 32. MLOps integra três áreas

``` text
Engenharia de Dados
        +
Ciência de Dados
        +
DevOps
        ↓
      MLOps
```

### Engenharia de Dados

Dados, ingestão, transformação, armazenamento e qualidade.

### Ciência de Dados

Features, treinamento, avaliação e modelos.

### DevOps

Automação, infraestrutura, deploy e observabilidade.

------------------------------------------------------------------------

# 33. CD4ML

**CD4ML --- Continuous Delivery for Machine Learning** --- adapta a
ideia de entrega contínua ao contexto de ML.

Três elementos precisam evoluir juntos:

``` text
Código
Dados
Modelos
```

## Código

Pipelines, APIs e regras versionadas.

## Dados

Dados brutos e transformados precisam ser rastreáveis e, quando
necessário, versionados.

## Modelos

Artefatos precisam ser catalogados com informações como métricas,
hiperparâmetros e linhagem.

------------------------------------------------------------------------

# 34. Git x DVC

Uma divisão útil:

``` text
Git
 ↓
Código + configuração

DVC
 ↓
Dados + versões dos dados
```

Git é excelente para código. DVC pode complementar o processo quando
datasets e artefatos de dados são grandes ou precisam de versionamento
específico.

### Benefício

Permite reconstruir melhor o contexto de um experimento.

------------------------------------------------------------------------

# 35. MLflow

MLflow pode ser utilizado para **experiment tracking**.

Podemos ter:

``` text
Run 1
Run 2
Run 3
Run 4
```

Cada execução pode registrar:

-   hiperparâmetros;
-   métricas;
-   artefatos;
-   modelo.

Isso permite responder:

> Qual configuração gerou este resultado?

------------------------------------------------------------------------

# 36. Model Registry

O Model Registry organiza versões e estágios dos modelos.

Conceitualmente:

``` text
Treinado
 ↓
Validado
 ↓
Candidato
 ↓
Produção
```

### Por que usar?

Evita a situação:

``` text
modelo_final.pkl
modelo_final_v2.pkl
modelo_final_novo.pkl
modelo_final_real.pkl
```

O registro cria uma referência mais organizada para os modelos.

------------------------------------------------------------------------

# 37. Níveis de maturidade em MLOps

## Nível 0 --- Processo manual

Características:

-   notebooks;
-   scripts avulsos;
-   treinamento manual;
-   deploy manual;
-   arquivos compartilhados.

Problema:

``` text
Intervenção humana
 ↓
Erros
 ↓
Baixa reprodutibilidade
```

------------------------------------------------------------------------

## Nível 1 --- Continuous Training

Começamos a automatizar o treinamento:

``` text
Novos dados
 ↓
Pipeline
 ↓
Validação
 ↓
Treinamento
 ↓
Avaliação
```

------------------------------------------------------------------------

## Nível 2 --- CI/CD para ML

Existe uma automação mais completa:

``` text
Código
Dados
Modelo
 ↓
Testes
 ↓
Validação
 ↓
Build
 ↓
Deploy
 ↓
Monitoramento
```

Também podem existir mecanismos como Champion/Challenger, deploy
controlado e retreinamento orientado por eventos.

------------------------------------------------------------------------

# 38. Champion x Challenger

``` text
Champion
→ modelo atualmente utilizado

Challenger
→ novo modelo candidato
```

O Challenger é avaliado antes de substituir o Champion.

Isso reduz o risco de colocar uma nova versão em produção sem validação
suficiente.

------------------------------------------------------------------------

# 39. Shadow Deployment

No Shadow Mode, o novo modelo recebe dados reais, mas suas respostas não
são utilizadas para a decisão final.

``` text
Tráfego real
   ├──► Champion → decisão real
   └──► Challenger → avaliação
```

### Benefício

Permite comparar o novo modelo com dados reais sem colocá-lo
imediatamente no controle da decisão.

------------------------------------------------------------------------

# 40. Canary Deployment

No Canary, uma parcela do tráfego é direcionada à nova versão.

Exemplo:

``` text
95% → versão atual
5%  → versão nova
```

Se o comportamento for adequado, a nova versão pode receber mais
tráfego.

### Benefício

Reduz o impacto potencial de um problema.

------------------------------------------------------------------------

# 41. Blue-Green Deployment

Mantemos dois ambientes:

``` text
Blue  → versão atual
Green → versão nova
```

Depois de validar o novo ambiente, o tráfego pode ser direcionado para
ele.

### Benefício

Facilita trocas controladas e rollback.

------------------------------------------------------------------------

# 42. Observabilidade

Observabilidade é a capacidade de entender o comportamento do sistema a
partir dos sinais que ele produz.

Em ML podemos observar:

-   latência;
-   erros;
-   número de requisições;
-   distribuição das features;
-   qualidade dos dados;
-   métricas do modelo;
-   drift.

Uma API pode estar tecnicamente saudável e ainda assim produzir
previsões ruins.

------------------------------------------------------------------------

# 43. Healthcheck x monitoramento do modelo

São coisas diferentes.

### Healthcheck

Pergunta:

> O serviço está funcionando?

### Monitoramento de ML

Pergunta:

> O modelo continua se comportando de forma adequada?

É possível ter:

``` text
Healthcheck → OK
```

e simultaneamente:

``` text
Drift → alto
```

Por isso os dois são necessários.

------------------------------------------------------------------------

# 44. Data Drift

Data Drift ocorre quando a distribuição dos dados observados em produção
muda em relação à referência.

Por exemplo:

``` text
Treinamento
idade média = 35
```

Depois:

``` text
Produção
idade média = 55
```

A API pode continuar funcionando normalmente, mas o contexto dos dados
mudou.

------------------------------------------------------------------------

# 45. Concept Drift

Data Drift e Concept Drift não são a mesma coisa.

### Data Drift

Mudança na distribuição das entradas:

``` text
P(X) muda
```

### Concept Drift

Mudança na relação entre entrada e resultado:

``` text
P(Y|X) muda
```

Por exemplo, o mesmo perfil de cliente pode passar a ter uma
probabilidade diferente de inadimplência devido a uma mudança econômica.

------------------------------------------------------------------------

# 46. Por que drift é perigoso?

Um sistema pode estar:

``` text
HTTP 200
CPU normal
Memória normal
Container saudável
```

e mesmo assim:

``` text
qualidade das previsões ↓
```

Esse é o motivo de monitoramento de infraestrutura não ser suficiente
para ML.

------------------------------------------------------------------------

# 47. Teste Kolmogorov-Smirnov

O teste **KS** pode comparar duas distribuições.

A estatística apresentada é:

``` text
D = sup |F_ref(x) - F_prod(x)|
```

De maneira intuitiva:

> Quanto maior a diferença entre as distribuições acumuladas, maior a
> evidência de que elas não são iguais.

É especialmente útil para comparar distribuições de variáveis numéricas.

------------------------------------------------------------------------

# 48. PSI --- Population Stability Index

O PSI é muito utilizado em cenários de risco e crédito.

Uma forma de representá-lo é:

``` text
PSI = Σ (P - Q) × ln(P / Q)
```

Ele compara a proporção observada em determinadas faixas com a proporção
de referência.

O material apresenta a seguinte régua:

  PSI             Interpretação
  --------------- -----------------------
  `< 0,10`        Estável
  `0,10 – 0,25`   Mudança moderada
  `>= 0,25`       Mudança significativa

Esses valores são referências e devem ser interpretados de acordo com o
contexto do modelo.

------------------------------------------------------------------------

# 49. KL Divergence

A **Divergência de Kullback-Leibler** mede a diferença entre uma
distribuição e uma referência.

A fórmula apresentada no material é:

``` text
D_KL(P || Q) = ∫ p(x) log(p(x) / q(x)) dx
```

O importante é entender o propósito:

> Medir quão diferente uma distribuição está em relação a outra segundo
> essa medida.

------------------------------------------------------------------------

# 50. KS x PSI x KL

  -----------------------------------------------------------------------
  Técnica                 Ideia                   Uso
  ----------------------- ----------------------- -----------------------
  KS                      Distância entre         Comparação estatística
                          distribuições           
                          acumuladas              

  PSI                     Diferença entre         Estabilidade
                          proporções em bins      populacional

  KL                      Divergência entre       Diferença informacional
                          distribuições           
  -----------------------------------------------------------------------

O ponto principal não é decorar fórmulas isoladas.

É saber que são ferramentas para investigar **mudanças na distribuição
dos dados**.

------------------------------------------------------------------------

# 51. Drift não significa automaticamente modelo ruim

Se houve drift:

``` text
Os dados mudaram.
```

Isso não significa automaticamente:

``` text
O modelo está inútil.
```

É necessário investigar:

-   magnitude da mudança;
-   quais features mudaram;
-   impacto nas métricas;
-   contexto de negócio;
-   duração da mudança;
-   possibilidade de mudança temporária.

------------------------------------------------------------------------

# 52. CI/CD para Machine Learning

Em software:

``` text
Código
 ↓
Teste
 ↓
Build
 ↓
Deploy
```

Em ML:

``` text
Código
+
Dados
+
Modelo
 ↓
Testes
 ↓
Validação
 ↓
Build
 ↓
Deploy
```

Além dos testes tradicionais, podem existir critérios relacionados às
métricas do modelo.

------------------------------------------------------------------------

# 53. Quality Gates

Um **quality gate** é uma condição que precisa ser satisfeita para o
pipeline continuar.

Exemplo:

``` text
F1 >= 0,80
```

Se o modelo candidato não satisfizer a condição:

``` text
Validação
 ↓
Falha
 ↓
Deploy bloqueado
```

### Benefício

Evita que uma regressão conhecida avance automaticamente para produção.

------------------------------------------------------------------------

# 54. Docker + CI/CD

Docker padroniza o ambiente.

CI/CD automatiza o processo.

Juntos:

``` text
Código
 ↓
Testes
 ↓
Build da imagem
 ↓
Imagem versionada
 ↓
Deploy
```

Isso reduz configuração manual e aumenta a reprodutibilidade.

------------------------------------------------------------------------

# 55. Health Check

Um processo pode estar ativo e ainda não estar pronto.

Um endpoint como:

``` text
GET /health
```

pode indicar se o serviço está funcionando corretamente.

Isso pode ser usado pela infraestrutura para verificar saúde e tomar
ações automáticas.

------------------------------------------------------------------------

# 56. Contratos de entrada e saída

Uma API de ML precisa definir:

``` text
Entrada
 ↓
Processamento
 ↓
Saída
```

Exemplo:

``` json
{
  "customer_id": "123",
  "age": 30,
  "annual_income": 80000,
  "credit_score": 720
}
```

O contrato permite que o sistema consumidor saiba exatamente o que deve
enviar e o que receber.

------------------------------------------------------------------------

# 57. Diagnóstico de uma API de ML

Quando uma API não funciona, investigue por camadas:

``` text
1. Container está rodando?
        ↓
2. Processo iniciou?
        ↓
3. Porta está correta?
        ↓
4. API responde?
        ↓
5. Entrada é válida?
        ↓
6. Modelo foi carregado?
        ↓
7. Pré-processamento funciona?
        ↓
8. Predição funciona?
        ↓
9. Métricas continuam adequadas?
```

Isso é melhor do que alterar várias partes simultaneamente.

------------------------------------------------------------------------

# 58. Logs

Logs são uma das primeiras fontes de evidência.

Procure por:

-   erros de import;
-   arquivo não encontrado;
-   modelo não carregado;
-   erro de porta;
-   erro de validação;
-   exceções durante inferência.

No Docker:

``` bash
docker logs nome-do-container
```

------------------------------------------------------------------------

# 59. Feature Store

Uma **Feature Store** centraliza features utilizadas por modelos.

Pode existir:

``` text
Offline Store
 ↓
Treinamento

Online Store
 ↓
Inferência em tempo real
```

### Por que usar?

Para evitar que uma feature seja calculada de uma maneira durante o
treinamento e de outra durante a inferência.

Esse problema é conhecido como inconsistência entre treino e serving.

------------------------------------------------------------------------

# 60. Orquestração distribuída

Quando o treinamento cresce, uma única máquina pode não ser suficiente.

O material cita ferramentas como:

-   Ray;
-   Kubeflow Pipelines.

A ideia é:

``` text
Problema grande
 ↓
Distribuição
 ↓
Vários recursos
 ↓
Execução coordenada
```

Isso permite escalar treinamento e pipelines.

------------------------------------------------------------------------

# 61. Edge Computing e TinyML

Nem todo modelo precisa executar em um servidor.

Alguns cenários exigem execução em:

-   celulares;
-   IoT;
-   veículos;
-   equipamentos industriais.

Nesses casos, latência, memória e energia podem ser críticos.

Por isso existem técnicas específicas de otimização e inferência na
borda.

------------------------------------------------------------------------

# 62. ONNX

ONNX é um formato para representar modelos de Machine Learning de
maneira interoperável.

A ideia é facilitar a execução de modelos em ambientes diferentes
daqueles utilizados no treinamento.

O ONNX Runtime pode ser utilizado para inferência otimizada.

------------------------------------------------------------------------

# 63. Quantização

Quantização reduz a precisão numérica dos pesos.

Por exemplo:

``` text
FP32 → FP16
```

ou:

``` text
FP32 → INT8
```

Pode reduzir:

-   memória;
-   tamanho do modelo;
-   custo computacional.

Mas existe um trade-off:

``` text
menor precisão numérica
        ↓
possível perda de qualidade
```

Por isso a técnica precisa ser validada no modelo real.

------------------------------------------------------------------------

# 64. Governança

Quando um modelo influencia decisões reais, não basta saber sua
acurácia.

Também precisamos saber:

-   para que foi criado;
-   onde pode ser usado;
-   quais são suas limitações;
-   como foi avaliado;
-   quais dados utiliza;
-   quais grupos foram considerados.

Isso faz parte da governança de Machine Learning.

------------------------------------------------------------------------

# 65. Model Cards

Model Cards são documentos utilizados para registrar informações
importantes sobre modelos.

Podem incluir:

-   objetivo;
-   contexto;
-   limitações;
-   métricas;
-   dados;
-   possíveis vieses;
-   grupos avaliados.

### Benefícios

-   transparência;
-   documentação;
-   auditabilidade;
-   comunicação das limitações.

------------------------------------------------------------------------

# 66. Fairness e viés

Uma métrica média pode esconder diferenças entre grupos.

Dependendo do contexto, pode ser necessário avaliar:

-   desempenho por grupo;
-   diferenças de erro;
-   métricas de equidade;
-   impacto das decisões.

Ferramentas como Fairlearn e AI Fairness 360 são exemplos de recursos
para esse tipo de análise.

------------------------------------------------------------------------

# 67. Privacidade e LGPD

Sistemas de ML que usam dados pessoais também precisam considerar:

-   quais dados são coletados;
-   por que são usados;
-   quem possui acesso;
-   como são armazenados;
-   por quanto tempo;
-   como são protegidos.

MLOps não resolve sozinho essas questões, mas uma arquitetura
profissional precisa incluí-las.

------------------------------------------------------------------------

# 68. O ciclo de atualização de um modelo

Imagine:

``` text
Modelo 1.0
```

em produção.

Surge:

``` text
Modelo 2.0
```

Um processo maduro pode ser:

``` text
Modelo 2.0
 ↓
Testes
 ↓
Avaliação
 ↓
Registro
 ↓
Challenger
 ↓
Comparação
 ↓
Deploy controlado
 ↓
Monitoramento
```

Isso é muito mais seguro do que simplesmente substituir um arquivo
`.pkl`.

------------------------------------------------------------------------

# 69. Rastreabilidade

Rastreabilidade significa conseguir responder:

> De onde veio este modelo?

Podemos ter:

``` text
Modelo v2.1
 ↓
Run 184
 ↓
Dataset v7
 ↓
Commit abc123
 ↓
Hiperparâmetros X
 ↓
Métricas Y
```

### Benefícios

-   debugging;
-   auditoria;
-   rollback;
-   comparação;
-   manutenção.

------------------------------------------------------------------------

# 70. Reprodutibilidade

Idealmente:

``` text
Código X
+
Dados Y
+
Hiperparâmetros Z
+
Ambiente W
 ↓
Modelo M
```

Se esses elementos estiverem registrados, o resultado pode ser
reconstruído ou investigado.

------------------------------------------------------------------------

# 71. Rollback

Se uma versão nova apresentar problema:

``` text
v2.1
 ↓
problema
 ↓
rollback
 ↓
v2.0
```

Isso só é possível de maneira confiável se as versões anteriores
estiverem identificadas e disponíveis.

------------------------------------------------------------------------

# 72. Por que versões importam?

Evite depender apenas de:

``` text
latest
```

Quando controle de release é importante, versões explícitas ajudam:

``` text
model-api:1.0
model-api:1.1
model-api:2.0
```

Isso facilita:

-   identificação;
-   reprodução;
-   rollback;
-   auditoria.

------------------------------------------------------------------------

# 73. Como todas as ferramentas se conectam

  Problema       Ferramenta/conceito   O que resolve
  -------------- --------------------- ------------------------
  Código         Git                   Versionamento
  Dados          DVC                   Versionamento de dados
  Experimentos   MLflow                Rastreamento
  Modelos        Model Registry        Versões e estágios
  API            FastAPI               Serving
  Contratos      Pydantic              Validação
  Ambiente       Docker                Empacotamento
  Entrega        CI/CD                 Automação
  Saúde          Healthcheck           Verificação do serviço
  Drift          KS / PSI / KL         Mudanças estatísticas
  Features       Feature Store         Centralização
  Governança     Model Cards           Documentação

------------------------------------------------------------------------

# 74. O ciclo completo de MLOps

O modelo mental mais importante é:

``` text
             DADOS
               ↓
          Preparação
               ↓
          Treinamento
               ↓
           Validação
               ↓
         Model Registry
               ↓
             CI/CD
               ↓
            Docker
               ↓
            Deploy
               ↓
          API / Serving
               ↓
            Usuários
               ↓
        Observabilidade
               ↓
             Drift
               ↓
        Retreinamento
               ↓
          Novo modelo
               │
               └──────────────► novo ciclo
```

------------------------------------------------------------------------

# 75. Roadmap prático de estudo

O material propõe uma sequência de laboratórios.

## Lab 1 --- DVC + Git

Aprender:

``` text
Código → Git
Dados → DVC
```

Objetivo: reprodutibilidade.

## Lab 2 --- MLflow

Aprender:

``` text
Experiment Tracking
+
Model Registry
```

Objetivo: rastrear experimentos e modelos.

## Lab 3 --- CI/CD

Aprender:

``` text
Testes
+
Quality Gates
```

Objetivo: impedir que versões inadequadas avancem.

## Lab 4 --- Docker + FastAPI

Aprender:

``` text
Serving
+
Containerização
```

Objetivo: empacotar e disponibilizar o modelo.

## Lab 5 --- Drift + Observabilidade

Aprender:

``` text
Monitoramento
+
KS / PSI
+
Alertas
```

Objetivo: detectar mudanças nos dados.

------------------------------------------------------------------------

# 76. Ordem ideal para estudar

Não é necessário decorar todas as ferramentas ao mesmo tempo.

Uma ordem lógica é:

``` text
1. Entender ML em produção
        ↓
2. Entender containers
        ↓
3. Criar uma API
        ↓
4. Containerizar a API
        ↓
5. Versionar dados e modelos
        ↓
6. Automatizar testes
        ↓
7. Fazer deploy
        ↓
8. Monitorar
        ↓
9. Detectar drift
        ↓
10. Automatizar retreinamento
```

Cada etapa resolve um problema que aparece naturalmente depois da
anterior.

------------------------------------------------------------------------

# 77. Mapa mental

``` text
                    MACHINE LEARNING
                           │
                           ▼
                  Modelo em produção
                           │
            ┌──────────────┼──────────────┐
            │              │              │
          Dados          Código         Modelo
            │              │              │
           DVC            Git        MLflow/Registry
            │              │              │
            └──────────────┼──────────────┘
                           ↓
                         MLOps
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
     Serving             CI/CD          Observabilidade
        │                  │                  │
     FastAPI             Testes             Drift
        │                  │                  │
      Docker          Quality Gates      KS / PSI / KL
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ↓
                     Retreinamento
                           ↓
                       Challenger
                           ↓
                         Deploy
                           ↓
                      Novo ciclo
```

------------------------------------------------------------------------

# 78. Checklist para prova

## Fundamentos

-   [ ] Por que ML em produção é diferente de um notebook?
-   [ ] Por que existe o problema "funciona na minha máquina"?
-   [ ] Qual a relação entre código, dados e hiperparâmetros?
-   [ ] Qual a diferença entre bare-metal, VM e container?
-   [ ] Por que containers são mais leves?
-   [ ] O que são namespaces?
-   [ ] O que são cgroups?
-   [ ] Qual a diferença entre visão e controle de recursos?

## Docker

-   [ ] O que é uma imagem?
-   [ ] O que é um container?
-   [ ] O que é um registry?
-   [ ] Por que imagens possuem camadas?
-   [ ] Para que serve Dockerfile?
-   [ ] Qual a diferença entre `FROM`, `RUN`, `COPY` e `CMD`?
-   [ ] Por que a ordem do Dockerfile influencia o cache?
-   [ ] O que é multi-stage build?
-   [ ] Por que usar usuário não-root?
-   [ ] O que `EXPOSE` faz?

## Serving

-   [ ] O que é model serving?
-   [ ] Para que serve FastAPI?
-   [ ] Por que validar entradas?
-   [ ] Qual o papel do Pydantic?
-   [ ] Por que o pré-processamento precisa ser consistente?
-   [ ] O que é healthcheck?

## MLOps

-   [ ] O que é MLOps?
-   [ ] O que é CD4ML?
-   [ ] Por que Git é importante?
-   [ ] Por que DVC pode ser usado?
-   [ ] Para que serve MLflow?
-   [ ] O que é Model Registry?
-   [ ] Quais são os níveis de maturidade?
-   [ ] O que são Champion e Challenger?
-   [ ] O que é Shadow Deployment?
-   [ ] O que é Canary?
-   [ ] O que é Blue-Green?

## Observabilidade

-   [ ] O que é Data Drift?
-   [ ] O que é Concept Drift?
-   [ ] Qual a diferença?
-   [ ] O que KS mede?
-   [ ] O que é PSI?
-   [ ] O que é KL Divergence?
-   [ ] Por que uma API pode estar saudável e o modelo estar ruim?

## Produção

-   [ ] O que é dívida técnica em ML?
-   [ ] O que é Boundary Erosion?
-   [ ] O que são Pipeline Jungles?
-   [ ] O que é Glue Code?
-   [ ] O que é Data Testing Debt?
-   [ ] Por que versionar dados, código e modelos?
-   [ ] Por que rollback é importante?
-   [ ] O que são Model Cards?
-   [ ] Por que governança e fairness importam?

------------------------------------------------------------------------

# 79. Diferenças que você precisa saber

  -----------------------------------------------------------------------
  Conceito A              Conceito B              Diferença
  ----------------------- ----------------------- -----------------------
  VM                      Container               VM virtualiza uma
                                                  máquina; container
                                                  isola processos

  Namespace               cgroup                  Namespace controla
                                                  visão; cgroup controla
                                                  recursos

  Imagem                  Container               Imagem é modelo;
                                                  container é execução

  Dockerfile              Compose                 Dockerfile constrói
                                                  imagem; Compose
                                                  organiza serviços

  Git                     DVC                     Git versiona código;
                                                  DVC ajuda a versionar
                                                  dados

  MLflow                  Model Registry          MLflow rastreia
                                                  experimentos; Registry
                                                  organiza modelos

  Data Drift              Concept Drift           Mudança em `P(X)`
                                                  vs. mudança em
                                                  `P(Y\|X)`

  Healthcheck             Drift monitoring        Saúde técnica
                                                  vs. comportamento
                                                  estatístico

  Champion                Challenger              Modelo atual
                                                  vs. candidato

  Shadow                  Canary                  Shadow observa sem
                                                  decidir; Canary recebe
                                                  parte do tráfego

  CI                      CD                      Integração/testes
                                                  vs. entrega/deploy
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 80. Resumo em uma frase

**Container:** empacota e isola a execução.

**Imagem:** modelo usado para criar containers.

**Dockerfile:** receita de construção da imagem.

**Namespaces:** controlam a visão do processo.

**cgroups:** controlam recursos.

**MLOps:** engenharia do ciclo de vida de ML em produção.

**DVC:** ajuda a versionar dados.

**MLflow:** registra experimentos e modelos.

**Model Registry:** organiza versões e estágios.

**FastAPI:** disponibiliza o modelo como API.

**Pydantic:** valida dados.

**CI/CD:** automatiza testes e entrega.

**Observabilidade:** acompanha o comportamento do sistema.

**Data Drift:** mudança na distribuição das entradas.

**Concept Drift:** mudança na relação entre entradas e resultados.

**KS:** compara distribuições.

**PSI:** mede estabilidade populacional.

**KL:** mede divergência entre distribuições.

**Champion:** modelo atual.

**Challenger:** modelo candidato.

**Shadow:** testa o candidato sem usá-lo na decisão final.

**Canary:** envia parte do tráfego para a nova versão.

**Blue-Green:** mantém dois ambientes para facilitar a troca.

**Feature Store:** centraliza features.

**Model Card:** documenta características, uso e limitações do modelo.

------------------------------------------------------------------------

# 81. O modelo mental definitivo

Se você precisar guardar apenas uma sequência, pense:

``` text
             MODELO
                │
                ▼
          "Como colocar
           em produção?"
                │
                ▼
             DOCKER
                │
                ▼
          "Como controlar
           o ambiente?"
                │
                ▼
              MLOPS
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     Código   Dados    Modelo
       │        │        │
      Git      DVC     MLflow
                         │
                         ▼
                       Registry
                         │
                         ▼
                       CI/CD
                         │
                         ▼
                       Deploy
                         │
                         ▼
                  Observabilidade
                         │
                         ▼
                        Drift
                         │
                         ▼
                  Retreinamento
                         │
                         ▼
                       Challenger
                         │
                         ▼
                    Novo Deploy
                         │
                         └──────► CICLO
```

------------------------------------------------------------------------

# 82. Conclusão

O ponto central é entender que **Machine Learning em produção é um
sistema, não apenas um modelo**.

Um modelo pode apresentar uma ótima métrica no notebook e ainda falhar
por problemas de:

-   ambiente;
-   dependências;
-   dados;
-   pré-processamento;
-   integração;
-   infraestrutura;
-   segurança;
-   qualidade;
-   monitoramento;
-   mudança de distribuição.

O Docker resolve principalmente o problema de **empacotamento e
isolamento do ambiente**.

O FastAPI transforma o modelo em um **serviço consumível**.

Git, DVC e MLflow aumentam a **rastreabilidade**.

O Model Registry organiza o **ciclo de vida dos modelos**.

CI/CD automatiza **testes, validações e entrega**.

Observabilidade mostra **o que está acontecendo em produção**.

KS, PSI e outras técnicas ajudam a identificar **mudanças estatísticas
nos dados**.

Champion/Challenger, Canary e Blue-Green permitem **atualizar modelos de
maneira controlada**.

E MLOps conecta tudo em um ciclo:

``` text
Desenvolver
 ↓
Versionar
 ↓
Treinar
 ↓
Validar
 ↓
Registrar
 ↓
Empacotar
 ↓
Testar
 ↓
Publicar
 ↓
Monitorar
 ↓
Detectar mudanças
 ↓
Retreinar
 ↓
Validar novamente
 ↓
Publicar novamente
```

O objetivo não é simplesmente ter um modelo funcionando.

É conseguir responder:

> **Qual modelo está em produção?**

> **Com quais dados ele foi treinado?**

> **Qual código o gerou?**

> **Qual ambiente o executa?**

> **Como sabemos que ele continua funcionando?**

> **O que acontece se os dados mudarem?**

> **Como substituímos o modelo com segurança?**

Quando essas perguntas podem ser respondidas sistematicamente, estamos
saindo de um projeto de ML experimental e entrando em **Engenharia de
Machine Learning e MLOps**.
