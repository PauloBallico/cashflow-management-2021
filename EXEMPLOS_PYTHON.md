# 🐍 Exemplos de Código Python - Análise de Dados do Fluxo de Caixa

## Introdução

Este documento contém scripts Python prontos para usar com o arquivo `Fluxo_2021.xlsx`, demonstrando capacidade de automação e análise de dados.

---

## 📦 Setup Inicial

### Instalar Bibliotecas Necessárias

```bash
pip install pandas openpyxl matplotlib seaborn plotly numpy scikit-learn
```

### Imports Padrão

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime
import openpyxl

# Configurações
plt.style.use('seaborn-v0_8-darkgrid')
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', 100)
```

---

## 🔧 ETL Básico

### 1. Carregar Dados de Uma Aba

```python
def carregar_aba_dados(arquivo, mes):
    """
    Carrega dados de uma aba específica do fluxo de caixa
    
    Args:
        arquivo (str): Caminho para o arquivo Excel
        mes (str): Mês no formato 'MM_21' (ex: '12_21')
    
    Returns:
        pd.DataFrame: DataFrame com os dados
    """
    
    # Carregar aba
    df = pd.read_excel(
        arquivo, 
        sheet_name=mes,
        header=0
    )
    
    # Renomear colunas para padrão
    df.columns = ['ID', 'TM', 'CodCli', 'Nome', 'dtvcto', 'Valor', 'tipo', 'Pagto']
    
    # Remover linhas vazias
    df = df.dropna(subset=['CodCli', 'Valor'])
    
    # Converter tipos
    df['dtvcto'] = pd.to_datetime(df['dtvcto'], errors='coerce')
    df['Valor'] = pd.to_numeric(df['Valor'], errors='coerce')
    
    # Adicionar coluna de mês para facilitar análises
    df['mes'] = mes.split('_')[0]
    df['ano'] = '2021'
    
    return df

# Uso
df_dezembro = carregar_aba_dados('Fluxo_2021.xlsx', '12_21')
print(df_dezembro.head())
print(f"Total de transações: {len(df_dezembro)}")
```

### 2. Consolidar Dados do Ano Inteiro

```python
def consolidar_ano_completo(arquivo, ano='2021'):
    """
    Consolida todas as abas de dados do ano em um único DataFrame
    
    Args:
        arquivo (str): Caminho para o arquivo Excel
        ano (str): Ano para filtrar (padrão: '2021')
    
    Returns:
        pd.DataFrame: DataFrame consolidado
    """
    
    meses = [f'{str(i).zfill(2)}_21' for i in range(1, 13)]
    
    dfs = []
    for mes in meses:
        try:
            df_mes = carregar_aba_dados(arquivo, mes)
            dfs.append(df_mes)
            print(f"✓ Carregado: {mes} ({len(df_mes)} registros)")
        except Exception as e:
            print(f"✗ Erro ao carregar {mes}: {e}")
    
    # Concatenar todos
    df_completo = pd.concat(dfs, ignore_index=True)
    
    print(f"\n📊 Total consolidado: {len(df_completo)} transações")
    
    return df_completo

# Uso
df_2021 = consolidar_ano_completo('Fluxo_2021.xlsx')
```

### 3. Limpeza e Validação

```python
def limpar_e_validar(df):
    """
    Aplica limpeza e validações nos dados
    
    Args:
        df (pd.DataFrame): DataFrame a limpar
    
    Returns:
        pd.DataFrame: DataFrame limpo
    """
    
    print("🧹 Iniciando limpeza...")
    
    # Estatísticas iniciais
    print(f"Registros iniciais: {len(df)}")
    print(f"Valores nulos:\n{df.isnull().sum()}")
    
    # Remover duplicatas
    duplicatas_antes = len(df)
    df = df.drop_duplicates(subset=['dtvcto', 'Valor', 'CodCli', 'Nome'])
    print(f"Duplicatas removidas: {duplicatas_antes - len(df)}")
    
    # Remover valores nulos críticos
    df = df.dropna(subset=['Valor', 'dtvcto'])
    
    # Validar tipos
    assert df['Valor'].dtype in ['float64', 'int64'], "Coluna Valor deve ser numérica"
    assert pd.api.types.is_datetime64_any_dtype(df['dtvcto']), "Coluna dtvcto deve ser datetime"
    
    # Validar valores
    valores_negativos = df[df['Valor'] < 0]
    if len(valores_negativos) > 0:
        print(f"⚠️ Atenção: {len(valores_negativos)} valores negativos encontrados")
    
    print(f"✅ Limpeza concluída. Registros finais: {len(df)}")
    
    return df

