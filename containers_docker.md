# Containers e Docker — Guia Completo de Estudo

> Material de estudo baseado nos conteúdos do Encontro 01 — Containers e no Guia de Estudos de Imagens e Docker Compose.

---

## 1. Por que containers existem?

Antes de entender Docker, é importante entender **qual problema ele resolve**.

O problema não é simplesmente transportar o código de uma aplicação. O problema é conseguir reproduzir o **ambiente necessário para executar esse código**.

Imagine uma aplicação que funciona no computador de uma pessoa, mas apresenta erro no computador de outra. Isso pode acontecer porque existem diferenças em:

- sistema operacional;
- versões de bibliotecas;
- dependências;
- configurações;
- versões de ferramentas;
- recursos disponíveis.

É daí que surge a famosa situação:

> **"Na minha máquina funciona."**

Na prática, isso significa que o ambiente em que a aplicação foi desenvolvida não está sendo reproduzido corretamente em outro lugar.

Além disso, existem outros problemas:

- duas aplicações podem precisar de versões diferentes da mesma biblioteca;
- um serviço com problema não deveria derrubar os demais;
- aplicações precisam ser iniciadas e encerradas rapidamente;
- servidores precisam aproveitar melhor seus recursos;
- aplicações precisam poder ser replicadas com facilidade.

A containerização tenta resolver justamente esse conjunto de problemas.

---

# 2. Servidor físico, máquina virtual e container

Existem diferentes maneiras de executar aplicações de forma isolada.

## 2.1 Servidor físico

No modelo mais simples, uma aplicação pode ficar diretamente em uma máquina física.

### Vantagens

- isolamento físico;
- acesso direto aos recursos da máquina.

### Problemas

- preparar uma máquina pode demorar;
- parte dos recursos pode ficar ociosa;
- manter uma máquina para cada aplicação pode ser caro;
- escalar rapidamente é mais difícil.

---

## 2.2 Máquina virtual

Uma máquina virtual, ou **VM**, utiliza um **hipervisor** para virtualizar o hardware.

Cada máquina virtual possui seu próprio sistema operacional e seu próprio kernel.

A estrutura pode ser pensada assim:

```text
Hardware
   ↓
Hipervisor
   ↓
Sistema operacional da VM
   ↓
Aplicação
```

Cada VM funciona como uma máquina separada.

### Benefícios

- isolamento forte;
- possibilidade de executar sistemas operacionais diferentes;
- maior separação entre ambientes;
- útil quando existe necessidade de um kernel específico.

### Desvantagens

Cada VM precisa carregar seu próprio sistema operacional.

Isso significa:

- mais memória;
- mais espaço em disco;
- inicialização mais lenta;
- maior custo de recursos.

---

# 3. O que é um container?

Um container **não é uma máquina virtual pequena**.

Ele é, essencialmente, um ou mais **processos executando no kernel do host**, mas com uma visão restrita do sistema e com limites de recursos.

O material define o container como um processo que possui:

- visão restrita por **namespaces**;
- consumo controlado por **cgroups**;
- sistema de arquivos formado por **camadas de imagem**.

Uma forma simples de pensar é:

```text
Sistema operacional
        ↓
     Docker
        ↓
 ┌───────────────┐
 │   Container   │
 │               │
 │   Processo    │
 └───────────────┘
```

O processo continua sendo um processo do sistema operacional.

A diferença é que ele possui determinadas restrições.

---

# 4. Container x máquina virtual

A principal pergunta para diferenciar os dois é:

> **O que está sendo virtualizado?**

### Máquina virtual

Virtualiza o **hardware**.

Cada VM possui seu próprio sistema operacional e kernel.

### Container

Utiliza o **kernel do sistema operacional do host** e restringe a visão e o consumo do processo.

| Característica | Máquina Virtual | Container |
|---|---|---|
| Virtualiza | Hardware | Sistema operacional/ambiente de execução |
| Kernel | Próprio | Compartilhado com o host |
| Sistema operacional | Cada VM possui um | Compartilhado |
| Inicialização | Mais lenta | Muito rápida |
| Consumo | Maior | Menor |
| Imagem | Geralmente maior | Geralmente menor |
| Isolamento | Mais forte | Depende da configuração |

Containers costumam iniciar em milissegundos ou segundos, enquanto máquinas virtuais podem levar dezenas de segundos ou minutos.

---

# 5. Quando usar VM em vez de container?

Container não substitui máquina virtual em todos os cenários.

Uma VM continua sendo importante quando, por exemplo:

- é necessário executar um kernel diferente;
- existe necessidade de isolamento mais forte;
- uma aplicação depende diretamente de hardware;
- são necessários módulos específicos do kernel.

