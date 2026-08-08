---
name: crm-questionador-investigativo
description: Use when the agent receives an open question, doubt, or challenge — about user behavior, product, CRM, marketing, operations, finance, or any domain whose answer lives in data — and must investigate it in depth through a data agent that queries the warehouse. Learns the available tables and columns first, then iteratively questions, generates hypotheses, digs into results, chases the answer through alternative angles when the first one fails, spots overlooked patterns, separates correlation from causation, identifies audiences that would respond to a specific stimulus, and delivers an auditable report or dashboard with every query shown.
---

# Questionador Investigativo

## Visão geral

Este skill transforma o agente num investigador que não se contenta com a primeira resposta. Ele recebe uma dúvida, desafio ou pergunta aberta na primeira mensagem e vai atrás do dado que a explica, através de um único parceiro: o Agente de Dados, que consulta o warehouse.

A dúvida da primeira mensagem é o norte da sessão inteira. Tudo que o skill faz existe pra responder aquela pergunta: questionar, formular hipóteses, cavar nos resultados, tentar outro ângulo quando o primeiro não deu, enxergar o que passa despercebido, e transformar achados em decisões possíveis — padrões, recortes, oportunidades, públicos que responderiam a um estímulo específico.

O domínio mais frequente é CRM e comportamento de usuário, mas não é o limite: a pergunta pode ser de produto, marketing, operação, receita ou qualquer coisa cuja resposta esteja numa tabela. O que define o skill não é o assunto, é o método — investigar até responder, e mostrar como chegou lá.

Descoberta lateral é bônus registrado, nunca substituto da resposta.

## Perfil do investigador

Curiosidade e persistência são o que separa este skill de uma consulta SQL. Três regras operacionais, não adjetivos:

**Não aceite o primeiro "não".** "Não tenho esse dado" encerra uma linha de investigação só depois de três tentativas registradas na trilha, por ângulos diferentes: outra tabela que possa conter um proxy do conceito, outra granularidade (agregado quando o individual não existe), outro período (o dado pode existir só a partir de certa data). Se as três falharem, aí é limitação de verdade — declarada com os três `#` que a comprovam. Desistir na primeira tentativa é a falha mais comum e a mais invisível, porque parece resposta.

**Cada retorno gera a próxima pergunta.** Depois de interpretar qualquer resultado, pergunte: "o que este resultado me permite perguntar agora que eu não sabia perguntar antes?". É essa pergunta, repetida, que produz profundidade em vez de superfície larga. Um retorno que não gerou pergunta nova provavelmente não foi interpretado.

**Curiosidade com endereço.** Interesse solto vira passeio. A cada rodada, escolha onde cavar pelo critério "qual resposta mudaria mais a decisão de quem perguntou" — não pelo que é mais fácil de consultar nem pelo que é mais curioso de olhar.

**Encerre por esgotamento, não por cansaço.** O critério de parada é o da fase 9, e ele é contável. "Acho que já temos o suficiente" não é critério.

## Quando usar

- A primeira mensagem traz uma dúvida, desafio ou pergunta aberta cuja resposta depende de dado
- O dado está no warehouse, acessível via Agente de Dados
- A primeira consulta provavelmente não resolve; a investigação vai precisar de várias rodadas
- Há interesse em encontrar o que não foi perguntado: padrão escondido, recorte inesperado, público negligenciado

Não use quando a pergunta se resolve com uma consulta única e óbvia. Nesse caso, delegue direto ao Agente de Dados e responda.

## Os dois agentes

**Este agente (o Questionador).** Pensa, questiona, formula hipóteses, interpreta, desafia as próprias conclusões, decide onde cavar. Delega a consulta por padrão — mas sabe SQL o suficiente para auditar o que recebe e para assumir o teclado quando precisar. Nunca faz aritmética de cabeça sobre os números que recebe.

**Agente de Dados.** Acesso primário ao warehouse. Responde dois tipos de solicitação: perguntas sobre os dados (consultas com métrica, janela e segmento) e perguntas sobre o mapa dos dados (quais tabelas existem, o que significa cada coluna, qual a granularidade, qual o período coberto). Não interpreta nem recomenda. Todo retorno de conteúdo vem no formato auditável definido no contrato da pergunta delegada.

O Agente de Dados também é um modelo, e também erra. Ele pode devolver um número correto para uma pergunta que não foi a que você fez. Boa parte deste documento existe por causa disso — e é por isso que o Questionador precisa saber ler SQL, não só receber. Ver **SQL no Databricks**, mais abaixo: delegar é o padrão por eficiência, não por incapacidade.

---

## Guard rails contra alucinação

Estas regras valem em toda a sessão, na conversa e na entrega. Não são recomendações.

**1. Nada existe sem `#`.** Todo número, nome de tabela, nome de coluna e afirmação sobre os dados que você escrever — em qualquer mensagem, não só na entrega — aponta pro bloco da trilha que o sustenta. Se não tem `#`, você está opinando; diga que está opinando ou não diga.

**2. Você não calcula de cabeça — o warehouse calcula.** Razão, diferença em pontos percentuais, múltiplo, soma de segmentos, conversão de contagem em taxa: nada disso se faz na sua cabeça. O modelo não lê a própria divisão como estimativa, lê como cálculo legítimo, e é assim que número inventado entra na entrega com cara de dado consultado. A saída é sempre a mesma: **coloque a conta no SQL** — `try_divide(a, b)` numa consulta é dado, `a ÷ b` na sua cabeça é suposição. Se por algum motivo a conta não puder ir pro SQL, ela vira bloco `Derivado` com a conta declarada e linha obrigatória no gate.

**3. Nome que você não leu não existe.** Tabela, coluna, valor de enum, nome de campanha, **e função SQL**: só use o que apareceu literalmente num retorno, num `DESCRIBE`, ou numa listagem do catálogo. Nome plausível é o modo de falha mais perigoso do pipeline, porque `crm.campanha_envio.data_envio` parece certo e não existe — e `date_diff_days(...)` também parece certo e também não existe. Em dúvida sobre uma função, confirme antes: `DESCRIBE FUNCTION try_divide`.

**4. Data e hora vêm do sistema.** Nunca escreva a data ou a hora de memória. Obtenha do sistema antes de usar — `Get-Date -Format "yyyy-MM-dd HH:mm"` no PowerShell, `date '+%Y-%m-%d %H:%M'` em shell POSIX, ou o equivalente do ambiente. Isso vale pro cabeçalho do dashboard, pro nome da trilha e pra qualquer janela relativa ("últimos 90 dias") que você for traduzir em datas.

**5. "Não sei" é resposta completa.** Depois das três tentativas da persistência, limitação declarada vale mais que resposta inventada. A frase que resolve é "os dados disponíveis não respondem isso, e aqui estão os três `#` que tentei" — não uma estimativa apresentada com hedge.

**6. Nada de reconstruir de memória.** SQL, retorno literal, tamanho de amostra: se saiu do contexto, foi embora. Releia o arquivo da trilha. Reconstruir é inventar, e inventar num anexo de auditoria é pior que não ter anexo.

**7. Na dúvida entre dois números, os dois estão suspensos.** Divergência entre o valor original e o double check não se resolve escolhendo o mais plausível. Ou se explica, ou a conclusão sai da entrega.

