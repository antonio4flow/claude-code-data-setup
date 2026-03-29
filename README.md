# claude-code-data-setup

Template de configuração profissional do Claude Code para projetos de análise de dados tabulares. Inclui skills automatizadas, agentes especializados e memória persistente entre sessões.

---

## O que é este projeto

Este repositório é um **template pronto para uso** que configura o Claude Code como um assistente de análise de dados com comportamentos definidos. Em vez de explicar o que fazer a cada sessão, você define uma vez como o Claude deve se comportar — e ele segue esse padrão automaticamente.

O que vem configurado:
- **2 skills** que rodam automaticamente ao receber um novo dataset (qualidade de dados + EDA)
- **2 agentes** especializados para revisão técnica e redação de relatórios executivos
- **Memória persistente** via claude-mem para manter contexto entre sessões
- **Convenções de código** para Python/pandas documentadas no `CLAUDE.md`

Ideal para analistas que trabalham com CSVs e Excel e precisam de um fluxo repetível: ingestão → validação → exploração → análise → relatório.

---

## Pré-requisitos

| Requisito | Versão mínima | Como verificar |
|---|---|---|
| Claude Code | Mais recente | `claude --version` |
| Python | 3.11+ | `python --version` |
| Node.js | 18+ | `node --version` |
| Git | Qualquer | `git --version` |

Bibliotecas Python usadas nas análises:

```bash
pip install pandas numpy matplotlib seaborn plotly jupyter openpyxl
```

---

## Como clonar e usar

### 1. Clone o repositório

```bash
git clone https://github.com/seu-usuario/claude-code-data-setup.git meu-projeto
cd meu-projeto
```

### 2. Instale as dependências Python

```bash
pip install pandas numpy matplotlib seaborn plotly jupyter openpyxl
```

### 3. Instale o claude-mem (memória persistente)

```bash
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

> Sem claude-mem o projeto funciona normalmente, mas o Claude não vai lembrar do trabalho de sessões anteriores.

### 4. Abra o projeto no Claude Code

```bash
claude .
```

O Claude vai ler o `CLAUDE.md` automaticamente e já vai saber as convenções, a estrutura de pastas e o fluxo de trabalho do projeto.

### 5. Coloque seus dados em `data/raw/`

```bash
cp ~/Downloads/meu_dataset.csv data/raw/
```

### 6. Inicie o processo

```
Chegou um novo dataset em data/raw/meu_dataset.csv. Pode começar o processo.
```

O Claude vai rodar a validação de qualidade, depois a EDA, e perguntar o que fazer com os problemas encontrados.

---

## Estrutura do projeto

```
/
├── CLAUDE.md                        # Instruções permanentes para o Claude
├── README.md                        # Este arquivo
│
├── data/
│   ├── raw/                         # Dados brutos — NUNCA modificar diretamente
│   └── outputs/
│       ├── eda/                     # Gráficos e summaries da EDA
│       └── reports/                 # Relatórios finais exportados
│
├── notebooks/                       # Análises exploratórias em Jupyter
│
├── src/                             # Módulos Python reutilizáveis
│   ├── ingestion.py
│   ├── cleaning.py
│   ├── analysis.py
│   └── visualization.py
│
└── .claude/
    ├── skills/
    │   ├── eda/SKILL.md             # Skill de análise exploratória
    │   ├── data-quality/SKILL.md    # Skill de validação de qualidade
    │   └── claude-mem/SKILL.md      # Documentação do plugin de memória
    └── agents/
        ├── data-reviewer.md         # Agente revisor de código
        └── report-writer.md         # Agente de relatórios executivos
```

---

## O que cada arquivo do `.claude/` faz

### Skills — comportamentos automáticos

Skills são acionadas automaticamente pelo Claude quando a situação corresponde ao gatilho definido. Você não precisa chamar explicitamente — o Claude reconhece o contexto e aplica.

---

#### `.claude/skills/data-quality/SKILL.md`

**Gatilho:** qualquer dado novo chega para análise, ou você diz "valida os dados", "checa a qualidade".

**O que faz:**
1. Verifica nulos por coluna com percentual e severidade
2. Detecta duplicatas exatas e por chave primária
3. Identifica colunas numéricas disfarçadas de texto e datas como string
4. Sinaliza outliers extremos via IQR × 3
5. Detecta valores negativos em colunas suspeitas (valor, preço, quantidade)

**Saída:** relatório categorizado em CRÍTICO / ATENÇÃO / INFO com linha de status final (`APROVADO`, `APROVADO COM RESSALVAS` ou `BLOQUEADO`). Nada avança enquanto houver CRÍTICO sem resolução.

---

#### `.claude/skills/eda/SKILL.md`

**Gatilho:** após data-quality aprovada, ou você diz "analisa esse arquivo", "faz uma EDA".

**O que faz:**
1. Carrega o dataset e inspeciona shape, tipos e amostra
2. Gera métricas de qualidade (nulos, duplicatas, cardinalidade)
3. Calcula estatísticas descritivas de todas as colunas
4. Gera 4 gráficos: mapa de nulos, distribuições, correlação, boxplots
5. Produz summary em Markdown com alertas e próximos passos

**Saída:** arquivo `data/outputs/eda/eda_[nome].md` + gráficos em `data/outputs/eda/` (dpi=150, bbox_inches='tight').

---

#### `.claude/skills/claude-mem/SKILL.md`

Documentação de referência do plugin claude-mem. Não é uma skill executável — é um guia para consultar quando precisar usar os tools MCP de memória (`search`, `timeline`, `smart_explore`, etc.).

---

### Agents — personas especializadas

Agentes são chamados explicitamente com `@nome-do-agente`. Cada um assume uma persona com comportamento e tom definidos.

---

#### `.claude/agents/data-reviewer.md`

**Como chamar:** `@data-reviewer revisa esse código`

**Persona:** cientista de dados sênior, crítico mas construtivo. Não elogia por elogiar.

**O que revisa:**
- Lógica estatística (métrica adequada para a distribuição? correlação vs. causalidade?)
- Armadilhas comuns do pandas (`inplace=True`, `.iterrows()`, `fillna(0)` incorreto, merge sem verificação)
- Clareza e reprodutibilidade (notebook roda do início ao fim? variáveis têm nomes descritivos?)
- Magic numbers sem explicação, funções longas demais, caminhos hardcoded

**Saída estruturada:** O que está correto → Problemas críticos com código corrigido → Melhorias recomendadas → Sugestões opcionais.

---

#### `.claude/agents/report-writer.md`

**Como chamar:** `@report-writer faz o relatório executivo`

**Persona:** consultor de negócios especializado em comunicação para não-técnicos.

**O que faz:**
- Transforma números em insights com contexto e impacto
- Usa linguagem de negócio (sem "outliers", "p-valor", "distribuição bimodal")
- Formata valores: R$ 1.200.000 → R$ 1,2M | 0.234 → 23%
- Sempre termina com tabela de próximos passos com responsável e prazo
- Adapta o tom ao público: CEO, gerência, equipe técnica ou cliente externo

**Saída:** relatório em Markdown salvo em `data/outputs/reports/`.

---

## Fluxo de trabalho padrão

```
1. INGESTÃO
   └─ Coloque o arquivo em data/raw/
      "Chegou um novo dataset em data/raw/arquivo.csv"

