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

| Join principal | `COUNT(*)` | `COUNT(DISTINCT customer_id)` | Razão | Data |
|----------------|-----------|-------------------------------|-------|------|
| | | | | |

**Âncora com o usuário.** Pergunte a expectativa dele antes de mostrar o número.

| Número de controle | Esperado pelo usuário | Retornado | Data | Fechou? |
|--------------------|-----------------------|-----------|------|---------|
| | | | | |

---

## 2. Hipóteses refutadas

Antes de testar hipótese nova, verifique se ela já está aqui. Se estiver, só reteste com justificativa explícita: mudou o produto, mudou o período, mudou o segmento.

| Data | Hipótese | Evidência que a derrubou | Consulta / fonte | Vale retestar se... |
|------|----------|--------------------------|------------------|---------------------|
| | | | | |

---

## 3. Resultados de experimentos executados

O dado mais valioso deste arquivo: experimento controlado é o único dado causal disponível, todo o resto é observacional. Hipótese confirmada aqui vira conhecimento causal estabelecido e pesa mais que qualquer correlação em investigações futuras.

| Data | Teste executado | Hipótese causal de origem | Público e tamanho | Métrica primária | Resultado | Confirmou ou refutou? |
|------|-----------------|---------------------------|-------------------|------------------|-----------|-----------------------|
| | | | | | | |

---

## 4. Histórico de investigações

Índice curto pra achar o que já foi feito e não repetir.

| Data | Dúvida inicial | Calibração | Onde está a entrega | Achado principal |
|------|----------------|------------|---------------------|------------------|
| | | | | |
