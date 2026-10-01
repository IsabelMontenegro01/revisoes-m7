# Engenharia de Machine Learning e MLOps — Guia de Estudo

> Material baseado no arquivo **Engenharia de Machine Learning e MLOps: Da Teoria Rigorosa à Implementação de Produção com Containers e Docker**.
>
> O foco deste guia é entender **o problema, o conceito, por que usar cada tecnologia, seus benefícios e como tudo se conecta**. O código aparece apenas quando ajuda a compreender uma ideia.

---

# 1. Machine Learning não termina no treinamento

&emsp;É comum imaginar ML assim:

```text
Dados → Treinamento → Modelo → Predição
```

&emsp;Isso funciona para um notebook. Em produção, o ciclo é maior:

```text
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

&emsp;A ideia central de **MLOps** é justamente transformar esse ciclo em um processo confiável, rastreável e automatizado.

> **Treinar um modelo é uma etapa. Manter esse modelo funcionando corretamente em produção é um problema de engenharia.**

---

# 2. Por que existe o problema "funciona na minha máquina"?

&emsp;Imagine um modelo treinado com:

- Python 3.11;
- determinada versão do scikit-learn;
- bibliotecas específicas;
- determinado pipeline de pré-processamento;
- arquivo de modelo treinado;
- configurações locais.

&emsp;Ao enviar apenas o código e o `.pkl` para outro servidor, podem existir:

- outra versão do Python;
- bibliotecas incompatíveis;
- bibliotecas do sistema ausentes;
- caminhos de arquivos diferentes;
- configurações de GPU diferentes;
- variáveis de ambiente diferentes.

&emsp;Em Machine Learning isso é ainda mais crítico porque não basta reproduzir o código. É necessário reproduzir o **ambiente, os dados, o pré-processamento e o modelo**.

---

# 3. Um sistema de ML é maior que o algoritmo

&emsp;Podemos pensar em:

```text
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

&emsp;O algoritmo matemático é apenas uma parte.

&emsp;Por isso, um modelo pode apresentar excelentes resultados no laboratório e ainda falhar quando integrado a um sistema real.

---

# 4. Código, dados e hiperparâmetros

&emsp;O material apresenta a ideia:

```text
Sistema Inteligente = f(Código, Dados, Hiperparâmetros)
```

## Código

&emsp;Define como os dados são tratados, quais transformações são feitas e como o modelo é utilizado.

## Dados

&emsp;Determinam aquilo que o modelo aprende. Alterar o conjunto de treinamento pode produzir um modelo diferente mesmo com o mesmo código.

## Hiperparâmetros

&emsp;São configurações do treinamento, como taxa de aprendizado, profundidade de árvores, regularização e número de estimadores.

### Por que isso importa?

&emsp;Para reproduzir um modelo precisamos saber **o que foi usado para produzi-lo**, não apenas possuir o arquivo final.

---

# 5. Bare-metal, máquinas virtuais e containers

## Bare-metal

&emsp;A aplicação executa diretamente no sistema operacional da máquina física:

```text
Aplicação
↓
Sistema Operacional
↓
Hardware
```

### Benefício

&emsp;Pouco overhead.

### Problema

&emsp;As aplicações compartilham o mesmo ambiente. Projetos que precisam de versões incompatíveis de Python, bibliotecas ou CUDA podem entrar em conflito.

---

## Máquina virtual

&emsp;Uma VM virtualiza o hardware:

```text
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

&emsp;Cada VM possui seu próprio sistema operacional.

### Benefícios

- isolamento forte;
- possibilidade de sistemas operacionais diferentes;
- separação maior entre ambientes.

### Desvantagens

- mais memória;
- mais disco;
- inicialização mais lenta;
- maior overhead.

---

## Container

&emsp;Containers utilizam o kernel do host e isolam processos:

```text
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

> **VM virtualiza uma máquina; container isola processos dentro de uma máquina.**

---

# 6. VM x Container

| Característica      | VM                | Container          |
| ------------------- | ----------------- | ------------------ |
| Virtualização       | Hardware          | Ambiente/processos |
| Sistema operacional | Cada VM possui um | Compartilhado      |
| Kernel              | Próprio           | Compartilhado      |
| Consumo             | Maior             | Menor              |
| Inicialização       | Mais lenta        | Mais rápida        |
| Isolamento          | Mais forte        | Mais leve          |

&emsp;Containers são especialmente úteis quando queremos empacotar aplicações e suas dependências de maneira reproduzível.

---

