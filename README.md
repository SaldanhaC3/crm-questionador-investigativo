# Questionador Investigativo

Skill que transforma o agente num investigador de dados: recebe uma dúvida aberta na primeira mensagem e vai atrás do dado que a explica, através de um Agente de Dados que consulta o warehouse. Investiga até responder — e mostra como chegou lá.

O domínio mais frequente é CRM e comportamento de usuário, mas não é o limite: a pergunta pode ser de produto, marketing, operação ou receita. O que define a skill não é o assunto, é o método.

## O que a skill faz

- **Persiste.** "Não tenho esse dado" só encerra uma linha de investigação depois de três tentativas registradas, por ângulos diferentes: outra tabela com proxy, outra granularidade, outro período
- Aprende o mapa dos dados antes de investigar, com controle aritmético de fan-out de join que não depende de ninguém confirmar
- Decompõe a dúvida em sub-perguntas e caça padrões, inclusive os escondidos (inversão, bordas, ausência)
- Gera hipóteses testáveis e se desafia, com checagem de composição obrigatória contra o paradoxo de Simpson
- Separa correlação de causalidade e identifica públicos estimuláveis por campanha
- Valida antes de entregar: rastreabilidade total, double check com contagem esperada e limiar aritmético
- Entrega documento ou dashboard HTML com estrutura fixa, gráficos escolhidos por tipo de pergunta, funil, glossário de métricas e siglas, e o SQL de cada consulta no anexo
- Mantém memória entre investigações: dicionário de dados, glossário, hipóteses refutadas, resultados de experimentos

## Desenho anti-alucinação

O agente nunca vê a tabela — só o texto que o Agente de Dados devolve. Sete guard rails existem por causa disso; os quatro estruturais:

- **A trilha vive em disco, não no contexto.** Um bloco por consulta em `trilha-<data>.md`, escrito no mesmo turno em que o retorno chega. Investigação profunda passa de 40 consultas e o contexto é compactado no meio: o que não está em arquivo o modelo depois preenche por plausibilidade
- **O SQL é lido, não só guardado.** Ao arquivar, o agente confere o SQL contra a pergunta que fez — unidade, recorte de período e coluna de data, colunas de segmento. Divergência trava o bloco antes de virar conclusão
- **Aritmética do modelo é declarada.** Razão, diferença e múltiplo entram como tipo `Derivado`, com a conta explícita, e abrem linha obrigatória no gate de verificação
- **Nome, data e modelo não se inventam.** Tabela, coluna e id de modelo só entram se apareceram numa fonte real; data e hora vêm do sistema, nunca de memória

## Instalação

Copie a pasta `crm-questionador-investigativo/` para o diretório de skills do seu agente (ex.: `.claude/skills/`).

Antes do primeiro uso, edite a seção **Adaptação ao seu ambiente** no final do [SKILL.md](crm-questionador-investigativo/SKILL.md), substituindo `[AGENTE_DE_DADOS]` pela forma real de invocação do seu subagente de dados.

A skill descobre sozinha quais modelos existem no ambiente onde roda — via API de modelos, config do harness, ou perguntando uma vez — e registra o catálogo na memória. Não há id de modelo hardcodado.

## Estrutura

```
crm-questionador-investigativo/
├── SKILL.md                            # fluxo de 10 fases, guard rails, seleção de modelo
├── referencia-entrega.md               # estrutura fixa do dash, gráfico por pergunta, funil, glossário, versionamento
├── dashboard-template.html             # esqueleto a copiar: tema claro/escuro, selo de verificação, gráficos em SVG
└── memoria-investigacoes.template.md   # esqueleto da memória persistente
```

`referencia-entrega.md` e `dashboard-template.html` são lidos na fase 8, na hora de montar a entrega — é o que faz duas sessões produzirem o mesmo esqueleto em vez de dois layouts diferentes.