# Uso
df_limpo = limpar_e_validar(df_2021)
```

---

## 📊 Análises Descritivas

### 4. Estatísticas por Categoria

```python
def estatisticas_por_categoria(df):
    """
    Calcula estatísticas descritivas por categoria
    
    Args:
        df (pd.DataFrame): DataFrame com dados
    
    Returns:
        pd.DataFrame: Estatísticas por categoria
    """
    
    stats = df.groupby('CodCli')['Valor'].agg([
        ('Total', 'sum'),
        ('Média', 'mean'),
        ('Mediana', 'median'),
        ('Desvio_Padrão', 'std'),
        ('Mínimo', 'min'),
        ('Máximo', 'max'),
        ('Contagem', 'count')
    ]).round(2)
    
    # Adicionar % do total
    stats['% do Total'] = (stats['Total'] / stats['Total'].sum() * 100).round(2)
    
    # Ordenar por total decrescente
    stats = stats.sort_values('Total', ascending=False)
    
    return stats

# Uso
stats = estatisticas_por_categoria(df_limpo)
print(stats.head(10))

# Exportar para Excel
stats.to_excel('estatisticas_categorias.xlsx')
```

### 5. Análise Temporal

```python
def analise_temporal(df, freq='M'):
    """
    Agrupa dados por período de tempo
    
    Args:
        df (pd.DataFrame): DataFrame com dados
        freq (str): Frequência ('D'=diário, 'W'=semanal, 'M'=mensal)
    
    Returns:
        pd.DataFrame: Dados agregados por período
    """
    
    # Agrupar por data e tipo
    df_temp = df.copy()
    df_temp['periodo'] = df_temp['dtvcto'].dt.to_period(freq)
    
    # Separar receitas e despesas
    receitas = df_temp[df_temp['Valor'] > 0].groupby('periodo')['Valor'].sum()
    despesas = df_temp[df_temp['Valor'] < 0].groupby('periodo')['Valor'].sum().abs()
    
    # Consolidar
    resultado = pd.DataFrame({
        'Receitas': receitas,
        'Despesas': despesas
    }).fillna(0)
    
    resultado['Saldo'] = resultado['Receitas'] - resultado['Despesas']
    resultado['Saldo_Acumulado'] = resultado['Saldo'].cumsum()
    
    return resultado

# Uso
df_mensal = analise_temporal(df_limpo, freq='M')
print(df_mensal)
```

---

## 📈 Visualizações

### 6. Gráfico de Evolução do Saldo

```python
def plot_evolucao_saldo(df_temporal):
    """
    Cria gráfico de linha mostrando evolução do saldo
    
    Args:
        df_temporal (pd.DataFrame): DataFrame com dados temporais
    """
    
    fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(14, 10))
    
    # Gráfico 1: Receitas vs Despesas
    ax1.plot(df_temporal.index.astype(str), df_temporal['Receitas'], 
             marker='o', linewidth=2, label='Receitas', color='green')
    ax1.plot(df_temporal.index.astype(str), df_temporal['Despesas'], 
             marker='s', linewidth=2, label='Despesas', color='red')
    ax1.set_title('Receitas vs Despesas Mensais - 2021', fontsize=16, fontweight='bold')
    ax1.set_xlabel('Mês')
    ax1.set_ylabel('Valor (R$)')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    ax1.tick_params(axis='x', rotation=45)
    
    # Gráfico 2: Saldo Acumulado
    ax2.plot(df_temporal.index.astype(str), df_temporal['Saldo_Acumulado'], 
             marker='D', linewidth=2.5, color='blue', label='Saldo Acumulado')
    ax2.axhline(y=0, color='black', linestyle='--', alpha=0.5)
    ax2.fill_between(range(len(df_temporal)), 
                      df_temporal['Saldo_Acumulado'], 
                      0, alpha=0.3, color='blue')
    ax2.set_title('Saldo Acumulado - 2021', fontsize=16, fontweight='bold')
    ax2.set_xlabel('Mês')
    ax2.set_ylabel('Saldo (R$)')
    ax2.legend()
    ax2.grid(True, alpha=0.3)
    ax2.tick_params(axis='x', rotation=45)
    
    plt.tight_layout()
    plt.savefig('evolucao_saldo.png', dpi=300, bbox_inches='tight')
    plt.show()