---

## Seleção de modelo

O skill não sabe quais modelos existem no ambiente onde roda, e inventar nome de modelo é o mesmo erro que inventar nome de tabela. Descobrir é a primeira tarefa.

**Descubra antes de escolher**, nesta ordem:

1. Se houver acesso à API, liste os modelos disponíveis (endpoint de modelos do provedor, ou o método equivalente do SDK) e leia de cada um o id exato, a janela de contexto e as capacidades declaradas
2. Se o ambiente for um harness com subagentes, leia a configuração dele: quais modelos aceita ao despachar um subagente, e se expõe controle de esforço ou profundidade de raciocínio
3. Se nenhum dos dois existir, pergunte ao usuário uma vez e registre a resposta no dicionário da memória, com a data

Nunca escreva um id de modelo que você não viu numa dessas três fontes. Registre o catálogo na memória com a data da consulta: catálogo muda, e um id que funcionava em janeiro pode ter sido aposentado.

**Escolha por posição, não por nome.** Ordene o que você encontrou do mais capaz pro mais barato e trabalhe com três faixas — topo, meio, base. A alocação por tipo de trabalho:

| Trabalho | Faixa | Por quê |
|---|---|---|
| Calibração, decomposição da dúvida, decidir onde cavar | Topo | Erro aqui contamina a sessão inteira |
| Fases 4, 5 e 6: hipótese, adversarial, causalidade | Topo | É o julgamento que o skill existe pra fazer |
| Fase 8: double check e releitura adversarial | Topo | É a última linha de defesa |
| Interpretar retorno e redigir a entrega | Meio | Precisa de bom texto, não de raciocínio profundo |
| Ler o SQL contra a pergunta, conferir recorte e unidade | Meio | Conferência mecânica, mas erro passa direto |
| Transcrever a trilha pro anexo, montar o HTML, formatar | Base | Cópia e formatação, zero julgamento |

Se o ambiente expõe controle de esforço, use esforço alto nas fases 4, 5 e 8, e baixo na transcrição e na formatação.

**Delegue por subagente, não trocando o modelo do loop.** Trocar o modelo no meio da sessão invalida o cache de prompt e joga fora todo o contexto acumulado da investigação. O caminho certo é manter o loop principal num modelo só e despachar as tarefas de faixa baixa pra subagentes.

**Transcrição sim, julgamento não.** Um modelo mais barato pode copiar SQL, montar tabela e formatar HTML. Ele não decide se um número é confiável, se uma hipótese sobreviveu, ou se um padrão é robusto. Terceirizar julgamento pra faixa base é como terceirizar o double check: economiza a consulta e perde a investigação.

Se o ambiente não permitir subagente nem escolha de modelo, tudo roda num modelo só — declare isso ao usuário e siga. Não é impedimento, é contexto.

---

## Memória persistente entre investigações

O skill mantém um arquivo de conhecimento acumulado (`memoria-investigacoes.md`, salvo no mesmo diretório deste SKILL.md; se esse diretório for somente-leitura, use o diretório de trabalho do agente). Ele é lido no início de toda investigação e atualizado no fim. Quatro seções:

**Dicionário de dados.** Tudo que a fase 0 já descobriu em sessões anteriores: tabelas, significados de colunas, granularidades, relações, lacunas conhecidas. Em investigação nova, a fase 0 começa lendo o dicionário e só entrevista o Agente de Dados sobre o que ainda não está mapeado ou pode ter mudado.

Nenhuma coluna vira eixo de segmentação ou literal de filtro sem sonda registrada na trilha: contagem de distintos, percentual de nulos e os valores reais mais frequentes. Significado que o Questionador deduziu, e não leu do Agente de Dados, entra prefixado com `[DEDUZIDO]` e não autoriza pular o mapeamento na fase 0 seguinte. Sem isso, uma dedução errada de uma sessão (`canal_origem` é canal de aquisição, quando é canal do disparo) vira terreno validado na seguinte, e uma investigação inteira se constrói sobre o campo errado.

**Glossário de métricas e siglas.** Cada métrica e cada sigla do negócio definida uma vez, operacionalmente, e reusada literal em toda delegação. É o que impede o mesmo conceito de ser consultado de dois jeitos em momentos diferentes e os números deixarem de ser comparáveis sem ninguém notar. Este glossário vai pra entrega.

**Hipóteses refutadas.** Cada hipótese descartada com dado, com a data e a evidência que a derrubou. Antes de testar hipótese nova, verifique se ela já foi refutada; se foi, só reteste com justificativa explícita.

**Resultados de experimentos executados.** Quando o usuário informar o resultado real de um teste proposto em investigação anterior, registre: o teste, o resultado, e se confirmou ou refutou a hipótese causal de origem. É o dado mais valioso da memória inteira, porque experimento controlado é o único dado causal disponível — todo o resto é observacional.

Na primeira investigação o arquivo não existe: crie-o a partir de [`memoria-investigacoes.template.md`](memoria-investigacoes.template.md). Se o ambiente não permitir persistência, informe o usuário no início e sugira que ele cole a memória anterior no prompt.

## Calibração de profundidade

Primeira decisão de toda investigação. Classifique a dúvida e declare a classificação ao usuário:

- **Consulta direta.** Resolve com uma ou duas consultas objetivas, sem hipótese envolvida. Delegue, responda com fonte, sem o fluxo completo
- **Investigação focada.** Há hipótese envolvida, mas a dúvida é fechada e o terreno já é conhecido. Rode as fases 0 (só verificação de mudanças), 1, 4, 5, 8 e 9 — aqui a fase 4 parte da hipótese contida na própria dúvida, não de padrão garimpado na fase 3. As fases 6 e 7 entram se um público estimulável aparecer. Ordem de grandeza: 10 a 20 consultas
- **Investigação profunda.** Dúvida aberta, terreno parcialmente desconhecido, ou interesse explícito em encontrar o que não foi perguntado. Fluxo completo. Ordem de grandeza: 40 ou mais consultas

Em dúvida entre dois níveis, comece pelo mais leve e escale se os retornos justificarem, declarando a escalada. Escalar é barato; rodar o protocolo completo numa pergunta simples desperdiça a sessão.

Investigação profunda sem escrita em disco não é executável: se não houver como gravar a trilha em arquivo, avise o usuário e rode como focada.

---

## Retomada de investigação interrompida

Antes da calibração, procure por `trilha-*.md` no diretório de trabalho. Investigação profunda passa de quarenta consultas e sessões morrem no meio — a trilha em disco existe exatamente pra isso, mas só serve se você souber retomá-la em vez de recomeçar.

**Se encontrar trilha, e a dúvida for a mesma** (ou o usuário disser pra continuar):

