# 📊 Sistema de Fluxo de Caixa 2021

[![Excel](https://img.shields.io/badge/Excel-Advanced-217346?logo=microsoft-excel)](https://www.microsoft.com/excel)
[![Status](https://img.shields.io/badge/Status-Complete-success)]()
[![License](https://img.shields.io/badge/License-MIT-blue.svg)]()

## 📋 Sobre o Projeto

Sistema completo de controle de fluxo de caixa desenvolvido para gestão financeira empresarial, cobrindo todo o ano de 2021. O projeto demonstra habilidades avançadas em Excel, análise financeira e automação de processos através de fórmulas complexas e estruturação de dados.

**Contexto**: Sistema desenvolvido para controle financeiro de empresas do setor industrial, com foco em gestão de múltiplas entidades e categorização detalhada de receitas e despesas.

## 🎯 Objetivos do Projeto

- ✅ Controle diário de entradas e saídas financeiras
- ✅ Projeção de saldos e fluxo de caixa
- ✅ Categorização inteligente de transações
- ✅ Separação por centros de custo (Empresa 1, Empresa 2, Distribuição de Lucros)
- ✅ Acompanhamento de clientes estratégicos (Marcopolo, Volare, Hidrover)
- ✅ Automação através de fórmulas avançadas

## 🚀 Funcionalidades Principais

### 1. **Estrutura Modular por Mês**
- 24 abas dedicadas (12 meses × 2 tipos)
- Aba de dados brutos (XX_21)
- Aba de fluxo visual (Fluxo_XX_21)

### 2. **Controle Diário de Transações**
Cada transação contém:
- Data de vencimento
- Valor
- Categoria (Salários, EPI, Combustível, etc.)
- Tipo/Centro de custo
- Status de pagamento (via VLOOKUP)

### 3. **Dashboard de Fluxo**
Visualização dia a dia incluindo:
- Saldo inicial do mês
- Recebimentos diários (operação)
- Pagamentos por categoria
- Saldo projetado
- Identificação de dias da semana

### 4. **Fórmulas Avançadas Implementadas**

```excel
# Identificação de dias da semana
=IF(B5=1,"Domingo",IF(B5=2,"Segunda",IF(B5=3,"Terça",...)))

# Busca de status de pagamento
=VLOOKUP(E2,$E$2783:$H$2814,4,FALSE)

# Cálculo de dia da semana
=WEEKDAY(B2)

# Totalização de recebimentos
=SUM(B7:B26)

# Propagação de saldos
=Fluxo_11_21!U38
```

## 📊 Estrutura de Dados

### Categorias de Despesas
- 💰 **Salários e Encargos**: Folha de pagamento, FGTS, benefícios
- 🛡️ **EPI**: Equipamentos de proteção individual
- ⛽ **Combustível**: Abastecimento de frota
- 🏦 **Bancos**: Juros, tarifas, despesas bancárias
- 📦 **Material**: Insumos e matéria-prima
- 🤝 **Acordos e Parcelamentos**: Compromissos financeiros
- 👥 **Sócios**: Retiradas e distribuição de lucros
- 🚚 **Transporte e Viagem**: Despesas logísticas

### Categorias de Receitas
- 🏭 **Clientes Principais**:
  - Marcopolo/Seman
  - Volare
  - Hidrover
  
### Centros de Custo
1. **Empresa 1**: Operações administrativas e corporativas
2. **Empresa 2**: Operações industriais e produção
3. **Distribuição de Lucros**: Retiradas societárias

## 📈 Métricas e KPIs

O sistema permite acompanhar:

- 📊 Saldo diário e projeções
- 💵 Total de recebimentos mensais
- 💸 Total de pagamentos por categoria
- 🔄 Fluxo de caixa acumulado
- 📉 Análise de tendências ao longo do ano
- ⚠️ Identificação de períodos críticos

## 🔧 Tecnologias e Técnicas

### Excel Avançado
- **Fórmulas**:  VLOOKUP, WEEKDAY, IF aninhados, SUM, referências entre abas
- **Estruturação**: Separação de dados e apresentação
- **Automação**: Cálculos automáticos de status e totalizações
- **Organização**: Nomenclatura padronizada e estrutura hierárquica

### Boas Práticas
- ✅ Separação clara entre dados brutos e visualização
- ✅ Nomenclatura consistente de abas
- ✅ Uso de referências absolutas e relativas apropriadamente
- ✅ Categorização detalhada para análise
- ✅ Propagação de saldos entre meses

## 📁 Estrutura do Arquivo

```
Fluxo_2021.xlsx
├── 01_21          # Dados brutos - Janeiro
├── Fluxo_01_21    # Dashboard - Janeiro
├── 02_21          # Dados brutos - Fevereiro
├── Fluxo_02_21    # Dashboard - Fevereiro
├── ...            # (repetição para todos os meses)
├── 12_21          # Dados brutos - Dezembro
├── Fluxo_12_21    # Dashboard - Dezembro
├── RECLAM JUDICIAL # Controle de processos trabalhistas
└── Planilha1      # Controles adicionais
```

## 💡 Insights e Aprendizados

Este projeto demonstra:

1. **Habilidade Analítica**: Capacidade de estruturar dados financeiros complexos
2. **Automação**: Uso de fórmulas para reduzir trabalho manual e erros
3. **Visão de Negócio**: Compreensão de fluxo de caixa e gestão financeira
4. **Atenção a Detalhes**: Categorização minuciosa e controle preciso
5. **Escalabilidade**: Estrutura que pode ser replicada para outros períodos

## 📊 Dados de Exemplo

### Resumo 2021 (Dezembro)

- **Total de linhas de transações**: ~2.860 registros
- **Categorias rastreadas**: 15+
- **Clientes principais**: 3+
- **Período de cobertura**: 365 dias

## 🎓 Habilidades Demonstradas

### Técnicas
- Modelagem de dados financeiros
- Criação de dashboards executivos
- Automação de processos
- Análise de fluxo de caixa

### Ferramentas
- Microsoft Excel (Fórmulas avançadas)
- Gestão de múltiplas fontes de dados
- Criação de relatórios gerenciais

## 🔍 Como Utilizar

1. **Visualização Rápida**: Abrir qualquer aba `Fluxo_XX_21` para ver o dashboard do mês
2. **Dados Detalhados**: Consultar abas `XX_21` para transações completas
3. **Análise de Categoria**: Filtrar por tipo de despesa/receita
4. **Projeções**: Acompanhar saldos acumulados ao longo dos dias

## 🚀 Possíveis Evoluções

- [ ] Migração para Power BI para visualizações interativas
- [ ] Automação com Python/Pandas para processamento em larga escala
- [ ] Integração com APIs bancárias
- [ ] Dashboard web com gráficos dinâmicos
- [ ] Machine Learning para previsão de fluxo

## 📝 Notas Técnicas

- **Dados sensíveis**: Todos os valores foram anonimizados
- **Período**: Ano completo de 2021
- **Formato**: Excel (.xlsx) com preservação de fórmulas
- **Compatibilidade**: Excel 2016+ ou LibreOffice Calc

## 📧 Contato

Desenvolvido como parte do portfólio profissional para demonstração de habilidades em análise de dados e Excel avançado.

---

**Tags**: `excel` `financial-analysis` `cashflow` `data-analysis` `dashboard` `formulas` `vlookup` `business-intelligence`

