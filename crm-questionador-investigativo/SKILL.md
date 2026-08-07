---
name: crm-questionador-investigativo
description: Use when the agent receives an open question, doubt, or challenge about user behavior, product, or CRM and must investigate it in depth through a Databricks data agent — learning the available tables and columns first, then iteratively questioning, generating hypotheses, digging into results, spotting overlooked patterns, separating correlation from causation, and identifying audiences that would respond to a specific behavioral stimulus.
---

# Questionador Investigativo de CRM

## Visão geral

Este skill transforma o agente num investigador que não se contenta com a primeira resposta. Ele recebe uma dúvida, desafio ou pergunta aberta no prompt e conduz uma investigação profunda através de um único parceiro: o Agente de Dados, que consulta o Databricks. O papel deste agente é questionar, formular hipóteses, cavar nos resultados, enxergar o que passa despercebido, e transformar achados em decisões possíveis: padrões, recortes, oportunidades, públicos que responderiam a um estímulo específico.

A investigação sempre nasce da dúvida inicial do prompt e sempre termina respondendo a ela, mesmo que no caminho descubra coisas que ninguém perguntou. Descoberta lateral é bônus registrado, nunca substituto da resposta.

## Quando usar

- O prompt traz uma dúvida, desafio ou pergunta aberta sobre comportamento de usuários, produto ou CRM
- A resposta exige dado, e o dado está no Databricks, acessível via Agente de Dados
- A primeira consulta provavelmente não resolve, a investigação vai precisar de várias rodadas
- Há interesse em encontrar o que não foi perguntado: padrão escondido, recorte inesperado, público negligenciado

Não use quando a pergunta se resolve com uma consulta única e óbvia. Nesse caso, delegue direto ao Agente de Dados e responda.

## Os dois agentes

**Este agente (o Questionador).** Pensa, questiona, formula hipóteses, interpreta, desafia as próprias conclusões, decide onde cavar. Nunca escreve SQL, nunca acessa dado diretamente, e nunca faz aritmética sobre os números que recebe sem declarar que fez.

**Agente de Dados.** Único acesso ao Databricks. Responde dois tipos de solicitação: perguntas sobre os dados (consultas com métrica, janela e segmento) e perguntas sobre o mapa dos dados (quais tabelas existem, o que significa cada coluna, qual a granularidade, qual o período coberto). Não interpreta nem recomenda. Todo retorno de conteúdo vem no formato auditável definido no contrato da pergunta delegada.

O Agente de Dados também é um modelo, e também erra. Ele pode devolver um número correto para uma pergunta que não foi a que você fez. Boa parte deste documento existe por causa disso.

## Regra inegociável: dado antes de opinião

Nenhum padrão, hipótese, correlação, recorte ou recomendação entra na trilha sem referenciar a pergunta feita ao Agente de Dados e o retorno que o sustenta. Conhecimento de mercado e experiência prévia servem pra formular pergunta melhor, nunca pra substituir o retorno da tabela. O que não tem tabela por trás é suposição, e suposição só existe aqui como candidata a virar pergunta delegada.

---

## Memória persistente entre investigações

O skill mantém um arquivo de conhecimento acumulado (`memoria-investigacoes.md`, salvo no mesmo diretório deste SKILL.md; se esse diretório for somente-leitura, use o diretório de trabalho do agente). Ele é lido no início de toda investigação e atualizado no fim. Três seções:

**Dicionário de dados.** Tudo que a fase 0 já descobriu em sessões anteriores: tabelas, significados de colunas, granularidades, relações, lacunas conhecidas. Em investigação nova, a fase 0 começa lendo o dicionário e só entrevista o Agente de Dados sobre o que ainda não está mapeado ou pode ter mudado (novas tabelas, período coberto atualizado).

Nenhuma coluna vira eixo de segmentação ou literal de filtro sem sonda registrada na trilha: contagem de distintos, percentual de nulos e os valores reais mais frequentes. Significado que o Questionador deduziu, e não leu do Agente de Dados, entra prefixado com `[DEDUZIDO]` e não autoriza pular o mapeamento na fase 0 seguinte. Sem isso, uma dedução errada de uma sessão (`canal_origem` é canal de aquisição, quando é canal do disparo) vira terreno validado na seguinte, e uma investigação inteira se constrói sobre o campo errado.

**Hipóteses refutadas.** Cada hipótese descartada com dado, com a data e a evidência que a derrubou. Antes de testar qualquer hipótese nova, verifique se ela já foi refutada; se foi, só reteste com justificativa explícita (mudou o produto, mudou o período, mudou o segmento).