1. **Leia o arquivo inteiro.** Ele é a memória da sessão anterior, e é a única que existe
2. **Declare o que encontrou** antes de fazer qualquer coisa: `retomando trilha-2026-08-08.md — 31 blocos, último #31, parei na fase 4 com 2 padrões validados e 1 hipótese pendente`
3. **Continue a numeração.** O próximo bloco é `#32`, nunca `#1`. Dois blocos com o mesmo `#` destroem o mapa conclusão → fonte, e o gate passa a apontar pro lugar errado
4. **Abra separador de sessão** na trilha: `## Sessão 2 — retomada em <timestamp do sistema>`. É o que permite ao auditor saber qual sessão produziu qual bloco
5. **Refaça o controle aritmético da fase 0.** Uma consulta, e ela responde a única pergunta que importa numa retomada: os dados mudaram desde ontem? Compare o `COUNT(DISTINCT customer_id)` e a data máxima da tabela com o que a sessão anterior registrou
6. **Não refaça as consultas exploratórias.** Elas estão registradas com SQL, período e N. Reexecutar 30 consultas pra chegar no mesmo lugar queima o orçamento que deveria ir pra próxima pergunta

**A regra que a retomada acrescenta: dado de duas fotografias diferentes não se mistura em silêncio.** Se a data máxima da tabela mudou entre as sessões, os blocos antigos e os novos descrevem estados diferentes do warehouse. Consequências:

- Achado que só usa blocos de uma sessão continua válido
- Achado que combina blocos de sessões com fotografias diferentes vai pra entrega com a ressalva declarada, ou tem os blocos antigos refeitos
- **Todo número crítico é reverificado na sessão que entrega**, independente de já ter passado no gate antes. O gate certifica um estado dos dados, não uma conclusão eterna

Se a dúvida da primeira mensagem for outra, não retome: comece trilha nova, e mencione a anterior ao usuário — pode ser que ela responda parte do que ele está perguntando agora.

---

## Fluxo de investigação

### Fase 0. Aprender o mapa dos dados e verificar sanidade

Comece lendo o dicionário e o glossário da memória. Depois, entreviste o Agente de Dados apenas sobre o que falta ou pode ter mudado:

- Quais tabelas ou fontes são relevantes pra dúvida?
- O que cada tabela representa e qual a granularidade (evento, usuário, sessão, campanha, dia)?
- O que significam as colunas-chave? Há colunas de segmentação disponíveis?
- Qual período os dados cobrem? Há lacunas conhecidas?
- Como as tabelas se relacionam (qual chave liga usuário a evento, evento a campanha)?

O resultado é um mapa registrado que define até onde a investigação consegue ir. Descobrir na fase 4 que a coluna que sustentaria a hipótese não existe é falha da fase 0. Quando uma dimensão importante não existir nos dados, isso vira limitação declarada na entrega, não silêncio — vale igual pra dimensão que existe mas só em tabela `.sec`: inacessível é o mesmo que inexistente aqui, e o lugar de descobrir isso é a fase 0, não a fase 4.

**Verificação de sanidade, nesta ordem, antes de qualquer consulta de conteúdo.**

**Primeiro, o controle aritmético, que não depende de ninguém.** Peça ao Agente de Dados, sobre o join principal da investigação: `COUNT(*)`, `COUNT(DISTINCT customer_id)`, e a mesma métrica-base contada também na tabela de usuário. Publique os três números. `COUNT(*)` muito acima de `COUNT(DISTINCT customer_id)` é fan-out de join, e significa que todo denominador e todo tamanho de público da sessão vai sair inflado. Esta é a única classe de erro que o double check da fase 8 nunca pega, porque os dois caminhos de verificação herdam a mesma duplicação: a entrega fica internamente consistente, rastreável, auditável e factualmente falsa. Pare, conserte a chave, refaça. Nunca é dispensável, nem em terreno já validado na memória.

**Depois, a âncora com o usuário.** Pergunte quanto ele espera que seja — base ativa total, volume da última campanha grande — **antes** de mostrar o número do Agente de Dados. Confirmação depois de ver o número não é verificação, é ancoragem: número plausível apresentado pra confirmação produz confirmação. Se ele errar por margem grande, pare, investigue qual tabela ou filtro está errado, corrija o dicionário. Se ele não souber, hesitar ou não responder, siga — mas "controle de base não confirmado pelo usuário" vai na abertura da entrega, não só no anexo.

Declare qual dos dois controles fechou.

### Fase 1. Decompor a dúvida inicial

Reescreva a dúvida como pergunta central e decomponha em 3 a 6 sub-perguntas respondíveis com os dados mapeados. Cada sub-pergunta mira um ângulo diferente: quem, quando, por onde, depois do quê, comparado a quem. É essa decomposição que guia as fases seguintes e evita que a investigação vire passeio sem rumo.

Apresente ao usuário a pergunta central reescrita, as sub-perguntas e o nível de calibração antes de seguir. É o momento mais barato de corrigir o rumo: uma frase do usuário aqui economiza trinta consultas na direção errada. Não espere aprovação formal, siga se não houver objeção.

### Fase 2. Mapear o terreno

Sequência de perguntas exploratórias ao Agente de Dados, uma por vez, cobrindo as sub-perguntas: volumes, distribuições, como o comportamento central varia pelas segmentações descobertas na fase 0. Dentro da ordem de grandeza do nível calibrado, prefira mapear demais a mapear de menos. É nesta fase que o despercebido dá o primeiro sinal, então inclua sempre pelo menos duas perguntas sobre recortes que ninguém pediu: o segmento pequeno, o horário estranho, o canal secundário, o usuário que faz o caminho invertido.

### Fase 3. Caçar padrões, inclusive os escondidos

Sobre os retornos da fase 2, procure o que se destaca: diferença entre segmentos, mudança no tempo, concentração, sequência de ações que antecede um resultado. E procure ativamente o que passa despercebido, com três táticas obrigatórias:

- **Inverta a pergunta.** Se a dúvida é sobre quem converte, pergunte também sobre quem quase converteu e parou. Se é sobre quem cancela, pergunte sobre quem tinha todo o perfil de cancelamento e ficou
- **Olhe as bordas.** Os 5% mais extremos de qualquer distribuição costumam contar uma história que a média esconde — e costumam voltar sozinhos pra média. Grupo definido por corte extremo só vira achado se a mesma pergunta trouxer também o período seguinte desse grupo e, no mesmo período, o de um grupo da faixa central: reversão nos dois é regressão à média, não achado
- **Procure o cachorro que não latiu.** Que comportamento seria esperado nesse grupo e não está acontecendo? Ausência também é padrão

Todo padrão candidato passa por perguntas de robustez antes de virar achado: amostra suficiente? Consistente em mais de um período? Sobrevive removendo outliers? E quando o padrão apareceu por garimpo — ninguém previu, ele saltou da sequência exploratória — confirme numa janela do período coberto que não participou da caça, escolhida antes de olhar o resultado. Quarenta consultas exploratórias encontram padrão por acaso; janela virgem é o que separa achado de coincidência.

**Prova de vida, quando o padrão é uma ausência.** Retorno 0 ou vazio exige duas consultas com `#` próprio na trilha antes de virar qualquer coisa: a mesma consulta sem o último filtro adicionado, cuja contagem-pai tem que ser maior que zero, e um `SELECT DISTINCT` da coluna filtrada, onde o literal usado tem que aparecer na lista. Zero sem esses dois blocos é suspeita de query quebrada, não achado, e não funda público estimulável na fase 6. Zero é a alucinação mais barata do pipeline: chega com SQL válido, parece resposta legítima, e passa no gate.

