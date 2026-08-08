# Memória de investigações

Conhecimento acumulado entre sessões do skill `crm-questionador-investigativo`. Lido no início de toda investigação, atualizado no fim.

Nenhum identificador individual entra neste arquivo. Só estrutura de dados, hipóteses e resultados agregados.

---

## 1. Dicionário de dados

O que a fase 0 já descobriu. Em investigação nova, leia daqui e só entreviste o Agente de Dados sobre o que falta ou pode ter mudado.

### Tabelas mapeadas

| Tabela | O que representa | Granularidade | Período coberto | Última verificação |
|--------|------------------|---------------|-----------------|--------------------|
| | | | | |

### Colunas-chave

Significado que o Questionador deduziu, e não leu do Agente de Dados, entra prefixado com `[DEDUZIDO]` e não autoriza pular o mapeamento na fase 0 seguinte. Nenhuma coluna vira eixo de segmentação ou literal de filtro sem sonda registrada: distintos, % de nulos e valores reais mais frequentes.

| Tabela | Coluna | Significado | Sonda (data + `#` da trilha) | Valores reais mais frequentes | % nulos |
|--------|--------|-------------|------------------------------|-------------------------------|---------|
| | | | | | |

### Relações entre tabelas

| De | Para | Chave de junção | Cardinalidade | Observações |
|----|------|-----------------|---------------|-------------|
| | | | | |

### Dimensões de segmentação disponíveis

| Dimensão | Onde vive | Valores | Cobertura (% preenchida) |
|----------|-----------|---------|--------------------------|
| | | | |

### Lacunas conhecidas

Dimensões que não existem, períodos faltantes, colunas não confiáveis. Cada uma com a data em que foi confirmada.

- 

### Números de controle

Usados na verificação de sanidade da fase 0.

**Controle aritmético** (nunca dispensável, não depende do usuário). `COUNT(*)` acima de `COUNT(DISTINCT customer_id)` é fan-out de join, e o double check da fase 8 nunca pega.

| Join principal | `COUNT(*)` | `COUNT(DISTINCT customer_id)` | Razão (fan-out) | Data máxima da tabela | Aferido em |
|----------------|-----------|-------------------------------|-----------------|-----------------------|------------|
| | | | | | |

A **data máxima da tabela** é o que permite saber, numa retomada, se os dados mudaram desde a sessão anterior. Sem ela, blocos de fotografias diferentes do warehouse se misturam em silêncio.

**Âncora com o usuário.** Pergunte a expectativa dele antes de mostrar o número.

| Número de controle | Esperado pelo usuário | Retornado | Data | Fechou? |
|--------------------|-----------------------|-----------|------|---------|
| | | | | |

---

### Catálogo de modelos disponíveis

Descoberto no ambiente (API de modelos, config do harness, ou pergunta ao usuário). Nunca escreva um id que não veio de uma dessas fontes. Catálogo muda — registre a data.

| Id exato do modelo | Faixa (topo / meio / base) | Janela de contexto | Descoberto em | Fonte da descoberta |
|--------------------|---------------------------|--------------------|---------------|---------------------|
| | | | | |

Controle de esforço disponível no ambiente? ( ) sim, valores: ______  ( ) não
Subagentes disponíveis? ( ) sim  ( ) não — se não, tudo roda num modelo só

---

## 2. Glossário de métricas e siglas

Cada termo definido uma vez, operacionalmente, e reusado literal em toda delegação. É o que impede o mesmo conceito de ser consultado de dois jeitos e os números deixarem de ser comparáveis. Vai inteiro para a entrega.

| Termo ou sigla | Definição operacional | Tabela / coluna de origem | `#` | Proxy? |
|----------------|-----------------------|---------------------------|-----|--------|
| | | | | |

Na coluna Proxy, marque quando a coluna disponível não é exatamente o conceito — e a entrega passa a escrever o nome da coluna, não o nome do conceito.

---

## 3. Hipóteses refutadas

Antes de testar hipótese nova, verifique se ela já está aqui. Se estiver, só reteste com justificativa explícita: mudou o produto, mudou o período, mudou o segmento.

| Data | Hipótese | Evidência que a derrubou | Consulta / fonte | Vale retestar se... |
|------|----------|--------------------------|------------------|---------------------|
| | | | | |

---

## 4. Resultados de experimentos executados

O dado mais valioso deste arquivo: experimento controlado é o único dado causal disponível, todo o resto é observacional. Hipótese confirmada aqui vira conhecimento causal estabelecido e pesa mais que qualquer correlação em investigações futuras.

| Data | Teste executado | Hipótese causal de origem | Público e tamanho | Métrica primária | Resultado | Confirmou ou refutou? |
|------|-----------------|---------------------------|-------------------|------------------|-----------|-----------------------|
| | | | | | | |

---

## 5. Histórico de investigações

Índice curto pra achar o que já foi feito e não repetir.

| Data | Dúvida inicial | Calibração | Entrega (arquivo + versão) | Trilha | Achado principal |
|------|----------------|------------|----------------------------|--------|------------------|
| | | | | | |
