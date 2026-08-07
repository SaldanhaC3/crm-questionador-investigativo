# CRM Questionador Investigativo

Skill de Claude Code que transforma o agente num investigador de dados de CRM: recebe uma dúvida aberta sobre comportamento de usuários, produto ou CRM e conduz uma investigação profunda e iterativa através de um Agente de Dados que consulta o Databricks.

## O que a skill faz

- Aprende o mapa dos dados (tabelas, colunas, granularidade) antes de investigar, com controle aritmético de fan-out de join que não depende de ninguém confirmar
- Decompõe a dúvida em sub-perguntas e caça padrões, inclusive os escondidos (inversão, bordas, ausência)
- Gera hipóteses testáveis e desafia as próprias conclusões, com checagem de composição obrigatória contra o paradoxo de Simpson
- Separa correlação de causalidade e identifica públicos estimuláveis por campanha
- Valida tudo antes de entregar: rastreabilidade total, double check de números críticos publicado como gate com contagem esperada e limiar numérico, e anexo de auditoria com o SQL de cada consulta
- Entrega em documento ou dashboard HTML autocontido, com gráficos que explicam variação ao longo do tempo e o anexo de auditoria recolhido no final
- Mantém memória persistente entre investigações (dicionário de dados, hipóteses refutadas, resultados de experimentos)

## Desenho anti-alucinação

O agente nunca vê a tabela — só o texto que o Agente de Dados devolve. Quatro mecanismos existem por causa disso:

- **A trilha vive em disco, não no contexto.** Um bloco por consulta em `trilha-<data>.md`, escrito no mesmo turno em que o retorno chega. Investigação profunda passa de 40 consultas e o contexto é compactado no meio: o que não está em arquivo o modelo depois preenche por plausibilidade
- **O SQL é lido, não só guardado.** Ao arquivar, o agente confere o SQL contra a pergunta que fez — unidade, recorte de período e coluna de data, colunas de segmento. Divergência trava o bloco antes de virar conclusão
- **Aritmética do modelo é declarada.** Razão, diferença e múltiplo entram como tipo `Derivado`, com a conta explícita, e abrem linha obrigatória no gate de verificação
- **Zero exige prova de vida.** Retorno vazio só vira achado depois da contagem-pai e do `SELECT DISTINCT` que confirma que o literal do filtro existe

## Instalação

Copie a pasta `crm-questionador-investigativo/` para o diretório de skills do seu agente (ex.: `.claude/skills/`).

Antes do primeiro uso, edite a seção **Adaptação ao seu ambiente** no final do [SKILL.md](crm-questionador-investigativo/SKILL.md), substituindo `[AGENTE_DE_DADOS]` pela forma real de invocação do seu subagente Databricks.

## Estrutura

```
crm-questionador-investigativo/
├── SKILL.md                            # definição completa (fluxo de 10 fases, templates, fundamentos estatísticos)
└── memoria-investigacoes.template.md   # esqueleto da memória persistente, copiado na primeira investigação
```
