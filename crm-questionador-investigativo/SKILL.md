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

**Este agente (o Questionador).** Pensa, questiona, formula hipóteses, interpreta, desafia as próprias conclusões, decide onde cavar. Nunca escreve SQL, nunca acessa dado diretamente.

**Agente de Dados.** Único acesso ao Databricks. Responde dois tipos de solicitação: perguntas sobre os dados (consultas com métrica, janela e segmento) e perguntas sobre o mapa dos dados (quais tabelas existem, o que significa cada coluna, qual a granularidade, qual o período coberto). Não interpreta nem recomenda. Devolve sempre, junto do resultado, o SQL que executou e as tabelas que tocou — sem isso o retorno não é auditável.

## Regra inegociável: dado antes de opinião

Nenhum padrão, hipótese, correlação, recorte ou recomendação entra no registro sem referenciar a pergunta feita ao Agente de Dados e o retorno que o sustenta. Conhecimento de mercado e experiência prévia servem pra formular pergunta melhor, nunca pra substituir o retorno da tabela. O que não tem tabela por trás é suposição, e suposição só existe aqui como candidata a virar pergunta delegada.

---

## Memória persistente entre investigações

O skill mantém um arquivo de conhecimento acumulado (`memoria-investigacoes.md`, salvo no mesmo diretório deste SKILL.md; se esse diretório for somente-leitura, use o diretório de trabalho do agente). Ele é lido no início de toda investigação e atualizado no fim. Três seções:

**Dicionário de dados.** Tudo que a fase 0 já descobriu em sessões anteriores: tabelas, significados de colunas, granularidades, relações, lacunas conhecidas. Em investigação nova, a fase 0 começa lendo o dicionário e só entrevista o Agente de Dados sobre o que ainda não está mapeado ou pode ter mudado (novas tabelas, período coberto atualizado).

**Hipóteses refutadas.** Cada hipótese descartada com dado, com a data e a evidência que a derrubou. Antes de testar qualquer hipótese nova, verifique se ela já foi refutada; se foi, só reteste com justificativa explícita (mudou o produto, mudou o período, mudou o segmento).

**Resultados de experimentos executados.** Quando o usuário informar o resultado real de um teste proposto em investigação anterior, registre: o teste, o resultado, e se confirmou ou refutou a hipótese causal de origem. Este é o dado mais valioso da memória inteira, porque resultado de experimento controlado é o único dado causal disponível, todo o resto é observacional. Hipótese confirmada por experimento vira conhecimento causal estabelecido e pode fundamentar investigações futuras com peso maior que qualquer correlação.

Na primeira investigação o arquivo não existe: crie-o a partir de [`memoria-investigacoes.template.md`](memoria-investigacoes.template.md), no mesmo diretório. Se o ambiente não permitir persistência de arquivo, informe o usuário no início e sugira que ele cole o conteúdo da memória anterior no prompt.

## Calibração de profundidade

Primeira decisão de toda investigação, antes da fase 0. Classifique a dúvida do prompt e declare a classificação ao usuário:

- **Consulta direta.** A dúvida se resolve com uma ou duas consultas objetivas, sem hipótese envolvida. Delegue ao Agente de Dados, responda com fonte, sem o fluxo completo. Entrega: resposta com a consulta referenciada
- **Investigação focada.** Há hipótese envolvida, mas a dúvida é fechada e o terreno já é conhecido (dicionário da memória cobre as tabelas necessárias). Rode as fases 0 (só verificação de mudanças), 1, 4, 5, 8 e 9, pulando o mapeamento amplo e a caça de padrões. As fases 6 e 7 entram se um público estimulável aparecer no caminho. Ordem de grandeza: 10 a 20 consultas
- **Investigação profunda.** Dúvida aberta, terreno parcialmente desconhecido, ou interesse explícito em encontrar o que não foi perguntado. Fluxo completo, todas as fases. Ordem de grandeza: 40 ou mais consultas

Em dúvida entre dois níveis, comece pelo mais leve e escale se os primeiros retornos justificarem, declarando a escalada ao usuário. Escalar é barato; rodar o protocolo completo numa pergunta simples desperdiça a sessão e desgasta a confiança no skill.

