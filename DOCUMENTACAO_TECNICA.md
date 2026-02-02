# 📘 Documentação Técnica - Sistema de Fluxo de Caixa 2021

## Índice
1. [Arquitetura do Sistema](#arquitetura-do-sistema)
2. [Estrutura de Dados](#estrutura-de-dados)
3. [Fórmulas e Lógicas](#fórmulas-e-lógicas)
4. [Fluxo de Trabalho](#fluxo-de-trabalho)
5. [Manutenção e Troubleshooting](#manutenção-e-troubleshooting)

---

## Arquitetura do Sistema

### Visão Geral

O sistema é composto por **26 abas** organizadas em pares para cada mês do ano:

```
Estrutura Hierárquica:
├── Abas de Dados Brutos (XX_21)
│   └── Contém todas as transações do mês
│       ├── Colunas: tm, CodCli, Nome, dtvcto, Valor, tipo, Pagto
│       └── ~2.800-3.000 linhas por mês
│
└── Abas de Dashboard (Fluxo_XX_21)
    └── Visualização consolidada dia a dia
        ├── Linha 1: Dias da semana (calculados)
        ├── Linha 2: Saldo localizado no banco
        ├── Linha 4: Saldo inicial (do mês anterior)
        ├── Linha 6-26: Recebimentos por cliente
        └── Linhas seguintes: Pagamentos por categoria
```

### Dependências Entre Abas

**Fluxo de Propagação de Saldos:**

```
Fluxo_01_21 → Fluxo_02_21 → Fluxo_03_21 → ... → Fluxo_12_21
     ↓              ↓              ↓                    ↓
  01_21         02_21         03_21                12_21
```

Cada aba de fluxo referencia:
- O saldo final do mês anterior
- Os dados brutos do próprio mês
- Tabela de lookup para status de pagamento

---

## Estrutura de Dados

### Aba de Dados Brutos (XX_21)

**Schema:**

| Coluna | Tipo | Descrição | Exemplo |
|--------|------|-----------|---------|
| A | Numérico | Marcador (sempre 0 ou None) | 0 |
| B (tm) | Texto | Tag/Marcador adicional | "tm" |
| C (CodCli) | Texto | Código/Categoria do cliente/despesa | "SALARIOS", "MARCOPOLO" |
| D (Nome) | Texto | Descrição detalhada da transação | "SALARIOS", "RETIRADA LUIS b" |
| E (dtvcto) | Data | Data de vencimento | 2021-12-01 |
| F (Valor) | Numérico/Fórmula | Valor da transação | 1020 ou =2193.19+157.32 |
| G (tipo) | Texto | Centro de custo | "Empresa 1", "Empresa 2", "Distribuição de Lucros" |
| H (Pagto.) | Fórmula | Status de pagamento (via VLOOKUP) | =VLOOKUP(...) |

**Tabela de Lookup (Final da aba):**
- Localizada nas últimas linhas (~2783:2814 no exemplo de Dezembro)
- Estrutura: Data | Valor1 | Valor2 | Status
- Usada para determinar se transação foi paga

### Aba de Dashboard (Fluxo_XX_21)

**Layout:**

```
   A          B        C        D        E    ...
1  [Label]  [Dia 1]  [Dia 2]  [Dia 3]  [Dia 4]
2  Saldo    =B29     =C29     =D29     ...
3  [Vazio]
4  SALDO    =Ant!U38 =B38     =C38     ...
5  [Cálc]   =WDAY..  =WDAY..  =WDAY..  ...
6  RECEB    =SUM(..) =SUM(..) =SUM(..) ...
7  Cliente1  valor    valor    valor   ...
...
27 PAGTOS   =SUM(..) =SUM(..) =SUM(..) ...
28 Cat1     valor    valor    valor   ...
...
```

**Linha por linha:**
- **Linha 1**: Nome do dia da semana (calculado via IF+WEEKDAY)
- **Linha 2**: Referência ao saldo confirmado no banco
- **Linha 4**: Saldo inicial (carregado do mês anterior)
- **Linha 5**: Código numérico do dia da semana
- **Linha 6**: Total de recebimentos do dia
- **Linhas 7-26**: Detalhamento de recebimentos por cliente
- **Linha 27+**: Total de pagamentos
- **Linhas seguintes**: Detalhamento de pagamentos por categoria
- **Última linha**: Saldo final do dia (Inicial + Recebimentos - Pagamentos)

---

## Fórmulas e Lógicas

### 1. Cálculo de Dia da Semana

**Linha 1 (Nome do dia):**
```excel
=IF(B5=1,"Domingo",
   IF(B5=2,"Segunda",
      IF(B5=3,"Terça",
         IF(B5=4,"Quarta",
            IF(B5=5,"Quinta",
               IF(B5=6,"Sexta",
                  IF(B5=7,"Sábado",0)))))))
```

**Linha 5 (Código do dia):**
```excel
=WEEKDAY(B2)
```

**Lógica**: 
- B2 contém a data
- WEEKDAY retorna 1-7 (Domingo=1, Segunda=2, etc.)
- IF aninhado converte número em texto

### 2. Busca de Status de Pagamento

**Nas abas de dados brutos:**
```excel
=VLOOKUP(E2,$E$2783:$H$2814,4,FALSE)
```

**Parâmetros:**
- `E2`: Data de vencimento a buscar
- `$E$2783:$H$2814`: Tabela de referência (endereços absolutos)
- `4`: Retornar valor da 4ª coluna da tabela
- `FALSE`: Correspondência exata

**Propósito**: Verificar se transação com determinada data foi efetivamente paga

### 3. Propagação de Saldos

**Saldo Inicial do Mês:**
```excel
=Fluxo_11_21!U38
```

**Saldo Diário (propagação horizontal):**
```excel
=B38    // Célula B do mesmo tipo de linha
```

**Fluxo completo:**
1. Fluxo_12_21 célula B4 busca saldo final de novembro
2. Células C4, D4, E4... referenciam o saldo do dia anterior
3. Cálculo: Saldo_Atual = Saldo_Anterior + Recebimentos - Pagamentos

### 4. Totalização de Recebimentos

```excel
=SUM(B7:B26)
```

**Observações:**
- Soma linhas 7-26 da coluna B (recebimentos do cliente)
- Range fixo para cada coluna de dia
- Resultado na linha 6

### 5. Cálculos Compostos

Muitas células de valor contêm fórmulas agregadas:

```excel
=2193.19+157.32+234.5+82.75+945+430.4
```

**Propósito**: 
- Consolidar múltiplos itens em uma única linha
- Facilitar rastreamento de composição de valores
- Documentar breakdown inline

---

## Fluxo de Trabalho

### Processo Mensal de Atualização

#### 1. Coleta de Dados
```
┌─────────────────┐
│ Extratos        │
│ Bancários       │──┐
└─────────────────┘  │
                     │
┌─────────────────┐  │    ┌──────────────────┐
│ Notas Fiscais   │──┼───→│  Aba XX_21       │
│ (Receitas)      │  │    │  (Dados Brutos)  │
└─────────────────┘  │    └──────────────────┘
                     │              │
┌─────────────────┐  │              ↓
│ Comprovantes    │──┘    ┌──────────────────┐
│ de Pagamento    │       │  Fluxo_XX_21     │
└─────────────────┘       │  (Dashboard)     │
                          └──────────────────┘
```

#### 2. Entrada de Transações

**Na aba XX_21:**
1. Adicionar nova linha após última transação
2. Preencher colunas C-G manualmente:
   - CodCli: Categoria
   - Nome: Descrição
   - dtvcto: Data
   - Valor: Montante
   - tipo: Centro de custo
3. Coluna H se preenche automaticamente (VLOOKUP)

#### 3. Atualização do Dashboard

**Na aba Fluxo_XX_21:**
1. Localizar data na linha 2 (cabeçalho)
2. Adicionar valor na linha correspondente ao cliente/categoria
3. Fórmulas de totalização se atualizam automaticamente
4. Saldo final é recalculado

#### 4. Reconciliação

**Diariamente:**
- Conferir linha 2 (Localizado no banco) com saldo real
- Ajustar discrepâncias

**Mensalmente:**
- Validar saldo final do mês
- Garantir que Fluxo_XX_21 linha final = Próximo mês linha 4

---

## Categorização Detalhada

### Categorias de Despesas

| Categoria | Subcategorias Comuns | Centro de Custo Típico |
|-----------|---------------------|------------------------|
| **SALARIOS** | Folha, 13º, Férias, Horas extras | Empresa 2 |
| **ENCARGOS** | FGTS, INSS | Empresa 2 |
| **EPI** | Zapas, Luvas, Capacetes, Uniformes | Empresa 2 |
| **COMBUSTIVEL** | Abastecimento frota, Vale-combustível | Empresa 2 |
| **BANCOS** | Tarifas, Juros, IOF | Empresa 1 |
| **MATERIAL** | Matéria-prima, Insumos | Empresa 1 |
| **TRANSPORTE** | Vale-transporte, Frete | Empresa 2 |
| **VIAGEM** | Hospedagem, Alimentação, Deslocamento | Empresa 2 |
| **ACORDO** | Parcelamentos judiciais | Variado |
| **RESCISAO** | Verbas rescisórias, FGTS rescisório | Empresa 2 |
| **SOCIOS** | Retiradas, Distribuição lucros | Distribuição de Lucros |
| **TERCEIROS** | Serviços externos | Variado |
| **ASSESSORIA** | Contabilidade, Jurídico | Empresa 1 |
| **INFORMATICA** | Licenças, Suporte | Empresa 1 |
| **PARCELAMENTO** | Refinanciamentos, Empréstimos | Variado |

### Categorias de Receitas

| Cliente | Características | Frequência |
|---------|----------------|------------|
| **MARCOPOLO/SEMAN** | Principal cliente industrial | Semanal/Quinzenal |
| **MARCOPOLO** | Projetos específicos | Eventual |
| **VOLARE** | Cliente secundário | Quinzenal |
| **HIDROVER** | Cliente pontual | Mensal |

---

## Manutenção e Troubleshooting

### Problemas Comuns

#### 1. #REF! Error

**Causa**: Referência a célula/aba deletada

**Solução**:
```excel
# Verificar fórmula
=Fluxo_11_21!U38

# Se aba foi renomeada/deletada, corrigir referência
=Fluxo_11_21_NEW!U38
```

#### 2. VLOOKUP Retorna #N/D

**Causas possíveis**:
- Data não existe na tabela de lookup
- Formato de data inconsistente
- Range da tabela incorreto

**Diagnóstico**:
```excel
# Verificar se a data está na tabela
=COUNTIF($E$2783:$E$2814, E2)    # Deve retornar > 0

# Verificar formato
=TEXT(E2, "dd/mm/yyyy")           # Ver formato atual
```

**Solução**:
- Adicionar data faltante na tabela de lookup
- Padronizar formato de datas
- Ajustar range na fórmula VLOOKUP

#### 3. Saldo Final Incorreto

**Checklist de diagnóstico**:

1. ✓ Verificar se todas as transações foram incluídas
   ```
   Última linha da aba XX_21 = último vencimento do mês?
   ```

2. ✓ Conferir totalizações
   ```excel
   =SUM(B7:B26)  # Range correto?
   ```

3. ✓ Validar propagação de saldo
   ```excel
   Fluxo_11_21 linha final = Fluxo_12_21 célula B4?
   ```

4. ✓ Conferir fórmulas de cálculo de saldo
   ```excel
   Saldo_Final = Saldo_Inicial + Recebimentos - Pagamentos
   ```

#### 4. Performance Lenta

**Sintomas**: Arquivo lento para abrir/salvar/calcular

**Otimizações**:
1. Substituir fórmulas voláteis (NOW, TODAY, RAND)
2. Limitar uso de arrays dinâmicos
3. Usar valores calculados ao invés de fórmulas repetidas
4. Desabilitar cálculo automático durante entrada em massa:
   ```
   Fórmulas → Opções de Cálculo → Manual
   ```

---

## Boas Práticas

### Para Manutenção

1. **Backup Regular**: Salvar cópia antes de grandes mudanças
2. **Documentar Mudanças**: Adicionar comentários em células com lógica complexa
3. **Nomenclatura Consistente**: Manter padrão de nomes de abas e categorias
4. **Validação Periódica**: Conferir saldos semanalmente
5. **Preservar Estrutura**: Não inserir/deletar linhas nas áreas de fórmulas

### Para Expansão

1. **Novo Ano**: Duplicar estrutura completa de 26 abas
2. **Nova Categoria**: Adicionar linha no dashboard e incluir em SUMs
3. **Novo Cliente**: Adicionar linha específica na seção de recebimentos
4. **Novo Centro de Custo**: Atualizar categorias na aba de dados

### Para Análise

1. **Tabelas Dinâmicas**: Criar em nova aba para não interferir em fórmulas
2. **Gráficos**: Usar dados consolidados, não células com fórmulas
3. **Filtros**: Preferir AutoFiltro ao invés de ocultar linhas
4. **Comparação entre Meses**: Usar fórmulas de referência cruzada

---

## Glossário

| Termo | Significado |
|-------|-------------|
| **Centro de Custo** | Divisão contábil (Empresa 1, 2, ou Distribuição de Lucros) |
| **dtvcto** | Data de vencimento |
| **VLOOKUP** | Função de busca vertical |
| **WEEKDAY** | Função que retorna dia da semana |
| **Range Absoluto** | Referência fixa ($A$1) que não muda ao copiar |
| **Range Relativo** | Referência variável (A1) que se ajusta ao copiar |
| **Aba de Dados Brutos** | Contém transações detalhadas (XX_21) |
| **Aba de Dashboard** | Visualização consolidada (Fluxo_XX_21) |

---

## Recursos Adicionais

### Fórmulas Auxiliares Úteis

**Contar transações por categoria:**
```excel
=COUNTIF(C:C, "SALARIOS")
```

**Somar por centro de custo:**
```excel
=SUMIF(G:G, "Empresa 1", F:F)
```

**Encontrar maior despesa do mês:**
```excel
=MAX(F:F)
```

**Média de despesas diárias:**
```excel
=AVERAGE(B27:AF27)    # Linha de total de pagamentos
```

### Templates de Validação

**Verificar integridade de dados:**
```excel
# Células vazias em colunas críticas
=COUNTBLANK(C:C)    # Deve ser 0 ou próximo

# Transações sem categoria
=COUNTIF(G:G, "")   # Deve ser 0

# Valores negativos inesperados  
=COUNTIF(F:F, "<0") # Verificar se esperado
```

---

**Última Atualização**: 2025-01
**Versão da Documentação**: 1.0
