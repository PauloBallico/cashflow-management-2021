# 📊 Análises e Insights - Ideias para Explorar os Dados

## Introdução

Este documento apresenta análises que podem ser realizadas com o sistema de fluxo de caixa, servindo como:
- 💡 Inspiração para quem está explorando o projeto
- 📚 Demonstração de capacidade analítica
- 🎯 Exemplos práticos para entrevistas técnicas

---

## 🔍 Análises Básicas (Nível Iniciante)

### 1. Total Gasto por Categoria em 2021

**Objetivo**: Identificar onde o dinheiro está sendo gasto

**Como fazer**:
```excel
# Na aba 12_21 (ou qualquer mês)
1. Selecionar coluna C (CodCli) e F (Valor)
2. Inserir → Tabela Dinâmica
3. Linhas: CodCli
4. Valores: Soma de Valor
5. Ordenar por valor decrescente
```

**Insights esperados**:
- Top 3 categorias de despesa representam ~60-70% do total
- Salários são provavelmente a maior despesa (40-50%)
- Oportunidades de otimização nas maiores categorias

---

### 2. Receita por Cliente

**Objetivo**: Entender concentração de receitas

**Como fazer**:
```excel
# Filtrar apenas recebimentos (valores positivos de clientes)
1. Filtrar coluna C para clientes (MARCOPOLO, VOLARE, etc)
2. Criar Tabela Dinâmica
3. Linhas: Nome do cliente
4. Valores: Soma de Valor
```

**Métricas a calcular**:
- % de receita por cliente
- Índice de concentração (HHI)
- Risco de dependência de um cliente

**Fórmula de concentração**:
```
Se Cliente A = 70% da receita → Alto risco
Se Top 3 = 80% → Concentração moderada-alta
```

---

### 3. Evolução Mensal do Saldo

**Objetivo**: Ver tendência ao longo do ano

**Como fazer**:
1. Criar nova aba "Análise Mensal"
2. Listar meses de Jan a Dez
3. Para cada mês, extrair saldo final da última coluna de Fluxo_XX_21
4. Criar gráfico de linha

**Estrutura**:
```
Mês     | Saldo Inicial | Receitas | Despesas | Saldo Final | Variação
--------|---------------|----------|----------|-------------|----------
Jan/21  | 16.453        | 150.000  | 145.000  | 21.453      | +5.000
Fev/21  | 21.453        | 180.000  | 170.000  | 31.453      | +10.000
...
```

**Insights esperados**:
- Meses com maior geração de caixa
- Meses críticos (saldo negativo)
- Sazonalidade

---

## 📈 Análises Intermediárias (Nível Médio)

### 4. Análise de Liquidez e Dias de Caixa

**Objetivo**: Avaliar saúde financeira da empresa

**Métricas**:

**a) Dias de Caixa**
```
Dias de Caixa = Saldo Médio / (Despesas Totais / 365)
```

**Interpretação**:
- < 30 dias: Risco alto
- 30-60 dias: Adequado para PMEs
- > 60 dias: Confortável

**b) Índice de Liquidez Imediata**
```
Liquidez Imediata = Saldo Atual / Despesas Mensais Médias
```

**c) Burn Rate (se aplicável)**
```
Burn Rate = Despesas Mensais - Receitas Mensais
```

---

### 5. Análise ABC de Despesas

**Objetivo**: Priorizar esforços de controle de custos

**Método**:
1. Listar todas as categorias de despesa do ano
2. Calcular % acumulado
3. Classificar:
   - **Classe A**: 80% dos gastos (controle rígido)
   - **Classe B**: 15% dos gastos (controle moderado)
   - **Classe C**: 5% dos gastos (controle básico)

**Exemplo**:
```
Categoria      | Valor Anual | % Total | % Acum | Classe
---------------|-------------|---------|--------|-------
SALARIOS       | 1.200.000   | 42%     | 42%    | A
MATERIAL       | 800.000     | 28%     | 70%    | A
COMBUSTIVEL    | 300.000     | 11%     | 81%    | B
TRANSPORTE     | 150.000     | 5%      | 86%    | B
EPI            | 100.000     | 4%      | 90%    | B
...
```