# Uso
plot_evolucao_saldo(df_mensal)
```

### 7. Gráfico Pizza - Top Despesas

```python
def plot_top_despesas(df, top_n=10):
    """
    Cria gráfico de pizza com as maiores despesas
    
    Args:
        df (pd.DataFrame): DataFrame com dados
        top_n (int): Número de categorias a mostrar
    """
    
    # Filtrar apenas despesas (valores negativos)
    despesas = df[df['Valor'] < 0].copy()
    despesas['Valor'] = despesas['Valor'].abs()
    
    # Agrupar por categoria
    top_desp = despesas.groupby('CodCli')['Valor'].sum().nlargest(top_n)
    
    # Criar gráfico
    fig, ax = plt.subplots(figsize=(12, 8))
    
    colors = plt.cm.Set3(np.linspace(0, 1, len(top_desp)))
    wedges, texts, autotexts = ax.pie(
        top_desp, 
        labels=top_desp.index,
        autopct='%1.1f%%',
        startangle=90,
        colors=colors,
        textprops={'fontsize': 10}
    )
    
    # Melhorar textos
    for autotext in autotexts:
        autotext.set_color('white')
        autotext.set_fontweight('bold')
    
    ax.set_title(f'Top {top_n} Categorias de Despesas - 2021', 
                 fontsize=16, fontweight='bold')
    
    plt.tight_layout()
    plt.savefig('top_despesas.png', dpi=300, bbox_inches='tight')
    plt.show()
    
    # Retornar valores
    return top_desp

# Uso
top_despesas = plot_top_despesas(df_limpo)
print(top_despesas)
```

### 8. Heatmap de Despesas por Categoria e Mês

```python
def plot_heatmap_despesas(df):
    """
    Cria heatmap mostrando despesas por categoria e mês
    
    Args:
        df (pd.DataFrame): DataFrame com dados
    """
    
    # Filtrar despesas
    despesas = df[df['Valor'] < 0].copy()
    despesas['Valor'] = despesas['Valor'].abs()
    
    # Pivotar dados
    pivot = despesas.pivot_table(
        values='Valor',
        index='CodCli',
        columns='mes',
        aggfunc='sum',
        fill_value=0
    )
    
    # Selecionar top 15 categorias
    top_categorias = despesas.groupby('CodCli')['Valor'].sum().nlargest(15).index
    pivot = pivot.loc[top_categorias]
    
    # Criar heatmap
    plt.figure(figsize=(14, 10))
    sns.heatmap(
        pivot, 
        annot=True, 
        fmt='.0f',
        cmap='YlOrRd',
        linewidths=0.5,
        cbar_kws={'label': 'Valor (R$)'}
    )
    
    plt.title('Heatmap de Despesas por Categoria e Mês - 2021', 
              fontsize=16, fontweight='bold')
    plt.xlabel('Mês')
    plt.ylabel('Categoria')
    plt.tight_layout()
    plt.savefig('heatmap_despesas.png', dpi=300, bbox_inches='tight')
    plt.show()