Na prática, os dois podem ser usados juntos.

Um cenário comum em nuvem é:

```text
Máquina física
      ↓
Máquina virtual
      ↓
Docker
      ↓
Containers
```

Assim, os containers compartilham o kernel da máquina virtual.

---

# 6. As três bases da containerização

Existem três conceitos fundamentais para entender o que realmente acontece dentro de um container:

1. **Namespaces**
2. **cgroups**
3. **Sistema de arquivos em camadas**

Uma forma simples de lembrar:

> **Namespaces → o que o processo consegue enxergar.**

> **cgroups → quanto o processo pode consumir.**

> **Camadas → quais arquivos o processo enxerga.**

---

# 7. Namespaces

Namespaces são recursos do kernel Linux utilizados para **isolar a visão que um processo possui do sistema**.

Isso permite que processos dentro de containers tenham a impressão de estar em um ambiente separado.

Por exemplo, um container pode possuir:

- seus próprios processos;
- seu próprio hostname;
- sua própria rede;
- seus próprios pontos de montagem.

Os principais namespaces apresentados são:

| Namespace | O que isola |
|---|---|
| `pid` | Árvore de processos |
| `mnt` | Pontos de montagem |
| `net` | Interfaces, IPs e portas |
| `uts` | Hostname |
| `ipc` | Comunicação entre processos |
| `user` | Usuários e grupos |
| `cgroup` | Hierarquia de cgroups |
| `time` | Determinados relógios do sistema |

---

## 7.1 Namespace PID

O namespace `pid` isola a árvore de processos.

Isso permite que o processo principal do container seja visto como:

```text
PID 1
```

dentro do container.

Porém, no host, o mesmo processo possui outro PID.

Ou seja:

```text
Dentro do container:
PID 1

No host:
PID 12345
```

Não são dois processos diferentes.

É o **mesmo processo sendo observado por perspectivas diferentes**.

---

## 7.2 Namespace de rede

O namespace `net` permite que o container tenha uma visão própria da rede.

Ele pode possuir:

- interfaces;
- endereços IP;
- portas;
- tabelas de roteamento.

Isso é fundamental para que diferentes containers possam executar serviços sem simplesmente compartilharem a mesma configuração de rede.

---

## 7.3 Namespace UTS

O namespace `uts` está relacionado principalmente ao hostname.

Por isso, um container pode possuir um nome de máquina diferente daquele visto no host.

---

## 7.4 Por que namespaces são importantes?

Sem namespaces, os processos de diferentes aplicações enxergariam muito mais do ambiente uns dos outros.

Os namespaces ajudam a criar a sensação de que cada container possui seu próprio ambiente.

Isso traz:

- isolamento;
- organização;
- menor interferência entre serviços;
- possibilidade de executar aplicações diferentes no mesmo host.

---

# 8. cgroups

Enquanto namespaces respondem:

> **"O que o processo pode enxergar?"**

os **cgroups** respondem:

> **"Quanto o processo pode consumir?"**

Cgroups, ou **control groups**, são recursos do kernel Linux utilizados para agrupar processos e aplicar:

- limites;
- contabilização;
- priorização de recursos.

---

# 9. Por que cgroups são necessários?

Imagine dois containers:

```text
Container A → aplicação normal
Container B → processo consumindo muita CPU e memória
```

Se não houver controle, o Container B pode consumir tantos recursos que prejudique o Container A.

Isso é conhecido como problema do **vizinho barulhento**.

Os cgroups ajudam a controlar isso.

---

# 10. Recursos controlados pelos cgroups

Entre os principais controladores estão:

### CPU

Controla a quantidade e o peso relativo de processamento.

### Memory

Define limites de memória.

### PIDs

Limita a quantidade de processos e threads.

### I/O

Controla operações e banda de entrada e saída.

### cpuset

Define em quais núcleos de CPU um grupo pode executar.

---

# 11. CPU x memória

É importante não confundir o comportamento desses dois limites.

## Limite de memória

A memória possui um limite rígido.

Quando o processo tenta ultrapassar o limite e o sistema não consegue recuperar memória suficiente, pode ocorrer um **OOM kill**.

Um container encerrado dessa maneira pode apresentar:

```text
exit code 137
```

## Limite de CPU

O comportamento é diferente.

O processo não é necessariamente encerrado.

O kernel pode aplicar **throttling**, reduzindo a quantidade de CPU disponível.

Na prática:

```text
Memória → pode causar encerramento
CPU → pode causar lentidão
```

Essa diferença é muito importante para diagnóstico.

