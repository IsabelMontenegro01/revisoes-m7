# Do perceptron à LSTM/GRU: o fio da meada

> Resumo do raciocínio do prof. Murilo (transcrição), amarrado ao material do prof. Jeff (slides e1–e3, notebooks e1–e4 e os HTMLs interativos da célula LSTM e GRU).
> A ideia central: **cada arquitetura nasceu para resolver um problema que a anterior não resolvia.**

---

## 0. O mapa em uma frase

```
Perceptron  →  MLP (+ backprop)  →  CNN  →  RNN  →  LSTM / GRU  →  (Transformer, vem depois)
  1 neurônio     resolve XOR       reduz      ganha       resolve o        "estamos construindo
  só separa      mas treina        dimensão   memória     esquecimento     os bloquinhos até lá"
  o linear       com gradiente     de imagem  (tempo)     da RNN
```

O professor começa com a pergunta: *"RNN, LSTM e GRU são redes neurais de quê? Recorrentes. O que faz uma rede ser recorrente? Ela tem memória."* A partir daí ele volta ao começo para mostrar **por que** chegamos nelas.

---

## 1. Rede neural = aproximador

- Uma rede neural é um **aproximador de funções** (o "neurônio universal"). A inspiração é o neurônio biológico: representar matematicamente como ele funciona.
- Um **único neurônio** (perceptron) calcula uma soma ponderada + viés e passa por uma ativação. Ele traça **uma reta** no espaço.

&emsp;**O que ele resolve:** problemas **linearmente separáveis** (classe A de um lado da reta, classe B do outro). Dá para descrever **AND** e **OR**.

&emsp;**O que ele não resolve:** o **XOR**. Não existe uma reta única que separe as classes; seriam necessárias mais de uma. Esse limite foi o que "quebrou" as redes neurais no início.

---

## 2. Rede neural de várias camadas (MLP) e backpropagation

&emsp;**Solução do XOR:** em vez de um neurônio, **várias camadas de neurônios** (e ativações não lineares). Várias retas combinadas separam o que uma só não separa.

&emsp;**Novo problema:** com vários neurônios, é preciso **treinar**. Cada conexão tem um **peso**.

&emsp;**Solução:** o erro da saída é propagado **de trás para frente**, atualizando os pesos: **backpropagation**. A técnica de cálculo por trás é o **gradiente descendente**: reduzir o erro ajustando cada peso na direção que mais o diminui.

> 🔑 Guarde: **peso → erro → gradiente → atualização de peso.** Tudo que vem depois (vanishing/exploding gradient, LSTM) gira em torno de *o gradiente conseguir ou não chegar aos pesos certos*.

---

## 3. Por que a MLP não bastou: dimensionalidade → CNN

- A MLP reconhece bem coisas simples (o professor cita caracteres), mas ao receber **uma foto**, cada pixel vira uma entrada. A rede **explode em dimensionalidade**: fica gigantesca.
- Exemplo do notebook e1 (CIFAR-10): uma imagem 32×32×3 tem **3.072 números**; a MLP do notebook tem **~1,7 milhão de parâmetros**.

&emsp;**Solução: convoluções (CNN).**
- Cada **filtro** extrai uma **característica** da imagem (bordas, cores…). Um filtro 3×3×3 tem só 27 pesos + 1 viés, e **desliza pela imagem inteira** reaproveitando os mesmos pesos.
- No final dos blocos convolucionais, liga-se uma rede densa (MLP) já com dimensão reduzida.
- Resultado no notebook: a CNN tem **~357 mil parâmetros** (≈ 5× menos que a MLP) e ainda **acerta mais**.

---

## 4. O que nem MLP nem CNN fazem: lidar com **tempo**

&emsp;O ponto-chave do professor: *"nenhum desses consegue lidar com informação de tempo"*. Para a rede, uma previsão não tem ligação com a outra. Exemplo dele: prever "isso é um quadro branco" e depois "isso está encostado"… sem relação entre as duas previsões.