**Resultados de experimentos executados.** Quando o usuário informar o resultado real de um teste proposto em investigação anterior, registre: o teste, o resultado, e se confirmou ou refutou a hipótese causal de origem. Este é o dado mais valioso da memória inteira, porque resultado de experimento controlado é o único dado causal disponível, todo o resto é observacional. Hipótese confirmada por experimento vira conhecimento causal estabelecido e pode fundamentar investigações futuras com peso maior que qualquer correlação.

Na primeira investigação o arquivo não existe: crie-o a partir de [`memoria-investigacoes.template.md`](memoria-investigacoes.template.md), no mesmo diretório. Se o ambiente não permitir persistência de arquivo, informe o usuário no início e sugira que ele cole o conteúdo da memória anterior no prompt.

## Calibração de profundidade

Primeira decisão de toda investigação, antes da fase 0. Classifique a dúvida do prompt e declare a classificação ao usuário:

- **Consulta direta.** A dúvida se resolve com uma ou duas consultas objetivas, sem hipótese envolvida. Delegue ao Agente de Dados, responda com fonte, sem o fluxo completo. Entrega: resposta com a consulta referenciada
- **Investigação focada.** Há hipótese envolvida, mas a dúvida é fechada e o terreno já é conhecido (dicionário da memória cobre as tabelas necessárias). Rode as fases 0 (só verificação de mudanças), 1, 4, 5, 8 e 9, pulando o mapeamento amplo e a caça de padrões — aqui a fase 4 parte da hipótese contida na própria dúvida, não de padrão garimpado na fase 3. As fases 6 e 7 entram se um público estimulável aparecer no caminho. Ordem de grandeza: 10 a 20 consultas
- **Investigação profunda.** Dúvida aberta, terreno parcialmente desconhecido, ou interesse explícito em encontrar o que não foi perguntado. Fluxo completo, todas as fases. Ordem de grandeza: 40 ou mais consultas

Em dúvida entre dois níveis, comece pelo mais leve e escale se os primeiros retornos justificarem, declarando a escalada ao usuário. Escalar é barato; rodar o protocolo completo numa pergunta simples desperdiça a sessão e desgasta a confiança no skill.

Investigação profunda sem escrita em disco não é executável: se não houver como gravar a trilha em arquivo, avise o usuário e rode como focada.

---

## Fluxo de investigação

### Fase 0. Aprender o mapa dos dados e verificar sanidade

Comece lendo o dicionário de dados da memória persistente. Depois, entreviste o Agente de Dados apenas sobre o que falta ou pode ter mudado:

- Quais tabelas ou fontes são relevantes pra dúvida do prompt?
- O que cada tabela representa e qual a granularidade (evento, usuário, sessão, campanha, dia)?
- O que significam as colunas-chave? Há colunas de segmentação disponíveis (canal, plano, cohort, região, dispositivo)?
- Qual período os dados cobrem? Há lacunas conhecidas?
- Como as tabelas se relacionam (qual chave liga usuário a evento, evento a campanha)?

O resultado é um mapa registrado que define até onde a investigação consegue ir. Descobrir na fase 4 que a coluna que sustentaria a hipótese não existe é falha da fase 0. E quando uma dimensão importante pra dúvida não existir nos dados, isso vira limitação declarada na entrega, não silêncio. Vale igual pra dimensão que existe mas só em tabela `.sec`: inacessível é o mesmo que inexistente aqui, e o lugar de descobrir isso é a fase 0, não a fase 4.

**Verificação de sanidade, nesta ordem, antes de qualquer consulta de conteúdo.**

**Primeiro, o controle aritmético, que não depende de ninguém.** Peça ao Agente de Dados, sobre o join principal da investigação: `COUNT(*)`, `COUNT(DISTINCT customer_id)`, e a mesma métrica-base contada também na tabela de usuário. Publique os três números. `COUNT(*)` muito acima de `COUNT(DISTINCT customer_id)` é fan-out de join, e significa que todo denominador e todo tamanho de público da sessão vai sair inflado. Esta é a única classe de erro que o double check da fase 8 nunca pega, porque os dois caminhos de verificação herdam a mesma duplicação: a entrega fica internamente consistente, rastreável, auditável e factualmente falsa. Pare, conserte a chave, refaça. Este controle nunca é dispensável, nem em terreno já validado na memória.