# Uso
plot_heatmap_despesas(df_limpo)
```

---

## 🎯 Análises Avançadas

### 9. Análise de Outliers

```python
def detectar_outliers(df, n_std=3):
    """
    Detecta outliers usando método de desvio padrão
    
    Args:
        df (pd.DataFrame): DataFrame com dados
        n_std (int): Número de desvios padrão para considerar outlier
    
    Returns:
        pd.DataFrame: DataFrame com outliers detectados
    """
    
    outliers_list = []
    
    for categoria in df['CodCli'].unique():
        df_cat = df[df['CodCli'] == categoria]
        
        if len(df_cat) < 3:
            continue
        
        media = df_cat['Valor'].mean()
        std = df_cat['Valor'].std()
        
        # Identificar outliers
        limite_superior = media + n_std * std
        limite_inferior = media - n_std * std
        
        outliers = df_cat[
            (df_cat['Valor'] > limite_superior) | 
            (df_cat['Valor'] < limite_inferior)
        ].copy()
        
        if len(outliers) > 0:
            outliers['z_score'] = ((outliers['Valor'] - media) / std).abs()
            outliers_list.append(outliers)
    
    if outliers_list:
        df_outliers = pd.concat(outliers_list, ignore_index=True)
        df_outliers = df_outliers.sort_values('z_score', ascending=False)
        return df_outliers
    else:
        return pd.DataFrame()

# Uso
outliers = detectar_outliers(df_limpo)
print(f"Outliers detectados: {len(outliers)}")
print(outliers[['CodCli', 'Nome', 'Valor', 'z_score']].head(10))
```

### 10. Análise de Correlação

```python
def analise_correlacao(df):
    """
    Calcula correlação entre receitas e despesas ao longo do tempo
    
    Args:
        df (pd.DataFrame): DataFrame com dados
    
    Returns:
        dict: Dicionário com métricas de correlação
    """
    
    # Agregar por mês
    df_temp = df.copy()
    df_temp['mes_num'] = pd.to_datetime(df_temp['dtvcto']).dt.month
    
    # Separar receitas e despesas
    df_rec = df_temp[df_temp['Valor'] > 0].groupby('mes_num')['Valor'].sum()
    df_desp = df_temp[df_temp['Valor'] < 0].groupby('mes_num')['Valor'].sum().abs()
    
    # Criar DataFrame consolidado
    df_corr = pd.DataFrame({
        'mes': range(1, 13),
        'receitas': df_rec,
        'despesas': df_desp
    }).fillna(0)
    
    # Calcular correlação
    correlacao = df_corr['receitas'].corr(df_corr['despesas'])
    
    # Plotar
    fig, ax = plt.subplots(figsize=(10, 6))
    
    ax.scatter(df_corr['receitas'], df_corr['despesas'], 
               s=100, alpha=0.6, color='blue')
    
    # Linha de tendência
    z = np.polyfit(df_corr['receitas'], df_corr['despesas'], 1)
    p = np.poly1d(z)
    ax.plot(df_corr['receitas'], p(df_corr['receitas']), 
            "r--", alpha=0.8, label=f'Tendência (r={correlacao:.2f})')
    
    ax.set_xlabel('Receitas (R$)')
    ax.set_ylabel('Despesas (R$)')
    ax.set_title('Correlação Receitas vs Despesas', fontsize=14, fontweight='bold')
    ax.legend()
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig('correlacao_receitas_despesas.png', dpi=300, bbox_inches='tight')
    plt.show()
    
    return {
        'correlacao': correlacao,
        'coef_angular': z[0],
        'intercepto': z[1]
    }

