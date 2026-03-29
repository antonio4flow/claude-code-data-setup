# Skill: claude-mem — Memória Persistente entre Sessões

## O que é

**claude-mem** é um plugin do Claude Code que cria memória persistente e inteligente entre sessões. Ele captura automaticamente tudo que acontece durante o trabalho, comprime com IA, e injeta contexto relevante em sessões futuras — sem precisar reexplicar o histórico toda vez.

**Benefício principal:** redução de 6–12x no uso de tokens ao retomar projetos, via *progressive disclosure* (carrega só o que for necessário).

---

## Quando Acionar

- Usuário pergunta "como fizemos X da última vez?", "já resolvemos isso antes?", "onde estava o problema de Y?"
- Início de uma sessão num projeto com histórico de trabalho
- Antes de planejar ou executar uma tarefa multi-etapa
- Quando precisar entender a jornada de desenvolvimento de um projeto
- Para navegar um codebase grande sem ler arquivos inteiros

---

## Tools MCP Disponíveis

O claude-mem expõe 6 ferramentas MCP que seguem um padrão de **3 camadas progressivas** — use da mais leve para a mais detalhada.

```
Camada 1: search / smart_search    → ~50–100 tokens  (IDs e índice)
Camada 2: timeline / smart_outline → ~200 tokens     (contexto e estrutura)
Camada 3: get_observations / smart_unfold → 500–1000 tokens (detalhes completos)
```

### `search`
Busca no banco de memória por linguagem natural.

```
search(query="bug de autenticação", type="bugfix", dateStart="2024-01-01")
```
- **Saída:** índice compacto com IDs de observações
- **Filtros:** `type`, `dateStart`, `dateEnd`, `project`, `obs_type`, `orderBy`, `limit`, `offset`
- **Usar quando:** quiser encontrar trabalho passado antes de pedir detalhes

---

### `smart_search`
Busca estrutural em código via AST parsing (tree-sitter). Não lê arquivos — lê símbolos.

```
smart_search(query="autenticar", file_pattern=".py")
```
- **Saída:** funções, classes e métodos com assinaturas (corpo dobrado)
- **Linguagens suportadas:** Python, TypeScript, JavaScript, Go, Rust, Java, C, C++, Ruby, PHP
- **Tokens:** ~1–2k vs ~12k de ler o arquivo completo
- **Usar quando:** procurar onde algo está implementado sem ler tudo

---

### `smart_outline`
Retorna o esquema estrutural completo de um arquivo — sem os corpos das funções.

```
smart_outline(file_path="src/analysis.py")
```
- **Saída:** todos os símbolos (funções, classes, imports, tipos) com assinaturas
- **Usar quando:** quiser entender a estrutura de um arquivo antes de decidir o que ler

---

### `smart_unfold`
Expande um símbolo específico para seu código-fonte completo.

```
smart_unfold(file_path="src/analysis.py", symbol_name="calcular_media_movel")
```
- **Saída:** implementação completa incluindo docstring, decoradores e corpo
- **Usar quando:** após identificar o símbolo com `smart_outline`, precisar ler a implementação
- **Workflow típico:** `smart_outline` → identificar símbolo → `smart_unfold`

---

### `timeline`
Retorna o contexto cronológico ao redor de uma observação — o que aconteceu antes e depois.

```
timeline(anchor=456, depth_before=3, depth_after=3)
# ou com query:
timeline(query="refatoração do módulo de limpeza", depth_before=5, depth_after=2)
```
- **Saída:** sequência temporal de eventos ao redor do âncora
- **Usar quando:** precisar entender "como chegamos até aqui" ou "o que mudou depois"

---

### `get_observations`
Busca os detalhes completos de observações por ID. Sempre batchear múltiplos IDs juntos.

```
get_observations(ids=[123, 456, 789])
```
- **Saída:** dados completos de cada observação (narrativa, código, contexto)
- **Usar quando:** após `search` ou `timeline` retornarem IDs, e precisar dos detalhes

---

### `__IMPORTANT` (metaferramenta)
Documenta o padrão de workflow em 3 camadas. Consultar quando não tiver certeza de qual tool usar.