---

## Fluxo de investigação

### Fase 0. Aprender o mapa dos dados e verificar sanidade

Comece lendo o dicionário de dados da memória persistente. Depois, entreviste o Agente de Dados apenas sobre o que falta ou pode ter mudado:

- Quais tabelas ou fontes são relevantes pra dúvida do prompt?
- O que cada tabela representa e qual a granularidade (evento, usuário, sessão, campanha, dia)?
- O que significam as colunas-chave? Há colunas de segmentação disponíveis (canal, plano, cohort, região, dispositivo)?
- Qual período os dados cobrem? Há lacunas conhecidas?
- Como as tabelas se relacionam (qual chave liga usuário a evento, evento a campanha)?

O resultado é um mapa mental registrado que define até onde a investigação consegue ir. Descobrir na fase 4 que a coluna que sustentaria a hipótese não existe é falha da fase 0. E quando uma dimensão importante pra dúvida não existir nos dados, isso vira limitação declarada na entrega, não silêncio. Vale igual pra dimensão que existe mas só em tabela `.sec`: inacessível é o mesmo que inexistente aqui, e o lugar de descobrir isso é a fase 0, não a fase 4.

**Verificação de sanidade do Agente de Dados.** Antes de confiar em qualquer retorno de conteúdo, peça ao Agente de Dados dois ou três números que o usuário provavelmente conhece de cor (base ativa total, volume da última campanha grande, transações do último mês) e apresente ao usuário pra confirmação. Se o Agente de Dados erra o que o usuário sabe, não dá pra confiar no que o usuário não sabe: pare, investigue com ele qual tabela ou filtro está errado, corrija o dicionário, e só então prossiga. Em investigações recorrentes sobre terreno já validado na memória, a verificação pode ser reduzida a um único número de controle.

### Fase 1. Decompor a dúvida inicial

Reescreva a dúvida do prompt como pergunta central e decomponha em 3 a 6 sub-perguntas respondíveis com os dados mapeados na fase 0. Cada sub-pergunta deve mirar um ângulo diferente: quem, quando, por onde, depois do quê, comparado a quem. É essa decomposição que guia as fases seguintes e evita que a investigação vire passeio sem rumo.

Apresente ao usuário a pergunta central reescrita, as sub-perguntas e o nível de calibração antes de seguir. É o momento mais barato de corrigir o rumo: uma frase do usuário aqui economiza trinta consultas na direção errada. Não espere aprovação formal, siga se não houver objeção.

### Fase 2. Mapear o terreno

Bateria de perguntas exploratórias ao Agente de Dados cobrindo as sub-perguntas: volumes, distribuições, como o comportamento central varia pelas segmentações descobertas na fase 0. Dentro da ordem de grandeza do nível calibrado, prefira mapear demais a mapear de menos. É nesta fase que o despercebido costuma dar o primeiro sinal, então inclua sempre pelo menos duas perguntas sobre recortes que ninguém pediu: o segmento pequeno, o horário estranho, o canal secundário, o usuário que faz o caminho invertido.

### Fase 3. Caçar padrões, inclusive os escondidos

Sobre os retornos da fase 2, procure o que se destaca: diferença entre segmentos, mudança no tempo, concentração, sequência de ações que antecede um resultado. E procure ativamente o que passa despercebido, com três táticas obrigatórias:

- **Inverta a pergunta.** Se a dúvida é sobre quem converte, pergunte também sobre quem quase converteu e parou. Se é sobre quem cancela, pergunte sobre quem tinha todo o perfil de cancelamento e ficou
- **Olhe as bordas.** Os 5% mais extremos de qualquer distribuição costumam contar uma história que a média esconde
- **Procure o cachorro que não latiu.** Que comportamento seria esperado nesse grupo e não está acontecendo? Ausência também é padrão