---

### 6. Análise de Prazo Médio de Recebimento

**Objetivo**: Entender ciclo de conversão de caixa

**Como calcular**:
1. Para cada cliente, listar datas de venda (inferir de recebimentos)
2. Calcular dias entre venda e recebimento
3. Calcular média por cliente

**Estrutura**:
```
Cliente    | Data Venda | Data Receb | Dias | Prazo Médio
-----------|------------|------------|------|-------------
Marcopolo  | 01/12      | 15/12      | 14   | 18 dias
Marcopolo  | 05/12      | 20/12      | 15   |
Marcopolo  | 10/12      | 01/01      | 22   |
```

**Insights**:
- Cliente mais rápido de pagar
- Cliente mais lento (risco de inadimplência)
- Oportunidades de negociação de prazos

---

### 7. Comparação Realizado vs Orçado

**Objetivo**: Avaliar acuracidade de planejamento

**Pré-requisito**: Ter orçamento anual ou usar média dos 3 primeiros meses

**Estrutura**:
```
Categoria    | Orçado Anual | Realizado | Variação | % Var
-------------|--------------|-----------|----------|-------
SALARIOS     | 1.100.000    | 1.200.000 | +100.000 | +9%
MATERIAL     | 750.000      | 800.000   | +50.000  | +7%
COMBUSTIVEL  | 280.000      | 300.000   | +20.000  | +7%
```

**Análise de variações**:
- Variações favoráveis (economia)
- Variações desfavoráveis (estouro)
- Causas (volume, preço, mix)

---

## 🎯 Análises Avançadas (Nível Expert)

### 8. Análise de Correlação entre Vendas e Despesas

**Objetivo**: Entender elasticidade de custos

**Método estatístico**:
```python
import pandas as pd
import numpy as np

# Carregar dados mensais
df = pd.DataFrame({
    'mes': range(1, 13),
    'receitas': [150000, 180000, ...],  # extrair do Excel
    'despesas_variaveis': [...],
    'despesas_fixas': [...]
})

# Calcular correlação
corr_receita_desp_var = df['receitas'].corr(df['despesas_variaveis'])
corr_receita_desp_fix = df['receitas'].corr(df['despesas_fixas'])

print(f"Correlação Receitas x Despesas Variáveis: {corr_receita_desp_var:.2f}")
print(f"Correlação Receitas x Despesas Fixas: {corr_receita_desp_fix:.2f}")
```

**Interpretação**:
- Correlação alta (>0.7): Despesas crescem com vendas (bom)
- Correlação baixa (<0.3): Despesas não escalam (investigar)

---

### 9. Previsão de Fluxo de Caixa (Machine Learning)

**Objetivo**: Prever saldo futuro com base em histórico

**Abordagem 1: ARIMA (Série Temporal)**
```python
from statsmodels.tsa.arima.model import ARIMA
import pandas as pd

# Preparar dados
df = pd.read_excel('Fluxo_2021.xlsx', sheet_name='Análise Mensal')
df['data'] = pd.to_datetime(df['mes'], format='%m/%Y')
df.set_index('data', inplace=True)

# Treinar modelo
model = ARIMA(df['saldo_final'], order=(1,1,1))
results = model.fit()

# Prever próximos 3 meses
forecast = results.forecast(steps=3)
print(f"Previsão próximos 3 meses: {forecast}")
```

**Abordagem 2: Regressão Linear Múltipla**
```python
from sklearn.linear_model import LinearRegression

# Features: receitas, despesas fixas, despesas variáveis, mes
X = df[['receitas', 'desp_fixas', 'desp_variaveis', 'mes_numerico']]
y = df['saldo_final']

# Treinar modelo
model = LinearRegression()
model.fit(X, y)

# Prever
previsao = model.predict([[180000, 80000, 60000, 13]])  # Jan 2022
```

---

### 10. Análise de Sensibilidade