---

## Skills Disponíveis

### `claude-mem:mem-search`
Busca memória com progressive disclosure.

**Acionar quando:** o usuário perguntar sobre trabalho anterior ("já fizemos isso?", "como resolvemos X?")

**Workflow interno:**
1. `search` para encontrar observações relevantes
2. `timeline` para contexto cronológico
3. `get_observations` para detalhes completos se necessário

---

### `claude-mem:make-plan`
Cria plano de implementação detalhado com descoberta de documentação.

**Acionar quando:** o usuário pedir para planejar uma feature, tarefa ou implementação multi-passo — *antes* de executar.

**O que faz:**
- Explora o codebase com subagente
- Descobre documentação relevante
- Cria plano faseado pronto para revisão
- Deixa tudo preparado para execução com `do`

---

### `claude-mem:do`
Executa planos faseados usando subagentes.

**Acionar quando:** o usuário pedir para executar um plano — especialmente um criado por `make-plan`.

**O que faz:**
- Executa cada fase do plano
- Usa subagentes para trabalho paralelo quando possível
- Injeta memória relevante em cada fase
- Rastreia progresso

---

### `claude-mem:smart-explore`
Exploração estrutural de código otimizada para tokens via AST.

**Acionar quando:** precisar navegar um codebase grande para entender estrutura sem ler arquivos inteiros.

**Diferença para `smart_search`:** mais rápido quando o objetivo é só a estrutura (não uma busca por keyword específica).

---

### `claude-mem:timeline-report`
Gera relatório narrativo completo da jornada de desenvolvimento de um projeto.

**Acionar quando:** o usuário pedir "histórico do projeto", "como chegamos aqui", "relatório de desenvolvimento".

**Saída:** documento narrativo tipo "Journey Into [Projeto]" com decisões arquiteturais, pontos de virada e cronologia completa.

---

## Como Funciona a Memória

### Captura automática via 5 lifecycle hooks

| Hook | Quando dispara | O que captura |
|---|---|---|
| `SessionStart` | Início de cada sessão | Injeta contexto das últimas 50 observações |
| `UserPromptSubmit` | A cada mensagem do usuário | Prepara contexto para resposta |
| `PostToolUse` | Após cada tool executada | Leitura, escrita, bash, git |
| `Summary` | Fim do processamento | Gera resumo com IA |
| `SessionEnd` | Encerramento da sessão | Salva tudo no banco |

### Estrutura de cada observação armazenada

- **ID único** para referência
- **Título** gerado por IA ("Corrigido bug de nulos em cleaning.py")
- **Narrativa** em linguagem natural do que aconteceu
- **Categoria** (bugfix, feature, refactor, análise…)
- **Timestamp**
- **Embedding semântico** para busca por similaridade

### Onde os dados ficam

```
~/.claude-mem/
├── claude-mem.db        # Banco SQLite com todas as observações
├── settings.json        # Configurações
├── worker.pid           # Processo do worker
├── worker.port          # Porta (padrão: 37777)
└── logs/
    └── 2026-03-29.log
```

Web viewer disponível em **http://localhost:37777** — visualização em tempo real de todo o histórico.

---

## Privacidade

Use a tag `<private>` para excluir qualquer conteúdo da memória:

```
<private>
API_KEY=sk-xxx
Senha do banco: ...
</private>
```

O conteúdo é removido antes de chegar ao banco de dados.

---

## Padrão de Uso Recomendado

```
Sessão 1 → Você trabalha normalmente
           → claude-mem captura automaticamente

Sessão 2 → Contexto anterior injeta no início
           → "Como fizemos a limpeza do CSV?" → mem-search responde com histórico
           → Próxima tarefa grande? → make-plan → revisar → do

Semanas depois → timeline-report mostra toda a jornada
               → smart-explore para navegar o código crescido
```

---

## Instalação

```bash
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

A instalação configura automaticamente os hooks, o worker service e o banco de dados. Não requer configuração manual para funcionar.

**Requisitos:** Node.js 18+, Claude Code atualizado.