# 7. Container como ambiente reproduzível

&emsp;A ideia da "cesta" ajuda a entender a principal vantagem do container: em vez de depender do que existe na máquina de destino, o ambiente necessário é empacotado junto com a aplicação.

&emsp;Para ML, essa cesta pode conter Python, bibliotecas, código, modelo treinado, pesos, pré-processamento e configurações. Assim, o ambiente de execução deixa de depender tanto da máquina onde o software foi instalado.

> **A ideia central é empacotar o ambiente necessário para tornar a execução mais reproduzível.**

---

# 8. Namespaces

&emsp;Namespaces controlam **o que um processo consegue enxergar**.

&emsp;Um container pode possuir uma visão isolada de:

- processos;
- rede;
- pontos de montagem;
- usuários;
- hostname.

### PID namespace

&emsp;Process ID. Isola a árvore de processos. O processo principal pode ser visto como PID 1 dentro do container.

### Network namespace

&emsp;Isola interfaces, IPs, portas e rotas.

### Mount namespace

&emsp;Isola a visão dos pontos de montagem e do sistema de arquivos.

### User namespace

&emsp;Permite separar identificadores de usuários entre container e host.

### Regra para lembrar

> **Namespaces = visão do sistema.**

---

# 9. cgroups

&emsp;Cgroups, ou Control Groups, controlam **quanto de recurso um processo pode consumir**.

&emsp;Podemos estabelecer limites de:

- CPU;
- memória;
- número de processos;
- I/O.

&emsp;Por exemplo:

```text
Container
├── CPU: limite definido
└── Memória: limite definido
```

### Por que usar?

&emsp;Evita que um serviço consuma recursos indefinidamente e prejudique os demais.

&emsp;Isso melhora:

- isolamento;
- previsibilidade;
- estabilidade;
- utilização da infraestrutura.

---

# 10. CPU x memória

&emsp;Os limites possuem comportamentos diferentes.

### CPU

&emsp;Um processo pode sofrer **throttling** e ficar mais lento.

### Memória

&emsp;Quando o limite é ultrapassado e não há memória suficiente para recuperar, pode ocorrer **OOM kill**.

&emsp;Isso pode aparecer associado ao:

```text
exit code 137
```

&emsp;Portanto:

```text
CPU → pode gerar lentidão
Memória → pode provocar encerramento
```

---

# 11. Imagem, container e registry

## Imagem

&emsp;É o modelo utilizado para criar containers.

```text
Imagem → modelo
```

## Container

&emsp;É uma execução de uma imagem.

```text
Container → instância em execução
```

&emsp;Uma analogia:

```text
Imagem = classe
Container = objeto
```

## Registry

&emsp;É um repositório de imagens.

&emsp;O fluxo pode ser:

```text
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

---

# 12. Camadas de uma imagem

&emsp;Imagens Docker são formadas por camadas:

```text
Camada base
↓
Dependências
↓
Código
↓
Configuração
```

### Por que isso é útil?

&emsp;Camadas que não mudaram podem ser reutilizadas. Isso:

- acelera builds;
- reduz downloads;
- economiza espaço;
- evita trabalho repetido.

---

# 13. Dockerfile

&emsp;O Dockerfile descreve **como construir uma imagem**.

&emsp;É uma receita automatizada do ambiente. Principais instruções:

## `FROM`

&emsp;Define a imagem base.

```dockerfile
FROM python:3.11-slim
```

&emsp;Evita começar o ambiente do zero.

## `WORKDIR`

&emsp;Define o diretório de trabalho.

## `COPY`

&emsp;Copia arquivos para a imagem.

## `RUN`

&emsp;Executa comandos durante o build.

```dockerfile
RUN pip install -r requirements.txt
```

## `CMD`

&emsp;Define o processo principal executado quando o container inicia.

```dockerfile
CMD ["python", "app.py"]
```

## `EXPOSE`

&emsp;Documenta a porta usada pela aplicação.

> `EXPOSE` não publica a porta no computador.

---

# 14. `RUN` x `CMD`

&emsp;Essa diferença precisa estar muito clara:

```text
RUN
→ acontece durante o build