**Objetivo**: Simular cenários (otimista, realista, pessimista)

**Variáveis chave**:
- Receitas: -20%, 0%, +20%
- Custos variáveis: -10%, 0%, +10%
- Custos fixos: 0%, +5%, +10%

**Estrutura da análise**:
```
Cenário      | Receitas | Custos Var | Custos Fix | Resultado
-------------|----------|------------|------------|------------
Pessimista   | -20%     | +10%       | +10%       | -R$ 500k
Base         | 0%       | 0%         | 0%         | +R$ 100k
Otimista     | +20%     | -10%       | 0%         | +R$ 800k
```

**Insight**: Identificar variável com maior impacto

---

### 11. Análise de Break-Even (Ponto de Equilíbrio)

**Objetivo**: Identificar nível mínimo de receita para não ter prejuízo

**Fórmula**:
```
Break-Even = Custos Fixos / (1 - (Custos Variáveis / Receitas))
```

**Cálculo com dados reais**:
```
Custos Fixos Mensais = R$ 90.000
Custos Variáveis = 45% da Receita

Break-Even = 90.000 / (1 - 0.45) = R$ 163.636/mês
```

**Análise**:
- Quantos meses em 2021 ficaram acima do break-even?
- Qual a margem de segurança? [(Receita Atual - BE) / Receita Atual]

---

### 12. Análise de Composição de Custos Fixos vs Variáveis

**Objetivo**: Entender alavancagem operacional

**Classificação**:
```
CUSTOS FIXOS:
- Salários administrativos
- Aluguel
- Seguros
- Depreciação

CUSTOS VARIÁVEIS:
- Material direto
- Comissões de vendas
- Combustível (proporcional a vendas)
```

**Métricas**:
```
% Custos Fixos = Custos Fixos / Custos Totais
% Custos Variáveis = Custos Variáveis / Custos Totais

Alavancagem Operacional = Margem Contrib / Lucro Operacional
```

**Interpretação**:
- Alto % de fixos: Mais risco, mais upside
- Alto % de variáveis: Menos risco, menos upside

---

## 🧪 Análises Especializadas

### 13. Detecção de Anomalias

**Objetivo**: Identificar transações suspeitas ou atípicas

**Método**:
1. Calcular média e desvio padrão por categoria
2. Identificar valores que estão a >3 desvios padrão da média
3. Investigar causas

**Exemplo em Python**:
```python
import pandas as pd
import numpy as np

df = pd.read_excel('Fluxo_2021.xlsx', sheet_name='12_21')

# Para cada categoria
for categoria in df['CodCli'].unique():
    subset = df[df['CodCli'] == categoria]['Valor']
    
    media = subset.mean()
    std = subset.std()
    
    # Outliers
    outliers = subset[(subset > media + 3*std) | (subset < media - 3*std)]
    
    if len(outliers) > 0:
        print(f"Categoria {categoria}: {len(outliers)} anomalias detectadas")
```

---

### 14. Análise de Ciclo de Conversão de Caixa (CCC)

**Objetivo**: Medir eficiência operacional

**Fórmula**:
```
CCC = DIO + DSO - DPO

Onde:
DIO = Days Inventory Outstanding (estoque)
DSO = Days Sales Outstanding (recebíveis)
DPO = Days Payables Outstanding (pagáveis)
```

**Simplificado para este projeto**:
```
DSO = (Contas a Receber / Receitas) × 365

Exemplo:
Receita anual = R$ 2.000.000
Recebível médio = R$ 300.000
DSO = (300.000 / 2.000.000) × 365 = 55 dias
```

**Interpretação**:
- DSO baixo (<30): Clientes pagam rápido
- DSO alto (>60): Risco de inadimplência

---

### 15. Dashboard Executivo em Excel

**Objetivo**: Criar visão consolidada para tomada de decisão

**Estrutura do Dashboard**:

```
┌─────────────────────────────────────────────────────────┐
│                 DASHBOARD FINANCEIRO 2021                │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  💰 Saldo Atual: R$ 150.000        📈 vs Mês Ant: +12%  │
│                                                          │
│  📊 Receitas 2021: R$ 2.000.000    📊 Despesas: 1.850k  │
│  📈 Resultado: R$ 150.000          📊 Margem: 7.5%      │
│                                                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  [Gráfico Evolução Saldo]    [Gráfico Pizza Despesas]  │
│                                                          │
│  [Tabela Top 5 Clientes]     [Tabela Top 5 Despesas]   │
│                                                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ⚠️ Alertas:                                             │
│  • Saldo projetado negativo em 15/02                    │
│  • Despesa com combustível +25% vs mês anterior         │
│  • Cliente Volare com 3 pagamentos atrasados            │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Funcionalidades**:
- Atualização automática ao mudar mês
- Condicional formatting (vermelho/verde)
- Sparklines para mini-gráficos
- Slicers para filtrar por período

---

## 💡 Ideias de Análises para Entrevistas

### Pergunta 1: "Como você identificaria oportunidades de redução de custos?"

**Resposta estruturada**:
1. Análise ABC para priorizar categorias
2. Comparação com benchmarks de mercado
3. Análise de tendência (custos crescendo acima da inflação?)
4. Detecção de outliers e investigação
5. Avaliação de alternativas (fornecedores, processos)

**Demonstração com dados reais**:
- "Salários representam 42% dos custos"
- "Combustível teve variação de 30% entre meses"
- "Posso negociar contratos de maior volume para Material"

---

### Pergunta 2: "Como você preveria o fluxo de caixa para o próximo trimestre?"

**Resposta técnica**:
1. **Análise de sazonalidade**: Média dos Q1 dos últimos anos
2. **Regressão linear**: Receitas = f(mês, sazonalidade)
3. **Cenários**: Otimista/Realista/Pessimista
4. **Validação**: Conferir com pipeline de vendas

**Excel**:
```excel
=AVERAGE(Jan:Mar receitas) × (1 + taxa_crescimento)
```

**Python**:
```python
from sklearn.linear_model import LinearRegression
# Código de exemplo acima
```

---

### Pergunta 3: "Quais KPIs você monitoraria mensalmente?"

**Resposta prática**:

**KPIs Financeiros**:
- 📊 Saldo de caixa
- 💰 Receitas vs orçado
- 💸 Despesas vs orçado
- 📈 Margem EBITDA
- 🔄 Dias de caixa

**KPIs Operacionais**:
- 📅 DSO (prazo médio recebimento)
- 🚚 Giro de estoque (se aplicável)
- 👥 Produtividade (receita/funcionário)

**KPIs de Risco**:
- ⚠️ Concentração de clientes
- 📉 Liquidez imediata
- 🔴 Dias com saldo negativo

---

## 📚 Recursos para Aprofundamento

### Cursos Recomendados
- **Excel Avançado**: Microsoft Learn, Udemy
- **Power BI**: DataCamp, Pluralsight
- **Python para Finanças**: Coursera, edX
- **Análise Financeira**: FGV, Insper

### Livros
- "Financial Modeling" - Simon Benninga
- "Python for Finance" - Yves Hilpisch
- "Storytelling with Data" - Cole Nussbaumer Knaflic

### Ferramentas
- **Análise**: Excel, Python (Pandas), R
- **Visualização**: Power BI, Tableau, Plotly
- **Estatística**: SPSS, Minitab, R

---

## ✅ Checklist de Análises para Portfólio

Para impressionar em entrevistas, tenha prontas:

- [ ] Top 5 insights do projeto
- [ ] Uma análise de tendência (com gráfico)
- [ ] Uma análise de correlação
- [ ] Uma previsão simples (próximo mês)
- [ ] Dashboard executivo em Excel ou Power BI
- [ ] Script Python para ETL básico
- [ ] Apresentação de 5 slides sobre o projeto

---

**Última Atualização**: 2025-01-27  
**Nível**: Iniciante a Avançado  
**Tempo para implementar todas**: 8-16 horas