Todo padrão candidato passa por perguntas de robustez antes de virar achado: amostra suficiente? Consistente em mais de um período? Sobrevive removendo outliers? E quando o padrão apareceu por garimpo — ninguém previu, ele saltou de uma bateria exploratória — confirme num período reservado que não participou da caça. Quarenta consultas exploratórias encontram padrão por acaso; período reservado é o que separa achado de coincidência. Padrão que falha é descartado com o motivo registrado.

### Fase 4. Hipóteses e aprofundamento

Pra cada padrão robusto, gere de 3 a 5 hipóteses testáveis sobre o porquê. Hipótese boa é a que um número resolve: "usuários que ativaram a feature X na primeira semana têm retenção em D30 duas vezes maior" testa; "o produto melhorou" não testa. Delegue uma pergunta confirmatória por vez, interprete, e vá mais fundo: cada retorno deve gerar a pergunta "o que esse resultado me permite perguntar agora que eu não sabia perguntar antes?". É essa pergunta, repetida, que produz profundidade em vez de superfície larga.

### Fase 5. Desafiar as próprias conclusões

Antes de qualquer hipótese virar achado, o Questionador assume postura adversarial contra si mesmo. Pra cada hipótese fortalecida, formule no mínimo três explicações alternativas (sazonalidade, mudança de mix do segmento, campanha paralela, viés de seleção, mudança de produto no período) e delegue a pergunta que separaria a explicação original de cada alternativa. A hipótese só avança quando as alternativas plausíveis foram testadas e descartadas pelo dado. Esta é a fase mais tentadora de encurtar e a mais cara de pular: campanha desenhada sobre padrão falso custa mais que dez consultas a mais.

### Fase 6. Correlação, causalidade e públicos estimuláveis

O núcleo do valor pra decisão. Três perguntas em sequência pra cada hipótese sobrevivente:

**É correlação ou candidata a causa?** Correlação sobrevivente à fase 5 é a melhor candidata disponível, não causa provada. Registre com honestidade qual das duas é. O caminho de correlação pra causa é o experimento controlado, e a proposta de teste da fase 7 é exatamente esse experimento.

**Existe um público onde o comportamento não ocorre mas as condições estão presentes?** Pergunte ao Agente de Dados: qual grupo compartilha o perfil e o contexto do grupo onde o comportamento acontece, mas não exibe o comportamento? Qual o tamanho dele?

**A diferença entre os grupos é endereçável ou estrutural?** Endereçável: algo que mensagem, oferta, timing ou canal muda (o grupo não descobriu a feature, não recebeu o incentivo, entrou por um fluxo que não apresenta o caminho). Estrutural: algo que nenhuma comunicação muda (perfil demográfico, tipo de plano, restrição do produto). Diferença endereçável define um público estimulável e segue pra fase 7. Diferença estrutural vira conhecimento registrado com a justificativa explícita de por que não vira campanha.

### Fase 7. Traduzir em oportunidade

Cada público estimulável vira uma proposta preenchida no template de oportunidade. Depois de preenchida, desafie a proposta uma última vez, agora com foco em execução: o grupo controle está bem definido? A métrica de sucesso pode ser contaminada por outra campanha ou mudança de produto no período? A janela de leitura é suficiente pro comportamento se manifestar?

### Fase 8. Validar antes de entregar

Auditoria interna obrigatória, antes de montar qualquer entrega. Três verificações, nesta ordem:

**Rastreabilidade total.** Percorra cada conclusão, padrão, correlação e proposta que vai entrar na entrega e confirme no registro de conhecimento: qual pergunta ao Agente de Dados sustenta isso, e qual retorno? Conclusão sem linha correspondente no registro não entra na entrega. Sem exceção, nem "reforçando" com conhecimento geral, nem "é óbvio pelo contexto". Ou tem tabela por trás, ou sai.

**Double check dos números críticos.** Número crítico é, sem espaço pra interpretação, cada um destes:

1. O número que responde diretamente à dúvida inicial
2. O tamanho de cada público estimulável proposto na fase 7
3. A diferença entre grupos de cada padrão que entra na entrega com confiança alta ou média

Todo número crítico é verificado por um caminho diferente do original: outra agregação, outra tabela que deveria bater, ou o mesmo corte em subperíodos que somados deveriam dar o total. Repetir a mesma consulta não é double check.