# Uso
metricas = analise_correlacao(df_limpo)
print(f"Correlação: {metricas['correlacao']:.3f}")
print(f"Para cada R$ 1 de receita, despesas aumentam: R$ {metricas['coef_angular']:.2f}")
```

### 11. Previsão Simples (Média Móvel)

```python
def previsao_media_movel(df, janela=3, periodos_futuro=3):
    """
    Faz previsão usando média móvel
    
    Args:
        df (pd.DataFrame): DataFrame com dados temporais
        janela (int): Tamanho da janela para média móvel
        periodos_futuro (int): Quantos períodos prever
    
    Returns:
        pd.DataFrame: DataFrame com previsões
    """
    
    # Calcular médias móveis
    df = df.copy()
    df['MA_Receitas'] = df['Receitas'].rolling(window=janela).mean()
    df['MA_Despesas'] = df['Despesas'].rolling(window=janela).mean()
    
    # Fazer previsões
    previsoes = []
    for i in range(periodos_futuro):
        prox_receita = df['MA_Receitas'].iloc[-1]
        prox_despesa = df['MA_Despesas'].iloc[-1]
        prox_saldo = prox_receita - prox_despesa
        
        previsoes.append({
            'periodo': f'Previsão +{i+1}',
            'Receitas': prox_receita,
            'Despesas': prox_despesa,
            'Saldo': prox_saldo
        })
    
    df_prev = pd.DataFrame(previsoes)
    
    # Plotar
    fig, ax = plt.subplots(figsize=(12, 6))
    
    # Dados históricos
    ax.plot(range(len(df)), df['Receitas'], 
            marker='o', label='Receitas (Real)', color='green')
    ax.plot(range(len(df)), df['Despesas'], 
            marker='s', label='Despesas (Real)', color='red')
    
    # Previsões
    inicio_prev = len(df)
    ax.plot(range(inicio_prev, inicio_prev + len(df_prev)), df_prev['Receitas'],
            marker='o', linestyle='--', label='Receitas (Previsto)', 
            color='lightgreen', linewidth=2)
    ax.plot(range(inicio_prev, inicio_prev + len(df_prev)), df_prev['Despesas'],
            marker='s', linestyle='--', label='Despesas (Previsto)', 
            color='lightcoral', linewidth=2)
    
    ax.axvline(x=inicio_prev-0.5, color='black', linestyle=':', alpha=0.5)
    ax.text(inicio_prev-0.5, ax.get_ylim()[1]*0.9, 'Previsão →', 
            fontsize=12, fontweight='bold')
    
    ax.set_xlabel('Período')
    ax.set_ylabel('Valor (R$)')
    ax.set_title('Previsão de Receitas e Despesas (Média Móvel)', 
                 fontsize=14, fontweight='bold')
    ax.legend()
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig('previsao_media_movel.png', dpi=300, bbox_inches='tight')
    plt.show()
    
    return df_prev

# Uso
df_temporal = analise_temporal(df_limpo)
previsoes = previsao_media_movel(df_temporal, janela=3, periodos_futuro=3)
print(previsoes)
```

---

## 📄 Relatórios Automatizados

### 12. Gerar Relatório Completo em PDF

```python
from matplotlib.backends.backend_pdf import PdfPages

def gerar_relatorio_pdf(df, nome_arquivo='relatorio_financeiro.pdf'):
    """
    Gera relatório completo em PDF com todas as análises
    
    Args:
        df (pd.DataFrame): DataFrame com dados
        nome_arquivo (str): Nome do arquivo PDF a criar
    """
    
    with PdfPages(nome_arquivo) as pdf:
        
        # Página 1: Estatísticas Gerais
        fig, ax = plt.subplots(figsize=(11, 8.5))
        ax.axis('off')
        
        stats_text = f"""
        RELATÓRIO FINANCEIRO - 2021
        ===========================
        
        Total de Transações: {len(df):,}
        Período: {df['dtvcto'].min().strftime('%d/%m/%Y')} a {df['dtvcto'].max().strftime('%d/%m/%Y')}
        
        RECEITAS:
        Total: R$ {df[df['Valor'] > 0]['Valor'].sum():,.2f}
        Média Mensal: R$ {df[df['Valor'] > 0]['Valor'].sum() / 12:,.2f}
        
        DESPESAS:
        Total: R$ {df[df['Valor'] < 0]['Valor'].sum():,.2f}
        Média Mensal: R$ {df[df['Valor'] < 0]['Valor'].sum() / 12:,.2f}
        
        RESULTADO:
        Saldo: R$ {df['Valor'].sum():,.2f}
        Margem: {(df['Valor'].sum() / df[df['Valor'] > 0]['Valor'].sum() * 100):.2f}%
        """
        
        ax.text(0.1, 0.5, stats_text, fontsize=12, fontfamily='monospace',
                verticalalignment='center')
        pdf.savefig(fig, bbox_inches='tight')
        plt.close()
        
        # Página 2: Gráfico de evolução
        df_temporal = analise_temporal(df)
        plot_evolucao_saldo(df_temporal)
        pdf.savefig(bbox_inches='tight')
        plt.close()
        
        # Página 3: Top despesas
        plot_top_despesas(df)
        pdf.savefig(bbox_inches='tight')
        plt.close()
        
        print(f"✅ Relatório gerado: {nome_arquivo}")

