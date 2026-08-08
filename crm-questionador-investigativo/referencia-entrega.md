# Referência de entrega

Lida na fase 8, antes de montar qualquer entrega. Define a estrutura fixa, a escolha de gráfico, o versionamento e como explicar métricas, siglas e queries. É daqui que vem a consistência entre entregas: mesma dúvida investigada duas vezes deve produzir dois arquivos com o mesmo esqueleto.

Para o HTML, copie [`dashboard-template.html`](dashboard-template.html) e preencha. Não comece de uma página em branco — o template já carrega a estrutura, o tema claro/escuro, o cabeçalho de versão, o selo de verificação e os renderizadores de gráfico.

---

## 1. Cabeçalho e versionamento

Todo arquivo abre com quatro informações, na mesma ordem, sempre:

```
<título curto da investigação>
Dúvida investigada: <a pergunta da primeira mensagem, literal>
Gerado em: 2026-08-08 14:32  ·  v3  ·  47 consultas  ·  6/6 verificados
```

**A data e a hora vêm do sistema, nunca de memória.** Obtenha antes de escrever:

```bash
date '+%Y-%m-%d %H:%M'
```

```powershell
Get-Date -Format "yyyy-MM-dd HH:mm"
```

**Número de versão.** `v1` na primeira geração. Se você regerar a mesma investigação — porque a fase 9 abriu rodada nova, porque o usuário pediu ajuste, ou porque um double check reprovou e a conclusão mudou — incremente. Grave o número no rodapé do arquivo anterior antes de sobrescrever, ou salve como `dashboard-<data>-v<n>.html`. Duas versões do mesmo dashboard sem data e hora visíveis são indistinguíveis, e é exatamente aí que alguém decide sobre o número velho.

**O que muda de versão para versão** entra numa linha só, abaixo do cabeçalho, quando houver versão anterior: `v3: público estimulável recalculado após double check (era 41.200, é 33.800)`.

---

## 2. Estrutura fixa

A mesma ordem em todas as entregas. Seção sem conteúdo é omitida, nunca reordenada.

| # | Seção | Conteúdo |
|---|-------|----------|
| 1 | Cabeçalho | Título, dúvida, timestamp, versão, contagem de consultas, selo de verificação |
| 2 | Resposta direta | A resposta à dúvida inicial em até três frases, com o número principal e seu `#`. Antes de qualquer gráfico |
| 3 | Como cheguei aqui | Três a cinco linhas sobre o caminho: quantas consultas, quais ângulos, o que foi descartado. É o que dá confiança de que houve investigação, não uma query |
| 4 | Achados | Um bloco por padrão validado, ordenados por confiança. Cada um: afirmação, gráfico que a explica, linha de leitura, N e janela, selo de confiança, `#` |
| 5 | Funil | Quando a dúvida envolver um percurso com etapas. Ver seção 4 abaixo |
| 6 | Comparação de grupos | Grupo onde o comportamento ocorre × público estimulável, lado a lado, mesmas métricas nas mesmas posições |
| 7 | Oportunidades | Cards, um por público estimulável, com o template de oportunidade preenchido |
| 8 | O que foi refutado | Hipóteses descartadas com o dado que as derrubou. Evita retrabalho na próxima sessão |
| 9 | Limitações | Dimensões que não existiam, consultas vazias, blocos `[SEM SQL]`, controle de base não confirmado, conclusões parciais |
| 10 | Glossário | Métricas e siglas. Ver seção 5 |
| 11 | Anexo de auditoria | Recolhido. Trilha completa com SQL e explicação. Ver seção 6 |

No documento de texto, as mesmas onze seções na mesma ordem.

---

## 3. Gráficos: qual usar para qual pergunta

Duas regras que vêm antes de todas:

**Gráfico entra quando responde melhor que o texto sozinho responderia.** Gráfico decorativo dilui os que importam. Uma entrega com quatro gráficos que explicam vale mais que uma com doze.

**Barras é a forma para comparar segmentos, não o padrão para tudo.** Cair em barras por falta de escolha é o erro mais comum e o mais invisível — o gráfico fica legível e responde a pergunta errada. A pergunta define a forma; a tabela abaixo é o mapeamento, e a coluna `tipo` é literalmente o campo que você preenche em `DADOS.achados[].grafico.tipo` no template.