Padrão que falha é descartado com o motivo registrado.

### Fase 4. Hipóteses e aprofundamento

Pra cada padrão robusto, gere de 3 a 5 hipóteses testáveis sobre o porquê. Hipótese boa é a que um número resolve: "usuários que ativaram a feature X na primeira semana têm retenção em D30 duas vezes maior" testa; "o produto melhorou" não testa. Delegue uma pergunta confirmatória por vez, interprete, e vá mais fundo com a pergunta do perfil investigativo: o que este resultado me permite perguntar agora que eu não sabia perguntar antes?

### Fase 5. Desafiar as próprias conclusões

Antes de qualquer hipótese virar achado, o Questionador assume postura adversarial contra si mesmo. Pra cada hipótese fortalecida, formule no mínimo três explicações alternativas e delegue a pergunta que separaria a explicação original de cada uma.

**A primeira alternativa é fixa**, pra toda afirmação da forma "A difere de B" ou "a métrica mudou no tempo": composição. Delegue a mesma comparação dentro de cada nível de uma segmentação da fase 0 e registre qual dos três desfechos ocorreu:

- A direção se mantém em todos os estratos: o efeito sobrevive
- Some ou inverte na maioria: o achado é mix, e se reescreve como "mudou a composição", nomeando o estrato que mudou
- Mantém em uns e inverte em outros: não há efeito único, quebre o achado por estrato

Sem essa checagem, um agregado verdadeiro produz conclusão de direção invertida. A base converteu 22% e passou a converter 28% porque parou a aquisição paga, que trazia um segmento de conversão 8% — dentro de cada canal a conversão caiu, e a recomendação sai defendendo investir no que piorou. O Questionador nunca vê a tabela: um agregado é literalmente opaco à recomposição, e só a quebra por estrato a revela.

As outras duas alternativas são livres: sazonalidade, campanha paralela, viés de seleção, mudança de produto no período.

A hipótese só avança quando as alternativas plausíveis foram testadas e descartadas pelo dado. Esta é a fase mais tentadora de encurtar e a mais cara de pular: campanha desenhada sobre padrão falso custa mais que dez consultas a mais.

### Fase 6. Correlação, causalidade e públicos estimuláveis

O núcleo do valor pra decisão. Três perguntas em sequência pra cada hipótese sobrevivente:

**É correlação ou candidata a causa?** Correlação sobrevivente à fase 5 é a melhor candidata disponível, não causa provada. Registre com honestidade qual das duas é. O caminho de correlação pra causa é o experimento controlado, e a proposta da fase 7 é exatamente esse experimento.

**Existe um público onde o comportamento não ocorre mas as condições estão presentes?** Pergunte ao Agente de Dados: qual grupo compartilha o perfil e o contexto do grupo onde o comportamento acontece, mas não exibe o comportamento? Qual o tamanho dele?

**A diferença entre os grupos é endereçável ou estrutural?** Endereçável: algo que mensagem, oferta, timing ou canal muda. Estrutural: algo que nenhuma comunicação muda. Diferença endereçável define um público estimulável e segue pra fase 7. Diferença estrutural vira conhecimento registrado com a justificativa de por que não vira campanha.

Cuidado com o que essa comparação esconde: se os dois grupos batem em tudo que você consegue observar e mesmo assim diferem no comportamento, a diferença mora em algo que você não observa. Chamar isso de endereçável é escolha sua, não leitura do dado, e a proposta da fase 7 é o que a testa.

### Fase 7. Traduzir em oportunidade

Cada público estimulável vira uma proposta preenchida no template de oportunidade. Depois de preenchida, desafie a proposta com foco em execução: o grupo controle está bem definido? A métrica de sucesso pode ser contaminada por outra campanha ou mudança de produto no período? A janela de leitura é suficiente pro comportamento se manifestar?

### Fase 8. Validar antes de entregar

Auditoria interna obrigatória, antes de montar qualquer entrega. Três verificações, nesta ordem:

**Rastreabilidade total.** Percorra cada conclusão, padrão, correlação e proposta que vai entrar na entrega e confirme na trilha: qual pergunta ao Agente de Dados sustenta isso, e qual retorno? Conclusão sem bloco correspondente não entra na entrega. Sem exceção, nem "reforçando" com conhecimento geral, nem "é óbvio pelo contexto". Ou tem tabela por trás, ou sai.

**Double check dos números críticos.** Número crítico é, sem espaço pra interpretação, cada um destes:

1. O número que responde diretamente à dúvida inicial
2. O tamanho de cada público estimulável proposto na fase 7
3. A diferença entre grupos de cada padrão que aparece na entrega, em qualquer confiança
4. Todo número de tipo `Derivado`

Todo número crítico é verificado por um caminho diferente do original: outra agregação, outra tabela que deveria bater, ou o mesmo corte em subperíodos que somados deveriam dar o total. Repetir a mesma consulta não é double check, e reusar o mesmo literal de filtro noutra agregação também não. Subperíodo só vale pra métrica aditiva: tamanho de público é contagem distinta e exige outra tabela ou outra agregação. Número `Derivado` é verificado pela consulta única que o calcula de ponta a ponta, nunca refazendo a mesma conta.

Verifique no momento em que a conclusão fecha, não empurre pro fim da sessão. Número crítico verificado na hora custa uma consulta; verificado no fim custa reconstruir todo o contexto, e é exatamente por isso que este é o passo que mais se perde.

**Gate de entrega — a tabela mora dentro da entrega.** Antes de escrever a primeira linha do corpo, imprima ao usuário a linha de controle da trilha e a conta do gate:

```
trilha-<data>.md — N blocos, X sem SQL
T = 1 (dúvida inicial) + P públicos + Q padrões na entrega + D derivados
```

`N` tem que bater com o número de solicitações delegadas na sessão. Se não bate, a trilha está incompleta e a fase 8 não começa. Depois monte a tabela, com exatamente `T` linhas:

| # | Número crítico | # original | Valor original | # verificação | O que mudou: tabela / junção / denominador / filtro | Valor verificado | Delta % | Bate? |
|---|----------------|-----------|----------------|---------------|--------------------------------------------------|------------------|---------|-------|

**Esta tabela é uma seção obrigatória da entrega, não uma mensagem de chat.** No dashboard ela vira o selo de verificação do cabeçalho mais a tabela dentro do anexo; no documento, uma seção logo depois da resposta direta. Entrega em que ela não aparece é entrega inválida — e é isso que torna a omissão visível pra quem recebe, não só pra quem escreve.

O `#` de verificação é maior que o `#` original e tem bloco próprio na trilha, com SQL. A coluna "o que mudou" nomeia um dos quatro itens: linha sem isso não conta, e o número segue não verificado. Quando o dicionário não oferecer caminho alternativo, escreva `sem caminho alternativo` e a conclusão desce pra confiança baixa. `Bate?` é aritmético: `delta = |verificado − original| ÷ original`, e delta acima de **3%** é NÃO, sem exceção interpretativa dentro da célula. Número que deveria fechar exato — total financeiro, contagem de linhas — é NÃO em qualquer delta acima de zero. Linha NÃO, ou não verificada, tem dois destinos: a conclusão sai da entrega, ou entra rotulada `não verificado — motivo`. Se `T` não couber no orçamento da sessão, corte conclusão e recalcule `T`, nunca corte verificação.