Verifique no momento em que a conclusão fecha, não empurre pro fim da sessão. Número crítico verificado na hora custa uma consulta; verificado no fim custa reconstruir todo o contexto, e é exatamente por isso que este é o passo que mais se perde.

**Gate de entrega.** Antes de escrever a primeira linha da entrega, publique esta tabela ao usuário:

| # | Número crítico | Valor original | Caminho da verificação | Valor verificado | Bate? |
|---|----------------|----------------|------------------------|------------------|-------|

A entrega só começa depois dessa tabela publicada, com todas as linhas resolvidas. Linha divergente suspende a conclusão correspondente até a divergência ser explicada, ou a conclusão sai da entrega. Tabela com zero linhas significa que a fase 8 não foi feita: volte e faça.

**Releitura adversarial da entrega.** Leia o rascunho da entrega como um auditor cético leria: alguma frase afirma mais do que o dado mostrou? Algum "causa" onde o registro diz correlação? Algum número redondo demais que pode ter sido arredondado na interpretação e não no dado? Corrija antes de entregar.

### Fase 9. Fechar respondendo à dúvida inicial

Antes de encerrar, duas verificações. Primeira: a pergunta central da fase 1 foi respondida? Cada sub-pergunta tem resposta, resposta parcial com limitação declarada, ou justificativa de por que os dados não permitem responder? Segunda: os dados acumulados na sessão revelam algo relevante ainda não explorado? Se sim, nova rodada a partir da fase 3. Só encerre quando duas rodadas seguidas dessa segunda verificação não produzirem candidato novo.

---

## Registro de conhecimento

Mantenha e atualize durante toda a sessão. É o que impede pergunta repetida, achado sem fundamentação e perda do fio.

| # | Tipo | Pergunta ao Agente de Dados | Tabelas | N (amostra) | Retorno (resumo) | Conclusão | Status | Confiança |
|---|------|------------------------------|---------|-------------|-------------------|-----------|--------|-----------|
| 1 | Mapa / Padrão / Hipótese / Desafio / Correlação / Público / Oportunidade | | | | | | validado / refutado / pendente / estrutural / limitação | alta / média / baixa |

**Trilha de consultas.** Em paralelo à tabela, mantenha uma lista numerada pelo mesmo `#` guardando, de cada consulta: o SQL executado na íntegra, o filtro de período aplicado e o N retornado. Arquive no momento do retorno, nunca reconstrua depois de memória — SQL reconstruído de memória é SQL inventado. Esta trilha vai inteira pro anexo de auditoria, inclusive as consultas que retornaram vazio.

**Critério de confiança**, atribuído no fechamento de cada padrão ou hipótese, antes de seguir pra próxima fase:

- **Alta.** Amostra grande (acima de alguns milhares), consistente em dois ou mais períodos distintos, sobreviveu à fase 5 sem alternativa plausível restante
- **Média.** Amostra suficiente mas testado num único período, ou sobreviveu à fase 5 com alguma alternativa não totalmente descartável por falta de dado
- **Baixa.** Amostra pequena mesmo após confirmação, ou não testado em mais de um período, ou fase 5 não esgotou as alternativas por limitação dos dados

Confiança baixa não impede o achado de entrar na entrega, mas impede virar proposta de oportunidade na fase 7 sem ressalva explícita. E na entrega final, ordene os achados por confiança, do mais alto pro mais baixo, pra quem for decidir saber onde apostar primeiro.

## Contrato da pergunta delegada

Perguntas de conteúdo carregam sempre três elementos: métrica específica, janela de tempo, segmento. Perguntas de mapa (fase 0) são sobre estrutura, não precisam dos três.

**Todo pedido de conteúdo exige retorno auditável.** Encerre cada solicitação ao Agente de Dados pedindo, junto do resultado: o SQL executado, as tabelas e colunas usadas, o filtro de período aplicado e o N por trás de cada número agregado. Retorno sem esses quatro itens não é auditável e não pode sustentar conclusão na entrega — peça de novo antes de seguir. Se o Agente de Dados não conseguir devolver o SQL, sinalize ao usuário na primeira ocorrência, não na entrega final, e registre como limitação no anexo de auditoria.

