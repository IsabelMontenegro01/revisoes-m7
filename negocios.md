# Tratado Integrado de Gestão de Ativos, Engenharia Orçamentária e Demonstrações Financeiras

## Módulo 1: Manutenção Preditiva e Gestão Estratégica de Ativos

### 1.1. Fundamentos da Gestão de Ativos (ISO 55000)
De acordo com a norma **ISO 55000**, um ativo é qualquer bem ou recurso com valor real ou potencial para uma organização durante seu ciclo de vida. O ecossistema abrange equipamentos industriais, infraestruturas físicas, sistemas de software, bases de dados e capital humano.

O ciclo de vida completo de um ativo engloba cinco etapas encadeadas:
1. **Planejamento**
2. **Aquisição**
3. **Operação**
4. **Manutenção**
5. **Renovação/Descarte** (incluindo responsabilidades remanescentes)

O objetivo central não é o reparo pontual, mas sustentar um equilíbrio contínuo entre **custo, risco e desempenho** ao longo da vida útil.

### 1.2. Qualidade Percebida pelo Cliente e Métricas Operacionais
A excelência da manutenção impacta a percepção do cliente em duas dimensões:
*   **Disponibilidade:** Porcentagem do tempo em que o ativo está operável (ex.: servidor com 99,99% de uptime). É influenciada pelo tempo de reparo.
*   **Confiabilidade:** Probabilidade de o equipamento funcionar sem falhas durante um intervalo especificado (ex.: motor operando milhares de horas contínuas). É afetada pela frequência de ocorrências.

### 1.3. Matriz de Criticidade de Ativos
Para evitar a instrumentação indiscriminada da planta, classifica-se a criticidade:
*   **Classe A (Crítico):** Falha causa interrupção imediata da produção. Impacto financeiro altíssimo (medido em minutos). 
    *   *Estratégia:* Manutenção Preditiva com redundância (ex.: motores principais, transformadores).
*   **Classe B (Importante):** Falha causa degradação ou retrabalho. Impacto financeiro moderado (medido em horas).
    *   *Estratégia:* Manutenção Preventiva + Monitoramento (ex.: bombas secundárias, sensores).
*   **Classe C (Baixo Impacto):** Paradas não comprometem a produção imediata. Impacto baixo (medido em dias).
    *   *Estratégia:* Manutenção Corretiva Planejada (ex.: ventiladores auxiliares).

### 1.4. Análise Comparativa dos Tipos de Manutenção
*   **Corretiva:** Acionada por falha inesperada. Custo elevado (paradas, danos em cascata) e risco máximo à segurança e qualidade.
*   **Preventiva:** Intervalos regulares (tempo ou uso). Custo moderado e risco reduzido, previne falhas catastróficas.
*   **Preditiva:** Monitoramento contínuo (sensores, IoT, IA). Custo inicial moderado/alto (infraestrutura), mas reduz o risco ao mínimo, permitindo intervenção antes da quebra.

### 1.5. Estrutura do PCM e Sinergia com o PCP
O **Planejamento e Controle de Manutenção (PCM)** coordena recursos (mão de obra, peças, ferramentas) em quatro pilares:
1. **Planejamento:** Tipos, frequências e recursos.
2. **Programação:** Agendamento sem conflito com a produção.
3. **Execução:** Realização com qualidade e segurança.
4. **Controle:** Monitoramento de custos, prazos e efetividade.

O PCM atua em sinergia com o **PCP (Planejamento e Controle da Produção)**, alinhando paradas e alimentando dados para evitar atrasos.

### 1.6. Manutenção Produtiva Total (TPM) e Quality 4.0
O TPM busca os **ZERO** defeitos, quebras e acidentes através de:
*   **Manutenção Autônoma:** Operadores realizam limpeza, lubrificação e inspeções visuais.
*   **Manutenção Planejada:** Especialistas focam em preditiva, causa raiz e confiabilidade.
*   **Melhoria Específica:** Grupos multidisciplinares focados em reduzir perdas crônicas (ex.: SMED para tempo de setup).