&emsp;Quando a entrada é **uma série temporal** (ou qualquer coisa que se expande ao longo do tempo), queremos que **a primeira previsão esteja ligada à segunda**. É aí que entram as **redes recorrentes**.

&emsp;**Exemplo dos slides do prof. Jeff (e2):** dataset de fraude com `m1, m2, m3`. Numa rede feedforward, a **ordem das entradas não faz diferença** (`[52,51,4]` e `[4,51,52]` entram nas mesmas "gavetas" com pesos fixos; o slide pergunta justamente "a ordem das entradas faz diferença?"). Para dados em que a ordem importa, precisamos de outra estrutura.

---

## 5. RNN: a rede com memória

&emsp;**Ideia básica (slide e2):** usar **loops** para passar informação de uma etapa para a próxima. A saída/estado do passo anterior volta como entrada do passo seguinte.

Na fórmula do notebook e2:

```
h_t = tanh( W_h · h_(t-1)  +  W_x · x_t  +  b )
```

- `x_t` = entrada atual, `h_(t-1)` = estado (memória) anterior.
- Os **mesmos pesos** `W_h` e `W_x` são **reaplicados em todos os passos** (o professor diz "aplicar os pesos de forma recursiva").
- Exemplo do slide: **previsão de consumo de água (m³)**. A rede "desenrolada" no tempo (t1, t2, t3…) usa os mesmos pesos em cada passo, e a previsão em t3 depende do que ela viu em t1 e t2.

&emsp;**O que a RNN resolve:** dá à rede **memória** → serve para séries temporais e sequências.

---

## 6. O problema da RNN: por que ela "esquece"

### 6.1 A pergunta que o experimento quer responder

&emsp;Os notebooks e os HTMLs usam sempre a mesma tarefa de brinquedo: uma sequência de `a` e `b` com um marcador `M`, e a rede deve responder **qual símbolo estava `gap` posições antes do `M`**.

```
b b a b a M        gap = 3 → resposta: a
    ↑     ↑
   alvo   M
```

&emsp;Essa tarefa foi escolhida porque **isola uma única dificuldade: lembrar de algo que ficou para trás**. Não há padrão estatístico para "adivinhar"; a informação necessária está na entrada, só que **longe da hora de responder**. O parâmetro `gap` funciona como um botão de dificuldade: aumenta só a distância.

> **Objetivo do experimento:** descobrir a partir de que distância a RNN deixa de conseguir usar uma informação que, comprovadamente, está na entrada.

&emsp;**Resultado (notebook e2):** gaps de 1 a 10 → 100%; gap 20 → ~96%; gap 40 → ~77%; e a tendência é cair até 50% (que é o chute, pois são só duas respostas possíveis).

&emsp;**Conclusão:** o problema da RNN **não é falta de informação**, é que a informação **não sobrevive ao caminho** até a saída. Por isso o notebook pede no final: explique por que a RNN falha mesmo com a informação presente.

### 6.2 Por que a informação não sobrevive

&emsp;A RNN, a cada passo, faz `h_t = tanh(W_h·h_(t-1) + W_x·x_t + b)`. Ou seja, **a memória é reescrita por inteiro a cada passo**, passando pela mesma matriz `W_h` e por um `tanh`.

&emsp;O slide "Por que a memória esvanece?" mostra a versão simplificada: a cada passo a memória antiga é cortada pela metade (a → a/2 → a/4 → a/8…). Depois de 8 passos, o primeiro `a` vale menos de 1% do original. Está lá, mas é ruído perto das entradas recentes.

&emsp;**Moral:** se a cada passo você multiplica a memória por um número menor que 1, **a informação antiga decai exponencialmente com a distância**. A RNN não tem como dizer "este valor aqui eu quero guardar intacto".

### 6.3 O mesmo problema visto pelo treinamento (gradiente)