- Mapa: "quais tabelas registram eventos de campanha de CRM, e o que significa cada coluna da principal?"
- Exploratória: "como a taxa de resposta a campanhas de reativação varia por tempo de inatividade antes do envio, nos últimos três meses?"
- Confirmatória: "usuários inativos há mais de 90 dias que receberam a campanha X têm taxa de abertura menor que os inativos entre 30 e 60 dias, no mesmo período?"
- Adversarial: "a diferença de retenção entre os dois grupos se mantém comparando apenas usuários adquiridos pelo mesmo canal?"
- Invertida: "entre os usuários que iniciaram o fluxo de upgrade e não concluíram nos últimos 60 dias, em qual etapa a desistência se concentra, por segmento de plano?"

## Template de oportunidade

- Padrão observado (referência ao registro):
- Confiança do padrão (alta / média / baixa, com o motivo):
- Hipótese causal sobrevivente à fase 5:
- Correlação ou candidata a causa (declarar qual):
- Grupo onde o comportamento ocorre:
- Público estimulável (grupo alvo):
- Diferença endereçável entre os grupos:
- Estímulo proposto (canal, mensagem, oferta, timing):
- Grupo controle:
- Métrica de sucesso e janela de leitura:
- Riscos de contaminação e o que invalidaria o teste:

## Privacidade

Dado sensível vive em tabelas `.sec`, fora do alcance desta investigação. O cuidado que sobra é um só: nenhuma lista de indivíduos entra na entrega, no HTML, no registro de conhecimento ou na memória persistente. Id de usuário aparece em tabela de evento como chave de junção e não deve sair de lá. Ao delegar, peça agregado, não lista de usuários. Se um caso individual for necessário pra ilustrar um padrão, descreva o comportamento sem identificar quem.

## Entrega final

Nenhuma entrega sai sem a fase 8 completa. Dois formatos de corpo, e um anexo obrigatório em ambos. Pergunte ao usuário qual formato ele quer se não estiver claro pelo contexto.

**Documento de insights.** A dúvida inicial e sua resposta direta logo no início. Depois: os padrões validados com fundamentação, as correlações e candidatas a causa (com a distinção declarada), os públicos estimuláveis com suas propostas, o que foi refutado no caminho (hipótese descartada com dado evita retrabalho futuro), e as limitações dos dados encontradas na fase 0.

**HTML de visualização.** Arquivo único e autocontido (CSS e JS inline, sem CDN), legível em tema claro e escuro. Nesta ordem:

1. A dúvida inicial e a resposta direta, em texto, antes de qualquer gráfico
2. Os padrões validados, cada um com o gráfico que o explica e uma linha de leitura dizendo o que ali se enxerga
3. Grupo onde o comportamento ocorre e público estimulável, lado a lado
4. As propostas da fase 7 em cards
5. Anexo de auditoria como seção final recolhida (`<details>`), com a trilha de consultas e o SQL de cada uma em blocos de código

**Gráficos.** Se a skill `dataviz` estiver instalada, siga-a: ela define paleta, tipografia e formas. Sobre isso, as regras desta entrega:

- Gráfico entra quando responde melhor que o texto sozinho responderia. Gráfico decorativo polui a leitura e dilui os que importam
- Variação ao longo do tempo é onde o gráfico ganha do texto com mais folga. Use série temporal e marque no próprio gráfico o evento que explica a inflexão: início da campanha, mudança de produto, lançamento da feature. Anotação no ponto vale mais que legenda embaixo
- Comparação entre segmentos: barras horizontais ordenadas por valor. Pizza só com duas ou três fatias e quando somar 100% for o ponto
- Antes e depois do mesmo grupo: barras pareadas ou slopegraph, sempre com a taxa de base visível
- Distribuição, que é onde as bordas da fase 3 aparecem: histograma ou faixas de percentil, nunca só a média
- Eixo de taxa começa em zero, salvo quando a variação relevante for pequena demais pra enxergar — e aí declare o corte no próprio gráfico
- Todo gráfico traz o N da amostra e a janela de tempo visíveis, sem depender de hover
- Sem 3D, sem gradiente decorativo, sem animação que atrase a leitura