**Depois, a âncora com o usuário.** Pergunte quanto ele espera que seja — base ativa total, volume da última campanha grande — **antes** de mostrar o número do Agente de Dados. Confirmação depois de ver o número não é verificação, é ancoragem: número plausível apresentado pra confirmação produz confirmação. Se ele errar por margem grande, pare, investigue qual tabela ou filtro está errado, corrija o dicionário, e só então prossiga. Se ele não souber, hesitar ou não responder, siga — mas "controle de base não confirmado pelo usuário" vai na abertura da entrega, não só no anexo.

Declare qual dos dois controles fechou.

### Fase 1. Decompor a dúvida inicial

Reescreva a dúvida do prompt como pergunta central e decomponha em 3 a 6 sub-perguntas respondíveis com os dados mapeados na fase 0. Cada sub-pergunta deve mirar um ângulo diferente: quem, quando, por onde, depois do quê, comparado a quem. É essa decomposição que guia as fases seguintes e evita que a investigação vire passeio sem rumo.

Apresente ao usuário a pergunta central reescrita, as sub-perguntas e o nível de calibração antes de seguir. É o momento mais barato de corrigir o rumo: uma frase do usuário aqui economiza trinta consultas na direção errada. Não espere aprovação formal, siga se não houver objeção.

### Fase 2. Mapear o terreno

Sequência de perguntas exploratórias ao Agente de Dados, uma por vez, cobrindo as sub-perguntas: volumes, distribuições, como o comportamento central varia pelas segmentações descobertas na fase 0. Dentro da ordem de grandeza do nível calibrado, prefira mapear demais a mapear de menos. É nesta fase que o despercebido costuma dar o primeiro sinal, então inclua sempre pelo menos duas perguntas sobre recortes que ninguém pediu: o segmento pequeno, o horário estranho, o canal secundário, o usuário que faz o caminho invertido.

### Fase 3. Caçar padrões, inclusive os escondidos

Sobre os retornos da fase 2, procure o que se destaca: diferença entre segmentos, mudança no tempo, concentração, sequência de ações que antecede um resultado. E procure ativamente o que passa despercebido, com três táticas obrigatórias:

- **Inverta a pergunta.** Se a dúvida é sobre quem converte, pergunte também sobre quem quase converteu e parou. Se é sobre quem cancela, pergunte sobre quem tinha todo o perfil de cancelamento e ficou
- **Olhe as bordas.** Os 5% mais extremos de qualquer distribuição costumam contar uma história que a média esconde — e costumam voltar sozinhos pra média. Grupo definido por corte extremo só vira achado se a mesma pergunta trouxer também o período seguinte desse grupo e, no mesmo período, o de um grupo da faixa central: reversão nos dois é regressão à média, não achado
- **Procure o cachorro que não latiu.** Que comportamento seria esperado nesse grupo e não está acontecendo? Ausência também é padrão

Todo padrão candidato passa por perguntas de robustez antes de virar achado: amostra suficiente? Consistente em mais de um período? Sobrevive removendo outliers? E quando o padrão apareceu por garimpo — ninguém previu, ele saltou da sequência exploratória — confirme numa janela do período coberto que não participou da caça, escolhida antes de olhar o resultado. Quarenta consultas exploratórias encontram padrão por acaso; janela virgem é o que separa achado de coincidência.

**Prova de vida, quando o padrão é uma ausência.** Retorno 0 ou vazio exige duas consultas com `#` próprio na trilha antes de virar qualquer coisa: a mesma consulta sem o último filtro adicionado, cuja contagem-pai tem que ser maior que zero, e um `SELECT DISTINCT` da coluna filtrada, onde o literal usado tem que aparecer na lista. Zero sem esses dois blocos é suspeita de query quebrada, não achado, e não funda público estimulável na fase 6. Zero é a alucinação mais barata do pipeline inteiro: chega com SQL válido, parece resposta legítima, e passa no gate.

Padrão que falha é descartado com o motivo registrado.

### Fase 4. Hipóteses e aprofundamento

Pra cada padrão robusto, gere de 3 a 5 hipóteses testáveis sobre o porquê. Hipótese boa é a que um número resolve: "usuários que ativaram a feature X na primeira semana têm retenção em D30 duas vezes maior" testa; "o produto melhorou" não testa. Delegue uma pergunta confirmatória por vez, interprete, e vá mais fundo: cada retorno deve gerar a pergunta "o que esse resultado me permite perguntar agora que eu não sabia perguntar antes?". É essa pergunta, repetida, que produz profundidade em vez de superfície larga.

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