| A pergunta é sobre | `tipo` | Não use |
|---|---|---|
| Variação ao longo do tempo | `serie` — linha, com o evento anotado no ponto de inflexão | Barras por mês, tabela de meses |
| A mesma métrica em vários estratos ao longo do tempo — **a checagem de composição da fase 5** | `multiserie` — uma linha por estrato, o agregado tracejado | Um gráfico por estrato, lado a lado |
| Comparação entre segmentos | `barras` — horizontais, ordenadas por valor | Pizza, barras verticais com rótulo girado |
| Percurso com etapas e perda entre elas | `funil` — com taxa de passagem em cada degrau | Barras soltas por etapa |
| Antes e depois do mesmo grupo, poucos grupos | `pareadas` — com a diferença em pp declarada | Duas séries temporais separadas |
| Antes e depois, muitos grupos, e quem cruzou importa | `slopegraph` | Barras agrupadas |
| Distribuição, bordas, concentração — **as bordas da fase 3** | `histograma` — com as faixas extremas destacadas | Só a média |
| Composição que muda no tempo | `area100` — no máximo cinco faixas | Pizza por período |
| Duas métricas por segmento | `scatter` — com os dois eixos rotulados | Barras duplas |
| Dois ou três valores que somam o todo | `pizza` — só aqui ela serve | Pizza com quatro fatias ou mais |

O template implementa os dez. Se a pergunta não couber em nenhum, o gráfico provavelmente não é a forma certa — descreva em texto ou tabela e diga por quê.

**Dois tipos existem porque duas fases do fluxo produzem exatamente esse dado:** `multiserie` é o gráfico da checagem de composição (o agregado sobe, os estratos descem — é isso que a linha tracejada mostra), e `histograma` é o gráfico das bordas. Se você rodou a fase 5 e a fase 3, esses dois achados já existem; não os entregue em texto.

**Série temporal é onde o gráfico ganha do texto com mais folga.** Duas exigências:

- Anote no próprio gráfico o evento que explica a inflexão: início da campanha, mudança de produto, lançamento da feature, parada da aquisição paga. Anotação no ponto vale mais que legenda embaixo, e é o que transforma o gráfico de descrição em explicação
- Coorte incompleta não entra, ou entra marcada como parcial. Sem isso o gráfico desenha uma queda que é só censura de janela — a métrica não fechou ainda

**Regras que valem para todos:**

- Eixo de taxa começa em zero, salvo quando a variação relevante for pequena demais pra enxergar — e aí declare o corte no próprio gráfico
- Todo gráfico traz o N da amostra **com a unidade** (`3.140 clientes distintos`, não `3.140`) e a janela de tempo, visíveis, sem depender de hover
- Todo gráfico traz o `#` do bloco da trilha que o alimenta, em nota abaixo
- Toda taxa traz o denominador declarado
- Máximo de sete categorias num gráfico de barras; o resto agrupa em "outros", com o número de itens agrupados
- Sem 3D, sem gradiente decorativo, sem animação que atrase a leitura
- Se houver skill de visualização de dados instalada no ambiente, siga-a para paleta e tipografia; estas regras são sobre qual forma escolher, não sobre cor

**Declare os números uma vez só.** No HTML, todos os dados vivem num bloco `DADOS` no topo do script, cada entrada carregando o `#` de origem, e os gráficos renderizam a partir dele. Número digitado direto no meio do markup aparece duas ou três vezes e diverge na terceira.

**Confiança usa chave sem acento.** No campo `confianca` escreva `alta`, `media` ou `baixa` — sem acento. É essa chave que ordena os achados e que vira classe CSS; escrever `média` quebra a ordenação e a cor do selo. O acento aparece só no rótulo renderizado.

---

## 4. Funil

Use funil quando a dúvida envolver um percurso: onboarding, checkout, fluxo de upgrade, jornada de campanha (enviado → entregue → aberto → clicado → convertido).

O que um funil precisa mostrar:

- **Valor absoluto de cada etapa**, com a unidade — usuários distintos, não eventos, salvo quando a etapa é intrinsecamente por evento
- **Taxa de passagem entre etapas consecutivas** (etapa N ÷ etapa N−1), não só a taxa contra o topo. A perda mora entre dois degraus, e é a taxa de passagem que a localiza
- **Taxa acumulada contra o topo**, como segunda leitura
- **A etapa onde a maior perda acontece**, destacada visualmente. É a resposta que o funil existe pra dar

Três erros que invalidam um funil:

1. **Etapas de granularidades diferentes.** `envios` no topo e `usuários que converteram` na base produz uma taxa que não significa nada. Todas as etapas na mesma unidade, ou declare explicitamente a troca e por quê
2. **Etapas que não são sequenciais de verdade.** Se um usuário pode chegar à etapa 3 sem passar pela 2, não é funil — é distribuição de estados, e o gráfico certo é barras
3. **Coorte aberta.** Se o topo do funil inclui gente que entrou ontem, a base do funil ainda não teve tempo de acontecer. Feche a coorte: só quem entrou com tempo suficiente pra percorrer o percurso inteiro

Cada número do funil tem seu `#`, e a taxa de passagem, se você não a recebeu pronta, é `Derivado` — com a conta declarada e linha no gate.