# Uso
gerar_relatorio_pdf(df_limpo)
```

---

## 🚀 Script Completo de Análise

### 13. Pipeline Completo

```python
def pipeline_analise_completa(arquivo_excel):
    """
    Executa pipeline completo de análise
    
    Args:
        arquivo_excel (str): Caminho para o arquivo Excel
    """
    
    print("=" * 60)
    print("INICIANDO ANÁLISE COMPLETA DO FLUXO DE CAIXA")
    print("=" * 60)
    
    # 1. Carregar dados
    print("\n[1/6] Carregando dados...")
    df = consolidar_ano_completo(arquivo_excel)
    
    # 2. Limpar dados
    print("\n[2/6] Limpando dados...")
    df = limpar_e_validar(df)
    
    # 3. Estatísticas descritivas
    print("\n[3/6] Calculando estatísticas...")
    stats = estatisticas_por_categoria(df)
    stats.to_excel('output_estatisticas.xlsx')
    print("Salvo: output_estatisticas.xlsx")
    
    # 4. Análise temporal
    print("\n[4/6] Análise temporal...")
    df_temporal = analise_temporal(df, freq='M')
    df_temporal.to_excel('output_evolucao_mensal.xlsx')
    print("Salvo: output_evolucao_mensal.xlsx")
    
    # 5. Visualizações
    print("\n[5/6] Gerando visualizações...")
    plot_evolucao_saldo(df_temporal)
    plot_top_despesas(df)
    plot_heatmap_despesas(df)
    print("Salvo: evolucao_saldo.png, top_despesas.png, heatmap_despesas.png")
    
    # 6. Relatório PDF
    print("\n[6/6] Gerando relatório PDF...")
    gerar_relatorio_pdf(df)
    
    print("\n" + "=" * 60)
    print("✅ ANÁLISE CONCLUÍDA COM SUCESSO!")
    print("=" * 60)
    
    return df, stats, df_temporal

# Executar pipeline completo
if __name__ == "__main__":
    df, stats, df_temporal = pipeline_analise_completa('Fluxo_2021.xlsx')
```

---

## 💡 Dicas e Best Practices

### Performance

```python
# Use chunks para arquivos grandes
for chunk in pd.read_excel('arquivo.xlsx', chunksize=1000):
    processar(chunk)

# Cache de dados
import pickle

# Salvar
with open('df_cache.pkl', 'wb') as f:
    pickle.dump(df, f)

# Carregar
with open('df_cache.pkl', 'rb') as f:
    df = pickle.load(f)
```

### Logs

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('analise.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)
logger.info("Iniciando análise...")
```

### Tratamento de Erros

```python
try:
    df = pd.read_excel('arquivo.xlsx')
except FileNotFoundError:
    print("❌ Erro: Arquivo não encontrado")
    exit(1)
except Exception as e:
    print(f"❌ Erro inesperado: {e}")
    exit(1)
```

---

## 📚 Recursos Adicionais

### Documentação
- Pandas: https://pandas.pydata.org/docs/
- Matplotlib: https://matplotlib.org/stable/contents.html
- Seaborn: https://seaborn.pydata.org/
- Plotly: https://plotly.com/python/

### Tutoriais
- Real Python: https://realpython.com/pandas-python-explore-dataset/
- DataCamp: Python for Finance
- Kaggle: Learn Pandas

---

**Última Atualização**: 2025-01-27  
**Python Version**: 3.8+  
**Dependências**: pandas, numpy, matplotlib, seaborn, openpyxl