2. QUALIDADE  [skill: data-quality — automática]
   └─ Nulos, duplicatas, tipos, outliers
      Status: APROVADO / APROVADO COM RESSALVAS / BLOQUEADO

3. LIMPEZA
   └─ "Corrija os problemas encontrados e salve uma versão limpa"
      Saída em data/outputs/arquivo_limpo.csv

4. EDA  [skill: eda — automática]
   └─ Distribuições, correlações, alertas, próximos passos
      Saída em data/outputs/eda/

5. ANÁLISE
   └─ Desenvolva no notebook com base nos achados da EDA
      notebooks/analise_[tema].ipynb

6. REVISÃO  [agent: data-reviewer]
   └─ "@data-reviewer dá uma olhada no que fiz"
      Feedback técnico com código corrigido

7. RELATÓRIO  [agent: report-writer]
   └─ "@report-writer faz o relatório executivo para a diretoria"
      Saída em data/outputs/reports/
```

---

## Como instalar o claude-mem

O claude-mem mantém memória do que foi feito em sessões anteriores. Com ele, o Claude lembra de análises passadas, decisões tomadas e código escrito — sem precisar reexplicar a cada sessão nova.

### Instalação

```bash
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

A instalação é automática: configura os hooks, inicia o worker em background e cria o banco em `~/.claude-mem/`.

### Como usar na prática

Após instalar, você pode perguntar:

```
"Como fizemos a limpeza do dataset da semana passada?"
"Já resolvemos algum problema parecido com esse antes?"
"Mostra o histórico do que foi feito nesse projeto"
```

Para planejar e executar tarefas grandes:

```
/claude-mem:make-plan   → cria plano faseado para uma tarefa complexa
/claude-mem:do          → executa o plano com subagentes
/claude-mem:mem-search  → busca trabalho anterior
/claude-mem:timeline-report → relatório completo da jornada do projeto
```

Para navegar código sem ler arquivos inteiros:

```
/claude-mem:smart-explore → estrutura do codebase via AST (6–12x menos tokens)
```

### Privacidade

Use a tag `<private>` para excluir qualquer conteúdo da memória:

```
<private>
chave_api=sk-xxx
senha_banco=...
</private>
```

---

## Convenções do projeto

Definidas no `CLAUDE.md` e seguidas automaticamente pelo Claude em todas as respostas:

- Variáveis e funções: `snake_case`
- Classes: `PascalCase`
- Constantes: `UPPER_SNAKE_CASE`
- Comentários e docstrings: **em português**
- Nunca usar `inplace=True` — sempre reatribuição explícita
- `data/raw/` é somente leitura — outputs sempre em `data/outputs/`
- Gráficos: `dpi=150` mínimo, `bbox_inches='tight'`
- Outliers: sempre documentar o critério usado (IQR, Z-score, domínio)

---

## Dúvidas frequentes

**O Claude não está seguindo as convenções.**
Verifique se o `CLAUDE.md` está na raiz do projeto e se você abriu o Claude Code com `claude .` dentro da pasta correta.

**A skill não disparou automaticamente.**
Skills dependem do Claude reconhecer o contexto. Se não disparar, chame explicitamente: "rode a skill de data-quality no arquivo X".

**O claude-mem não está lembrando de sessões anteriores.**
Confirme que o worker está rodando: acesse `http://localhost:37777` no navegador. Se não abrir, reinstale o plugin.

**Quero adaptar para outro tipo de projeto.**
Edite o `CLAUDE.md` com o contexto do seu projeto, substitua as skills e agents pelo que fizer sentido, e mantenha a estrutura de pastas ou ajuste conforme necessário.
