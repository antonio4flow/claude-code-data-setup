# Skill: Validação de Qualidade dos Dados

## Anúncio obrigatório
Sempre inicie sua resposta com este bloco exato, antes de qualquer análise:

---
🔍 **[SKILL: data-quality]** Ativada
Arquivo detectado: {nome_do_arquivo}
Executando validação de qualidade antes de liberar para EDA...
---

## Quando Acionar

- Antes de qualquer análise em dados novos
- Quando o usuário disser "valida os dados", "checa a qualidade", "tem algum problema nesse dataset"
- Após uma etapa de limpeza/transformação para confirmar que o resultado está correto
- Antes de gerar relatório final

---

## O Que Esta Skill Verifica

| Categoria | O Que Checa |
|---|---|
| Completude | Nulos por coluna, linhas completamente vazias |
| Unicidade | Duplicatas exatas, duplicatas por chave |
| Tipos | Colunas numéricas com strings, datas como texto |
| Consistência | Valores fora de domínio esperado, negativos onde não faz sentido |
| Outliers | IQR e Z-score para colunas numéricas |
| Formato | Datas em formatos mistos, CEPs/CPFs mal formatados se aplicável |

---

## Protocolo de Execução

### Módulo de Validação Completa

```python
import pandas as pd
import numpy as np
from typing import Dict, List

def verificar_qualidade(df: pd.DataFrame, chave_primaria: List[str] = None) -> Dict:
    """
    Executa verificação completa de qualidade do DataFrame.

    Retorna dicionário com resultados categorizados por severidade:
    - CRITICO: impede análise correta
    - ATENCAO: pode distorcer resultados
    - INFO: observação sem impacto direto
    """
    resultados = {"CRITICO": [], "ATENCAO": [], "INFO": []}

    # --- 1. COMPLETUDE ---
    nulos = df.isnull().sum()
    pct_nulos = (nulos / len(df)) * 100

    for col, pct in pct_nulos.items():
        if pct > 50:
            resultados["CRITICO"].append(
                f"[NULOS] '{col}': {pct:.1f}% de valores nulos — coluna pode ser inutilizável"
            )
        elif pct > 10:
            resultados["ATENCAO"].append(
                f"[NULOS] '{col}': {pct:.1f}% de valores nulos"
            )
        elif pct > 0:
            resultados["INFO"].append(
                f"[NULOS] '{col}': {pct:.1f}% de valores nulos ({nulos[col]} linhas)"
            )

    # Linhas completamente vazias
    linhas_vazias = df.isnull().all(axis=1).sum()
    if linhas_vazias > 0:
        resultados["CRITICO"].append(
            f"[LINHAS VAZIAS] {linhas_vazias} linhas completamente nulas encontradas"
        )

    # --- 2. UNICIDADE ---
    n_duplicatas = df.duplicated().sum()
    if n_duplicatas > 0:
        pct_dup = (n_duplicatas / len(df)) * 100
        nivel = "CRITICO" if pct_dup > 10 else "ATENCAO"
        resultados[nivel].append(
            f"[DUPLICATAS] {n_duplicatas} linhas duplicadas ({pct_dup:.1f}% do total)"
        )

    if chave_primaria:
        n_dup_chave = df.duplicated(subset=chave_primaria).sum()
        if n_dup_chave > 0:
            resultados["CRITICO"].append(
                f"[CHAVE DUPLICADA] {n_dup_chave} duplicatas na chave {chave_primaria}"
            )

    # --- 3. TIPOS ---
    for col in df.select_dtypes(include="object").columns:
        # Detectar numérico disfarçado de texto
        amostra = df[col].dropna().head(100)
        try:
            pd.to_numeric(amostra)
            resultados["ATENCAO"].append(
                f"[TIPO] '{col}' parece numérica mas está como texto — considerar conversão"
            )
        except (ValueError, TypeError):
            pass

        # Detectar data disfarçada de texto
        if any(kw in col.lower() for kw in ["data", "date", "dt", "mes", "ano", "year"]):
            try:
                pd.to_datetime(amostra, infer_datetime_format=True, errors="raise")
                resultados["ATENCAO"].append(
                    f"[TIPO] '{col}' parece data mas está como texto — considerar pd.to_datetime()"
                )
            except Exception:
                pass

    # --- 4. OUTLIERS (IQR) ---
    for col in df.select_dtypes(include=np.number).columns:
        serie = df[col].dropna()
        if len(serie) < 10:
            continue
        Q1 = serie.quantile(0.25)
        Q3 = serie.quantile(0.75)
        IQR = Q3 - Q1
        if IQR == 0:
            continue
        limite_inf = Q1 - 3 * IQR
        limite_sup = Q3 + 3 * IQR
        n_outliers = ((serie < limite_inf) | (serie > limite_sup)).sum()
        if n_outliers > 0:
            pct_out = (n_outliers / len(serie)) * 100
            nivel = "ATENCAO" if pct_out > 1 else "INFO"
            resultados[nivel].append(
                f"[OUTLIERS] '{col}': {n_outliers} outliers extremos ({pct_out:.1f}%) "
                f"fora de [{limite_inf:.2f}, {limite_sup:.2f}]"
            )

    # --- 5. VALORES NEGATIVOS SUSPEITOS ---
    colunas_suspeitas = [c for c in df.select_dtypes(include=np.number).columns
                         if any(kw in c.lower() for kw in ["valor", "preco", "price", "quantidade", "qty", "amount"])]
    for col in colunas_suspeitas:
        n_neg = (df[col] < 0).sum()
        if n_neg > 0:
            resultados["ATENCAO"].append(
                f"[NEGATIVO] '{col}': {n_neg} valores negativos — verificar se é esperado"
            )

    return resultados


def imprimir_relatorio(resultados: Dict, nome_dataset: str = "Dataset"):
    """Imprime relatório formatado de qualidade."""
    total_problemas = sum(len(v) for v in resultados.values())

    print(f"\n{'='*60}")
    print(f"RELATÓRIO DE QUALIDADE — {nome_dataset}")
    print(f"{'='*60}")

    if total_problemas == 0:
        print("✅ Nenhum problema encontrado!")
        return

    for nivel in ["CRITICO", "ATENCAO", "INFO"]:
        itens = resultados[nivel]
        if not itens:
            continue
        emoji = {"CRITICO": "🔴", "ATENCAO": "🟡", "INFO": "🔵"}[nivel]
        print(f"\n{emoji} {nivel} ({len(itens)} ocorrências):")
        for item in itens:
            print(f"   • {item}")

    print(f"\n{'='*60}")
    criticos = len(resultados["CRITICO"])
    if criticos > 0:
        print(f"⛔ {criticos} problema(s) CRÍTICO(s) — resolver antes de prosseguir")
    else:
        print("✅ Sem problemas críticos — análise pode prosseguir com cautela")


# --- USO ---
# df = pd.read_csv("data/raw/arquivo.csv")
# resultados = verificar_qualidade(df, chave_primaria=["id_pedido"])
# imprimir_relatorio(resultados, nome_dataset="Pedidos 2024")
```

---

## Checklist de Decisão Pós-Validação

Após rodar a validação, orientar o usuário sobre cada problema encontrado:

**Nulos:**
- < 5%: imputar com mediana/moda ou deixar se não for variável-chave
- 5–30%: decidir estratégia com o usuário (imputação, flag binário, exclusão)
- > 30%: recomendar exclusão da coluna ou obter dado na fonte

**Duplicatas exatas:**
- Verificar se são legítimas (ex: mesmo cliente, pedidos diferentes) ou erro de ingestão
- Usar `df.drop_duplicates()` apenas após confirmar

**Tipos errados:**
- Converter explicitamente e validar com `.isna()` após conversão para detectar falhas

**Outliers:**
- Nunca remover automaticamente — documentar critério e obter aprovação
- Sugerir análise com e sem outliers para comparação

---

## Saída Esperada

A skill deve sempre terminar com uma linha de status:

```
STATUS FINAL: [BLOQUEADO / APROVADO COM RESSALVAS / APROVADO]
Razão: [resumo em 1 linha]
```
