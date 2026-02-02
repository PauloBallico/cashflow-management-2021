# 📚 Guia de Uso - Sistema de Fluxo de Caixa

## Início Rápido

### Para Recrutadores e Avaliadores

Quer entender rapidamente o projeto? Siga este caminho:

1. **Abra o arquivo** `Fluxo_2021.xlsx`
2. **Vá para a aba** `Fluxo_12_21` (última aba de dashboard)
3. **Observe**:
   - Linha 1: Dias da semana de dezembro
   - Linha 4: Saldo inicial (vindo de novembro)
   - Linhas 6-26: Recebimentos diários por cliente
   - Linhas 27+: Pagamentos diários por categoria
   - Última linha: Saldo projetado dia a dia

4. **Clique em qualquer célula com fórmula** para ver a lógica aplicada

5. **Compare com a aba** `12_21` para ver os dados brutos que alimentam o dashboard

**Tempo estimado**: 5-10 minutos

---

## Navegação por Perfil de Usuário

### 👔 Executivo / Gestor

**Objetivo**: Visão rápida da saúde financeira

**Onde ir**:
- Abas `Fluxo_XX_21` → Linha de saldo final
- Comparar saldos entre meses

**O que observar**:
```
Saldo Final Dezembro vs Janeiro = Fluxo anual
Dias com saldo negativo = Períodos críticos
Maiores recebimentos = Principais clientes
Maiores categorias de despesa = Oportunidades de corte
```

**Análise recomendada**:
1. Traçar gráfico do saldo final mensal (linha temporal)
2. Identificar meses com maior pressão de caixa
3. Avaliar sazonalidade de recebimentos

---

### 💼 Analista Financeiro

**Objetivo**: Análise detalhada e reconciliação

**Onde ir**:
- Abas `XX_21` para dados transacionais
- Usar filtros e tabelas dinâmicas

**Análises sugeridas**:

#### 1. Análise por Centro de Custo
```excel
# Na aba 12_21
Filtrar coluna G (tipo):
- Empresa 1
- Empresa 2  
- Distribuição de Lucros

Usar Tabela Dinâmica:
Linhas: tipo
Valores: Soma de Valor
```

#### 2. Top 10 Despesas
```excel
1. Ordenar coluna F (Valor) decrescente
2. Selecionar top 10
3. Analisar categorias (coluna C)
```

#### 3. Frequência de Pagamentos por Categoria
```excel
=COUNTIFS(C:C, "SALARIOS", E:E, ">=01/12/2021", E:E, "<=31/12/2021")
```

#### 4. Comparativo Mensal
```excel
# Criar nova aba "Análise"
# Consolidar saldos finais de todos os meses
Mês | Saldo Inicial | Recebimentos | Pagamentos | Saldo Final
```

---

### 💻 Analista de Dados / Cientista de Dados

**Objetivo**: Extração e transformação de dados

**Ferramentas recomendadas**:
- Python + Pandas
- Power BI
- SQL (importar para banco)

#### Exemplo de ETL com Python

```python
import pandas as pd
import openpyxl

# Carregar arquivo
wb = openpyxl.load_workbook('Fluxo_2021.xlsx', data_only=True)

# Extrair dados de uma aba específica
ws = wb['12_21']

# Converter para DataFrame
data = []
for row in ws.iter_rows(min_row=2, values_only=True):
    data.append(row)

df = pd.DataFrame(data, columns=['ID', 'tm', 'CodCli', 'Nome', 
                                  'dtvcto', 'Valor', 'tipo', 'Pagto'])

# Limpeza e transformação
df['dtvcto'] = pd.to_datetime(df['dtvcto'])
df['Valor'] = pd.to_numeric(df['Valor'], errors='coerce')

# Análises
print(df.groupby('tipo')['Valor'].sum())
print(df.groupby('CodCli')['Valor'].sum().sort_values(ascending=False))

# Exportar para CSV
df.to_csv('transacoes_2021.csv', index=False)
```

#### Power BI

**Passos**:
1. Obter Dados → Excel → Selecionar arquivo
2. Escolher abas XX_21 (dados brutos)
3. Transformar:
   - Remover linhas de cabeçalho duplicadas
   - Converter tipos de dados
   - Criar coluna de mês (extrair de data)
4. Criar relacionamentos entre tabelas (se necessário)
5. Construir visualizações:
   - Gráfico de linha: Saldo ao longo do tempo
   - Gráfico de pizza: Despesas por categoria
   - Tabela: Top clientes por receita
   - KPI: Saldo atual vs meta

---

### 🔧 Desenvolvedor / Automação

**Objetivo**: Automatizar processos e criar ferramentas

#### Script de Validação