&emsp;Aqui entra o gradiente (seção 2). Para ajustar os pesos, o erro da saída precisa **voltar no tempo**, passo a passo (isso se chama BPTT). A cada passo para trás, o gradiente é multiplicado de novo pela mesma matriz de pesos.

| Fator por passo | Efeito após muitos passos | Consequência |
|---|---|---|
| **< 1** | o gradiente **some** (vanishing). Com 0,5, dez passos → ≈ 0,001 | os passos antigos quase não recebem sinal de erro → a rede **nunca aprende** que o passo distante importa |
| **> 1** | o gradiente **explode**. Com 1,5, dez passos → ≈ 57,7; no experimento controlado, de 2,6 para ~10⁹ | atualizações gigantes, perda oscilando, `NaN` |
| **≈ 1** | gradiente estável | funciona, mas é um equilíbrio instável; a RNN não tem garantia de ficar aí |

&emsp;**Ligação com o esquecimento da 6.2:** são **a mesma coisa vista de dois lados**. Na execução (forward), a informação antiga é atenuada; no treino (backward), o sinal de erro que deveria ensinar a rede a **guardar** essa informação também é atenuado. Pior: a rede precisa do gradiente justamente para aprender a lembrar, e é o gradiente que desaparece. Por isso ela não só esquece, como **não consegue aprender a não esquecer**.

&emsp;**O que cada experimento do notebook e2 prova:**
- *Recorrência escalar* (`h_t = α·h_(t-1)`): mostra a causa matemática, bem simples. É só α elevado a T.
- *Gradiente por passo de tempo*: mostra que, na RNN real, o gradiente é **muito menor nos passos distantes de `M`**. É a evidência de que o 6.3 acontece de fato, e não só na teoria.
- *Gradiente explosivo controlado*: mostra o outro lado (fator > 1) sem depender de sorte na inicialização.
- *Gradient clipping*: mostra que explosão tem remédio simples (limitar a norma, mantendo a direção), **mas** o clipping não resolve o evanescente e não dá memória à rede. Ou seja, **o problema do esquecimento precisa de uma mudança na arquitetura**, e é isso que a LSTM faz.

---

## 7. LSTM: a ideia, porta por porta

### 7.1 O que o professor disse (e uma nota de precisão)

&emsp;O professor usa a analogia dos **máximos locais**: o treinamento pode "ficar preso" num pico que não é o melhor, e LSTM/GRU trariam um efeito de "janela deslizante", considerando só um intervalo relevante.

> ⚠️ **Nota para estudo:** a analogia é útil como imagem, mas tecnicamente o problema que a LSTM ataca é o **gradiente evanescente/explosivo** (seção 6.3), não máximos locais. E "janela deslizante" não é literal: a LSTM não corta uma janela fixa; ela tem **portas aprendidas** que decidem, a cada passo, o que manter, o que descartar e o que mostrar. Vale confirmar com o professor como ele quer que isso apareça na atividade.

### 7.2 O raciocínio de projeto

&emsp;A pergunta de projeto é: **como fazer a rede decidir o que guardar sem que a informação seja destruída a cada passo?** A LSTM responde com duas decisões:

1. **Separar duas memórias**, com papéis diferentes:
   - `c` (**longo prazo**): o "caderno". Só é alterado de forma controlada.
   - `h` (**curto prazo / saída**): o que a célula **mostra** neste passo para a próxima camada e para o próximo passo.
2. **Colocar portas** (valores entre 0 e 1 que funcionam como torneiras) para controlar o que entra, o que fica e o que sai do caderno. As portas são **calculadas pela própria rede** a cada passo, a partir do símbolo lido e de `h` anterior, e **aprendidas** no treino.

### 7.3 Cada peça e por que ela existe