CMD
→ acontece quando o container inicia
```

&emsp;Confundir os dois é um erro comum.

---

# 15. Por que a ordem do Dockerfile importa?

&emsp;Uma organização comum é:

```dockerfile
COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .
```

&emsp;As dependências geralmente mudam menos que o código.

&emsp;Se apenas o código mudar:

```text
requirements.txt não mudou
↓
camada de instalação pode ser reutilizada
↓
somente o código é reconstruído
```

&emsp;Isso aproveita melhor o cache do Docker.

---

# 16. Multi-stage build

&emsp;Multi-stage build usa diferentes etapas para construir a aplicação e gerar uma imagem final menor.

&emsp;A lógica é:

```text
Etapa de build
↓
Compilar / instalar / preparar
↓
Artefatos necessários
↓
Imagem final
```

&emsp;Ferramentas usadas somente para construir a aplicação não precisam permanecer na imagem final.

### Benefícios

- imagem menor;
- download mais rápido;
- menor armazenamento;
- menor superfície de ataque;
- deploy mais rápido.

---

# 17. Usuário não-root

&emsp;Aplicações não precisam necessariamente executar como `root`. Utilizar um usuário com menos privilégios reduz o impacto potencial de uma vulnerabilidade.

&emsp;Esse princípio é chamado de **least privilege**.

```text
Aplicação
↓
Privilégios mínimos
```

&emsp;É preferível a:

```text
Aplicação
↓
root
```

&emsp;quando o root não é necessário.

---

# 18. Docker aplicado a Machine Learning

&emsp;Em Machine Learning, a reprodutibilidade do ambiente também envolve garantir que o modelo receba os dados na representação esperada durante o treinamento.

&emsp;Por isso, a aplicação em produção precisa manter consistentes as etapas de validação, normalização e feature engineering utilizadas antes da inferência.

&emsp;Um fluxo de inferência pode ser:

```text
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

&emsp;Se o pré-processamento for diferente entre treino e produção, o modelo pode receber dados em um formato diferente daquele que aprendeu.

---

# 19. Model Serving

&emsp;Model serving é disponibilizar o modelo para receber entradas e retornar previsões.

```text
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

&emsp;Isso transforma o modelo em um serviço que pode ser consumido por outros sistemas.

---

# 20. FastAPI

&emsp;FastAPI pode ser utilizada para criar a API de inferência. Ela funciona como uma camada entre:

```text
Sistema consumidor
↓
API
↓
Modelo
```

&emsp;O consumidor não precisa conhecer os detalhes matemáticos do modelo.

---

# 21. Pydantic e validação de entradas

&emsp;Uma API precisa verificar se os dados recebidos são válidos. Por exemplo:

```text
customer_id
age
annual_income
credit_score
loan_amount
```

&emsp;Pydantic permite definir e validar esse contrato.

### Benefícios

- tipos explícitos;
- validação automática;
- erros mais claros;
- contrato de entrada bem definido.

&emsp;Isso evita que dados inválidos cheguem ao modelo.

### Por que validar antes da inferência?

&emsp;Sem validação:

```text
Entrada inválida
↓
Modelo
↓
Erro inesperado
```

&emsp;Com validação:

```text
Entrada
↓
Validação
├── inválida → erro controlado
└── válida → modelo
```

&emsp;Isso melhora a confiabilidade da API.

---

# 22. Por que ML é diferente de software tradicional?

&emsp;Em software tradicional, frequentemente pensamos:

```text
Código + Entrada → Saída
```

&emsp;Em ML:

```text
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

&emsp;Mesmo sem alterar o código, mudanças nos dados podem alterar o comportamento do sistema.

---

# 23. Dívida técnica em Machine Learning

&emsp;O material utiliza o trabalho de Sculley et al. para mostrar que o algoritmo é apenas uma pequena parte de um sistema de ML.

&emsp;O sistema pode envolver:

- coleta de dados;
- pipelines;
- features;
- infraestrutura;
- integração;
- monitoramento;
- configuração;
- segurança.

> **Um modelo funcionando em um notebook não significa que temos um sistema de ML pronto para produção.**

---

# 24. Boundary Erosion

&emsp;**Boundary Erosion** é a erosão dos limites entre responsabilidades.

&emsp;Um exemplo ruim seria um único script responsável por:

```text
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

&emsp;Fica difícil:

- testar;
- modificar;
- reutilizar;
- entender.

&emsp;Separar responsabilidades torna o sistema mais sustentável.

---

# 25. CACE — Changing Anything Changes Everything

&emsp;Em ML, uma pequena alteração pode gerar efeitos em várias partes do sistema. Exemplo:

```text
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

&emsp;Por isso mudanças precisam ser rastreáveis.

---

# 26. Pipeline Jungles

&emsp;Uma pipeline jungle acontece quando o fluxo de ML cresce sem organização. Exemplo:

```text
script.py
cron
SQL manual
notebook_final.ipynb
modelo_v2.pkl
dados_final.csv
dados_final_v2.csv
```

&emsp;Depois fica difícil responder:

> Qual processo realmente gerou o modelo em produção?

&emsp;MLOps tenta substituir esse conjunto de scripts por pipelines rastreáveis e automatizados.

---

# 27. Glue Code

&emsp;Glue code é código improvisado para conectar sistemas. Por exemplo:

```text
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

&emsp;Quanto mais conversões improvisadas existirem, maior a fragilidade do sistema.

&emsp;Uma arquitetura bem definida reduz esse acoplamento.

---

# 28. Data Testing Debt

&emsp;Testar apenas o código não basta.

&emsp;Podemos ter:

```text
Código → 100% dos testes passando
```

&emsp;e:

```text
Dataset → coluna inteira nula
```

&emsp;Por isso também precisamos testar:

- presença de colunas;
- tipos;
- valores ausentes;
- intervalos;
- distribuição;
- qualidade dos dados.

---

# 29. O que é MLOps?

&emsp;MLOps significa **Machine Learning Operations**. Não é uma única ferramenta.

&emsp;É a combinação de:

- práticas;
- processos;
- automação;
- arquitetura;
- ferramentas.

&emsp;O objetivo é permitir que modelos sejam:

- entregues;
- reproduzidos;
- monitorados;
- auditados;
- atualizados;
- mantidos em produção.

> **MLOps aplica princípios de engenharia e operações ao ciclo de vida de Machine Learning.**

### MLOps integra três áreas

```text
Engenharia de Dados
        +
Ciência de Dados
        +
DevOps
        ↓
      MLOps
```

#### Engenharia de Dados

&emsp;Dados, ingestão, transformação, armazenamento e qualidade.

#### Ciência de Dados

&emsp;Features, treinamento, avaliação e modelos.

#### DevOps

&emsp;Automação, infraestrutura, deploy e observabilidade.

---

# 30. CD4ML

&emsp;**CD4ML — Continuous Delivery for Machine Learning** — adapta a ideia de entrega contínua ao contexto de ML. Três elementos precisam evoluir juntos:

```text
Código
Dados
Modelos
```

## Código

&emsp;Pipelines, APIs e regras versionadas.

## Dados

&emsp;Dados brutos e transformados precisam ser rastreáveis e, quando necessário, versionados.

## Modelos

&emsp;Artefatos precisam ser catalogados com informações como métricas, hiperparâmetros e linhagem.

---

# 31. Git x DVC

&emsp;Uma divisão útil:

```text
Git
↓
Código + configuração

DVC
↓
Dados + versões dos dados
```

&emsp;Git é excelente para código. DVC pode complementar o processo quando datasets e artefatos de dados são grandes ou precisam de versionamento específico.

### Benefício

&emsp;Permite reconstruir melhor o contexto de um experimento.

---

# 32. MLflow

&emsp;MLflow pode ser utilizado para **experiment tracking**.

&emsp;Podemos ter:

```text
Run 1
Run 2
Run 3
Run 4
```

&emsp;Cada execução pode registrar:

- hiperparâmetros;
- métricas;
- artefatos;
- modelo.

&emsp;Isso permite responder:

> Qual configuração gerou este resultado?

---

# 33. Model Registry

&emsp;O Model Registry organiza versões e estágios dos modelos. Conceitualmente:

```text
Treinado
↓
Validado
↓
Candidato
↓
Produção
```

### Por que usar?

&emsp;Evita a situação:

```text
modelo_final.pkl
modelo_final_v2.pkl
modelo_final_novo.pkl
modelo_final_real.pkl
```

&emsp;O registro cria uma referência mais organizada para os modelos.

---

# 34. Níveis de maturidade em MLOps

## Nível 0 — Processo manual

&emsp;Características:

- notebooks;
- scripts avulsos;
- treinamento manual;
- deploy manual;
- arquivos compartilhados.

&emsp;Problema:

```text
Intervenção humana
↓
Erros
↓
Baixa reprodutibilidade
```

---

## Nível 1 — Continuous Training

&emsp;Começamos a automatizar o treinamento:

```text
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

---

## Nível 2 — CI/CD para ML

&emsp;Existe uma automação mais completa:

```text
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

&emsp;Também podem existir mecanismos como Champion/Challenger, deploy controlado e retreinamento orientado por eventos.

---

# 35. Champion x Challenger

```text
Champion
→ modelo atualmente utilizado