---

# 12. Por que limitar recursos?

Definir limites traz benefícios importantes.

## Previsibilidade

O comportamento da aplicação sob carga fica mais controlado.

## Isolamento

Um serviço não consegue consumir indefinidamente os recursos utilizados pelos demais.

## Densidade

Mais serviços podem compartilhar a mesma máquina.

## Controle de custos

A máquina pode ser utilizada de forma mais eficiente.

## Diagnóstico

Os sintomas ajudam a entender o problema.

Por exemplo:

```text
Exit 137 → investigar memória

Lentidão sob carga → investigar CPU/throttling
```

---

# 13. Imagem, container e registry

Esses três conceitos são fundamentais no Docker.

## Imagem

A imagem é um **modelo somente leitura** utilizado para criar containers.

Podemos pensar nela como uma planta.

```text
Imagem
   ↓
Container
```

Uma mesma imagem pode gerar vários containers.

---

## Container

O container é uma **instância em execução criada a partir de uma imagem**.

A imagem descreve o ambiente.

O container representa uma execução desse ambiente.

Uma analogia útil:

```text
Imagem = classe
Container = objeto
```

---

## Registry

O registry é um repositório de imagens.

Um exemplo conhecido é o **Docker Hub**.

O fluxo pode ser:

```text
Registry
   ↓ docker pull
Imagem local
   ↓ docker run
Container
```

---

# 14. `docker pull` x `docker run`

Essa diferença é importante.

### `docker pull`

Baixa uma imagem.

Exemplo:

```bash
docker pull nginx:alpine
```

### `docker run`

Cria e inicia um novo container a partir de uma imagem.

Exemplo:

```bash
docker run nginx:alpine
```

Portanto:

```text
pull → obtém a imagem

run → cria uma execução
```

---

# 15. Imagens são formadas por camadas

Uma imagem Docker não precisa ser um único bloco gigante.

Ela é formada por **camadas**.

Por exemplo:

```text
Camada base
    ↓
Dependências
    ↓
Código
    ↓
Metadados
```

Um Dockerfile pode gerar mudanças que correspondem a diferentes camadas.

---

# 16. Por que usar camadas?

O principal benefício é o **reaproveitamento**.

Se uma camada já existe localmente e não mudou, ela pode ser reutilizada.

Isso ajuda a:

- reduzir downloads;
- acelerar builds;
- economizar espaço;
- reaproveitar partes comuns entre imagens.

Por isso, a organização do Dockerfile influencia o aproveitamento do cache.

---

# 17. Camada de escrita

As camadas da imagem são somente leitura.

Quando um container é iniciado, o Docker adiciona uma camada de escrita por cima.

Podemos representar assim:

```text
Camada de escrita
-----------------
Camada da aplicação
-----------------
Camada de dependências
-----------------
Camada base
```

Alterações feitas durante a execução ficam nessa camada de escrita.

Quando o container é removido, essas alterações são descartadas.

É justamente por isso que **volumes** são importantes quando precisamos preservar dados.

---

# 18. Dockerfile

O **Dockerfile** descreve como construir uma imagem.

Ele funciona como uma receita.

Em vez de configurar manualmente uma máquina toda vez, podemos registrar as etapas necessárias.

Isso aumenta a:

- reprodutibilidade;
- padronização;
- automação;
- facilidade de manutenção.

---

# 19. Principais instruções do Dockerfile

## `FROM`

Define a imagem base.

Exemplo:

```dockerfile
FROM python:3.12-slim
```

### Por que usar?

A aplicação não precisa começar do zero.

Uma imagem base já pode fornecer:

- sistema de arquivos;
- runtime;
- ferramentas necessárias.

---

## `WORKDIR`

Define o diretório de trabalho.

```dockerfile
WORKDIR /app
```

### Por que usar?

Evita precisar escrever caminhos completos constantemente e deixa explícito onde a aplicação será executada.

---

## `COPY`

Copia arquivos para dentro da imagem.

```dockerfile
COPY . .
```

### Por que usar?

É como colocar dentro da imagem os arquivos necessários para executar a aplicação.

---

## `RUN`

Executa comandos **durante a construção da imagem**.

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

### Por que usar?

É utilizado para preparar a imagem.

Por exemplo:

- instalar dependências;
- criar diretórios;
- configurar componentes necessários.

---

## `EXPOSE`

Documenta a porta utilizada pela aplicação.

```dockerfile
EXPOSE 5050
```

### Atenção

`EXPOSE` **não publica a porta no computador**.

Ele apenas documenta qual porta o processo utiliza.