> **Impacto:** Soluções preditivas geram reduções de 10% a 20% nos custos de manutenção e no tempo de inatividade (McKinsey).

### 1.7. Custo do Ciclo de Vida (LCC) e Modelagem Financeira
O Life Cycle Cost (LCC) consolida todos os custos desde a concepção até o descarte:

$$LCC = CAPEX (Aquisição/Instalação) + OPEX (Energia/Pessoal/Manutenção) + Custos de Falha + Custos de Descarte$$

**Demonstração (Motor Industrial - 10 anos):**
*   **Sem Preditiva:** CAPEX ($R\$ 50.000$) + OPEX ($R\$ 80.000$) + Falha ($R\$ 150.000$) = **$R\$ 280.000$**
*   **Com Preditiva:** CAPEX ($R\$ 50.000$) + OPEX/Sensores ($R\$ 100.000$) + Falha ($R\$ 20.000$) = **$R\$ 170.000$**
*   **Ganho:** Economia de $R\$ 110.000$ (redução de 39%).

### 1.8. Roadmap de Implementação em 4 Fases
Para capturar valor e evitar o "purgatório dos pilotos":
1. **Diagnóstico & Criticidade:** Mapear ativos Classe A, calcular LCC e projetar ROI.
2. **Infraestrutura & Dados:** Instalar sensores, garantir IoT/segurança e limpar dados.
3. **Inteligência & Modelagem:** Treinar algoritmos de Machine Learning (anomalias).
4. **Integração & Ação:** Conectar alertas ao PCM e promover mudança cultural.

---

## Módulo 2: Planejamento Orçamentário e Análise de Investimentos

### 2.1. O Papel Estratégico do Orçamento
Projetos são aprovados pelo valor financeiro demonstrado, não apenas pela elegância técnica.
*   **Inputs:** Escopo, cronograma, riscos, recursos.
*   **Ferramentas:** Consolidação de custos, controle, definição de margens.
*   **Outputs:** Planilha orçamentária e documentos formais.

### 2.2. CAPEX vs. OPEX
*   **CAPEX (Capital Expenditure):** Investimento duradouro em ativo fixo (equipamentos, licenças, sensores).
*   **OPEX (Operational Expenditure):** Despesas diárias da operação (manutenção, energia, salários).

### 2.3. Matemática Financeira de Projetos
*   **Margem de Contribuição (MC):** Preço de Venda – Custos/Despesas Variáveis.
*   **Ponto de Equilíbrio (PE):** O "zero a zero" financeiro.
    *   Unidades:
    $$PE (unidades) = \frac{Custos Fixos Totais}{Margem de Contribuição Unitária}$$
    *   Valor:
    $$PE (valor R\$) = \frac{Custos Fixos Totais}{Índice da Margem de Contribuição (\%)}$$

*   **Retorno sobre o Investimento (ROI):** 
    $$ROI = \frac{Resultado Operacional - Custo do Investimento}{Custo do Investimento}$$
    *(O Resultado Operacional equivale à economia gerada subtraída do OPEX correspondente. ROI negativo no Ano 1 é comum).*

*   **Payback (Tempo de Retorno de Capital):** 
    $$Payback = \frac{CAPEX}{Lucro Operacional Anual Médio}$$

### 2.4. Gestão de Incertezas por Análise de Cenários
Um orçamento maduro projeta três cenários: **Conservador**, **Base** (mais provável) e **Otimista**.
*   **Case Motiva (PSDs):** A migração para preditiva orientada a dados em portas de plataforma de metrô exige um plano orçamentário que justifique financeiramente a redução de atrasos e corretivas.

---

## Módulo 3: DFC e Análise de Liquidez

### 3.1. Demonstração dos Fluxos de Caixa (DFC)
Registra a movimentação real de dinheiro (entradas e saídas). É o principal indicador de **liquidez e solvência** (Regulamentada pela Lei nº 11.638/2007).