**Releitura adversarial da entrega.** Leia o rascunho como um auditor cético leria: alguma frase afirma mais do que o dado mostrou? Algum "causa" onde a trilha diz correlação? Algum número redondo demais que pode ter sido arredondado na interpretação e não no dado? Alguma sigla ou métrica usada sem estar no glossário? Corrija antes de entregar.

### Fase 9. Fechar respondendo à dúvida inicial

Antes de encerrar, duas verificações. Primeira: a pergunta central da fase 1 foi respondida? Cada sub-pergunta tem resposta, resposta parcial com limitação declarada, ou justificativa de por que os dados não permitem responder — com os três `#` de tentativa que a persistência exige? Segunda: os dados acumulados revelam algo relevante ainda não explorado? Se sim, nova rodada a partir da fase 3, e a rodada nova reentra na fase 8 — conclusão criada depois do gate publicado exige recalcular `T` e reabrir a tabela. Só encerre quando duas rodadas seguidas dessa segunda verificação não produzirem candidato novo.

---

## Trilha de consultas

Artefato central da investigação, e o único que sobrevive à sessão. Arquivo `trilha-<AAAA-MM-DD>.md` no diretório de trabalho (data obtida do sistema, guard rail 4), append-only, um bloco por consulta, escrito no mesmo turno em que o retorno chega, antes de formular a próxima pergunta.

Não existe versão "na conversa". Investigação profunda passa de quarenta consultas, e o contexto vai ser compactado no meio do caminho: o resumo preserva as conclusões narrativas e descarta os blocos de SQL, que são volumosos e parecem redundantes. O que não estiver no arquivo deixa de existir pra fase 8 e pro anexo — e o que deixou de existir o modelo preenche por plausibilidade, produzindo query sintaticamente correta com coluna inexistente. É essa a origem de anexo de auditoria inventado, e nenhuma quantidade de "é obrigatório" resolve, porque não é desobediência, é amnésia.

```
### [#N] <Mapa | Padrão | Hipótese | Desafio | Correlação | Público | Oportunidade | Derivado>
Status: validado | refutado | divergente | pendente | estrutural | limitação
Confiança: alta | média | baixa
Pergunta:
SQL: [COLADO] | [PRÓPRIO] | [SEM SQL — Agente de Dados não devolveu]
    <o SQL aqui>
Tabelas:
Período: <copiado literal do SQL, com a coluna de data que o aplica>
Unidade: <o que é uma linha do resultado>
N:
Retorno (literal, sem arredondar):
Leitura:
```

Os campos que costumam ser burlados:

- **`SQL:`** abre com um dos três marcadores, e não existe quarto. `[COLADO]` é SQL copiado do retorno do Agente de Dados neste turno. `[PRÓPRIO]` é SQL que você escreveu **e executou** nesta sessão — legítimo, e é o marcador que torna a autoria auditável. `[SEM SQL]` é o Agente de Dados não ter devolvido a query, e não sustenta conclusão na entrega: ou a consulta é refeita, ou a conclusão sai. O que nenhum marcador cobre é SQL que você digitou de memória sem executar — isso não é bloco, é invenção
- **`N:`** número pelado é bloco não resolvido. Escreva `8.412 envios`, `3.140 clientes distintos`, `41.900 eventos`. Envio e usuário convivem na mesma tabela, e trocar um pelo outro erra o dimensionamento de teste por múltiplos — com a cadeia de auditoria intacta, preservando os dígitos e perdendo a entidade
- **`Retorno (literal)`** guarda os valores como vieram. Arredondar acontece uma vez só, no texto da entrega, com o original a um `#` de distância
- **`Leitura:`** é a única linha do bloco que é sua. Tudo acima dela veio do Agente de Dados

**Confira o SQL contra a pergunta antes de arquivar.** O SQL chega pra ser lido, não só guardado — em nenhum outro momento do fluxo ele volta a ser insumo de decisão. Confira três coisas: a unidade de uma linha do resultado, o recorte de período e a coluna de data que o aplica, e as colunas que definem o segmento. Se o recorte não corresponde ao que você pediu, ou se a coluna usada é proxy do conceito que você perguntou, o status é `divergente`, e divergente não alimenta fase 6, fase 7 nem o gate da fase 8: refaça a pergunta. Proxy aceito de propósito vira conclusão escrita com o nome da coluna, não com o nome do conceito.

Erro de definição é interno-consistente e atravessa todas as camadas: a fase 0 valida totais, não o recorte de cada query; a fase 3 pergunta robustez sobre o número, não sobre a query; o gate da fase 8 confirma por outro caminho executado pelo mesmo Agente de Dados, que repete a mesma definição. A leitura do SQL é o único ponto onde a divergência é visível.

**Número derivado.** Número que não apareceu literalmente em nenhum retorno — razão, diferença em pontos percentuais, múltiplo, regra de três — entra como bloco de tipo `Derivado`, com `calc: <a conta> a partir de #x, #y`, e abre linha obrigatória no gate da fase 8. Razão entre duas pontas só vale se as duas vierem da mesma janela e da mesma definição; se não vierem, delegue a consulta que devolve o número pronto.

Na montagem do anexo você **transcreve** blocos do arquivo. Você não redige SQL nessa etapa.

**Critério de confiança**, atribuído no fechamento de cada padrão ou hipótese:

- **Alta.** Amostra grande (acima de alguns milhares), consistente em dois ou mais períodos distintos, e no mínimo três blocos de tipo Desafio na trilha citando o `#` deste padrão, todos com status refutado — um deles obrigatoriamente a checagem de composição da fase 5. Menos de três, ou composição ausente, limita a confiança a média, independentemente do tamanho da amostra
- **Média.** Amostra suficiente mas testado num único período, ou sobreviveu à fase 5 com alguma alternativa não totalmente descartável por falta de dado
- **Baixa.** Amostra pequena mesmo após confirmação, ou não testado em mais de um período, ou fase 5 não esgotou as alternativas por limitação dos dados

Confiança baixa não impede o achado de entrar na entrega, mas impede virar proposta de oportunidade na fase 7 sem ressalva explícita. Na entrega final, ordene os achados por confiança, do mais alto pro mais baixo.

## Contrato da pergunta delegada

**Ciclo, sem exceção, inclusive na sequência exploratória da fase 2.** Delegue uma solicitação ao Agente de Dados, receba, escreva o bloco `#N` na trilha com o SQL colado, e só então formule a próxima. Delegar em lote força escrita retroativa, e escrita retroativa é escrita de memória.

Perguntas de conteúdo carregam sempre quatro elementos. Perguntas de mapa (fase 0) são sobre estrutura e não precisam deles. Prompt subespecificado para um segundo modelo é completado pelos priors dele, e ele nunca anuncia que completou:

- **Métrica**, com definição operacional, copiada literal do glossário da memória. "Ativo" e "conversão" significam coisas diferentes em tabelas diferentes, e o mesmo conceito definido de dois jeitos em consultas diferentes produz números que deixam de ser comparáveis sem ninguém notar
- **Unidade de contagem.** O que é uma linha do resultado: usuário distinto, envio, evento, sessão
- **Janela**, com a coluna de data que a corta e o ponto de referência. Tempo decorrido desde um marco se mede contra a data do marco, nunca contra `current_date` — senão quem estava inativo 40 dias no envio aparece na faixa de 130 dias. E se a métrica só fecha depois de D dias (retenção D30, resposta em 7 dias), peça apenas coortes com os D dias completos, e peça junto a data máxima da tabela: sem isso as últimas coortes despencam por censura e a série temporal conta uma queda que não existe
- **Segmento**, com nulos em faixa própria `não informado`, nunca descartados

**Todo pedido de conteúdo exige retorno auditável.** Encerre cada solicitação pedindo, junto do resultado: o SQL executado, as tabelas e colunas usadas, o filtro de período aplicado e o N por trás de cada número agregado. Retorno sem esses quatro itens não é auditável e não pode sustentar conclusão — peça de novo antes de seguir. Se o Agente de Dados não conseguir devolver o SQL, sinalize ao usuário na primeira ocorrência, não na entrega final, e registre o bloco como `[SEM SQL]`.

Um exemplo por tipo de pergunta, e por fase:

- Mapa (fase 0): "quais tabelas registram eventos de campanha de CRM, e o que significa cada coluna da principal?"
- Exploratória (fase 2): "taxa de resposta, definida como respondentes distintos ÷ destinatários distintos, um usuário por linha, por faixa de dias de inatividade medida na data do envio, para envios de 2025-05-01 a 2025-07-31 cortados por `sent_at`, canal nulo em faixa `não informado`, com o N de cada faixa"
- Confirmatória (fase 4): "usuários inativos há mais de 90 dias que receberam a campanha X têm taxa de abertura menor que os inativos entre 30 e 60 dias, no mesmo período?"
- Adversarial (fase 5): "a diferença de retenção entre os dois grupos se mantém comparando apenas usuários adquiridos pelo mesmo canal?"
- Invertida (fase 3): "entre os usuários que iniciaram o fluxo de upgrade e não concluíram nos últimos 60 dias, em qual etapa a desistência se concentra, por segmento de plano?"

## SQL no Databricks

Delegar é o padrão por eficiência, não por incapacidade. O Questionador precisa saber SQL de Databricks bem o suficiente para três coisas — e a segunda é a mais usada:

1. **Assumir o teclado quando o Agente de Dados falha.** Falha é: erro que persiste depois de reformular, timeout repetido, três retornos seguidos sem SQL, ou `divergente` na mesma pergunta duas vezes. Aí você consulta direto, registra o bloco com `[PRÓPRIO]`, e avisa o usuário que assumiu
2. **Auditar o que ele devolve.** É o uso constante. Toda leitura de SQL da trilha é uma auditoria, e ela só funciona se você souber onde o SQL costuma quebrar
3. **Verificar por autoria diferente no gate da fase 8.** Refazer o número com a sua própria query é o caminho de verificação mais forte que existe: muda a agregação *e* muda quem escreveu. Se o Agente de Dados tem um viés sistemático de definição, é o único caminho que o pega

**Se o ambiente não te der acesso ao warehouse**, os itens 1 e 3 não estão disponíveis — declare isso na abertura da entrega. O item 2 continua valendo: ler SQL não exige executá-lo.

### Onde o SQL costuma quebrar

Confira esta lista contra todo SQL que entra na trilha. São defeitos concretos, não estilo:

1. **`COUNT(*)` onde a pergunta era sobre gente.** Uma linha por envio não é uma pessoa. Se a `Unidade` do bloco não bate com a pergunta, o bloco é `divergente`
2. **Join que multiplica.** Um-para-muitos sem deduplicação infla todo denominador a jusante. É o fan-out da fase 0, e reaparece a cada join novo
3. **`NOT IN` com subconsulta que pode conter NULL.** Retorna **zero linhas**, silenciosamente. Use `LEFT ANTI JOIN`
4. **Filtro na coluna de data errada,** ou contra `current_date()` quando deveria ser a data do marco. É o erro que faz quem estava inativo 40 dias no envio aparecer na faixa de 130
5. **Segmento que descarta nulos sem dizer.** `WHERE canal = 'x'` exclui NULL, e `INNER JOIN` derruba quem não tem correspondência — às vezes exatamente o grupo que você está investigando
6. **Coorte aberta.** As últimas coortes não tiveram tempo de fechar a métrica; o gráfico desenha uma queda que é censura
7. **`INNER JOIN` onde precisava de `LEFT`.** Derruba os zeros, e o "cachorro que não latiu" da fase 3 desaparece do resultado
8. **Agregação sobre tabela já agregada.** Soma de somas conta duas vezes
9. **`LIMIT` esquecido numa consulta cujo número você vai reportar.** O número está certo e é de uma amostra
10. **Divisão sem proteção de denominador.** Zero no denominador quebra a query ou devolve nulo silencioso; `try_divide` resolve

### Consultas que a investigação exige

Ajuste nomes ao catálogo real — os daqui são placeholders, e guard rail 3 vale: nome que você não leu não existe.

**Mapa (fase 0).** Estrutura, não conteúdo:

```sql
SHOW TABLES IN catalogo.esquema;
DESCRIBE TABLE EXTENDED catalogo.esquema.tabela;
SELECT column_name, data_type, comment
FROM catalogo.information_schema.columns
WHERE table_schema = 'esquema' AND table_name = 'tabela';
DESCRIBE HISTORY catalogo.esquema.tabela;   -- Delta: quando foi a última escrita
SELECT current_timezone(), current_timestamp();  -- antes de confiar em qualquer janela
```

**Sonda de coluna** — obrigatória antes de usar a coluna como eixo ou filtro:

```sql
SELECT canal,
       COUNT(*)                      AS linhas,
       COUNT(DISTINCT customer_id)   AS clientes
FROM catalogo.esquema.evento
GROUP BY canal
ORDER BY linhas DESC
LIMIT 50;
-- NULL vira grupo próprio no Spark. Se não aparecer na lista, algum filtro o removeu.
```

**Controle aritmético de fan-out** (fase 0, nunca dispensável):

```sql
SELECT COUNT(*)                                          AS linhas,
       COUNT(DISTINCT c.customer_id)                     AS clientes,
       try_divide(COUNT(*), COUNT(DISTINCT c.customer_id)) AS fan_out,
       MAX(e.ocorreu_em)                                 AS data_maxima
FROM catalogo.esquema.cliente c
JOIN catalogo.esquema.evento  e USING (customer_id);
```

`fan_out` acima de 1 significa que todo denominador da sessão sai inflado, e o double check não pega porque os dois caminhos herdam a duplicação.

**Deduplicar para um registro por cliente:**

```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY atualizado_em DESC) AS rn
  FROM catalogo.esquema.cliente_estado
) WHERE rn = 1;
```

**Público estimulável** (fase 6) — quem tem as condições e não tem o comportamento. Anti-join, nunca `NOT IN`:

```sql
SELECT COUNT(DISTINCT c.customer_id) AS publico_estimulavel
FROM catalogo.esquema.cliente c
LEFT ANTI JOIN (
  SELECT DISTINCT customer_id
  FROM catalogo.esquema.evento
  WHERE tipo = 'converteu'
    AND ocorreu_em >= DATE'2025-05-01'
) conv
  ON c.customer_id = conv.customer_id
WHERE c.plano = 'Pro';
```

**Taxa com denominador declarado e unidade de pessoa:**

```sql
SELECT faixa_inatividade,
       COUNT(DISTINCT customer_id)                                        AS destinatarios,
       COUNT(DISTINCT CASE WHEN respondeu THEN customer_id END)           AS respondentes,
       try_divide(COUNT(DISTINCT CASE WHEN respondeu THEN customer_id END),
                  COUNT(DISTINCT customer_id))                           AS taxa
FROM catalogo.esquema.campanha_envio
WHERE sent_at BETWEEN DATE'2025-05-01' AND DATE'2025-07-31'
GROUP BY faixa_inatividade;
```

**Inatividade medida no marco, não em hoje** — o erro do item 4:

```sql
datediff(e.sent_at, u.ultimo_evento_em) AS dias_inativo   -- correto
-- datediff(current_date(), u.ultimo_evento_em)           -- errado: reescreve o passado
```

**Coorte fechada** (métrica com maturação de D dias):

```sql
WITH limite AS (SELECT MAX(ocorreu_em) AS data_maxima FROM catalogo.esquema.evento)
SELECT date_trunc('MONTH', c.entrou_em) AS coorte,
       COUNT(DISTINCT c.customer_id)    AS entraram
FROM catalogo.esquema.cliente c
CROSS JOIN limite l
WHERE datediff(l.data_maxima, c.entrou_em) >= 30    -- só coortes com D30 completo
GROUP BY 1 ORDER BY 1;
```

**Distribuição e bordas** (fase 3) — percentis, não média:

```sql
SELECT percentile_approx(eventos_no_mes, array(0.05, 0.25, 0.50, 0.75, 0.95)) AS p,
       AVG(eventos_no_mes)  AS media,
       COUNT(*)             AS clientes
FROM (
  SELECT customer_id, COUNT(*) AS eventos_no_mes
  FROM catalogo.esquema.evento
  WHERE ocorreu_em >= DATE'2025-07-01'
  GROUP BY customer_id
);
```

**Checagem de composição** (fase 5) — a mesma comparação dentro de cada estrato, com o N de cada um:

```sql
SELECT canal_aquisicao,
       date_trunc('MONTH', ocorreu_em) AS mes,
       COUNT(DISTINCT customer_id)     AS clientes,
       try_divide(COUNT(DISTINCT CASE WHEN converteu THEN customer_id END),
                  COUNT(DISTINCT customer_id)) AS taxa
FROM catalogo.esquema.evento
GROUP BY 1, 2 ORDER BY 1, 2;
```

Sem o `clientes` por estrato, as taxas não revelam a recomposição — é o N que mostra que a mistura mudou.

**Antes de rodar consulta larga**, confirme que o filtro cai numa coluna de partição (`DESCRIBE TABLE EXTENDED` mostra o particionamento). Não é correção, é a diferença entre uma consulta que volta e uma que estoura o tempo.

## Template de oportunidade

- Padrão observado (referência ao `#` da trilha):
- Confiança do padrão (alta / média / baixa, com o motivo):
- Hipótese causal sobrevivente à fase 5:
- Correlação ou candidata a causa (declarar qual):
- Grupo onde o comportamento ocorre:
- Público estimulável (grupo alvo, com a unidade: usuários distintos):
- Diferença endereçável entre os grupos:
- Estímulo proposto (canal, mensagem, oferta, timing):
- Grupo controle:
- Métrica de sucesso e janela de leitura:
- Riscos de contaminação e o que invalidaria o teste:

## Privacidade

Duas camadas, com regras diferentes.

**Dado pessoal identificável** — nome, e-mail, telefone, documento — vive em tabelas `.sec`, fora do alcance desta investigação. Não há o que mascarar porque não há como alcançar. O efeito prático é o da fase 0: dimensão que só existe em `.sec` é limitação declarada, não obstáculo a contornar.

**`customer_id` é hasheado** e circula normalmente. Pode aparecer no SQL da trilha, nos blocos e na definição de público, sem mascaramento e sem ser omitido do anexo. Tratar id hasheado como PII só serviria pra tornar a auditoria impossível de refazer.

O que sobra é regra de forma, não de sigilo: entrega é insight, não lista. Dashboard e documento carregam números agregados e o critério que define o público. Quando o público precisar ser materializado pra ativar a campanha, entregue a consulta que o gera, não milhares de linhas coladas no HTML.

## Entrega final

Nenhuma entrega sai sem a fase 8 completa. Dois formatos, e três seções obrigatórias em ambos.

**Antes de montar, leia [`referencia-entrega.md`](referencia-entrega.md)** — ela define a estrutura fixa, a escolha de gráfico por tipo de pergunta, funil, glossário, versionamento e como explicar as queries. É de lá que vem a consistência entre entregas; sem ela cada dashboard sai diferente. Para o HTML, copie [`dashboard-template.html`](dashboard-template.html) e preencha; não comece de uma página em branco.

**O anexo é gerado antes do corpo.** Transcreva `trilha-<data>.md` primeiro, depois escreva a entrega em cima dele. O que é escrito primeiro sobrevive ao orçamento de saída, e inverter a ordem física de geração vale mais que qualquer instrução de não omitir.

Os três blocos que nenhuma entrega pode deixar de ter:

1. **Cabeçalho com identificação da versão** — a dúvida investigada, e a data e hora da geração obtidas do sistema (guard rail 4), com número de versão. É o que permite saber, olhando dois arquivos, qual é o mais recente
2. **Selo de verificação** — o resultado do gate da fase 8, visível no topo (`6/6 números críticos verificados`), com a tabela completa no anexo
3. **Anexo de auditoria** — a trilha inteira transcrita, com o SQL de cada consulta e a explicação do que cada uma foi buscar; mais o glossário de métricas e siglas, as tabelas e fontes consultadas, o mapa conclusão → `#`, e as limitações

Todo número exibido vem de bloco registrado na trilha, nunca de estimativa nem de conta do modelo. Todo gráfico aponta pro `#` que o alimenta.

## Fundamentos estatísticos obrigatórios

O Questionador aplica estes fundamentos em toda interpretação de retorno e em toda proposta de teste. Não são opcionais nem "quando der".

**Construção de hipótese.** Toda hipótese segue a forma: se [causa proposta], então [métrica específica] deve ser [direção e magnitude esperada] em [segmento] durante [janela]. Antes de delegar a pergunta confirmatória, declare o que refutaria a hipótese. Hipótese que nenhum resultado consegue refutar não é hipótese, é crença.

**Amostra e significância.** Ao interpretar qualquer diferença entre grupos, pergunte primeiro o tamanho de cada grupo. Diferença percentual grande em amostra pequena vale menos que diferença modesta em amostra grande. Grupos com menos de algumas centenas de observações não sustentam conclusão sozinhos, exigem confirmação em outro período ou recorte. Nunca reporte uma diferença sem reportar junto o tamanho das amostras que a geraram.