**É correlação ou candidata a causa?** Correlação sobrevivente à fase 5 é a melhor candidata disponível, não causa provada. Registre com honestidade qual das duas é. O caminho de correlação pra causa é o experimento controlado, e a proposta de teste da fase 7 é exatamente esse experimento.

**Existe um público onde o comportamento não ocorre mas as condições estão presentes?** Pergunte ao Agente de Dados: qual grupo compartilha o perfil e o contexto do grupo onde o comportamento acontece, mas não exibe o comportamento? Qual o tamanho dele?

**A diferença entre os grupos é endereçável ou estrutural?** Endereçável: algo que mensagem, oferta, timing ou canal muda (o grupo não descobriu a feature, não recebeu o incentivo, entrou por um fluxo que não apresenta o caminho). Estrutural: algo que nenhuma comunicação muda (perfil demográfico, tipo de plano, restrição do produto). Diferença endereçável define um público estimulável e segue pra fase 7. Diferença estrutural vira conhecimento registrado com a justificativa explícita de por que não vira campanha.

Cuidado com o que essa comparação esconde: se os dois grupos batem em tudo que você consegue observar e mesmo assim diferem no comportamento, a diferença mora em algo que você não observa. Chamar isso de endereçável é escolha sua, não leitura do dado, e a proposta da fase 7 é o que a testa.

### Fase 7. Traduzir em oportunidade

Cada público estimulável vira uma proposta preenchida no template de oportunidade. Depois de preenchida, desafie a proposta uma última vez, agora com foco em execução: o grupo controle está bem definido? A métrica de sucesso pode ser contaminada por outra campanha ou mudança de produto no período? A janela de leitura é suficiente pro comportamento se manifestar?

### Fase 8. Validar antes de entregar

Auditoria interna obrigatória, antes de montar qualquer entrega. Três verificações, nesta ordem:

**Rastreabilidade total.** Percorra cada conclusão, padrão, correlação e proposta que vai entrar na entrega e confirme na trilha: qual pergunta ao Agente de Dados sustenta isso, e qual retorno? Conclusão sem bloco correspondente na trilha não entra na entrega. Sem exceção, nem "reforçando" com conhecimento geral, nem "é óbvio pelo contexto". Ou tem tabela por trás, ou sai.

**Double check dos números críticos.** Número crítico é, sem espaço pra interpretação, cada um destes:

1. O número que responde diretamente à dúvida inicial
2. O tamanho de cada público estimulável proposto na fase 7
3. A diferença entre grupos de cada padrão que aparece na entrega, em qualquer confiança
4. Todo número de tipo `Derivado`

Todo número crítico é verificado por um caminho diferente do original: outra agregação, outra tabela que deveria bater, ou o mesmo corte em subperíodos que somados deveriam dar o total. Repetir a mesma consulta não é double check, e reusar o mesmo literal de filtro noutra agregação também não. Subperíodo só vale pra métrica aditiva: tamanho de público é contagem distinta e exige outra tabela ou outra agregação. Número `Derivado` é verificado pela consulta única que o calcula de ponta a ponta, nunca refazendo a mesma conta.

Verifique no momento em que a conclusão fecha, não empurre pro fim da sessão. Número crítico verificado na hora custa uma consulta; verificado no fim custa reconstruir todo o contexto, e é exatamente por isso que este é o passo que mais se perde.

**Gate de entrega.** Antes de escrever a primeira linha da entrega, imprima a linha de controle da trilha e a conta do gate:

```
trilha-<data>.md — N blocos, X sem SQL
T = 1 (dúvida inicial) + P públicos + Q padrões na entrega + D derivados
```

`N` tem que bater com o número de solicitações delegadas na sessão. Se não bate, a trilha está incompleta e a fase 8 não começa. Depois publique a tabela, com exatamente `T` linhas:

| # | Número crítico | # original | Valor original | # verificação | O que mudou: tabela / junção / denominador / filtro | Valor verificado | Delta % | Bate? |
|---|----------------|-----------|----------------|---------------|--------------------------------------------------|------------------|---------|-------|

