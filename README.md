# CRM Questionador Investigativo

Skill de Claude Code que transforma o agente num investigador de dados de CRM: recebe uma dúvida aberta sobre comportamento de usuários, produto ou CRM e conduz uma investigação profunda e iterativa através de um Agente de Dados que consulta o Databricks.

## O que a skill faz

- Aprende o mapa dos dados (tabelas, colunas, granularidade) antes de investigar
- Decompõe a dúvida em sub-perguntas e caça padrões, inclusive os escondidos (inversão, bordas, ausência)
- Gera hipóteses testáveis e desafia as próprias conclusões com explicações alternativas
- Separa correlação de causalidade e identifica públicos estimuláveis por campanha
- Valida tudo antes de entregar: rastreabilidade total, double check de números críticos publicado como gate, e anexo de auditoria com o SQL de cada consulta
- Entrega em documento ou dashboard HTML autocontido, com gráficos que explicam variação ao longo do tempo e o anexo de auditoria recolhido no final
- Mantém memória persistente entre investigações (dicionário de dados, hipóteses refutadas, resultados de experimentos)

## Instalação

Copie a pasta `crm-questionador-investigativo/` para o diretório de skills do seu agente (ex.: `.claude/skills/`).

Antes do primeiro uso, edite a seção **Adaptação ao seu ambiente** no final do [SKILL.md](crm-questionador-investigativo/SKILL.md), substituindo `[AGENTE_DE_DADOS]` pela forma real de invocação do seu subagente Databricks.

## Estrutura

```
crm-questionador-investigativo/
├── SKILL.md                            # definição completa (fluxo de 10 fases, templates, fundamentos estatísticos)
└── memoria-investigacoes.template.md   # esqueleto da memória persistente, copiado na primeira investigação
```