Challenger
→ novo modelo candidato
```

&emsp;O Challenger é avaliado antes de substituir o Champion.

&emsp;Isso reduz o risco de colocar uma nova versão em produção sem validação suficiente.

---

# 36. Shadow Deployment

&emsp;No Shadow Mode, o novo modelo recebe dados reais, mas suas respostas não são utilizadas para a decisão final.

```text
Tráfego real
├──► Champion → decisão real
└──► Challenger → avaliação
```

### Benefício

&emsp;Permite comparar o novo modelo com dados reais sem colocá-lo imediatamente no controle da decisão.

---

# 37. Canary Deployment

&emsp;No Canary, uma parcela do tráfego é direcionada à nova versão. Exemplo:

```text
95% → versão atual
5%  → versão nova
```

&emsp;Se o comportamento for adequado, a nova versão pode receber mais tráfego.

### Benefício

&emsp;Reduz o impacto potencial de um problema.

---

# 38. Blue-Green Deployment

&emsp;Mantemos dois ambientes:

```text
Blue  → versão atual
Green → versão nova
```

&emsp;Depois de validar o novo ambiente, o tráfego pode ser direcionado para ele.

### Benefício

&emsp;Facilita trocas controladas e rollback.

---

# 39. Observabilidade

&emsp;Observabilidade é a capacidade de entender o comportamento do sistema a partir dos sinais que ele produz.

&emsp;Em ML podemos observar:

- latência;
- erros;
- número de requisições;
- distribuição das features;
- qualidade dos dados;
- métricas do modelo;
- drift.

&emsp;Uma API pode estar tecnicamente saudável e ainda assim produzir previsões ruins.

---

# 40. Healthcheck x monitoramento do modelo

&emsp;São coisas diferentes.

### Healthcheck

&emsp;Pergunta:

> O serviço está funcionando?

### Monitoramento de ML

&emsp;Pergunta:

> O modelo continua se comportando de forma adequada?

&emsp;É possível ter:

```text
Healthcheck → OK
```

&emsp;e simultaneamente:

```text
Drift → alto
```

&emsp;Por isso os dois são necessários.

&emsp;Isso pode acontecer, por exemplo, quando:

```text
HTTP 200
CPU normal
Memória normal
Container saudável

↓

qualidade das previsões ↓
```

&emsp;Esse é um dos motivos pelos quais monitorar apenas a infraestrutura não é suficiente para ML.

---

# 41. Data Drift

&emsp;Data Drift ocorre quando a distribuição dos dados observados em produção muda em relação à referência. Por exemplo:

```text
Treinamento

idade média = 35
```

&emsp;Depois:

```text
Produção

idade média = 55
```

&emsp;A API pode continuar funcionando normalmente, mas o contexto dos dados mudou.

---

# 42. Concept Drift

&emsp;Data Drift e Concept Drift não são a mesma coisa.

### Data Drift

&emsp;Mudança na distribuição das entradas:

```text
P(X) muda
```

### Concept Drift

&emsp;Mudança na relação entre entrada e resultado:

```text
P(Y|X) muda
```

&emsp;Por exemplo, o mesmo perfil de cliente pode passar a ter uma probabilidade diferente de inadimplência devido a uma mudança econômica.

---

# 43. Teste Kolmogorov-Smirnov

&emsp;O teste **KS** pode comparar duas distribuições.

&emsp;A estatística apresentada é:

```text
D = sup |F_ref(x) - F_prod(x)|
```

&emsp;De maneira intuitiva:

> Quanto maior a diferença entre as distribuições acumuladas, maior a evidência de que elas não são iguais.

&emsp;É especialmente útil para comparar distribuições de variáveis numéricas.

---

# 44. PSI — Population Stability Index

&emsp;O PSI é muito utilizado em cenários de risco e crédito.

&emsp;Uma forma de representá-lo é:

```text
PSI = Σ (P - Q) × ln(P / Q)
```

&emsp;Ele compara a proporção observada em determinadas faixas com a proporção de referência.

&emsp;O material apresenta a seguinte referência:

| PSI           | Interpretação         |
| ------------- | --------------------- |
| `< 0,10`      | Estável               |
| `0,10 – 0,25` | Mudança moderada      |
| `>= 0,25`     | Mudança significativa |

&emsp;Esses valores são referências e devem ser interpretados de acordo com o contexto do modelo.

---

# 45. KL Divergence

&emsp;A **Divergência de Kullback-Leibler** mede a diferença entre uma distribuição e uma referência.

&emsp;A fórmula apresentada no material é:

```text
D_KL(P || Q) = ∫ p(x) log(p(x) / q(x)) dx
```

&emsp;O importante é entender o propósito:

> Medir quão diferente uma distribuição está em relação a outra segundo essa medida.

---

# 46. KS x PSI x KL

| Técnica | Ideia                                    | Uso                       |
| ------- | ---------------------------------------- | ------------------------- |
| KS      | Distância entre distribuições acumuladas | Comparação estatística    |
| PSI     | Diferença entre proporções em bins       | Estabilidade populacional |
| KL      | Divergência entre distribuições          | Diferença informacional   |

&emsp;O ponto principal não é decorar fórmulas isoladas.

&emsp;É saber que são ferramentas para investigar **mudanças na distribuição dos dados**.

---

# 47. Drift não significa automaticamente modelo ruim

&emsp;Se houve drift:

```text
Os dados mudaram.
```

&emsp;Isso não significa automaticamente:

```text
O modelo está inútil.
```

&emsp;É necessário investigar:

- magnitude da mudança;
- quais features mudaram;
- impacto nas métricas;
- contexto de negócio;
- duração da mudança;
- possibilidade de mudança temporária.

---

# 48. CI/CD para Machine Learning

&emsp;Em software:

```text
Código
↓
Teste
↓
Build
↓
Deploy
```

&emsp;Em ML:

```text
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