Para publicar uma porta é necessário configurar o container, por exemplo com `-p`.

---

## `CMD`

Define o processo principal que será executado quando o container iniciar.

```dockerfile
CMD ["gunicorn", "--bind", "0.0.0.0:5050", "main:app"]
```

Uma distinção importante:

```text
RUN → durante o build

CMD → quando o container inicia
```

---

# 20. Ordem do Dockerfile e cache

A organização do Dockerfile pode afetar bastante o tempo de build.

Um padrão útil é copiar primeiro os arquivos de dependência:

```dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
```

Por quê?

Imagine que apenas o código da aplicação mudou.

As dependências continuam iguais.

Assim, o Docker pode reaproveitar a camada relacionada à instalação das dependências e reconstruir apenas o que mudou depois dela.

Isso torna o build mais rápido.

---

# 21. Construindo uma imagem

Depois de criar o Dockerfile, podemos construir a imagem:

```bash
docker build -t api-recados:1.0 .
```

Aqui:

- `build` → constrói a imagem;
- `-t` → define nome e tag;
- `api-recados` → nome;
- `1.0` → versão;
- `.` → contexto utilizado no build.

---

# 22. Tags

Tags ajudam a identificar versões de imagens.

Exemplo:

```text
api-recados:1.0
api-recados:1.1
api-recados:2.0
```

Isso é mais informativo do que depender apenas de:

```text
latest
```

Uma tag explícita facilita:

- testes;
- identificação da versão;
- reprodução;
- retorno para uma versão anterior.

---

# 23. Imagem não é container

Essa é uma das distinções mais importantes do Docker.

### Imagem

É o resultado do build.

```text
Imagem
→ modelo
→ somente leitura
→ pode ser enviada para um registry
```

### Container

É uma execução da imagem.

```text
Container
→ instância
→ possui estado
→ executa um processo
→ pode possuir portas e variáveis
```

Alterar um arquivo dentro do container **não altera a imagem original**.

---

# 24. Portas

Um container pode possuir uma aplicação escutando em determinada porta.

Porém, isso não significa automaticamente que essa porta estará acessível pelo computador.

É necessário publicar a porta.

Exemplo:

```bash
docker run -d --name recados -p 9100:5050 api-recados:1.0
```

A estrutura é:

```text
-p PORTA_DO_HOST:PORTA_DO_CONTAINER
```

Nesse caso:

```text
Computador
localhost:9100
      ↓
Container
porta 5050
```

Portanto:

> A porta da esquerda pertence ao computador.

> A porta da direita pertence ao container.

---

# 25. Attached x Detached

Existem duas formas comuns de executar um container.

## Attached

O terminal fica conectado ao processo.

Exemplo:

```bash
docker run -it --rm alpine sh
```

É útil quando queremos interagir diretamente com o container.

---

## Detached

O container executa em segundo plano.

```bash
docker run -d nginx:alpine
```

O terminal continua disponível.

Para observar o que está acontecendo, usamos logs:

```bash
docker logs web
```

---

# 26. Comandos essenciais para observar containers

Alguns comandos são especialmente importantes para diagnóstico.

### Ver containers em execução

```bash
docker ps
```

### Ver também containers parados

```bash
docker ps -a
```

### Ver logs

```bash
docker logs nome-do-container
```

### Entrar no container

```bash
docker exec -it nome-do-container sh
```

### Parar

```bash
docker stop nome-do-container
```

### Iniciar novamente

```bash
docker start nome-do-container
```

### Remover

```bash
docker rm nome-do-container
```

---

# 27. Logs: uma das primeiras ferramentas de diagnóstico

Quando uma aplicação não funciona, não devemos simplesmente começar alterando configurações.

Primeiro precisamos descobrir **o que está acontecendo**.

Os logs são uma das primeiras fontes de evidência.

Podemos perguntar:

1. O container está rodando?
2. O processo iniciou corretamente?
3. A aplicação apresentou algum erro?
4. A requisição chegou?
5. Existe algum problema de configuração?

Por isso:

```bash
docker logs
```

é um dos comandos mais importantes para trabalhar com Docker.

---

# 28. Volumes

A camada de escrita do container é temporária.

Se o container for removido, os dados armazenados apenas nessa camada também podem desaparecer.

Isso é um problema para dados que precisam sobreviver à vida do container.

Exemplos:

- banco de dados;
- arquivos enviados por usuários;
- dados gerados pela aplicação.

Para isso existem os **volumes**.

A ideia é:

```text
Container
    ↓
Volume
    ↓
Dados persistentes
```

O volume mantém os dados fora da camada gravável temporária do container.

---

# 29. Por que volumes são importantes?