```python
def validar_fluxo(arquivo):
    """
    Valida integridade do arquivo de fluxo de caixa
    """
    wb = openpyxl.load_workbook(arquivo)
    issues = []
    
    # Verificar se todas as abas existem
    expected_sheets = [f'{str(i).zfill(2)}_21' for i in range(1, 13)] + \
                      [f'Fluxo_{str(i).zfill(2)}_21' for i in range(1, 13)]
    
    for sheet in expected_sheets:
        if sheet not in wb.sheetnames:
            issues.append(f"Aba ausente: {sheet}")
    
    # Verificar continuidade de saldos
    for i in range(1, 12):
        current_sheet = f'Fluxo_{str(i).zfill(2)}_21'
        next_sheet = f'Fluxo_{str(i+1).zfill(2)}_21'
        
        # Lógica para verificar se saldo final = saldo inicial próximo mês
        # (simplificado)
        
    if issues:
        print(f"❌ Encontradas {len(issues)} inconsistências:")
        for issue in issues:
            print(f"  - {issue}")
    else:
        print("✅ Arquivo validado com sucesso!")
    
    return len(issues) == 0

# Uso
validar_fluxo('Fluxo_2021.xlsx')
```

#### Dashboard Web (Flask + Plotly)

```python
from flask import Flask, render_template
import pandas as pd
import plotly.express as px

app = Flask(__name__)

# Carregar dados (executar uma vez)
df = carregar_dados_excel('Fluxo_2021.xlsx')

@app.route('/')
def dashboard():
    # Gráfico de saldo mensal
    saldos_mensais = calcular_saldos_mensais(df)
    fig1 = px.line(saldos_mensais, x='mes', y='saldo', 
                   title='Evolução do Saldo 2021')
    
    # Gráfico de despesas por categoria
    despesas = df.groupby('CodCli')['Valor'].sum().reset_index()
    fig2 = px.bar(despesas, x='CodCli', y='Valor',
                  title='Despesas por Categoria')
    
    return render_template('dashboard.html', 
                          grafico_saldo=fig1.to_html(),
                          grafico_despesas=fig2.to_html())

if __name__ == '__main__':
    app.run(debug=True)
```

---

## Casos de Uso Práticos

### Caso 1: Planejamento de Fluxo de Caixa para Próximo Mês

**Cenário**: Projetar necessidades de caixa para o próximo mês

**Passos**:
1. Analisar histórico dos últimos 3 meses (abas Fluxo_10_21, 11_21, 12_21)
2. Identificar despesas fixas (salários, encargos, aluguel)
3. Estimar receitas baseado em carteira de pedidos
4. Criar projeção no modelo:

```
Saldo Inicial (mês atual linha final)
+ Receitas Estimadas
- Despesas Fixas
- Despesas Variáveis Estimadas (média dos últimos 3 meses)
= Saldo Projetado
```

5. Identificar gaps de caixa
6. Planejar fontes de recursos se necessário

---

### Caso 2: Análise de Lucratividade por Cliente

**Objetivo**: Entender qual cliente é mais rentável

**Método**:
1. Criar nova aba "Análise_Clientes"
2. Listar clientes principais (Marcopolo, Volare, Hidrover)
3. Para cada cliente:
   ```excel
   Receita Total = SUMIF em todas abas XX_21, coluna C = nome cliente
   ```
4. Estimar custos diretos por cliente (se possível categorizar)
5. Calcular margem:
   ```
   Margem = (Receita - Custos Diretos) / Receita * 100
   ```

**Insights esperados**:
- Cliente com maior volume ≠ cliente mais rentável
- Custos de servir variam significativamente
- Base para estratégia comercial

---

### Caso 3: Preparação para Auditoria

**Necessidade**: Demonstrar controle financeiro robusto

**Checklist**:
- [x] Conferir reconciliação bancária (linha 2 vs extratos)
- [x] Validar categorização de despesas
- [x] Verificar se todos os comprovantes estão arquivados
- [x] Confirmar centros de custo corretos
- [x] Analisar transações acima de R$ 10.000
- [x] Documentar variações significativas mês a mês

**Documentos auxiliares**:
1. Extratos bancários digitalizados
2. Notas fiscais organizadas por mês
3. Contratos de fornecedores principais
4. Acordos de parcelamento
5. Relatório de reconciliação mensal

---

### Caso 4: Identificação de Oportunidades de Redução de Custos

**Método**:
1. Gerar Tabela Dinâmica:
   - Linhas: CodCli (categoria)
   - Valores: Soma de Valor
   - Filtro: tipo = "Empresa 1" ou "Empresa 2"

2. Ordenar por valor decrescente