&emsp;Além dos testes tradicionais, podem existir critérios relacionados às métricas do modelo.

---

# 49. Quality Gates

&emsp;Um **quality gate** é uma condição que precisa ser satisfeita para o pipeline continuar. Exemplo:

```text
F1 >= 0,80
```

&emsp;Se o modelo candidato não satisfizer a condição:

```text
Validação
↓
Falha
↓
Deploy bloqueado
```

### Benefício

&emsp;Evita que uma regressão conhecida avance automaticamente para produção.

---

# 50. Diagnóstico de uma API de ML

&emsp;Quando uma API não funciona, investigue por camadas:

```text
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

&emsp;Isso é melhor do que alterar várias partes simultaneamente.

---

# 51. Logs

&emsp;Logs são uma das primeiras fontes de evidência. Procure por:

- erros de import;
- arquivo não encontrado;
- modelo não carregado;
- erro de porta;
- erro de validação;
- exceções durante inferência.

&emsp;No Docker:

```bash
docker logs nome-do-container
```

---

# 52. Feature Store

&emsp;Uma **Feature Store** centraliza features utilizadas por modelos. Pode existir:

```text
Offline Store
↓
Treinamento

Online Store
↓
Inferência em tempo real
```

### Por que usar?

&emsp;Para evitar que uma feature seja calculada de uma maneira durante o treinamento e de outra durante a inferência.

&emsp;Esse problema é conhecido como inconsistência entre treino e serving.

---

# 53. Orquestração distribuída

&emsp;Quando o treinamento cresce, uma única máquina pode não ser suficiente.

&emsp;O material cita ferramentas como:

- Ray;
- Kubeflow Pipelines.

&emsp;A ideia é:

```text
Problema grande
↓
Distribuição
↓
Vários recursos
↓
Execução coordenada
```

&emsp;Isso permite escalar treinamento e pipelines.

---

# 54. Edge Computing e TinyML

&emsp;Nem todo modelo precisa executar em um servidor. Alguns cenários exigem execução em:

- celulares;
- IoT;
- veículos;
- equipamentos industriais.

&emsp;Nesses casos, latência, memória e energia podem ser críticos.

&emsp;Por isso existem técnicas específicas de otimização e inferência na borda.

---

# 55. ONNX

&emsp;ONNX é um formato para representar modelos de Machine Learning de maneira interoperável.

&emsp;A ideia é facilitar a execução de modelos em ambientes diferentes daqueles utilizados no treinamento.

&emsp;O ONNX Runtime pode ser utilizado para inferência otimizada.

---

# 56. Quantização

&emsp;Quantização reduz a precisão numérica dos pesos. Por exemplo:

```text
FP32 → FP16
```

&emsp;ou:

```text
FP32 → INT8
```

&emsp;Pode reduzir:

- memória;
- tamanho do modelo;
- custo computacional.

&emsp;Mas existe um trade-off:

```text
menor precisão numérica
↓
possível perda de qualidade
```

&emsp;Por isso a técnica precisa ser validada no modelo real.

---

# 57. Governança

&emsp;Quando um modelo influencia decisões reais, não basta saber sua acurácia.

&emsp;Também precisamos saber:

- para que foi criado;
- onde pode ser usado;
- quais são suas limitações;
- como foi avaliado;
- quais dados utiliza;
- quais grupos foram considerados.

&emsp;Isso faz parte da governança de Machine Learning.

---

# 58. Model Cards

&emsp;Model Cards são documentos utilizados para registrar informações importantes sobre modelos. Podem incluir:

- objetivo;
- contexto;
- limitações;
- métricas;
- dados;
- possíveis vieses;
- grupos avaliados.

### Benefícios

- transparência;
- documentação;
- auditabilidade;
- comunicação das limitações.

---

# 59. Fairness e viés

&emsp;Uma métrica média pode esconder diferenças entre grupos. Dependendo do contexto, pode ser necessário avaliar:

- desempenho por grupo;
- diferenças de erro;
- métricas de equidade;
- impacto das decisões.

&emsp;Ferramentas como Fairlearn e AI Fairness 360 são exemplos de recursos para esse tipo de análise.

---

# 60. Privacidade e LGPD

&emsp;Sistemas de ML que usam dados pessoais também precisam considerar:

- quais dados são coletados;
- por que são usados;
- quem possui acesso;
- como são armazenados;
- por quanto tempo;
- como são protegidos.

&emsp;MLOps não resolve sozinho essas questões, mas uma arquitetura profissional precisa incluí-las.

---

# 61. Rastreabilidade

&emsp;Rastreabilidade significa conseguir responder:

> De onde veio este modelo?

&emsp;Podemos ter:

```text
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