### 3.2. Regime de Competência vs. Regime de Caixa
*   **Competência (DRE):** Registra receitas/despesas no fato gerador (ex.: emissão da nota). Mede o **Lucro**.
*   **Caixa (DFC):** Registra entradas/saídas reais da conta bancária. Mede a **Liquidez**.

### 3.3. Atividades da DFC e Fluxo de Caixa Livre
1. **FCO (Operacionais):** Dinheiro da operação principal (clientes, fornecedores, salários).
2. **FCI (Investimentos):** Compra/venda de CAPEX (imóveis, máquinas). Negativo é comum em expansão.
3. **FCF (Financiamentos):** Movimentos com financiadores (empréstimos, dividendos, debêntures).

**Fluxo de Caixa Livre (FCL):**
$$FCL = FCO - CAPEX$$

### 3.4. Ciclo de Conversão de Caixa (CCC)
Tempo (em dias) que o dinheiro investido volta ao caixa:

$$CCC = PMR (Prazo Médio de Recebimento) + PME (Prazo Médio de Estoques) - PMP (Prazo Médio de Pagamento)$$

*   **Saudável (Walmart/MRV):** CCC negativo apoiado em previsibilidade ou poder de negociação.
*   **Frágil (Hurb/123milhas):** CCC negativo baseado em adiantamentos flexíveis (Passivo Operacional). O aumento nos custos inviabiliza a entrega futura.

### 3.5. Sinais de Crise e Cases
*   **Lojas Americanas:** Inflou o FCO ocultando dívidas de "risco sacado" em fornecedores (PCO).
*   **Oi S.A.:** Mesmo após reduzir dívidas em RJs, o caixa continuou sendo consumido pela operação e passivos, culminando em falência.
*   **Sinais de Alerta:** FCO cronicamente negativo; venda de ativos sem caixa real; passivo crescendo mais que a operação.

---

## Módulo 4: Demonstração do Resultado do Exercício (DRE)

### 4.1. Finalidade e Regulamentação
A DRE confronta receitas, custos e despesas para apurar o lucro ou prejuízo. Baseia-se no **Regime de Competência** e serve para medir a rentabilidade e apurar tributos (IRPJ/CSLL).

### 4.2. Estrutura Cascata Oficial
1. Receita Operacional Bruta
2. (-) Deduções da Receita Bruta (impostos, devoluções)
3. **(=) Receita Operacional Líquida**
4. (-) Custos das Vendas (CPV, CMV, CSP)
5. **(=) Resultado Operacional Bruto (Lucro Bruto)**
6. (-) Despesas Operacionais (vendas, administrativas)
7. (-) Resultado Financeiro Líquido (despesas - receitas financeiras)
8. **(=) Resultado Operacional Antes do IR/CSLL**
9. (-) Provisão para IR e CSLL
10. **(=) Resultado Líquido do Exercício**

---

## Módulo 5: Síntese e Aplicação em Projetos

### 5.1. Matriz Comparativa Financeira

| Demonstrativo | Regime Contábil | Foco Principal | Objetivo Estratégico |
| :--- | :--- | :--- | :--- |
| **DFC (Fluxo de Caixa)** | Caixa | Entradas e saídas efetivas de dinheiro | Medir liquidez e garantir compromissos |
| **DRE (Resultado)** | Competência | Confronto de receitas, custos e despesas | Medir rentabilidade e desempenho |
| **Balanço Patrimonial** | Estático (posição em um instante) | Ativos, Passivos e Patrimônio Líquido | Avaliar solvência e estrutura de capital |

### 5.2. Aplicação da DRE em Projetos de Engenharia
A estrutura da DRE pode justificar projetos de tecnologia (como preditiva):
*   **Créditos por Eficiência:** Ganhos entram como "receitas" geradas por economia (menos multas, redução de peças e inatividade).
*   **Confronto:** Deduzem-se os custos diretos do projeto (amortização de sensores) e despesas contínuas de operação.
*   **Resultado Líquido do Projeto:** O lucro final gerado baseia os cálculos de ROI e Payback necessários para a aprovação executiva.