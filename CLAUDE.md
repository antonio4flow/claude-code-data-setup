# Projeto de Análise de Dados

## Contexto do Projeto

Pipeline de análise de dados tabulares (CSV/Excel) que cobre desde a ingestão e validação da qualidade até a geração de relatórios executivos e visualizações. O foco é produzir insights acionáveis a partir de dados brutos, com rastreabilidade completa de cada etapa do processo.

## Stack Técnica

| Camada | Tecnologia |
|---|---|
| Linguagem | Python 3.11+ |
| Manipulação de dados | pandas, numpy |
| Visualizações | matplotlib, seaborn, plotly |
| Notebooks | Jupyter |
| Relatórios | Markdown, HTML |
| Qualidade de dados | great_expectations (opcional), pandas-profiling |

## Estrutura de Pastas

```
/
├── CLAUDE.md
├── notebooks/           # Análises exploratórias e experimentação
├── data/
│   ├── raw/             # Dados brutos — NUNCA modificar diretamente
│   └── outputs/
│       ├── eda/         # Gráficos e summaries de EDA
│       └── reports/     # Relatórios finais exportados
├── src/                 # Módulos Python reutilizáveis
│   ├── ingestion.py     # Leitura e normalização de fontes
│   ├── cleaning.py      # Transformações e limpeza
│   ├── analysis.py      # Funções de análise estatística
│   └── visualization.py # Funções de plotagem padronizadas
└── .claude/
    ├── skills/          # Skills automatizadas
    └── agents/          # Personas especializadas
```

## Convenções de Código

### Nomenclatura
- Variáveis e funções: `snake_case` (ex: `calcular_media_movel`)
- Classes: `PascalCase` (ex: `RelatorioMensal`)
- Constantes: `UPPER_SNAKE_CASE` (ex: `CAMINHO_DADOS_BRUTOS`)
- Arquivos: `snake_case.py`

### Comentários
- **Todos os comentários e docstrings em português**
- Docstrings em funções que recebem parâmetros não-óbvios
- Comentar o "por quê", não o "o quê"

### Tipagem
- Usar type hints sempre que possível em funções novas
- `pd.DataFrame` para DataFrames, `pd.Series` para Series
- Exemplo:
  ```python
  def calcular_percentual_nulos(df: pd.DataFrame) -> pd.Series:
      """Retorna a porcentagem de valores nulos por coluna."""
      return (df.isnull().sum() / len(df)) * 100
  ```

### DataFrames
- Nunca usar `inplace=True` — preferir reatribuição explícita
- Não modificar `data/raw/` — sempre criar cópia antes de transformar
- Nomear DataFrames intermediários com sufixo descritivo: `df_limpo`, `df_agregado`

## Comandos Mais Usados

```bash
# Iniciar Jupyter
jupyter notebook notebooks/

# Rodar análise de qualidade num arquivo
python -c "import pandas as pd; df = pd.read_csv('data/raw/arquivo.csv'); print(df.info()); print(df.describe())"

# Ver estrutura de um CSV rapidamente
python -c "import pandas as pd; print(pd.read_csv('data/raw/arquivo.csv').head())"

# Instalar dependências
pip install pandas numpy matplotlib seaborn plotly jupyter openpyxl
```

## Fluxo de Trabalho Padrão

1. **Ingestão** → Colocar dado bruto em `data/raw/`
2. **Qualidade** → Acionar skill `data-quality` para validação inicial
3. **EDA** → Acionar skill `eda` para análise exploratória
4. **Análise** → Desenvolver análise específica no notebook
5. **Revisão** → Acionar agente `data-reviewer` para revisão do código
6. **Relatório** → Acionar agente `report-writer` para output executivo

## Regras Importantes

- `data/raw/` é somente leitura — nunca salvar arquivos modificados lá
- Todo output de análise vai em `data/outputs/`
- Gráficos salvos com `dpi=150` mínimo e `bbox_inches='tight'`
- Ao tratar outliers, **sempre documentar** o critério usado (IQR, Z-score, domínio)