3. Aplicar análise ABC:
   - Categoria A: 80% dos gastos (focar aqui)
   - Categoria B: 15% dos gastos
   - Categoria C: 5% dos gastos

4. Para cada categoria A:
   - Analisar tendência ao longo do ano
   - Comparar com benchmark de mercado
   - Identificar variações anormais
   - Avaliar negociações com fornecedores

**Exemplo**:
```
Top 5 Despesas 2021:
1. SALARIOS: R$ 1.200.000 (35%)
   → Avaliar produtividade, considerar terceirização
2. MATERIAL: R$ 800.000 (24%)
   → Negociar volume com fornecedor, buscar alternativas
3. COMBUSTIVEL: R$ 300.000 (9%)
   → Otimizar rotas, avaliar cartão corporativo
...
```

---

## FAQ - Perguntas Frequentes

### Sobre Funcionalidades

**P: Como adicionar um novo mês?**
R: Copie as abas XX_21 e Fluxo_XX_21 de um mês existente. Renomeie para o novo mês. Ajuste referências de saldo inicial.

**P: Posso adicionar novas categorias de despesa?**
R: Sim. Adicione linhas nas abas XX_21 com a nova categoria na coluna C. Adicione linha correspondente no dashboard Fluxo_XX_21.

**P: Como exportar dados para outro formato?**
R: Salvar Como → CSV (para dados tabulares) ou usar script Python para conversão automatizada.

### Sobre Interpretação

**P: O que significa saldo negativo?**
R: Projeção indica que despesas superarão receitas naquele dia. Necessário aporte de capital ou renegociação de prazos.

**P: Por que há diferença entre "Localizado no banco" e saldo calculado?**
R: Possíveis causas: transações não lançadas, erros de categorização, pagamentos/recebimentos pendentes. Necessita reconciliação.

**P: Como interpretar os centros de custo?**
R:
- Empresa 1: Despesas administrativas e overhead
- Empresa 2: Despesas operacionais e produção
- Distribuição de Lucros: Retiradas dos sócios

### Sobre Problemas

**P: Fórmulas retornam #REF!**
R: Célula ou aba referenciada foi deletada/movida. Verifique dependências e corrija referências.

**P: VLOOKUP retorna #N/D**
R: Data não encontrada na tabela de lookup. Verifique se tabela está completa e formato de datas está consistente.

**P: Arquivo está lento**
R: Muitas fórmulas voláteis ou arquivo muito grande. Considere desabilitar cálculo automático temporariamente ou otimizar fórmulas.

---

## Recursos Adicionais

### Tutoriais Recomendados

- **Excel Avançado**: 
  - VLOOKUP e HLOOKUP
  - Fórmulas de Data e Hora
  - IF aninhados
  - Tabelas Dinâmicas

- **Power BI**:
  - Importação de dados do Excel
  - Transformação de dados (Power Query)
  - Criação de visualizações
  - DAX básico para cálculos

- **Python para Finanças**:
  - Pandas para análise de dados
  - Openpyxl para manipulação de Excel
  - Matplotlib/Plotly para visualizações
  - Numpy para cálculos financeiros

### Templates e Ferramentas

1. **Template de Análise Mensal**: [Link placeholder]
2. **Script de Validação**: [Link placeholder]
3. **Dashboard Power BI**: [Link placeholder]
4. **Notebook Jupyter com Análises**: [Link placeholder]

### Comunidade e Suporte

- Dúvidas sobre Excel: Stack Overflow, r/excel
- Análise de Dados: Kaggle, Medium
- Automação: GitHub, Dev.to

---

## Próximos Passos

### Para Usuários Iniciantes
1. ✅ Ler este guia completamente
2. ✅ Abrir arquivo e navegar pelas abas
3. ✅ Entender estrutura de uma aba de dados
4. ✅ Analisar uma aba de dashboard
5. ✅ Criar primeira análise simples (soma por categoria)

### Para Usuários Intermediários
1. ✅ Dominar fórmulas principais (VLOOKUP, SUMIF)
2. ✅ Criar tabelas dinâmicas
3. ✅ Gerar relatórios mensais automatizados
4. ✅ Construir gráficos de tendência
5. ✅ Implementar validações adicionais

### Para Usuários Avançados
1. ✅ Automatizar com Python/VBA
2. ✅ Integrar com Power BI
3. ✅ Criar pipeline de ETL
4. ✅ Desenvolver previsões (forecasting)
5. ✅ Implementar dashboards em tempo real

---

**Última Atualização**: 2025-01  
**Versão do Guia**: 1.0  
**Compatibilidade**: Excel 2016+, LibreOffice Calc, Google Sheets (limitado)