Containers são frequentemente tratados como ambientes que podem ser destruídos e recriados.

Se os dados estiverem presos ao container, recriar o container pode significar perder esses dados.

Com volumes:

```text
Container A
     ↓
 Volume
     ↑
Container B
```

Podemos remover o container e criar outro utilizando os mesmos dados.

Isso é especialmente importante para serviços como bancos de dados.

---

# 30. Docker Compose

Uma aplicação real frequentemente possui mais de um serviço.

Por exemplo:

```text
Frontend
   ↓
Backend
   ↓
Banco de dados
```

Executar todos esses serviços manualmente com vários `docker run` pode ficar difícil de manter.

O **Docker Compose** resolve esse problema permitindo descrever os serviços em um único arquivo.

---

# 31. Dockerfile x Docker Compose

É importante não confundir as responsabilidades.

### Dockerfile

Responde:

> **Como construir a imagem da minha aplicação?**

### Docker Compose

Responde:

> **Como os diferentes serviços da aplicação devem ser executados juntos?**

Uma forma de memorizar:

```text
Dockerfile
→ construção da imagem

Compose
→ organização da execução
```

---

# 32. O que o Compose descreve?

Um arquivo Compose pode definir:

- serviços;
- imagens;
- builds;
- portas;
- variáveis de ambiente;
- volumes;
- dependências;
- configurações de execução.

Um exemplo conceitual:

```yaml
services:
  frontend:
    build: ./frontend
    ports:
      - "8080:80"

  backend:
    build: ./backend
```

A ideia principal não é decorar YAML.

É entender que o arquivo registra **como os serviços trabalham juntos**.

---

# 33. `services`

A seção `services` define os serviços da aplicação.

Por exemplo:

```yaml
services:
  frontend:
  backend:
```

Nesse caso existem dois serviços:

```text
frontend
backend
```

Cada serviço normalmente corresponde a um container.

---

# 34. `build`

Define onde está a configuração usada para construir a imagem.

```yaml
build: ./backend
```

Isso significa que o Compose deve construir a imagem a partir daquele diretório.

---

# 35. `image`

Também podemos utilizar uma imagem já existente.

Conceitualmente:

```yaml
image: nginx:alpine
```

Nesse caso, em vez de construir a imagem a partir de um Dockerfile local, o serviço utiliza a imagem especificada.

---

# 36. `ports`

Publica uma porta do container no computador.

Exemplo:

```yaml
ports:
  - "8080:80"
```

A lógica é a mesma do `docker run`:

```text
8080 → computador
80   → container
```

---

# 37. Rede interna do Compose

Uma das partes mais importantes do Compose é a comunicação entre serviços.

Suponha:

```text
frontend
backend
```

O frontend não precisa descobrir manualmente o IP do backend.

Dentro da rede do Compose, o nome do serviço pode funcionar como endereço.

Por exemplo:

```text
backend:5050
```

Assim:

```text
Frontend
   ↓
backend:5050
   ↓
Backend
```

Isso facilita muito a configuração da aplicação.

---

# 38. Porta publicada x porta interna

Esse conceito costuma causar confusão.

Imagine:

```yaml
frontend:
  ports:
    - "8080:80"
```

O usuário acessa:

```text
localhost:8080
```

Mas dentro do ambiente do serviço, o processo do frontend utiliza:

```text
porta 80
```

Agora imagine o backend:

```text
backend:5050
```

O frontend conversa com o backend usando a **porta interna** do backend.

O mapeamento:

```text
8080:80
```

não altera a porta utilizada para a comunicação interna entre os serviços.

---

# 39. Por que usar nomes de serviço?

Usar nomes de serviço é melhor do que configurar IPs manualmente.

Em vez de:

```text
http://192.168.x.x:5050
```

podemos utilizar:

```text
http://backend:5050
```

Isso traz mais flexibilidade porque o serviço pode ser recriado e seu IP pode mudar.

O nome do serviço continua sendo a referência usada pela aplicação.

---

# 40. Proxy reverso

Um cenário comum é colocar um Nginx na frente do backend.

Por exemplo:

```text
Navegador
   ↓
localhost:8080
   ↓
Frontend / Nginx
   ↓
backend:5050
```

O navegador conversa com o frontend.

O frontend encaminha determinadas requisições para o backend.

Por exemplo:

```text
/api/rotas
      ↓
backend:5050/rotas
```

Isso é útil porque o frontend pode funcionar como uma porta de entrada para a aplicação.

---

# 41. `depends_on`

O `depends_on` permite declarar uma dependência entre serviços.

Exemplo:

```yaml
depends_on:
  - backend
```

Isso registra que o frontend depende do backend.

### Atenção

`depends_on` sozinho **não significa que a aplicação já está pronta para receber requisições**.

Ele trata da ordem de inicialização, mas uma aplicação pode ainda estar iniciando internamente.

Quando a prontidão realmente importa, o material recomenda combinar isso com **healthcheck** e uma condição de dependência adequada.

---

# 42. `environment`

Variáveis de ambiente permitem passar configurações para o serviço.

Isso é importante porque alguns valores mudam entre ambientes.

Por exemplo:

```text
desenvolvimento
produção
testes
```

Em vez de criar uma imagem diferente para cada configuração, podemos manter a mesma imagem e alterar os valores fornecidos ao container.

Isso ajuda na separação entre:

```text
Código da aplicação
        +
Configuração do ambiente
```

---

# 43. Healthcheck

Um serviço pode estar com o container em execução e, mesmo assim, ainda não estar pronto para receber requisições.

Por isso existe o conceito de **healthcheck**.

A ideia é verificar:

> "O serviço está realmente saudável e pronto?"

Isso é diferente de simplesmente perguntar:

> "O processo foi iniciado?"

Essa distinção é importante em aplicações com múltiplos serviços.

---

# 44. Fluxo básico do Docker Compose

Um fluxo comum é:

```bash
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs -f
docker compose down
```

A lógica é:

```text
Validar
   ↓
Construir e iniciar
   ↓
Verificar estado
   ↓
Observar logs
   ↓
Encerrar
```

---

# 45. Quando usar `--build`?

Se o Dockerfile ou arquivos copiados para dentro da imagem mudaram, precisamos garantir que a imagem seja reconstruída.

Por isso:

```bash
docker compose up -d --build
```

é útil quando houve alterações na construção da imagem.

Sem reconstruir, podemos continuar executando uma imagem antiga.

---

# 46. Diagnóstico de uma aplicação com vários containers

Imagine:

```text
Frontend funciona
Backend não responde
```

Não devemos assumir imediatamente que o problema está no backend.

Podemos seguir uma sequência.

### 1. Verificar os serviços

```bash
docker compose ps
```

Pergunta:

> Todos os containers estão em execução?

---

### 2. Ver os logs

```bash
docker compose logs -f
```

Verifique frontend e backend.

---

### 3. Conferir a porta

Pergunte:

> Em qual porta o backend está escutando?

Depois:

> Qual porta o frontend está tentando acessar?

---

### 4. Conferir o nome do serviço

A comunicação interna deve utilizar o nome do serviço:

```text
backend:5050
```

e não necessariamente:

```text
localhost:5050
```

---

### 5. Testar em etapas

Primeiro:

```text
Backend
```

Depois:

```text
Frontend → Backend
```

E finalmente:

```text
Navegador → Frontend → Backend
```

Isso ajuda a descobrir exatamente em qual parte está o problema.

---

# 47. HTTP 502

Um erro `502` pode aparecer quando um proxy recebeu uma requisição, mas não conseguiu conversar corretamente com o serviço de destino.

Em uma arquitetura:

```text
Navegador
   ↓
Nginx
   ↓
Backend
```

um `502` pode indicar um problema na comunicação:

```text
Nginx ──X──> Backend
```

Por isso, ao encontrar `502`, vale conferir:

- nome do serviço;
- porta interna;
- serviço está rodando?;
- logs do backend;
- configuração do proxy.

---

# 48. O que containers NÃO isolam completamente?

É importante não tratar container como uma fronteira de segurança absoluta.

O kernel continua sendo compartilhado.

Isso significa que:

> Uma vulnerabilidade no kernel do host pode afetar os containers.

Além disso, configurações de usuário e privilégios importam.

O uso de:

```bash
--privileged
```

remove parte importante das proteções e, por isso, não deve ser utilizado sem necessidade.

---

# 49. Root dentro do container

Também é importante entender que `root` dentro de um container não significa automaticamente isolamento perfeito.

Dependendo da configuração, um processo privilegiado pode representar riscos maiores.

Por isso, segurança em containers depende de várias decisões:

- privilégios;
- namespaces;
- usuários;
- capacidades;
- configuração do host;
- configuração do runtime.

A ideia principal é:

> **Isolamento não é uma propriedade automática simplesmente porque algo está em um container.**

---

# 50. O modelo mental completo

Agora podemos juntar tudo.

Imagine uma aplicação web com frontend e backend.