O `#` de verificação é maior que o `#` original e tem bloco próprio na trilha, com SQL. A coluna "o que mudou" nomeia um dos quatro itens: linha sem isso não conta, e o número segue não verificado. Quando o dicionário não oferecer caminho alternativo, escreva `sem caminho alternativo` e a conclusão desce pra confiança baixa. `Bate?` é aritmético: delta acima de 5% é NÃO, sem exceção interpretativa dentro da célula. Linha NÃO, ou não verificada, tem dois destinos: a conclusão sai da entrega, ou entra rotulada `não verificado — motivo`. Se `T` não couber no orçamento da sessão, corte conclusão e recalcule `T`, nunca corte verificação.

**Releitura adversarial da entrega.** Leia o rascunho da entrega como um auditor cético leria: alguma frase afirma mais do que o dado mostrou? Algum "causa" onde a trilha diz correlação? Algum número redondo demais que pode ter sido arredondado na interpretação e não no dado? Corrija antes de entregar.

### Fase 9. Fechar respondendo à dúvida inicial

Antes de encerrar, duas verificações. Primeira: a pergunta central da fase 1 foi respondida? Cada sub-pergunta tem resposta, resposta parcial com limitação declarada, ou justificativa de por que os dados não permitem responder? Segunda: os dados acumulados na sessão revelam algo relevante ainda não explorado? Se sim, nova rodada a partir da fase 3, e a rodada nova reentra na fase 8 — conclusão criada depois do gate publicado exige recalcular `T` e reabrir a tabela. Só encerre quando duas rodadas seguidas dessa segunda verificação não produzirem candidato novo.

---

## Trilha de consultas

Artefato central da investigação, e o único que sobrevive à sessão. Arquivo `trilha-<AAAA-MM-DD>.md` no diretório de trabalho, append-only, um bloco por consulta, escrito no mesmo turno em que o retorno chega, antes de formular a próxima pergunta.

Não existe versão "na conversa". Investigação profunda passa de quarenta consultas, e o contexto vai ser compactado no meio do caminho: o resumo preserva as conclusões narrativas e descarta os blocos de SQL, que são volumosos e parecem redundantes. O que não estiver no arquivo deixa de existir pra fase 8 e pro anexo — e o que deixou de existir o modelo preenche por plausibilidade, produzindo query sintaticamente correta com coluna inexistente. É essa a origem de anexo de auditoria inventado, e nenhuma quantidade de "é obrigatório" resolve, porque não é desobediência, é amnésia.

```
### [#N] <Mapa | Padrão | Hipótese | Desafio | Correlação | Público | Oportunidade | Derivado>
Status: validado | refutado | divergente | pendente | estrutural | limitação
Confiança: alta | média | baixa
Pergunta:
SQL: [COLADO] | [SEM SQL — Agente de Dados não devolveu]
    <o SQL aqui>
Tabelas:
Período: <copiado literal do SQL, com a coluna de data que o aplica>
Unidade: <o que é uma linha do resultado>
N:
Retorno (literal, sem arredondar):
Leitura:
```

Os campos que costumam ser burlados:

- **`SQL:`** abre com um dos dois marcadores, e não existe terceiro. `[COLADO]` significa copiado do retorno neste turno; SQL que você digitou é SQL inventado. Bloco `[SEM SQL]` não sustenta conclusão na entrega: ou a consulta é refeita, ou a conclusão sai
- **`N:`** número pelado é bloco não resolvido. Escreva `8.412 envios`, `3.140 clientes distintos`, `41.900 eventos`. Em CRM, envio e usuário convivem na mesma tabela, e trocar um pelo outro erra o dimensionamento de teste por múltiplos — com a cadeia de auditoria intacta, preservando os dígitos e perdendo a entidade
- **`Retorno (literal)`** guarda os valores como vieram. Arredondar acontece uma vez só, no texto da entrega, com o original a um `#` de distância
- **`Leitura:`** é a única linha do bloco que é sua. Tudo acima dela veio do Agente de Dados

**Confira o SQL contra a pergunta antes de arquivar.** O SQL chega pra ser lido, não só guardado — em nenhum outro momento do fluxo ele volta a ser insumo de decisão. Confira três coisas: a unidade de uma linha do resultado, o recorte de período e a coluna de data que o aplica, e as colunas que definem o segmento. Se o recorte não corresponde ao que você pediu, ou se a coluna usada é proxy do conceito que você perguntou, o status é `divergente`, e divergente não alimenta fase 6, fase 7 nem o gate da fase 8: refaça a pergunta. Proxy aceito de propósito vira conclusão escrita com o nome da coluna, não com o nome do conceito.