---

## 5. Glossário de métricas e siglas

Seção obrigatória. Vem do glossário da memória persistente e cresce a cada sessão.

Toda métrica que aparece na entrega tem entrada, com **definição operacional** — não descrição:

| Termo | Definição operacional | Fonte |
|-------|----------------------|-------|
| Taxa de resposta | Respondentes distintos ÷ destinatários distintos, por campanha, contando um usuário uma vez | `#12` |
| Usuário ativo | Cliente com ao menos um evento de sessão nos últimos 30 dias, contra a data máxima da tabela | `#4` |
| D30 | Retenção em 30 dias: percentual da coorte de entrada com ao menos um evento entre o dia 23 e o dia 37 após a entrada | `#31` |
| Inatividade | Dias entre a data do evento e a data do último evento anterior do mesmo cliente — medida na data do envio, nunca contra hoje | `#12` |

**Toda sigla é expandida na primeira aparição do corpo e tem entrada no glossário.** CRM, D30, LTV, CAC, MQL, ARPU, churn: o que é óbvio pra quem investigou não é óbvio pra quem vai decidir três semanas depois. Se a sigla é interna da empresa, com mais razão.

Se uma métrica foi calculada com um proxy — a coluna disponível não é exatamente o conceito — a entrada do glossário diz isso, com o nome real da coluna. É a diferença entre "taxa de resposta" e "taxa de resposta, aproximada por eventos de clique porque não há tabela de resposta".

---

## 6. Anexo de auditoria

Recolhido no HTML (`<details>`), seção final no documento. Cinco blocos:

**a) Selo e tabela de verificação.** A tabela completa do gate da fase 8, íntegra, com a linha de controle da trilha acima dela.

**b) Trilha de consultas.** Todas as consultas da sessão, na ordem, cada uma com:

- O `#` e o tipo do bloco
- **O que esta consulta foi buscar**, em uma frase — não o SQL parafraseado, a intenção. `#23: verificar se a diferença de abertura entre os dois grupos sobrevive comparando só quem entrou pelo mesmo canal`
- O SQL executado, na íntegra, em bloco de código copiável
- Tabelas tocadas, filtro de período com a coluna de data, unidade da linha, N retornado
- O retorno resumido e a leitura

Sem resumir SQL e sem omitir consulta que não deu em nada: consulta vazia também é informação de auditoria, e as três tentativas que viraram limitação são a prova de que houve persistência.

**c) Tabelas e fontes.** Nome de cada tabela usada, com a descrição obtida na fase 0, a granularidade e o período coberto.

**d) Mapa conclusão → fonte.** Cada conclusão da entrega apontando pro(s) `#` que a sustentam. É o que permite auditar de cima pra baixo, não só de baixo pra cima.

**e) Limitações e lacunas.** Dimensões que não existiam nos dados, consultas vazias, blocos `[SEM SQL]`, controle de base não confirmado pelo usuário, e qualquer conclusão entregue como parcial por causa disso.

Se a trilha inteira não couber no arquivo, entregue o anexo como arquivo separado e abra o corpo com a contagem: `anexo: 47 consultas, arquivo trilha-2026-08-08.md`.

---

## 7. Checklist antes de fechar o arquivo

- [ ] Data e hora obtidas do sistema, não escritas de memória
- [ ] Número de versão presente, e a linha de mudança se houver versão anterior
- [ ] Resposta direta à dúvida inicial no topo, antes de qualquer gráfico
- [ ] Selo de verificação visível no cabeçalho, com a tabela do gate no anexo
- [ ] Todo número exibido tem `#`; nenhum número saiu de conta feita de cabeça
- [ ] Todo gráfico tem N com unidade, janela, `#` e linha de leitura
- [ ] Série temporal tem o evento anotado no ponto de inflexão; coorte incompleta marcada
- [ ] Funil, se houver: mesma unidade em todas as etapas, taxa de passagem entre degraus, coorte fechada
- [ ] Toda sigla expandida na primeira aparição e no glossário
- [ ] Toda métrica do glossário com definição operacional, não descrição
- [ ] Cada gráfico usa o `tipo` que a pergunta pede — barras só para comparar segmentos
- [ ] Se a fase 5 rodou, a checagem de composição está em `multiserie`; se a fase 3 olhou as bordas, está em `histograma`
- [ ] Anexo com o SQL de cada consulta e a frase de intenção de cada uma
- [ ] Limitações declaradas, incluindo as três tentativas que não deram
- [ ] Nenhuma lista de indivíduos no arquivo; público estimulável entregue como critério ou consulta
- [ ] Abre em tema claro e escuro sem quebrar; nenhuma barra de rolagem horizontal na página