```text
                 COMPUTADOR
                     │
             ┌───────┴───────┐
             │     Docker    │
             └───────┬───────┘
                     │
       ┌─────────────┴─────────────┐
       │                            │
 ┌─────▼─────┐                ┌─────▼─────┐
 │ Frontend  │                │  Backend  │
 │ Container │                │ Container │
 └─────┬─────┘                └─────┬─────┘
       │                            │
       └──────── rede ──────────────┘
```

Cada container possui:

```text
Namespaces
    ↓
o que o processo enxerga

cgroups
    ↓
quanto o processo pode consumir

Camadas da imagem
    ↓
arquivos disponíveis
```

E a imagem é construída por um:

```text
Dockerfile
```

Enquanto o conjunto de serviços é organizado pelo:

```text
Docker Compose
```

---

# 51. Fluxo completo de uma aplicação Dockerizada

Podemos visualizar o processo inteiro:

```text
Dockerfile
    ↓
docker build
    ↓
Imagem
    ↓
Registry (opcional)
    ↓
docker pull
    ↓
docker run / docker compose
    ↓
Container
    ↓
Namespaces + cgroups + filesystem
    ↓
Aplicação em execução
```

Para aplicações com vários serviços:

```text
Dockerfiles
    ↓
Imagens
    ↓
Docker Compose
    ↓
Frontend + Backend + Banco
    ↓
Rede interna
    ↓
Aplicação completa
```

---

# 52. Por que Docker é útil?

Docker não é apenas uma ferramenta para "rodar containers".

O principal benefício está na **padronização do ambiente de execução**.

Isso pode ajudar a diminuir problemas como:

> "No meu computador funciona."

O ambiente necessário para executar a aplicação passa a ser descrito de forma mais explícita.

Isso melhora:

### Reprodutibilidade

O mesmo ambiente pode ser reconstruído.

### Portabilidade

A aplicação pode ser executada em diferentes máquinas que possuem suporte adequado ao Docker.

### Isolamento

Serviços podem possuir ambientes separados.

### Escalabilidade

É relativamente simples criar novas instâncias de um serviço.

### Padronização

O time pode compartilhar a mesma definição de ambiente.

### Automação

Builds e execuções podem ser reproduzidos sem configurar tudo manualmente.

---

# 53. Por que não colocar tudo em um único container?

Uma aplicação pode possuir diversos componentes:

```text
Frontend
Backend
Banco
Worker
Cache
```

Separar os componentes em serviços diferentes pode trazer vantagens.

Por exemplo:

```text
Frontend → escala separadamente
Backend  → escala separadamente
Worker   → escala separadamente
```

Além disso, cada serviço pode possuir responsabilidades mais claras.

O Compose ajuda a representar essa arquitetura.

---

# 54. Conceitos que mais merecem atenção

Se você estiver estudando para uma prova ou avaliação, estes são alguns dos conceitos que precisam estar muito claros:

## 1. Container não é VM

Container compartilha o kernel do host.

VM possui seu próprio sistema operacional/kernel.

---

## 2. Namespace não é cgroup

```text
Namespace → o que pode enxergar

cgroup → quanto pode consumir
```

---

## 3. Imagem não é container

```text
Imagem → modelo

Container → execução
```

---

## 4. Dockerfile não é Compose

```text
Dockerfile → constrói imagem

Compose → organiza serviços
```

---

## 5. `EXPOSE` não publica porta

`EXPOSE` documenta a porta.

A publicação acontece através da configuração de portas, como:

```text
8080:80
```

---

## 6. `RUN` não é `CMD`

```text
RUN → build

CMD → execução
```

---

## 7. Porta externa não é porta interna

```text
8080:80
```

significa:

```text
host:8080
   ↓
container:80
```

---

## 8. `depends_on` não significa prontidão

Ele organiza a dependência/inicialização, mas não garante sozinho que o serviço já esteja pronto.

---

## 9. Dados importantes não devem depender da camada de escrita

Para dados persistentes, utilize volumes.

---

# 55. Tabela de conceitos essenciais

| Conceito | Pergunta que responde | Por que existe? |
|---|---|---|
| Container | Onde a aplicação executa? | Isolar e padronizar execução |
| Namespace | O que o processo enxerga? | Isolamento de visão |
| cgroup | Quanto o processo pode consumir? | Controle de recursos |
| Imagem | Qual ambiente será executado? | Criar containers reproduzíveis |
| Camada | O que mudou na imagem? | Cache e reutilização |
| Dockerfile | Como construir a imagem? | Automatizar e padronizar build |
| Registry | Onde guardar imagens? | Distribuição de imagens |
| Volume | Onde ficam dados persistentes? | Sobreviver à remoção do container |
| Porta | Como acessar o serviço? | Comunicação externa |
| Compose | Como os serviços trabalham juntos? | Orquestrar aplicação local/multi-serviço |
| `depends_on` | Qual serviço depende de outro? | Declarar dependências |
| `environment` | Qual configuração varia? | Separar configuração da imagem |
| Healthcheck | O serviço está pronto? | Verificar saúde/prontidão |
| Logs | O que está acontecendo? | Diagnóstico |