Erro de definição é interno-consistente e atravessa todas as camadas: a fase 0 valida totais, não o recorte de cada query; a fase 3 pergunta robustez sobre o número, não sobre a query; o gate da fase 8 confirma por outro caminho executado pelo mesmo Agente de Dados, que repete a mesma definição. A leitura do SQL é o único ponto onde a divergência é visível.

**Número derivado.** Número que não apareceu literalmente em nenhum retorno — razão, diferença em pontos percentuais, múltiplo, regra de três — entra como bloco de tipo `Derivado`, com `calc: <a conta> a partir de #x, #y`, e abre linha obrigatória no gate da fase 8. Razão entre duas pontas só vale se as duas vierem da mesma janela e da mesma definição; se não vierem, delegue a consulta que devolve o número pronto. Sua aritmética não é dado de tabela: o modelo não lê a própria divisão como estimativa, lê como cálculo legítimo, e é aqui que o Questionador vira a fonte da alucinação que o resto do documento existe pra evitar.

Na montagem do anexo você **transcreve** blocos do arquivo. Você não redige SQL nessa etapa.

**Critério de confiança**, atribuído no fechamento de cada padrão ou hipótese, antes de seguir pra próxima fase:

- **Alta.** Amostra grande (acima de alguns milhares), consistente em dois ou mais períodos distintos, e no mínimo três blocos de tipo Desafio na trilha citando o `#` deste padrão, todos com status refutado — um deles obrigatoriamente a checagem de composição da fase 5. Menos de três, ou composição ausente, limita a confiança a média, independentemente do tamanho da amostra
- **Média.** Amostra suficiente mas testado num único período, ou sobreviveu à fase 5 com alguma alternativa não totalmente descartável por falta de dado
- **Baixa.** Amostra pequena mesmo após confirmação, ou não testado em mais de um período, ou fase 5 não esgotou as alternativas por limitação dos dados

Confiança baixa não impede o achado de entrar na entrega, mas impede virar proposta de oportunidade na fase 7 sem ressalva explícita. E na entrega final, ordene os achados por confiança, do mais alto pro mais baixo, pra quem for decidir saber onde apostar primeiro.

## Contrato da pergunta delegada

**Ciclo, sem exceção, inclusive na sequência exploratória da fase 2.** Delegue uma solicitação ao Agente de Dados, receba, escreva o bloco `#N` na trilha com o SQL colado, e só então formule a próxima. Delegar em lote força escrita retroativa, e escrita retroativa é escrita de memória.

Perguntas de conteúdo carregam sempre quatro elementos. Perguntas de mapa (fase 0) são sobre estrutura e não precisam deles. Prompt subespecificado para um segundo modelo é completado pelos priors dele, e ele nunca anuncia que completou:

- **Métrica**, com definição operacional. "Ativo" e "conversão" significam coisas diferentes em tabelas diferentes, e o mesmo conceito definido de dois jeitos em consultas diferentes produz números que deixam de ser comparáveis sem ninguém notar. Diga qual
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

**`customer_id` é hasheado** e circula normalmente. Pode aparecer no SQL da trilha, nos blocos e na definição de público, sem mascaramento e sem ser omitido do anexo de auditoria. Tratar id hasheado como PII só serviria pra tornar a auditoria impossível de refazer.

O que sobra é regra de forma, não de sigilo: entrega é insight, não lista. Dashboard e documento carregam números agregados e o critério que define o público. Quando o público estimulável precisar ser materializado pra ativar a campanha, entregue a consulta que o gera, não milhares de linhas coladas no HTML.

## Entrega final

Nenhuma entrega sai sem a fase 8 completa. Dois formatos de corpo, e um anexo obrigatório em ambos. Pergunte ao usuário qual formato ele quer se não estiver claro pelo contexto.

**O anexo é gerado antes do corpo.** Transcreva `trilha-<data>.md` primeiro, depois escreva a entrega em cima dele. O que é escrito primeiro sobrevive ao orçamento de saída, e inverter a ordem física de geração vale mais que qualquer instrução de não omitir.

**Documento de insights.** A dúvida inicial e sua resposta direta logo no início. Depois: os padrões validados com fundamentação, as correlações e candidatas a causa (com a distinção declarada), os públicos estimuláveis com suas propostas, o que foi refutado no caminho (hipótese descartada com dado evita retrabalho futuro), e as limitações dos dados encontradas na fase 0.

**HTML de visualização.** Arquivo único e autocontido (CSS e JS inline, sem CDN), legível em tema claro e escuro. Nesta ordem:

1. A dúvida inicial e a resposta direta, em texto, antes de qualquer gráfico
2. Os padrões validados, cada um com o gráfico que o explica e uma linha de leitura dizendo o que ali se enxerga
3. Grupo onde o comportamento ocorre e público estimulável, lado a lado
4. As propostas da fase 7 em cards
5. Anexo de auditoria como seção final recolhida (`<details>`), com a trilha e o SQL de cada consulta em blocos de código

Declare os números uma vez só, num bloco de dados no topo do script, cada entrada carregando o `#` da trilha de onde veio, e renderize tudo a partir dele. Número digitado direto no meio do markup aparece duas ou três vezes e diverge na terceira. Se a trilha inteira não couber no arquivo, entregue o anexo como arquivo separado e abra o corpo com a contagem: "anexo: N consultas, arquivo `trilha-<data>.md`".

**Gráficos.** Se a skill `dataviz` estiver instalada, siga-a: ela define paleta, tipografia e formas. Sobre isso, as regras desta entrega:

- Gráfico entra quando responde melhor que o texto sozinho responderia. Gráfico decorativo polui a leitura e dilui os que importam
- Variação ao longo do tempo é onde o gráfico ganha do texto com mais folga. Use série temporal e marque no próprio gráfico o evento que explica a inflexão: início da campanha, mudança de produto, lançamento da feature. Anotação no ponto vale mais que legenda embaixo
- Comparação entre segmentos: barras horizontais ordenadas por valor. Pizza só com duas ou três fatias e quando somar 100% for o ponto
- Antes e depois do mesmo grupo: barras pareadas ou slopegraph, sempre com a taxa de base visível
- Distribuição, que é onde as bordas da fase 3 aparecem: histograma ou faixas de percentil, nunca só a média
- Eixo de taxa começa em zero, salvo quando a variação relevante for pequena demais pra enxergar — e aí declare o corte no próprio gráfico
- Todo gráfico traz o N da amostra, com a unidade, e a janela de tempo visíveis, sem depender de hover
- Coorte incompleta não entra em série temporal, ou entra marcada como parcial. Sem isso o gráfico desenha uma queda que é só censura
- Sem 3D, sem gradiente decorativo, sem animação que atrase a leitura

Todo número exibido vem de bloco registrado na trilha, nunca de estimativa nem de conta do modelo. Todo gráfico aponta pro `#` que o alimenta, em nota abaixo dele, pra que qualquer ponto do dashboard possa ser rastreado até a consulta no anexo.

**Anexo de auditoria (obrigatório nos dois formatos).** É o que permite auditar a investigação depois, sem depender da memória de ninguém. Contém:

- Tabelas e fontes consultadas: nome de cada tabela usada, com a descrição obtida na fase 0, a granularidade e o período coberto
- A trilha inteira, transcrita, na ordem. Sem resumir SQL e sem omitir consulta que não deu em nada: consulta vazia também é informação de auditoria
- Mapa conclusão → fonte: cada conclusão da entrega apontando pro(s) `#` que a sustentam
- Double checks realizados: a linha de controle e a tabela do gate da fase 8, íntegras
- Limitações e lacunas: dimensões que não existiam nos dados, consultas que retornaram vazio, blocos `[SEM SQL]`, controle de base não confirmado pelo usuário, e qualquer conclusão entregue como parcial por causa disso

No HTML, o anexo entra como seção final expansível do mesmo arquivo, com cada SQL em bloco de código copiável. No documento, como seção final. Entrega sem anexo de auditoria é entrega incompleta.

## Fundamentos estatísticos obrigatórios

O Questionador aplica estes fundamentos em toda interpretação de retorno e em toda proposta de teste. Não são opcionais nem "quando der".

**Construção de hipótese.** Toda hipótese segue a forma: se [causa proposta], então [métrica específica] deve ser [direção e magnitude esperada] em [segmento] durante [janela]. Antes de delegar a pergunta confirmatória, declare o que refutaria a hipótese. Hipótese que nenhum resultado consegue refutar não é hipótese, é crença, e não entra na fase 4.

**Amostra e significância.** Ao interpretar qualquer diferença entre grupos, pergunte primeiro ao Agente de Dados o tamanho de cada grupo. Diferença percentual grande em amostra pequena vale menos que diferença modesta em amostra grande. Como regra prática: grupos com menos de algumas centenas de observações não sustentam conclusão sozinhos, exigem confirmação em outro período ou recorte. Nunca reporte uma diferença sem reportar junto o tamanho das amostras que a geraram.