| Peça | Pergunta que responde | Por que é necessária |
|---|---|---|
| **Esquecimento `f`** (0 a 1) | "Quanto do caderno antigo eu mantenho?" | Sem ela, o caderno enche de lixo; com ela a rede pode **manter por muito tempo (f≈1)** ou **limpar (f≈0)** quando um novo contexto começa |
| **Candidato `g`** (−1 a +1) | "O que eu **poderia** escrever agora?" | É a proposta de nova informação, calculada do símbolo atual e de `h` anterior |
| **Entrada `i`** (0 a 1) | "Dessa proposta, quanto eu **realmente** escrevo?" | Separa **propor** de **aceitar**. Evita que qualquer símbolo irrelevante sobrescreva o caderno |
| **Saída `o`** (0 a 1) | "Do caderno, quanto eu **mostro** agora?" | Permite guardar algo por muito tempo **sem** que isso interfira na resposta até a hora certa |

&emsp;Em fórmula (um passo):

```
c_t = f · c_(t-1)  +  i · g        ← atualiza o caderno
h_t = o · tanh(c_t)                 ← decide o que mostrar
```

### 7.4 Por que isso resolve o problema da RNN (a parte mais importante)

&emsp;Compare as duas atualizações:

- **RNN:** a memória passa por **matriz + tanh** a cada passo → o sinal (e o gradiente) é **multiplicado repetidamente** → some ou explode.
- **LSTM:** `c_t = f·c_(t-1) + i·g`. O caminho do caderno é uma **soma** com uma multiplicação apenas por `f`. Se a rede aprende `f ≈ 1`, o conteúdo de `c` e o gradiente que volta por ele **atravessam muitos passos quase intactos**.

&emsp;Em outras palavras: a RNN **não consegue escolher** não esquecer; a LSTM **consegue**, porque o esquecimento virou um parâmetro que a rede controla. Ela troca "esquecer sempre um pouco" por "esquecer só quando decidir".

&emsp;Detalhe de treino que reforça isso: o HTML inicializa o **viés do esquecimento em 1,0** (`vies_esquecimento: 1.0`). Isso faz a porta começar **aberta** (mantendo a memória) e a rede só precisa aprender **quando fechar**. É mais fácil aprender a esquecer do que aprender a lembrar do zero.

### 7.5 Aplicando à tarefa: o que a rede precisa fazer

&emsp;Raciocínio que ajuda a entender o experimento: a rede **não sabe quando o `M` vai chegar**. Logo, ela precisa **ir guardando continuamente os últimos `gap` símbolos** e, quando o `M` aparece, **escolher o que estava `gap` posições atrás** e **congelar** essa informação até o fim da sequência (porque ainda chegam símbolos depois do `M`, que devem ser ignorados).

&emsp;Isso explica o critério que o HTML usou para escolher o treino a exibir: "entre as que atingiram o alvo, maior legibilidade (**f − i sobe após M**)". Traduzindo: após o `M`, a porta de esquecimento fica alta (**mantém**) e a de entrada fica baixa (**bloqueia**), ou seja, o caderno é **congelado**. É a leitura mais clara de como as portas resolvem o problema. (Em cada neurônio treinado isso pode aparecer de forma diferente; o HTML escolhe a execução em que fica mais legível.)

### 7.6 Armadilhas de interpretação (e a lição de cada uma)

&emsp;O notebook e3 monta pequenos testes de uma única unidade, sem treino, para "pegar" erros comuns de raciocínio:

| Armadilha | O que o teste mostra | Lição |
|---|---|---|
| "0,9 por passo é alto, então guarda bem" | com 0,9 por passo: 10 passos → ≈ 35%; 30 passos → ≈ 4% | **Pequenas perdas se acumulam.** É exatamente o efeito "a/2, a/4, a/8" da RNN, só que agora a rede pode escolher f≈1 |
| "Saída zero = memória apagada" | com `o = 0`, `h = 0`, mas `c` continua guardado | `o` controla **o que aparece**, não o que existe. Permite **guardar sem se manifestar** |
| "Candidato = informação que entra" | candidato 0,8 com `i` = 0 / 0,5 / 1 → entra 0 / 0,4 / 0,8 | `g` é só proposta; **quem decide é `i`** |
| "Se f = 1 a memória não diminui" | com `f=1`, `i=1` e candidato −0,2, a memória cai de 1 até ≈ 0 em 5 passos | A queda vem da **informação nova negativa**, não de esquecimento. `c` = parte mantida **+** parte nova |
| "Memória negativa = lembrança negativa" | `c` pode chegar a ≈ −1 | `c` é um **valor interno**, não uma porcentagem |
| "Trocar RNN por LSTM resolve" | na execução de referência do e3 (gap 40, 10 épocas), a LSTM começou perto de 50% na validação, enquanto a RNN já estava acima | **As portas também precisam ser aprendidas**; a arquitetura dá a *possibilidade*, o treino (épocas, capacidade) precisa torná-la realidade |
| "Mais unidades = sequência maior" | unidades são o tamanho do estado; o comprimento vem dos dados | Não confundir **capacidade da rede** com **tamanho do problema** |

### 7.7 Por que a conta de pesos tem "× 4"

&emsp;Cada uma das 4 peças (f, i, g, o) tem **seus próprios pesos**: um para o símbolo de entrada, um para cada unidade de `h` anterior e um viés. Para `U` unidades e `S` sinais: `4 × [U·(S+U) + U]`. Com 32 unidades e 9 sinais: 4 × 1.344 = **5.376**. A LSTM é mais "cara" que a RNN exatamente porque tem as portas.

---

## 8. O que os experimentos gap × neurônios concluem (HTML "LSTM por dentro")

### 8.1 O que o HTML é e para que serve

&emsp;É uma LSTM **treinada de verdade** (os pesos de várias épocas estão guardados na página) com três telas que acompanham o raciocínio:
1. **Problema:** mostra a tarefa e o gap (o "porquê" da rede).
2. **Treinamento:** mostra a rede aprendendo, época a época.
3. **Por dentro:** abre um neurônio e mostra `c`, `h` e as portas reagindo a cada símbolo (o "como" da rede).

&emsp;O valor dele é **conectar a fórmula do slide a um comportamento observável**: dá para ver a porta de esquecimento subir, a de entrada fechar, e o caderno `c` congelar.

### 8.2 Os resultados gravados

&emsp;(✅ = atingiu 99% na validação, com a época em que aprendeu; ❌ = não atingiu em 300 épocas)

| gap | 1 neurônio | 2 neurônios | 4 neurônios | 8 neurônios |
|---|---|---|---|---|
| 1 | ✅ (ép. 150) | ✅ (24) | ✅ (16) | ✅ (10) |
| 2 | ❌ 67% | ✅ (41) | ✅ (31) | ✅ (18) |
| 3 | ❌ 61% | ❌ 75% | ✅ (61) | ✅ (26) |
| 4 | ❌ 65% | ❌ 81% | ✅ (177) | ✅ (43) |

### 8.3 O que concluir
1. **Maior distância → mais difícil.** Dentro de cada coluna, o aprendizado fica mais lento ou deixa de acontecer. Mesma conclusão da RNN (seção 6), mas agora a LSTM empurra o limite para longe.
2. **Mais neurônios → aprende mais rápido e resolve gaps maiores.** Com 8 neurônios, o gap 1 aprende em 10 épocas e todos os gaps testados são resolvidos.
3. **Padrão notável:** nas colunas de 1, 2 e 4 neurônios, o **maior gap resolvido coincide com o número de neurônios** (1→1, 2→2, 4→4). Isso é compatível com o raciocínio da 7.5: a rede precisa guardar uma janela móvel dos últimos `gap` símbolos, e cada neurônio funciona como **um "espaço" de memória**. É uma **hipótese coerente com os dados**, não uma prova (os valores guardados são contínuos e não só 0/1, então a regra é aproximada; por exemplo, a GRU de 2 neurônios resolveu o gap 3, ver seção 9).
4. **Aprender é lento perto do limite.** 1 neurônio resolve o gap 1, mas só na época 150; 4 neurônios resolvem o gap 4, mas só na época 177. Perto do limite de capacidade, o gradiente leva muito mais tempo para achar a solução.
5. **Lição geral:** a LSTM **permite** resolver o problema, mas o resultado depende de **capacidade + distância + tempo de treino**. Isso conecta com a armadilha "trocar RNN por LSTM não garante solução" (7.6).

