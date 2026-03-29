# EDA — consultas_clinica.csv — 2026-03-28

## Visão Geral

| Métrica | Valor |
|---|---|
| Linhas totais | 30 |
| Linhas únicas (sem dups) | 28 |
| Colunas | 11 |
| Período | 15/01/2024 a 03/02/2024 |
| Duplicatas | 2 linhas (6.7%) |

---

## Qualidade dos Dados

| Coluna | Tipo | Nulos | % Nulos | Obs |
|---|---|---|---|---|
| id_consulta | int | 0 | 0% | Chave única (sem dups) |
| id_paciente | string | 0 | 0% | P001–P028 |
| nome_paciente | string | 0 | 0% | — |
| data_nascimento | date (texto) | 0 | 0% | Formato YYYY-MM-DD, consistente |
| data_consulta | date (texto) | 1 | 3.3% | 2 formatos distintos + 1 data inválida |
| medico | string | 0 | 0% | 4 médicos distintos |
| especialidade | string | 0 | 0% | 4 especialidades |
| diagnostico | string | 3 | 10.0% | ⚠️ Avaliar preenchimento |
| valor_consulta | float | 0 | 0% | R$ 150–650 |
| convenio | string | 2 | 6.7% | Pode ser particular não registrado |
| status_pagamento | string | 0 | 0% | Pago / Pendente / Cancelado |

---

## Distribuição por Especialidade

| Especialidade | Consultas | % | Valor unitário | Receita potencial |
|---|---|---|---|---|
| Cardiologia | 9 | 30% | R$ 320–650 | R$ 3.750 |
| Clínica Geral | 8 | 27% | R$ 150 (fixo) | R$ 1.200 |
| Ortopedia | 7 | 23% | R$ 280–380 | R$ 2.160 |
| Dermatologia | 6 | 20% | R$ 200 (fixo) | R$ 1.200 |
| **TOTAL** | **30** | | | **R$ 8.310*** |

*Excluindo as 2 consultas duplicadas (R$ 470 de distorção se não removidas).*

---

## Valor por Consulta — Estatísticas Descritivas (28 registros únicos)

| Métrica | Valor |
|---|---|
| Mínimo | R$ 150,00 |
| Q1 (25%) | R$ 150,00 |
| Mediana | R$ 280,00 |
| Média | R$ 291,43 |
| Q3 (75%) | R$ 380,00 |
| Máximo | R$ 650,00 |
| Desvio padrão | ~R$ 141,00 |

**Observação:** A distribuição é bimodal — existe um grupo de baixo valor (Clínica Geral R$ 150 + Dermatologia R$ 200) e um grupo de alto valor (Cardiologia R$ 320–650). A média de R$ 291 não representa bem nenhum dos dois grupos.

---

## Status de Pagamento

| Status | Consultas | % |
|---|---|---|
| Pago | 22 | 73.3% |
| Pendente | 5 | 16.7% |
| Cancelado | 3 | 10.0% |

**Receita a receber (Pendente):**
- 1003 Carla Dias — Ortopedia R$ 280
- 1009 Henrique Barros — Cardiologia R$ 520
- 1015 Nelson Prado — Ortopedia R$ 280
- 1023 Ursula Freitas — Dermatologia R$ 200
- 1030 Bernardo Cunha — Clínica Geral R$ 150
- **Total pendente: R$ 1.430**

**Consultas canceladas (receita perdida):**
- 1008 Gabriela Nunes — Dermatologia R$ 200
- 1018 Quezia Matos — Clínica Geral R$ 150
- 1028 Zara Monteiro — Ortopedia R$ 280
- **Total cancelado: R$ 630**

---

## Convênio

| Convênio | Consultas | % |
|---|---|---|
| Unimed | 9 | 30% |
| Bradesco Saúde | 7 | 23% |
| SulAmérica | 7 | 23% |
| Particular | 5 | 17% |
| Não informado | 2 | 7% |

---

## Distribuição por Médico

| Médico | Especialidade | Consultas | Receita |
|---|---|---|---|
| Dr. Carlos Souza | Cardiologia | 9 | R$ 3.750 |
| Dra. Fernanda Rocha | Clínica Geral | 8 | R$ 1.200 |
| Dr. Marcus Alves | Ortopedia | 7 | R$ 2.160 |
| Dra. Juliana Park | Dermatologia | 6 | R$ 1.200 |

**Cada médico atende exclusivamente sua especialidade** — não há sobreposição.

---

## Faixa Etária dos Pacientes

| Grupo | Faixa | Pacientes | Diagnósticos mais comuns |
|---|---|---|---|
| Jovem adulto | 19–34 anos | ~8 | Gripe, Acne, Ansiedade, Eczema |
| Adulto | 35–54 anos | ~12 | Tendinite, Diabetes, Lombalgia, Dermatite |
| Meia-idade/Sênior | 55+ anos | ~8 | Hipertensão grave, Artrose, Infarto prévio |

Paciente mais novo: Karen Oliveira (19 anos) | Mais velho: Nelson Prado (~62 anos)

---

## Alertas EDA

⚠️ **Cardiologia concentra 45% da receita** com apenas 30% das consultas — é a especialidade de maior valor médio (R$ 468/consulta vs. média geral R$ 291).

⚠️ **R$ 1.430 em contas pendentes** — Cardiologia sozinha representa R$ 520 desse valor (1 consulta de insuficiência cardíaca sem convênio informado).

⚠️ **2 datas problemáticas** na coluna `data_consulta` vão quebrar qualquer análise temporal sem tratamento prévio.

---

## Próximos Passos

1. **Remover duplicatas** (ids 1007 e 1022) antes de qualquer análise de receita
2. **Corrigir data_consulta** id 1025 (`2024-31-01` → `2024-01-31`) e id 1014 (`15/01/2024` → `2024-01-15`)
3. **Classificar os 2 convênios nulos** como "Particular" ou investigar na fonte
4. **Investigar os 3 diagnósticos nulos** — são consultas de retorno? Triagem?
5. **Análise de inadimplência por convênio** — verificar se algum convênio concentra os pendentes