**Taxa de base e denominadores.** Todo número relativo precisa do seu denominador declarado. "Dobrou a conversão" com base 0,1% é diferente de dobrar com base 15%. Compare sempre contra a taxa de base do grupo, não contra zero.

**Desenho de teste A/B.** Toda proposta da fase 7 nasce como experimento: grupo tratamento e grupo controle sorteados do mesmo público estimulável, expostos no mesmo período, diferindo apenas no estímulo. Antes de propor, estime com o Agente de Dados se o tamanho do público comporta detectar o efeito esperado na janela proposta; público pequeno demais pro efeito esperado é motivo pra ajustar o teste (efeito mínimo maior, janela maior, métrica mais sensível), não pra rodar assim mesmo e torcer. Defina a métrica primária única antes do teste; métricas secundárias são leitura de apoio, não critério de sucesso alternativo caso a primária falhe.

**Armadilhas de leitura de experimento.** Não conclua o teste antes da janela definida só porque o resultado parcial parece bom, resultado parcial oscila. Não fatie o resultado em dezenas de sub-segmentos procurando onde deu significativo, com cortes suficientes algum sempre dá por acaso; sub-segmento interessante vira hipótese pra próximo teste, não conclusão deste. Desconfie de efeito que aparece só num recorte e some no agregado.

**Regressão à média e sobrevivência.** O grupo controle é inegociável exatamente pelo motivo da fase 3: grupo selecionado por comportamento extremo volta pro normal sozinho, sem estímulo nenhum, e a campanha leva o crédito. E grupo definido por "quem ficou até o final" carrega viés de sobrevivência: compare cohorts completas desde a entrada, não apenas quem sobreviveu até a medição.

## Erros comuns

- Pular a fase 0 e descobrir tarde que a coluna necessária não existe
- Perder a dúvida inicial de vista, entregando descobertas laterais interessantes sem responder o que foi perguntado
- Registrar conhecimento de mercado como se fosse padrão observado nos dados
- Investigar só o que foi perguntado, sem as táticas de inversão, bordas e ausência da fase 3
- Deixar o double check pro fim e chegar no fim sem contexto pra fazê-lo. Verifique quando a conclusão fecha
- Delegar em lote e escrever a trilha depois. Trilha escrita depois é trilha escrita de memória, e SQL escrito de memória é SQL inventado
- Fazer a conta de cabeça e registrar o resultado como se fosse número de tabela. Razão, diferença e múltiplo são tipo `Derivado`, com a conta declarada
- Tratar retorno 0 ou vazio como comportamento inexistente sem a prova de vida
- Apresentar correlação como causa provada, em vez de declarar a distinção e propor o experimento que a resolveria
- Propor estímulo pra diferença estrutural, que nenhuma comunicação muda
- Aceitar "não tenho esse dado" como final sem reformular por outro ângulo, e sem registrar como limitação quando confirmado
- Preencher lacuna do HTML com número estimado. Lacuna se resolve com consulta ou se declara como limitação
- Entregar sem o anexo de auditoria, ou com anexo genérico que não mapeia cada conclusão à sua fonte
- Encher o dashboard de gráfico que não responde nada, ou entregar variação temporal em tabela quando uma série temporal anotada resolveria
- Ler resultado parcial de teste como definitivo, ou fatiar o resultado até achar um recorte significativo por acaso

## Skills complementares recomendadas

Se estiverem instaladas no ambiente, reforçam pontos específicos: `dataviz` na entrega em HTML (paleta e formas dos gráficos), `marketing-psychology` nas fases 3 e 6 (nomear o mecanismo comportamental por trás do padrão e do estímulo), e `churn-prevention` nas fases 6 e 7 quando a investigação for sobre inatividade e retenção.

Verifique se a skill existe antes de invocá-la. Skill ausente não é motivo pra parar a investigação, e nenhuma delas substitui as fases deste documento.

## Adaptação ao seu ambiente

Em todo este documento, "Agente de Dados" é um apelido genérico. Antes do primeiro uso, edite este parágrafo substituindo `[AGENTE_DE_DADOS]` pelo nome ou forma de invocação real do subagente Databricks no seu sistema — é essa instrução que o Questionador seguirá pra delegar:

> Invoque `[AGENTE_DE_DADOS]` passando uma solicitação por vez. Aguarde o retorno, escreva o bloco na trilha com o SQL colado, e só então formule a próxima.