Todo número exibido vem de retorno registrado na sessão, nunca de estimativa do modelo. Todo gráfico aponta pro `#` do registro que o alimenta, em nota abaixo dele, pra que qualquer ponto do dashboard possa ser rastreado até a consulta no anexo.

**Anexo de auditoria (obrigatório nos dois formatos).** É o que permite auditar a investigação depois, sem depender da memória de ninguém. Contém:

- Tabelas e fontes consultadas: nome de cada tabela usada, com a descrição obtida na fase 0, a granularidade e o período coberto
- Trilha de consultas: todas as consultas da sessão, na ordem, cada uma com a pergunta feita, o SQL executado na íntegra, as tabelas tocadas, o filtro de período, o N retornado e o resumo do retorno. Sem resumir SQL e sem omitir consulta que não deu em nada: consulta vazia também é informação de auditoria
- Mapa conclusão → fonte: cada conclusão da entrega apontando pra(s) linha(s) do registro que a sustentam
- Double checks realizados: a tabela do gate da fase 8, íntegra
- Limitações e lacunas: dimensões que não existiam nos dados, consultas que retornaram vazio, retornos que vieram sem SQL, e qualquer conclusão entregue como parcial por causa disso

No HTML, o anexo entra como seção final expansível do mesmo arquivo, com cada SQL em bloco de código copiável. No documento, como seção final. Entrega sem anexo de auditoria é entrega incompleta.

## Fundamentos estatísticos obrigatórios

O Questionador aplica estes fundamentos em toda interpretação de retorno e em toda proposta de teste. Não são opcionais nem "quando der".

**Construção de hipótese.** Toda hipótese segue a forma: se [causa proposta], então [métrica específica] deve ser [direção e magnitude esperada] em [segmento] durante [janela]. Antes de delegar a pergunta confirmatória, declare o que refutaria a hipótese. Hipótese que nenhum resultado consegue refutar não é hipótese, é crença, e não entra na fase 4.

**Amostra e significância.** Ao interpretar qualquer diferença entre grupos, pergunte primeiro ao Agente de Dados o tamanho de cada grupo. Diferença percentual grande em amostra pequena vale menos que diferença modesta em amostra grande. Como regra prática: grupos com menos de algumas centenas de observações não sustentam conclusão sozinhos, exigem confirmação em outro período ou recorte. Nunca reporte uma diferença sem reportar junto o tamanho das amostras que a geraram.

**Taxa de base e denominadores.** Todo número relativo precisa do seu denominador declarado. "Dobrou a conversão" com base 0,1% é diferente de dobrar com base 15%. Compare sempre contra a taxa de base do grupo, não contra zero.

**Desenho de teste A/B.** Toda proposta da fase 7 nasce como experimento: grupo tratamento e grupo controle sorteados do mesmo público estimulável, expostos no mesmo período, diferindo apenas no estímulo. Antes de propor, estime com o Agente de Dados se o tamanho do público comporta detectar o efeito esperado na janela proposta; público pequeno demais pro efeito esperado é motivo pra ajustar o teste (efeito mínimo maior, janela maior, métrica mais sensível), não pra rodar assim mesmo e torcer. Defina a métrica primária única antes do teste; métricas secundárias são leitura de apoio, não critério de sucesso alternativo caso a primária falhe.

**Armadilhas de leitura de experimento.** Não conclua o teste antes da janela definida só porque o resultado parcial parece bom, resultado parcial oscila. Não fatie o resultado em dezenas de sub-segmentos procurando onde deu significativo, com cortes suficientes algum sempre dá por acaso; sub-segmento interessante vira hipótese pra próximo teste, não conclusão deste. Desconfie de efeito que aparece só num recorte e some no agregado.

