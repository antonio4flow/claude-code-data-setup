# Skill: Análise Exploratória de Dados (EDA)

## Anúncio obrigatório
Sempre inicie sua resposta com este bloco exato, antes de qualquer análise:

---
📊 **[SKILL: eda]** Ativada
Data-quality concluída. Iniciando análise exploratória...
Outputs serão salvos em: data/outputs/eda/
---

## Quando Acionar

Acionar esta skill automaticamente quando:
- O usuário mencionar um novo dataset, arquivo CSV ou Excel
- O usuário disser "analisa esse arquivo", "o que tem nesse dataset", "faz uma EDA"
- Um novo arquivo aparecer em `data/raw/`

---

## O Que Esta Skill Faz

Executa uma análise exploratória completa e estruturada, gerando:
1. Um **summary em Markdown** com todas as métricas relevantes
2. **Gráficos salvos** em `data/outputs/eda/`
3. **Alertas** sobre problemas encontrados (nulos, outliers, distribuições estranhas)

---

## Protocolo de Execução

### Etapa 1 — Carregamento e Inspeção Inicial

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path

# Configuração de output
OUTPUT_DIR = Path("data/outputs/eda")
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

# Carregamento — detectar CSV ou Excel
caminho = Path("data/raw/NOME_DO_ARQUIVO")
if caminho.suffix == ".csv":
    df = pd.read_csv(caminho)
else:
    df = pd.read_excel(caminho)

print(f"Shape: {df.shape}")
print(f"Colunas: {df.columns.tolist()}")
print(df.dtypes)
print(df.head())
```

### Etapa 2 — Métricas de Qualidade

```python
# Nulos
nulos = df.isnull().sum()
pct_nulos = (nulos / len(df) * 100).round(2)

# Duplicatas
n_duplicatas = df.duplicated().sum()

# Cardinalidade por coluna
cardinalidade = df.nunique()

resumo_qualidade = pd.DataFrame({
    "tipo": df.dtypes,
    "nulos": nulos,
    "pct_nulos": pct_nulos,
    "valores_unicos": cardinalidade
})
print(resumo_qualidade)
```

### Etapa 3 — Estatísticas Descritivas

```python
# Numéricas
print(df.describe().round(2))

# Categóricas — top valores
for col in df.select_dtypes(include="object").columns:
    print(f"\n--- {col} ---")
    print(df[col].value_counts().head(10))
```

### Etapa 4 — Visualizações

```python
sns.set_theme(style="whitegrid", palette="muted")

# 1. Mapa de calor de nulos
plt.figure(figsize=(12, 6))
sns.heatmap(df.isnull(), cbar=False, yticklabels=False, cmap="viridis")
plt.title("Mapa de Valores Nulos")
plt.tight_layout()
plt.savefig(OUTPUT_DIR / "mapa_nulos.png", dpi=150, bbox_inches="tight")
plt.close()

# 2. Distribuição de variáveis numéricas
colunas_numericas = df.select_dtypes(include=np.number).columns.tolist()
if colunas_numericas:
    n_cols = min(3, len(colunas_numericas))
    n_rows = (len(colunas_numericas) + n_cols - 1) // n_cols
    fig, axes = plt.subplots(n_rows, n_cols, figsize=(6 * n_cols, 4 * n_rows))
    axes = np.array(axes).flatten()
    for i, col in enumerate(colunas_numericas):
        sns.histplot(df[col].dropna(), kde=True, ax=axes[i])
        axes[i].set_title(f"Distribuição: {col}")
    for j in range(i + 1, len(axes)):
        axes[j].set_visible(False)
    plt.tight_layout()
    plt.savefig(OUTPUT_DIR / "distribuicoes_numericas.png", dpi=150, bbox_inches="tight")
    plt.close()

# 3. Matriz de correlação
if len(colunas_numericas) >= 2:
    plt.figure(figsize=(10, 8))
    corr = df[colunas_numericas].corr()
    mask = np.triu(np.ones_like(corr, dtype=bool))
    sns.heatmap(corr, mask=mask, annot=True, fmt=".2f", cmap="coolwarm", center=0)
    plt.title("Matriz de Correlação")
    plt.tight_layout()
    plt.savefig(OUTPUT_DIR / "correlacao.png", dpi=150, bbox_inches="tight")
    plt.close()

# 4. Boxplots para detectar outliers
if colunas_numericas:
    fig, axes = plt.subplots(1, len(colunas_numericas), figsize=(5 * len(colunas_numericas), 5))
    if len(colunas_numericas) == 1:
        axes = [axes]
    for i, col in enumerate(colunas_numericas):
        sns.boxplot(y=df[col].dropna(), ax=axes[i])
        axes[i].set_title(col)
    plt.tight_layout()
    plt.savefig(OUTPUT_DIR / "boxplots_outliers.png", dpi=150, bbox_inches="tight")
    plt.close()
```

---

## Formato do Summary em Markdown

Após executar o código, gerar o seguinte relatório:

```markdown
# EDA — [Nome do Arquivo] — [Data]

## Visão Geral
- **Linhas:** X | **Colunas:** Y
- **Duplicatas:** Z linhas (W%)
- **Período:** [se houver coluna de data]

## Qualidade dos Dados
| Coluna | Tipo | Nulos | % Nulos | Valores Únicos |
|--------|------|-------|---------|----------------|
| ...    | ...  | ...   | ...     | ...            |

⚠️ **Alertas:**
- [Coluna X] tem Y% de nulos — avaliar imputação ou exclusão
- [Coluna Y] tem cardinalidade muito alta (Z valores) — verificar se é ID

## Estatísticas Descritivas
[Tabela com describe()]

## Correlações Notáveis
- [Coluna A] × [Coluna B]: r = 0.87 — forte correlação positiva
- ...

## Gráficos Gerados
- `data/outputs/eda/mapa_nulos.png`
- `data/outputs/eda/distribuicoes_numericas.png`
- `data/outputs/eda/correlacao.png`
- `data/outputs/eda/boxplots_outliers.png`

## Próximos Passos Sugeridos
1. Tratar nulos em [colunas com > 5% de nulos]
2. Investigar outliers em [colunas com boxplots anômalos]
3. [Observação específica do dataset]
```

---

## Comportamento Esperado do Agente

- Adaptar o código ao dataset real (não executar cegamente com nomes fixos)
- Detectar automaticamente colunas de data e calcular range temporal
- Se o dataset tiver > 50 colunas, priorizar as numéricas e as com mais nulos
- Sempre terminar com "Próximos Passos Sugeridos" baseados no que foi encontrado