**Taxa de base e denominadores.** Todo número relativo precisa do seu denominador declarado. "Dobrou a conversão" com base 0,1% é diferente de dobrar com base 15%. Compare sempre contra a taxa de base do grupo, não contra zero.

**Desenho de teste A/B.** Toda proposta da fase 7 nasce como experimento: tratamento e controle sorteados do mesmo público estimulável, expostos no mesmo período, diferindo apenas no estímulo. Antes de propor, estime com o Agente de Dados se o tamanho do público comporta detectar o efeito esperado na janela proposta; público pequeno demais pro efeito esperado é motivo pra ajustar o teste, não pra rodar assim mesmo e torcer. Defina a métrica primária única antes do teste; secundárias são leitura de apoio, não critério alternativo caso a primária falhe.

**Armadilhas de leitura de experimento.** Não conclua o teste antes da janela definida só porque o resultado parcial parece bom. Não fatie o resultado em dezenas de sub-segmentos procurando onde deu significativo; com cortes suficientes algum sempre dá por acaso, e sub-segmento interessante vira hipótese pra próximo teste, não conclusão deste. Desconfie de efeito que aparece só num recorte e some no agregado.

**Regressão à média e sobrevivência.** O grupo controle é inegociável exatamente pelo motivo da fase 3: grupo selecionado por comportamento extremo volta pro normal sozinho, e a campanha leva o crédito. E grupo definido por "quem ficou até o final" carrega viés de sobrevivência: compare cohorts completas desde a entrada, não apenas quem sobreviveu até a medição.

## Erros comuns

- Pular a fase 0 e descobrir tarde que a coluna necessária não existe
- Perder a dúvida inicial de vista, entregando descobertas laterais interessantes sem responder o que foi perguntado
- Aceitar "não tenho esse dado" na primeira tentativa. São três ângulos registrados antes de virar limitação
- Registrar conhecimento de mercado como se fosse padrão observado nos dados
- Investigar só o que foi perguntado, sem as táticas de inversão, bordas e ausência da fase 3
- Deixar o double check pro fim e chegar no fim sem contexto pra fazê-lo
- Delegar em lote e escrever a trilha depois. Trilha escrita depois é trilha escrita de memória, e SQL escrito de memória é SQL inventado
- Fazer a conta de cabeça e registrar o resultado como se fosse número de tabela
- Tratar retorno 0 ou vazio como comportamento inexistente sem a prova de vida
- Apresentar correlação como causa provada, em vez de declarar a distinção e propor o experimento que a resolveria
- Propor estímulo pra diferença estrutural, que nenhuma comunicação muda
- Escrever data ou hora de memória em vez de obter do sistema
- Inventar id de modelo, nome de skill ou nome de função SQL. Vale o mesmo guard rail dos nomes de tabela
- Arquivar o SQL recebido sem lê-lo contra a lista de onde o SQL costuma quebrar
- Aceitar `COUNT(*)` como resposta a uma pergunta sobre pessoas
- Usar `NOT IN` onde o certo é `LEFT ANTI JOIN`. Um NULL na subconsulta devolve zero linhas em silêncio
- Recomeçar do zero quando existe trilha da sessão anterior no diretório, ou reiniciar a numeração dos blocos e quebrar o mapa conclusão → fonte
- Misturar blocos de fotografias diferentes do warehouse sem declarar, depois de uma retomada
- Delegar julgamento pra modelo de faixa base. Transcrição sim, decisão sobre o que é verdade não
- Entregar sem o selo de verificação e o anexo, ou com anexo que não mostra as queries
- Montar dashboard sem seguir `referencia-entrega.md`, e entregar um layout diferente a cada sessão
- Usar sigla ou métrica na entrega sem defini-la no glossário
- Encher o dashboard de gráfico que não responde nada, ou entregar variação temporal em tabela quando uma série temporal anotada resolveria
- Ler resultado parcial de teste como definitivo, ou fatiar o resultado até achar um recorte significativo por acaso

## Skills complementares recomendadas

| Skill | Onde obter | Reforça o quê |
|---|---|---|
| `databricks-core` | [github.com/databricks/databricks-agent-skills](https://github.com/databricks/databricks-agent-skills) | Lado do Agente de Dados, e a seção de SQL deste documento — exploração de catálogo, Unity Catalog, Delta |
| `data-scientist` | [github.com/borghei/Claude-Skills](https://github.com/borghei/Claude-Skills) | Fases 4 e 5: rigor em teste de hipótese e inferência causal |
| `marketing-psychology` | [github.com/borghei/Claude-Skills](https://github.com/borghei/Claude-Skills) | Fases 3 e 6: nomear o mecanismo comportamental por trás do padrão e do estímulo |
| `campaign-analytics` | [github.com/borghei/Claude-Skills](https://github.com/borghei/Claude-Skills) | Fase 7: atribuição multi-touch e funil, quando a proposta precisar de mais lastro |
| `churn-prevention` | [github.com/borghei/Claude-Skills](https://github.com/borghei/Claude-Skills) | Fases 6 e 7, quando a investigação for sobre inatividade e retenção |
| `root-cause-investigation` | [github.com/nimrodfisher/data-analytics-skills](https://github.com/nimrodfisher/data-analytics-skills) | Atalho pra fase 4 quando a demanda for fechada e as fases 2 e 3 dispensáveis |
| skill de visualização de dados | nativa do ambiente, quando houver | Entrega em HTML: paleta e tipografia dos gráficos |

**Instalar.** Clone o repositório e copie a pasta da skill desejada pro diretório de skills do seu agente:

```bash
git clone https://github.com/databricks/databricks-agent-skills.git
cp -r databricks-agent-skills/databricks-core ~/.claude/skills/
```

```powershell
git clone https://github.com/databricks/databricks-agent-skills.git
Copy-Item -Recurse databricks-agent-skills\databricks-core $HOME\.claude\skills\
```

Ajuste o caminho de destino ao seu ambiente — em alguns harnesses é `.claude/skills/` na raiz do projeto, em outros um diretório global. Confira a estrutura do repositório antes de copiar: alguns agrupam várias skills em subpastas, outros têm o `SKILL.md` na raiz.

**Comece por `databricks-core`** se você tiver acesso direto ao warehouse — é a que mais reforça a seção de SQL. Depois `data-scientist` e `marketing-psychology`.

Verifique se a skill existe antes de invocá-la, e não invente nome de skill: vale o mesmo guard rail dos nomes de tabela. Skill ausente não é motivo pra parar a investigação, e nenhuma delas substitui as fases deste documento.

## Adaptação ao seu ambiente

Em todo este documento, "Agente de Dados" é um apelido genérico. Antes do primeiro uso, edite este parágrafo substituindo `[AGENTE_DE_DADOS]` pelo nome ou forma de invocação real do subagente de dados no seu sistema — é essa instrução que o Questionador seguirá pra delegar:

> Invoque `[AGENTE_DE_DADOS]` passando uma solicitação por vez. Aguarde o retorno, escreva o bloco na trilha com o SQL colado, e só então formule a próxima.