---

## 9. GRU: simplificar sem perder a ideia

### 9.1 O raciocínio de projeto

&emsp;A GRU parte da pergunta: **dá para ter o benefício das portas com menos peças?** A resposta foi:
- **Juntar as duas memórias em uma só** (`h`). Some o `c`.
- **Acoplar "manter" e "escrever"**: em vez de duas portas independentes (`f` e `i`), usa **uma só torneira `z`**, com duas saídas: `z` é o que se **mantém** e `1 − z` é o que **entra**. Faz sentido: se você mantém 90% do antigo, só sobram 10% de espaço para o novo.
- **Reset `r`**: controla quanto da memória antiga o candidato pode "consultar" ao ser calculado. Com `r = 0`, o candidato olha só o símbolo atual.

```
h_t = z · h_(t-1)  +  (1 − z) · g
```

### 9.2 Comparação com a LSTM

| | LSTM | GRU | Por que importa |
|---|---|---|---|
| Memórias | 2 (`c` e `h`) | 1 (`h`) | Menos estado para carregar |
| Portas | `f`, `i`, `o` + candidato | `z`, `r` + candidato | Menos parâmetros (≈ ×3 em vez de ×4) → treina mais rápido |
| Manter vs. escrever | independentes | acopladas (`z`, `1−z`) | Menos flexível: não dá para "manter tudo **e** escrever muito" |
| Saída | porta própria (`o`) | sem porta de saída: a memória inteira é a saída | Não dá para guardar algo "escondido" |

### 9.3 O que os resultados gravados mostram

&emsp;Mesmo padrão da LSTM: gap maior e menos neurônios → mais difícil. Dois pontos de interesse:
- A GRU de 1 neurônio aprendeu o gap 1 (época 215); a de 2 neurônios chegou a resolver o **gap 3** (época 274, lenta), enquanto a LSTM de 2 neurônios não resolveu. Isso mostra que a regra "gap máximo = nº de neurônios" é **aproximada**.
- Com 8 neurônios, todos os gaps testados aprendem rápido.

&emsp;**Cuidado com a conclusão:** são **uma execução por combinação** (uma semente). Dá para dizer que "as duas arquiteturas funcionam e seguem a mesma lógica", mas **não** que uma é melhor que a outra. Na prática, GRU e LSTM costumam ter desempenho parecido, e a escolha depende de dados e custo.

---

## 10. LSTM na prática com dados reais (notebook e4): o que ele ensina

### 10.1 Para que serve este experimento

&emsp;Os experimentos anteriores usam uma tarefa de brinquedo. O e4 pergunta: **isso vale no mundo real?** Tarefa: reconhecer 6 atividades (andar, subir/descer escada, sentado, em pé, deitado) pelo celular na cintura. Cada exemplo é uma sequência de 128 leituras × 9 sinais.

&emsp;O notebook é desenhado como uma **sequência de testes controlados, mudando uma coisa por vez**. Cada teste responde uma pergunta:

| Experimento | Pergunta | Resultado de referência | Conclusão |
|---|---|---|---|
| **Referência sem ordem** (média, desvio, mín, máx) | Quanto dá para acertar **sem** olhar a ordem do tempo? | ~82% | É o "piso" a superar. Só vale usar LSTM se ela passar disso |
| **LSTM com 32 unidades** | A ordem ajuda? | ~90% | Os ~8 pontos extras vêm de **ler a sequência** |
| **Tempo embaralhado** | A LSTM está usando a ordem mesmo? | cai para ~82% (igual à referência) | **Prova causal**: sem ordem, a LSTM empata com a referência. A vantagem **vem da ordem** |
| **8 / 32 / 64 unidades** | Quanta memória é necessária? | 82% / 90% / 90% | Com 8 faltou capacidade (volta à referência); 64 não traz informação nova. Existe um **tamanho suficiente**, e mais que isso não ajuda |
| **Só aceleração total** (3 sinais) | O que acontece se faltar informação? | treino e validação **juntos e baixos** (~76%) | **Falta de informação**: sem giroscópio, a rotação some. Rede não consegue nem nos exemplos que já viu |
| **Só 3 pessoas no treino** | O que acontece se houver pouca diversidade? | treino ~99%, validação bem menor, perda de validação **volta a subir** | **Overfitting**: decorou as 3 pessoas. A correção é mais pessoas, não mais épocas |

### 10.2 As duas lições de método (as mais importantes do notebook)

1. **Como diagnosticar uma curva:**
   - Treino alto **e** validação bem abaixo, com perda de validação subindo → **decorando** (overfitting). Remédio: mais dados diversos, regularização.
   - Treino e validação **juntos e baixos** → **falta de informação** ou capacidade. Remédio: mais sinais, rede maior.
   - A diferença entre os dois casos importa porque os **remédios são opostos**.
2. **Como validar sem se enganar:** separar **pessoas inteiras** para validação, e não janelas sorteadas. Como as janelas se sobrepõem 50% e são da mesma pessoa, sortear janelas deixa "parte da resposta" vazar para o treino: a validação sobe para ~97%, mas com pessoas realmente novas o acerto é ~79%. **Uma validação que promete mais do que o teste entrega não serve para decidir nada.**

### 10.3 A ponte com o resto do material

- A `LSTM(32)` entrega para a camada final **só o `h` do último passo** (32 números): a rede é forçada a **resumir os 128 passos** nesse vetor. É o mesmo princípio da tarefa `a/b`: a resposta só sai no fim, e o que importa é o que a memória conseguiu carregar até lá.
- O experimento do tempo embaralhado fecha o argumento de toda a unidade: **a razão de existirem redes recorrentes é a ordem importar**. Quando a ordem é destruída, a vantagem desaparece.

---

## 11. Fio condutor resumido (para revisar rápido)

| Etapa | Problema que resolve | Problema novo que cria |
|---|---|---|
| **Perceptron** | Separa classes lineares (AND, OR) | Não resolve XOR |
| **MLP** | Várias camadas resolvem XOR e problemas não lineares | Precisa ser treinada → **backprop / gradiente descendente**; explode com imagens |
| **CNN** | Filtros reduzem dimensionalidade e extraem características de imagens | **Não tem noção de tempo/sequência** |
| **RNN** | Memória: o passo atual usa o anterior (pesos reaplicados) | Memória **esvanece**; gradiente **evanescente/explosivo** em gaps longos |
| **LSTM** | Portas (esquecimento, entrada, saída) + memória longa `c` com caminho aditivo: guarda o que importa por muitos passos | Mais parâmetros; ainda precisa de treino/capacidade suficientes |
| **GRU** | Mesma ideia com menos peças (atualização + reset) | Menos flexível (memória única, portas acopladas) |
| **Transformer** | (próximos encontros) | — |

---

## Fontes usadas
- Transcrição do prof. Murilo (fluxo de raciocínio).
- Slides: `e03-LSTM.pdf`, `e02-intro-RNNs.pdf`.
- Notebooks: e1 (CNN/CIFAR-10), e2 (RNN), e3 (LSTM), e4 (LSTM com sensores).
- HTMLs interativos: `celula-lstm` e `celula-gru`.

> Observação: o PDF `e01-CNNs` não foi aberto diretamente; a parte de CNN se apoia no notebook e1 e na transcrição. Alguns termos da transcrição estavam com erro de reconhecimento de voz (ex.: "GRIU" = GRU, "MNP" = MLP) e foram corrigidos pelo contexto.