- debugging;
- auditoria;
- rollback;
- comparação;
- manutenção.

---

# 62. Reprodutibilidade

&emsp;Idealmente:

```text
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

&emsp;Se esses elementos estiverem registrados, o resultado pode ser reconstruído ou investigado.

---

# 63. Rollback

&emsp;Se uma versão nova apresentar problema:

```text
v2.1
↓
problema
↓
rollback
↓
v2.0
```

&emsp;Isso só é possível de maneira confiável se as versões anteriores estiverem identificadas e disponíveis.

---

# 64. Como todas as ferramentas se conectam

| Problema     | Ferramenta/conceito | O que resolve          |
| ------------ | ------------------- | ---------------------- |
| Código       | Git                 | Versionamento          |
| Dados        | DVC                 | Versionamento de dados |
| Experimentos | MLflow              | Rastreamento           |
| Modelos      | Model Registry      | Versões e estágios     |
| API          | FastAPI             | Serving                |
| Contratos    | Pydantic            | Validação              |
| Ambiente     | Docker              | Empacotamento          |
| Entrega      | CI/CD               | Automação              |
| Saúde        | Healthcheck         | Verificação do serviço |
| Drift        | KS / PSI / KL       | Mudanças estatísticas  |
| Features     | Feature Store       | Centralização          |
| Governança   | Model Cards         | Documentação           |

---

# 65. O ciclo completo de MLOps

&emsp;O modelo mental mais importante é:

```text
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

---

# 66. Roadmap e ordem de estudo

&emsp;O material propõe uma sequência de laboratórios.

## Lab 1 — DVC + Git

&emsp;Aprender:

```text
Código → Git
Dados → DVC
```

&emsp;Objetivo: reprodutibilidade.

## Lab 2 — MLflow

&emsp;Aprender:

```text
Experiment Tracking
+
Model Registry
```

&emsp;Objetivo: rastrear experimentos e modelos.

## Lab 3 — CI/CD

&emsp;Aprender:

```text
Testes
+
Quality Gates
```

&emsp;Objetivo: impedir que versões inadequadas avancem.

## Lab 4 — Docker + FastAPI

&emsp;Aprender:

```text
Serving
+
Containerização
```

&emsp;Objetivo: empacotar e disponibilizar o modelo.

## Lab 5 — Drift + Observabilidade

&emsp;Aprender:

```text
Monitoramento
+
KS / PSI
+
Alertas
```

&emsp;Objetivo: detectar mudanças nos dados.

&emsp;Uma ordem lógica para executar esses estudos é:

```text
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

&emsp;Cada etapa resolve um problema que aparece naturalmente depois da anterior.

---

# 67. Checklist para prova

## Fundamentos

- [ ] Por que ML em produção é diferente de um notebook?
- [ ] Por que existe o problema "funciona na minha máquina"?
- [ ] Qual a relação entre código, dados e hiperparâmetros?
- [ ] Qual a diferença entre bare-metal, VM e container?
- [ ] Por que containers são mais leves?
- [ ] O que são namespaces?
- [ ] O que são cgroups?
- [ ] Qual a diferença entre visão e controle de recursos?

## Docker

- [ ] O que é uma imagem?
- [ ] O que é um container?
- [ ] O que é um registry?
- [ ] Por que imagens possuem camadas?
- [ ] Para que serve Dockerfile?
- [ ] Qual a diferença entre `FROM`, `RUN`, `COPY` e `CMD`?
- [ ] Por que a ordem do Dockerfile influencia o cache?
- [ ] O que é multi-stage build?
- [ ] Por que usar usuário não-root?
- [ ] O que `EXPOSE` faz?

## Serving

- [ ] O que é model serving?
- [ ] Para que serve FastAPI?
- [ ] Por que validar entradas?
- [ ] Qual o papel do Pydantic?
- [ ] Por que o pré-processamento precisa ser consistente?
- [ ] O que é healthcheck?

## MLOps

- [ ] O que é MLOps?
- [ ] O que é CD4ML?
- [ ] Por que Git é importante?
- [ ] Por que DVC pode ser usado?
- [ ] Para que serve MLflow?
- [ ] O que é Model Registry?
- [ ] Quais são os níveis de maturidade?
- [ ] O que são Champion e Challenger?
- [ ] O que é Shadow Deployment?
- [ ] O que é Canary?
- [ ] O que é Blue-Green?

## Observabilidade

- [ ] O que é Data Drift?
- [ ] O que é Concept Drift?
- [ ] Qual a diferença?
- [ ] O que KS mede?
- [ ] O que é PSI?
- [ ] O que é KL Divergence?
- [ ] Por que uma API pode estar saudável e o modelo estar ruim?

## Produção

- [ ] O que é dívida técnica em ML?
- [ ] O que é Boundary Erosion?
- [ ] O que são Pipeline Jungles?
- [ ] O que é Glue Code?
- [ ] O que é Data Testing Debt?
- [ ] Por que versionar dados, código e modelos?
- [ ] Por que rollback é importante?
- [ ] O que são Model Cards?
- [ ] Por que governança e fairness importam?

---

# 68. Diferenças que você precisa saber

| Conceito A  | Conceito B       | Diferença                                                  |
| ----------- | ---------------- | ---------------------------------------------------------- |
| VM          | Container        | VM virtualiza uma máquina; container isola processos       |
| Namespace   | cgroup           | Namespace controla visão; cgroup controla recursos         |
| Imagem      | Container        | Imagem é modelo; container é execução                      |
| Dockerfile  | Compose          | Dockerfile constrói imagem; Compose organiza serviços      |
| Git         | DVC              | Git versiona código; DVC ajuda a versionar dados           |
| MLflow      | Model Registry   | MLflow rastreia experimentos; Registry organiza modelos    |
| Data Drift  | Concept Drift    | Mudança em `P(X)` vs. mudança em `P(Y\|X)`                 |
| Healthcheck | Drift monitoring | Saúde técnica vs. comportamento estatístico                |
| Champion    | Challenger       | Modelo atual vs. candidato                                 |
| Shadow      | Canary           | Shadow observa sem decidir; Canary recebe parte do tráfego |
| CI          | CD               | Integração/testes vs. entrega/deploy                       |

---

# 69. O modelo mental definitivo

&emsp;Se você precisar guardar apenas uma sequência, pense:

```text
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
Git      DVC      MLflow
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

---

# 70. Conclusão

&emsp;O ponto central é entender que **Machine Learning em produção é um sistema, não apenas um modelo**. O modelo depende de dados, código, ambiente, integração, monitoramento e processos de atualização, e cada uma dessas partes precisa ser tratada de forma organizada.

&emsp;Nesse contexto, Docker ajuda a tornar o ambiente reproduzível, as ferramentas de versionamento e rastreabilidade registram como o modelo foi construído, CI/CD automatiza validações e entrega, e a observabilidade permite acompanhar o comportamento do sistema depois do deploy.

&emsp;Quando um modelo entra em produção, o trabalho não termina. É necessário observar mudanças nos dados e no comportamento, investigar problemas, validar novas versões e atualizar o modelo de forma controlada. Esse ciclo contínuo é a base da Engenharia de Machine Learning e do MLOps.

> **A ideia central: um modelo pronto para produção precisa ser reproduzível, rastreável, observável e atualizável.**