---

# 56. Checklist final de estudo

Antes de considerar o conteúdo dominado, tente responder sem consultar o material:

- [ ] Por que containers existem?
- [ ] Qual problema o "na minha máquina funciona" representa?
- [ ] Qual é a diferença fundamental entre VM e container?
- [ ] O que é o kernel?
- [ ] O que são namespaces?
- [ ] O que o namespace `pid` faz?
- [ ] O que o namespace `net` faz?
- [ ] O que são cgroups?
- [ ] Qual a diferença entre limitar CPU e memória?
- [ ] O que significa exit code 137?
- [ ] O que é uma imagem?
- [ ] O que é um container?
- [ ] O que é um registry?
- [ ] Por que imagens possuem camadas?
- [ ] Por que o cache do Docker é importante?
- [ ] O que é Dockerfile?
- [ ] Para que servem `FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE` e `CMD`?
- [ ] Qual a diferença entre `RUN` e `CMD`?
- [ ] Por que copiar dependências antes do código pode acelerar o build?
- [ ] Qual a diferença entre `docker pull` e `docker run`?
- [ ] Qual a diferença entre attached e detached?
- [ ] Como verificar os logs de um container?
- [ ] Por que volumes existem?
- [ ] Qual a diferença entre Dockerfile e Docker Compose?
- [ ] O que é um service no Compose?
- [ ] Para que serve `ports`?
- [ ] Como um serviço encontra outro dentro do Compose?
- [ ] Qual a diferença entre porta interna e porta publicada?
- [ ] Para que serve `depends_on`?
- [ ] Por que `depends_on` não garante prontidão?
- [ ] Para que servem variáveis de ambiente?
- [ ] O que é healthcheck?
- [ ] Como investigar um HTTP 502?
- [ ] Por que container não deve ser tratado como uma VM?
- [ ] Quais são os limites do isolamento de containers?

---

# 57. Resumo em uma frase

Se fosse necessário resumir todo o conteúdo:

> **Docker permite empacotar aplicações em imagens e executá-las como containers, utilizando recursos do kernel como namespaces para controlar o que os processos enxergam, cgroups para controlar o que podem consumir e sistemas de arquivos em camadas para montar seu ambiente de execução; Dockerfile define como construir as imagens e Docker Compose define como vários serviços devem ser executados e conectados.**

---

# 58. Mapa mental

```text
                         DOCKER
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   CONTAINER            IMAGEM             COMPOSE
        │                  │                  │
        │              Dockerfile          Services
        │                  │                  │
        │          ┌───────┼───────┐     ┌────┼────┐
        │          │       │       │     │    │    │
        │         FROM    RUN     CMD  Front API  DB
        │
   ┌────┴─────────────┐
   │                  │
Namespaces          cgroups
   │                  │
   │                  ├── CPU
   ├── PID            ├── Memory
   ├── Network        ├── PIDs
   ├── Mount          └── I/O
   ├── Hostname
   └── Users
        │
        ▼
  Isolamento da visão

             + Camadas de filesystem
                       │
                       ▼
               Ambiente da aplicação

             + Volumes
                       │
                       ▼
                 Dados persistentes
```

---

## Conclusão

O mais importante é não decorar os comandos isoladamente.

O ideal é entender a relação entre eles:

```text
Problema de ambiente
        ↓
Containerização
        ↓
Imagem
        ↓
Dockerfile
        ↓
Container
        ↓
Namespaces + cgroups + camadas
        ↓
Portas + volumes
        ↓
Docker Compose
        ↓
Serviços comunicando pela rede
        ↓
Logs + diagnóstico
```

Quando esse fluxo faz sentido, os comandos deixam de ser uma lista para decorar e passam a representar ações dentro de um modelo mental.

O material do primeiro encontro coloca justamente essa base: entender o problema da containerização, diferenciar VM de container e compreender namespaces, cgroups e camadas antes de avançar para Dockerfile, volumes e composição de serviços. fileciteturn0file0L12-L19

O guia seguinte complementa essa base com imagens, Dockerfile, execução de containers, portas, logs e Docker Compose, reforçando que o objetivo é primeiro entender as decisões e só depois os comandos. fileciteturn0file1L28-L35