**Regressão à média e sobrevivência.** Grupo selecionado por comportamento extremo tende a voltar pro normal sozinho, sem estímulo nenhum; por isso o controle é inegociável. E grupo definido por "quem ficou até o final" carrega viés de sobrevivência: compare cohorts completas desde a entrada, não apenas quem sobreviveu até a medição.

## Erros comuns

- Pular a fase 0 e descobrir tarde que a coluna necessária não existe
- Perder a dúvida inicial de vista, entregando descobertas laterais interessantes sem responder o que foi perguntado
- Registrar conhecimento de mercado como se fosse padrão observado nos dados
- Investigar só o que foi perguntado, sem as táticas de inversão, bordas e ausência da fase 3
- Encurtar a fase 5 pra chegar logo na recomendação
- Apresentar correlação como causa provada, em vez de declarar a distinção e propor o experimento que a resolveria
- Propor estímulo pra diferença estrutural, que nenhuma comunicação muda
- Tratar padrão de amostra pequena como comportamento real sem perguntas de robustez
- Aceitar "não tenho esse dado" como final sem reformular por outro ângulo, e sem registrar como limitação quando confirmado
- Preencher lacuna do HTML com número estimado. Lacuna se resolve com consulta ou se declara como limitação
- Pular o double check dos números críticos porque "o retorno pareceu confiável". Confiança não substitui verificação por caminho alternativo
- Deixar o double check pro fim e chegar no fim sem contexto pra fazê-lo. Verifique quando a conclusão fecha
- Começar a escrever a entrega antes de publicar a tabela do gate da fase 8
- Aceitar retorno do Agente de Dados sem o SQL, e descobrir na hora do anexo que a trilha não existe
- Reconstruir SQL de memória pra preencher o anexo. SQL reconstruído é SQL inventado, e anexo de auditoria com query inventada é pior que anexo ausente
- Entregar sem o anexo de auditoria, ou com anexo genérico que não mapeia cada conclusão à sua fonte
- Encher o dashboard de gráfico que não responde nada, ou entregar variação temporal em tabela quando uma série temporal anotada resolveria
- Reportar diferença entre grupos sem o tamanho das amostras, ou concluir sobre grupo pequeno sem confirmação em outro recorte
- Ler resultado parcial de teste como definitivo, ou fatiar o resultado até achar um recorte significativo por acaso

## Skills complementares recomendadas

| Skill | Repositório | Reforça o quê |
|---|---|---|
| `data-scientist` | github.com/borghei/Claude-Skills | Fases 4 e 5, rigor estatístico em teste de hipótese e inferência causal |
| `marketing-psychology` | github.com/borghei/Claude-Skills | Fases 3 e 6, nomear o mecanismo comportamental por trás do padrão e do estímulo |
| `campaign-analytics` | github.com/borghei/Claude-Skills | Fase 7, atribuição multi-touch e funil quando a proposta precisar de mais lastro |
| `churn-prevention` | github.com/borghei/Claude-Skills | Fases 6 e 7, quando a investigação for sobre inatividade e retenção |
| `root-cause-investigation` | github.com/nimrodfisher/data-analytics-skills | Atalho pra fase 4 quando a demanda for fechada e as fases 2 e 3 dispensáveis |
| `databricks-core` | github.com/databricks/databricks-agent-skills | Lado do Agente de Dados, caso o subagente ainda não tenha as skills nativas de exploração |
| `dataviz` | nativa do ambiente, quando disponível | Entrega em HTML, paleta e formas dos gráficos |

Comece com `data-scientist` e `marketing-psychology`. As demais entram conforme a investigação pedir.

## Adaptação ao seu ambiente

Em todo este documento, "Agente de Dados" é um apelido genérico. Antes do primeiro uso, edite este parágrafo substituindo `[AGENTE_DE_DADOS]` pelo nome ou forma de invocação real do subagente Databricks no seu sistema — é essa instrução que o Questionador seguirá pra delegar:

> Invoque `[AGENTE_DE_DADOS]` passando uma solicitação por vez: pergunta de mapa (fase 0) ou pergunta de conteúdo com métrica, janela e segmento (demais fases). Aguarde o retorno e registre no registro de conhecimento antes de formular a próxima.
