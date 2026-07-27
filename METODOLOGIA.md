# Método CRIVO — metodologia própria para loterias brasileiras

Documento de referência do projeto. Julho de 2026.
Base de dados: histórico oficial completo das Loterias Caixa (API `portaldeloterias`), 9 modalidades.

---

## 1. A tese central

**Não modele a bola. Modele a multidão.**

Todo método vendido no mercado brasileiro tenta prever qual dezena vai sair. Isso é matematicamente impossível, e nós testamos e confirmamos isso com o histórico completo. Mas existe uma segunda variável que determina quanto dinheiro entra na sua conta e que **não é aleatória**: quantas outras pessoas escolheram a mesma combinação que você. Essa variável é gerada por seres humanos, tem padrão estável, é mensurável — e nós a medimos.

O sorteio é o "retorno" no jargão de mercado: imprevisível, esqueça. O rateio é a "estrutura de risco": previsível, trabalhe nela.

## 2. As cinco regras

| Letra | Regra | Conteúdo |
|---|---|---|
| **C** | Custo-teto | Orçamento fixo definido *antes* de olhar o prêmio. Nunca aumenta porque acumulou. |
| **R** | Rateio | Escolher combinações que a multidão evita. Não muda a chance; muda o quanto se leva. |
| **I** | Independência | Nenhuma dezena escolhida por frequência, atraso ou ciclo. Esses dados existem no painel para serem refutados, não usados. |
| **V** | Valor esperado | Só apostar quando prêmio ÷ (probabilidade × custo) passa do gatilho. |
| **O** | Observação auditável | Registrar toda aposta e comparar contra baseline aleatório. |

## 3. Os testes que rodamos — e o que deu

Todos os testes são **walk-forward**: em cada concurso a estratégia decide usando apenas os concursos anteriores; só depois o resultado é revelado. Isso elimina o viés de olhar o passado inteiro de uma vez.

### Teste 1 — "Dezenas quentes" e "atrasadas" batem o acaso?

Comparamos quatro estratégias ao longo de milhares de concursos, medindo acertos médios por concurso.

| Jogo | Mais frequentes | Mais atrasadas | Aleatório | Fixo | Esperado pela teoria |
|---|---|---|---|---|---|
| Mega-Sena (2.835 concursos) | 0,5940 | 0,5866 | 0,5936 | 0,5795 | 0,6000 |
| Lotofácil (3.544 concursos) | 9,0322 | 8,9932 | 8,9986 | 8,9856 | 9,0000 |
| Quina (6.874 concursos) | 0,3182 | 0,3064 | 0,3104 | 0,3234 | 0,3125 |

**Resultado: nenhuma estratégia se destaca.** Todas orbitam o valor teórico. Isso valida a regra **I** e refuta o produto mais vendido do mercado.

### Teste 2 — As bolas são honestas? (qui-quadrado com correção)

Aplicamos a correção para sorteio **sem reposição**, fator `(N−1)/(N−k)` — que a maioria das análises amadoras esquece, produzindo p-valores errados.

| Jogo | χ² corrigido | g.l. | p | Monte Carlo |
|---|---|---|---|---|
| Mega-Sena | 86,78 | 59 | 0,011 | 0,0093 |
| Lotofácil | 59,38 | 24 | < 0,001 | < 0,001 |
| Quina | 103,27 | 79 | 0,035 | 0,042 |

Os três deram significativo. **Não varremos isso para debaixo do tapete** — investigamos no teste seguinte.

### Teste 2b — O desvio é estável? (o teste que realmente decide)

Um qui-quadrado significativo não prova viés. Uma bola fisicamente mais pesada teria de aparecer demais *o tempo todo*. Dividimos o histórico em três eras e correlacionamos os desvios.

| Jogo | 1ª×2ª | 1ª×3ª | 2ª×3ª | Média |
|---|---|---|---|---|
| Mega-Sena | −0,019 | −0,029 | +0,251 | +0,068 |
| Lotofácil | −0,011 | +0,375 | +0,381 | +0,248 |

**Veredito: o desvio não é estável.** Pelo menos um par de eras tem correlação praticamente zero em ambos os jogos. Isso é assinatura de ruído não-estacionário, não de bola viciada.

E mesmo se fosse real: a dezena mais sobre-representada da Mega-Sena (10) aparece 16,6% acima do esperado; na Lotofácil (20), 4,3%. Contra uma margem da casa de ~56%, nem no melhor cenário isso vira o jogo.

**Verificação de integridade dos dados:** 200 concursos sorteados aleatoriamente da Lotofácil foram reconferidos contra a API oficial da Caixa. Dos 118 que responderam, **118 conferiram exatamente, zero divergências**. O desvio observado está nos dados oficiais, não em erro de coleta.

### Teste 3 — A prova brasileira da seleção consciente ⭐

Este é o achado principal do projeto, e responde uma pergunta que a literatura internacional levanta mas nunca mediu com dados da Caixa.

**Pergunta:** quando o sorteio sai com "cara de aniversário" (muitas dezenas ≤ 31), aparecem mais ganhadores?

**Método:** para cada concurso da Mega-Sena com arrecadação publicada, medimos ganhadores da faixa da quadra **por milhão de reais arrecadados** — normalizando pelo tamanho do concurso. Se as pessoas jogassem ao acaso, essa métrica seria plana.

| Dezenas ≤ 31 sorteadas | Concursos | Ganhadores por R$ 1 mi | Prêmio médio da quadra |
|---|---|---|---|
| 0 | 6 | 120,5 | R$ 723 |
| 1 | 43 | 144,2 | R$ 602 |
| 2 | 108 | 172,6 | R$ 526 |
| 3 | 203 | 206,0 | R$ 438 |
| 4 | 140 | 236,6 | R$ 379 |
| 5 | 56 | 310,9 | R$ 314 |
| 6 | 8 | 351,1 | R$ 258 |

Monotônico e perfeito nas duas colunas. Pearson r = +0,551; Spearman ρ = +0,50; teste de permutação (5.000 embaralhamentos) p < 0,0001; Welch t = 9,95 entre os extremos.

**Tradução em dinheiro:**
- Sorteio com ≤ 2 dezenas de data: prêmio médio da quadra **R$ 554,55**
- Sorteio com ≥ 5 dezenas de data: prêmio médio da quadra **R$ 306,64**
- **Diferença: +81%**

A diferença não vem de sorte. Vem de quantas pessoas apostaram naquelas dezenas. Você não escolhe como o sorteio sai — mas escolhe estar do lado certo dessa conta.

**Este é o único ganho real, mensurável e legítimo disponível na loteria brasileira.**

#### Replicação independente na Quina ⭐

Um achado só vale se replica em dado independente. Rodamos o mesmo teste na **Quina** (5 dezenas de 80, estrutura completamente diferente, 3.911 concursos com arrecadação publicada):

| Dezenas ≤ 31 sorteadas | Concursos | Ganhadores por R$ 1 mi | Prêmio médio |
|---|---|---|---|
| 0 | 334 | 8.046 | R$ 53,70 |
| 1 | 1.046 | 8.787 | R$ 49,38 |
| 2 | 1.366 | 9.808 | R$ 42,36 |
| 3 | 861 | 10.981 | R$ 35,60 |
| 4 | 277 | 12.383 | R$ 32,28 |
| 5 | 27 | 11.277 | R$ 30,69 |

Spearman ρ = +0,224; t = 14,35; p < 0,0001. Prêmio médio **+55%** fugindo de datas. A única quebra de monotonicidade está no grupo de n = 27, onde o ruído domina — exatamente onde se espera.

**Duas loterias independentes, com número de dezenas, tamanho de aposta e público diferentes, produziram o mesmo padrão na mesma direção.** Isso é replicação genuína, não pesca de resultado.

Na **Lotofácil** o teste corretamente não se aplica: todas as 25 dezenas são ≤ 31, então não existe contraste (ρ = −0,010, p = 0,66 — exatamente o nada esperado). Esse é um bom controle negativo: o método não inventa efeito onde não pode haver.

*Limitação declarada:* a Caixa não publica a distribuição de apostas por combinação. Inferimos popularidade pelo número de ganhadores — proxy indireto, mas o efeito é inequívoco neste tamanho de amostra. A amostra concentra-se em concursos mais recentes, que são os que têm arrecadação publicada na API.

## 4. Comparação com os modelos existentes

| Modelo | Promete | Prevê o sorteio? | O que a evidência diz | O que aproveitamos |
|---|---|---|---|---|
| Frequência / dezenas quentes | Números que saem mais tendem a sair | Não | Refutado no nosso Teste 1 | Nada |
| Atraso / dezenas frias | Número atrasado está para sair | Não | Falácia do apostador (Clotfelter & Cook, 1993) | Nada |
| Ciclos, soma, pares/ímpares | Filtrar combinações improváveis | Não | Filtrar reduz cobertura | Usamos ao contrário: como proxy de popularidade |
| "IA que prevê dezenas" | Rede neural aprende o padrão | Não | Não há função a aprender em sequência i.i.d. | ML para modelar a multidão, não a bola |
| Desdobramento / fechamento | "14 pontos garantidos" | Parcial | Garantia combinatória real, mas condicional e sem efeito no EV | Ferramenta de eficiência de custo, com a condição explicitada |
| Bolão / sindicato | "Aumenta suas chances" | Parcial | Aumenta P(ganhar algo), divide o prêmio, EV inalterado | Escolha de perfil de risco, nunca de retorno |
| Compra total (Mandel, 1992) | Comprar todas as combinações | Sim, historicamente | Funcionou quando jackpot > custo total | O raciocínio: só apostar quando prêmio/custo justifica |
| Falha de desenho (Cash WinFall) | Explorar roll-down | Sim, historicamente | O jogo devolvia US$ 1,15 por US$ 1 em sorteios específicos | Procurar o momento, não o número |
| Números impopulares | Evitar datas e padrões | Efeito real | Baker & McHale (2009); Simon (1998) | **É o coração do método — e nós o medimos com dado brasileiro** |
| **Método CRIVO** | Não prevê nada; otimiza custo, rateio e timing | **Não prevê** | Assume a imprevisibilidade e trabalha só onde há sinal | — |

### Por que o nosso é melhor — e em que sentido exato

Seria desonesto dizer que o CRIVO ganha mais vezes. Ele não ganha. Nenhum método ganha mais vezes. É melhor em quatro sentidos verificáveis:

1. **É falseável.** Faz previsões testáveis e aceita ser refutado. Os métodos vendidos só mostram prints de acertos e escondem os erros.
2. **Ataca a variável certa.** Os outros disputam uma variável aleatória (a bola). O CRIVO disputa uma variável humana (a escolha dos outros apostadores).
3. **Tem efeito econômico demonstrado.** +81% no prêmio médio, medido com dado oficial.
4. **Limita o dano.** O custo-teto garante que o pior caso é conhecido antes de jogar.

**O que o CRIVO não faz:** não aumenta a probabilidade de acertar, não torna a loteria lucrativa na média, e não substitui a decisão de simplesmente não jogar — que continua sendo, matematicamente, a melhor decisão financeira disponível.

## 5. A fronteira da previsibilidade — cartas, roleta e mercado

A pergunta certa não é "dá para prever?", é "**por que dá em uns e não em outros?**". A resposta é uma palavra: **reposição**.

| Sistema | Tem memória? | O que é previsível | Por quê | Vale a pena? |
|---|---|---|---|---|
| Blackjack (contagem) | Sim | Composição do baralho restante | Cartas saem **sem reposição**: cada carta vista muda a probabilidade da próxima. Informação se acumula. | Sim — vantagem histórica de ~0,5% a 1,5% |
| Pôquer | Sim | Comportamento dos adversários | Joga-se contra pessoas. O sinal está no oponente. | Sim — jogo de habilidade |
| Roleta | Não | Nada, salvo viés físico da roda | Cada giro é independente | Não, em roda moderna |
| Mercado financeiro | Parcial | Volatilidade (forte), retorno (muito fraco) | Preços gerados por agentes com feedback. Há estrutura, mas competida até quase sumir. | Marginalmente, com custo baixo e horizonte longo |
| **Loteria — o sorteio** | **Não** | **Nada** | Bolas voltam ao globo. Reposição total ⇒ independência total ⇒ informação zero. | **Não** |
| **Loteria — o rateio** | **Sim** | **Quantas pessoas escolhem cada combinação** | Gerado por humanos com vieses estáveis: datas, sequências, o número 7, bordas do volante. | **Sim — é aqui que o CRIVO joga** |

**A transposição que fizemos.** Do mercado tiramos a lição mais robusta dos últimos 50 anos: o retorno é quase imprevisível, mas a estrutura de risco é bem previsível — por isso a gestão profissional foca em risco e custo, não em adivinhar preço. Do blackjack tiramos a disciplina de bankroll: valor fixo, apostado por regra, nunca por emoção. Do pôquer tiramos o hábito de modelar o outro jogador em vez do baralho.

## 6. Referências consultadas

**Acadêmicas**
- Clotfelter & Cook (1993). *The "Gambler's Fallacy" in Lottery Play*. Management Science 39(12).
- Baker & McHale (2009). *Modelling the probability distribution of prize winnings in the UK National Lottery: consequences of conscious selection*. JRSS-A 172(4).
- Simon (1998). *An Analysis of the Distribution of Combinations Chosen by UK National Lottery Players*. Journal of Risk and Uncertainty 17(3).
- Farrell et al. (2000). *The Demand for Lotto: The Role of Conscious Selection*. JBES 18(2).
- Wang, Potter van Loon, van den Assem & van Dolder (2016). *Number preferences in lotteries*. Judgment and Decision Making 11(3).
- Polin, Ben Isaac & Aharon (2021). *Patterns in manually selected numbers in the Israeli lottery*. JDM 16(4).
- Chernoff (1981). *How to beat the Massachusetts numbers game*. Mathematical Intelligencer 3.
- Abrams & Garibaldi (2010). *Finding good bets in the lottery, and why you shouldn't take them*. American Mathematical Monthly 117(1).
- Genest, Lockhart & Stephens (2002). *Chi-square and the lottery*. JRSS-D 51.

**Institucionais e legais**
- Lei nº 13.756/2018, art. 16, II, "i" — 43,79% da arrecadação dos prognósticos numéricos para premiação.
- Lei nº 4.506/1964, art. 14 e Decreto nº 9.580/2018 (RIR), art. 732 — IR de 30% exclusivamente na fonte sobre prêmios de loteria.
- Caixa — página oficial da Mega-Sena (probabilidades e rateio).
- Office of the Inspector General, Massachusetts (2012) — relatório sobre o Cash WinFall.
- La Jolla Covering Repository — covering designs (base matemática dos desdobramentos).

**Mercado brasileiro (verificação de alegações)**
- Polícia Federal (nov/2025) — Operação Última Aposta: R$ 107 milhões bloqueados; plataforma cobrava acima da tarifa alegando "análises estatísticas para melhorar as chances".
- TRF4 (jan/2025) — apostadora que usou site não oficial recebeu R$ 3.740 em vez de R$ 64.232; ação julgada improcedente.
- Procon/MS (2022) — apuração sobre 15 sites e apps não autorizados a pedido da Caixa.
- Aos Fatos e Agência Lupa (dez/2024) — sites falsos de bolão da Mega da Virada.

---

## 8. Quando o método precisa mudar — gatilhos de revisão

Um método que nunca muda é dogma; um método que muda por qualquer motivo é perseguição de ruído. A saída é definir **antes** quais evidências obrigariam a mudar. Revisão de literatura de 2024 a 2026.

### A descoberta que organiza tudo

O método tem dois objetos com naturezas opostas:

| Objeto | Propriedade | Sofre deriva? |
|---|---|---|
| **O sorteio** (quais dezenas saem) | Aleatório, estacionário | **Não.** Se sofrer, é fraude, não oportunidade. |
| **A popularidade das dezenas** (o que as pessoas escolhem) | Comportamental, dinâmica, sensível a mídia | **Sim, e já sofreu comprovadamente.** |

Portanto **a regra R é a única exposta a envelhecimento**. As regras C, I e V só mudam se a regra do jogo mudar.

### Recalibrar — ajustar parâmetros, manter o método

| # | Gatilho | Dispara quando |
|---|---|---|
| R1 | Mudança de matriz, preço ou percentual de premiação | Qualquer alteração em fonte oficial |
| R2 | Deriva do mapa de popularidade | A cada 12 meses, ou imediato se um evento nacional se associar a uma dezena |
| R3 | Expansão ou contração do público | Nova jurisdição no mesmo pool, ou variação sustentada acima de 20% no volume |
| R4 | Correção monetária do custo-teto | Preço da aposta subir 10% ou mais, ou anualmente |
| R5 | Recalibrar o ponto ótimo de valor esperado | Sempre que R1 ou R3 disparar |

Precedente de R1: em 08/04/2025 a Mega Millions passou de US$ 2 para US$ 5 (+150%) melhorando as odds em apenas 4%. Precedente de R2: a escolha do número 19 despencou na Bélgica depois que a OMS nomeou a COVID-19. Precedente de R4: o reajuste da Caixa de 09/07/2025 (~21,7% médio) fez um teto fixo em reais comprar ~17% menos apostas.

### Aperfeiçoar — o método está certo, mas incompleto

| # | Gatilho | Dispara quando |
|---|---|---|
| A1 | Medir a diluição por surpresinha com dado próprio | Se em 12 meses não tivermos estimativa própria |
| A2 | Segmentar popularidade por tipo de concurso | Ao ter dados de concursos comuns × de pico |
| A3 | Tornar o custo-teto fisicamente vinculante | **Imediato — já disparado** |
| A4 | Formalizar a métrica de auditoria | Antes do próximo ciclo |

A métrica certa não é "ganhei ou não" (ruído puro), e sim prêmio médio por acerto ÷ prêmio médio de carteira aleatória simulada, com intervalo de confiança.

### Abandonar ou substituir — o método deixou de valer

| # | Gatilho | Dispara quando |
|---|---|---|
| X1 | Migração de prêmio rateado para prêmio fixo | Qualquer modalidade jogada adotar prêmio fixo |
| X2 | Efeito de rateio indistinguível de zero | Após 3 anos, se o IC 95% contiver 1,0 e o efeito for menor que 3% |
| X3 | Valor esperado negativo mesmo no melhor cenário | Por 12 meses seguidos |
| X4 | O teto de custo foi rompido | 2 ciclos seguidos, ou 1 estouro acima de 50% |
| X5 | Evidência revisada por pares de viés no sorteio | Publicação com método replicável |
| X6 | Modelo preditivo validado independentemente | Replicação revisada por pares, fora da amostra |

X4 não é gatilho estatístico, é de segurança: romper o teto sinaliza que a moldura racional deixou de governar o comportamento, e a resposta certa é parar, não recalibrar.

### O que a revisão 2024–2026 já mudou na prática

**Confirmado.** A regra I sai reforçada: o teste formal mais recente de aleatoriedade (2.706 sorteios da 6/49 romena, 1993–2024, qui-quadrado + Monte Carlo) não rejeitou a hipótese nula. Nenhum trabalho revisado por pares mostra ML batendo o acaso.

**Refinado — muda a prática.** O prêmio máximo *não* é o melhor momento. A demanda tem elasticidade de ~1,7 ao tamanho do prêmio contra ~0,5 ao preço; acumulações recordes atraem uma multidão que dilui o rateio. O ponto ótimo é uma acumulação boa *antes* de virar manchete nacional.

**Refinado.** O viés de datas é heterogêneo: mais forte em quem preenche um único bilhete e some a partir do segundo jogo na mesma sessão. Fugir de datas rende mais justamente nos concursos de pico, cheios de apostadores ocasionais.

**Reforçado com urgência.** A regra C precisa virar limite fisicamente vinculante — dinheiro separado antes, não valor anotado. Limites intransponíveis reduzem o gasto mais que limites voluntários, e entender de estatística não protege: campanhas educativas se mostraram ineficazes.

**Nada foi refutado.** Nenhum achado de 2024 a 2026 contradiz qualquer das cinco regras.

**Lacunas honestas.** Ninguém mediu o efeito da surpresinha em loterias grandes; nenhuma evidência nova de viés físico; nenhuma refutação metodológica formal de ML em loteria.

---

## 9. Operação — atualizar, verificar e fazer backup

O script `dashboard\atualizar_tudo.ps1` executa, nesta ordem:

1. baixa os concursos novos dos 9 jogos na API oficial
2. repara automaticamente qualquer concurso inconsistente (reconsulta a API)
3. **verifica** a base inteira: buracos na sequência, dezenas duplicadas, fora de faixa, datas faltando
4. **só então** refaz o backup único em `data_backup\`
5. regera o `bundle.json` que alimenta o painel

Se a verificação falhar, o backup anterior é preservado e o log avisa. É um backup só, sobrescrito a cada atualização bem-sucedida. Para restaurar, copie tudo de `data_backup\` para `data\`.

O painel online tem o botão **Atualizar estatísticas**, que faz o mesmo pela API direto do navegador (a Caixa envia `Access-Control-Allow-Origin: *`), guarda o delta no próprio navegador e roda a verificação de integridade ao final, invalidando todos os caches para nenhuma aba trabalhar com número velho.

Uma tarefa agendada roda essa rotina toda segunda-feira às 11h e envia o relatório com o índice CRIVO por modalidade.

---

## 10. Registro de apostas — a regra O em funcionamento

A quinta regra do método exige registrar toda aposta e comparar contra um baseline aleatório. Sem isso o CRIVO não é falseável na prática, só na teoria — e o gatilho X2, que aos três anos faz o teste decisivo da regra R, nunca poderia disparar.

A aba **Minhas apostas** implementa isso. Toda aposta registrada ganha uma **sombra**: um jogo do mesmo tamanho, sorteado ao acaso no mesmo instante, sem nenhum filtro. As duas carteiras correm juntas e o painel confere as duas contra o resultado oficial assim que o concurso sai. O registro fica salvo no navegador e pode ser exportado em JSON.

Um caso de borda que o painel trata explicitamente: se a sua aposta cai numa faixa que **acumulou** naquele concurso, o rateio registrado é zero — porque ninguém acertou. O painel mostra "acumulou" em vez de R$ 0 e explica que, se você tivesse mesmo aquele bilhete, teria sido o único ganhador. Não coloco um valor ali porque o prêmio de um ganhador que não existiu não é um dado, é uma hipótese.

Enquanto houver menos de 30 apostas conferidas, o painel avisa que o placar não deve ser lido: um único prêmio de faixa alta vira o resultado inteiro.

## 11. O backtest do próprio método

O teste que pode nos refutar, na aba Evidências. Para cada concurso de um período, gera jogos pelo CRIVO usando **apenas o que se sabia até o concurso anterior**, compara com uma carteira aleatória de tamanho idêntico, e calcula o retorno usando o **rateio realmente pago** em cada concurso.

Resultado com 100 concursos e 10 jogos por concurso: as duas carteiras perdem, e perdem quase o mesmo. Na Mega-Sena ambas terminaram em zero — ninguém fez quadra em mil jogos. Na Lotofácil, 25,6% contra 27,4%. Na Quina, 4,2% contra 10,3%, com a diferença inteira vindo de dois prêmios de faixa alta.

**É exatamente o previsto.** Se este teste mostrasse o CRIVO ganhando de forma consistente, a conclusão certa seria que há erro no teste, não que descobrimos algo.

Dois aprendizados de construção que valem registro. O filtro de "sem 3 dezenas consecutivas" é **impossível na Lotofácil** — marcando 15 de 25, três seguidas são inevitáveis; os limites agora se ajustam à densidade da aposta. E o retorno observado fica **sistematicamente abaixo** do RTP de longo prazo, porque a maior parte daquele percentual está em faixas raríssimas que uma amostra pequena nunca alcança. A média do jogo é uma miragem que só se materializa numa quantidade astronômica de apostas.

## 12. Por que algumas lotéricas acertam várias vezes

Pergunta frequente, com resposta inteiramente estatística — e mensurável.

Os 979 ganhadores da sena na história da Mega-Sena, com a cidade extraída da planilha oficial, distribuem-se assim: SP com 27,5% dos ganhadores para 21,6% da população, RJ 10,6% para 8,1%, MG 11,3% para 10,0%, RS 5,1% para 5,3%, SC 3,5% para 3,8%. A razão orbita 1 — onde tem mais gente, saem mais prêmios, na proporção.

A conta que fecha a questão: com 979 bilhetes vencedores distribuídos entre cerca de 13 mil lotéricas, mesmo que todas vendessem o mesmo volume seriam esperadas **35 lotéricas com dois ou mais prêmios**. Como o volume é desigual, se 10% das casas concentram 60% das vendas o esperado sobe para **99 com dois prêmios e 14 com três ou mais**. A existência de dezenas de "lotéricas da sorte" não é curiosa — seria estranho se elas não existissem.

O mecanismo é volume, não sorte. Uma casa que vende cem vezes mais bilhetes tem cem vezes mais chance de vender o vencedor, e lotéricas que organizam **bolões** concentram centenas de apostas numa compra só. Some-se o viés de sobrevivência (quem acerta pendura faixa; as 12.900 que nunca acertaram não anunciam nada) e o ciclo de retroalimentação: fama atrai movimento, movimento produz prêmios, prêmios confirmam a fama.

É o mesmo erro do Teste 2b em outra roupa — um padrão que parece extraordinário até você contar quantas oportunidades existiam para ele aparecer.

*Ressalva: o número de 13 mil lotéricas vem de reportagem, não de dado oficial da Caixa. Os 979 ganhadores, as cidades e a população são dado oficial.*

## 13. Aviso

Aposta de loteria não é investimento nem fonte de renda. O retorno esperado é estruturalmente negativo por lei. Este projeto serve para tomar decisões informadas sobre um gasto de entretenimento, e para não ser enganado por quem vende método. Se o jogo deixar de ser diversão, procure o programa Jogo Responsável da Caixa.

## 14. Filtro cruzado, régua do gatilho e a correção de co-ocorrência (v9)

Três mudanças na versão 9, sendo a terceira uma correção de um erro que estava no nosso próprio cálculo.

**A seleção passou a ser única e compartilhada.** Modalidade, janela de concursos e dezenas escolhidas ficam numa barra fixa no topo, e valem para todas as abas ao mesmo tempo. Cada mudança grava uma posição numa pilha, então desfazer e refazer funcionam para qualquer passo — inclusive troca de aba — sem precisar instrumentar botão por botão. Clicar numa dezena do volante ou da dispersão a joga na seleção; clicar de novo tira.

**A régua do gatilho** abre a aba Onde apostar. É uma linha de 0 a 1,10 com as nove modalidades posicionadas pelo índice CRIVO e a marca de break-even em 1,00. Responde de olhada a única pergunta que importa antes de apostar, e clicar num ponto abre a análise completa daquela modalidade.

**A correção que importa.** O painel novo mostra, para uma dezena selecionada, quais outras saíram junto com ela mais e menos do que o esperado. O esperado natural seria `freq(i) × freq(j) ÷ total` — mas essa fórmula supõe sorteio **com reposição**. Como a Caixa sorteia sem reposição, a probabilidade real de duas dezenas caírem no mesmo concurso é `k(k−1) / (N(N−1))`, e não `(k/N)²`. Na Mega-Sena a diferença é de 18%, o suficiente para empurrar **todos** os pares para o lado negativo e fazer qualquer combinação parecer "pouco trilhada". Seria um erro do mesmo tipo que este projeto critica nos outros: um artefato do cálculo apresentado como descoberta.

Em vez de assumir a fórmula fechada — que muda na Dupla-Sena, com dois sorteios por concurso, e na Timemania — calibramos a escala nos próprios dados: o total esperado é forçado a bater com o total observado, e só o desvio relativo sobra. A calibração empírica reencontrou o valor teórico com quatro casas: Mega-Sena 0,8475 contra 300/354 = 0,84746; Quina 0,8102 contra 320/395 = 0,81013; Lotofácil 0,9722 contra 350/360 = 0,97222; Timemania 0,8680 contra 480/553 = 0,86799. Na Dupla-Sena deu 0,9345, para a qual não há fórmula simples — que é justamente por que a calibração empírica foi a escolha certa.

Com a escala corrigida, o desvio médio passa a ser praticamente zero em todas as modalidades e os extremos ficam entre −2,2σ e +2,8σ. Testando 59 a 79 pares, o maior |z| que o acaso puro produz já fica perto de 2,9σ. **Ou seja: não há nenhuma dupla de dezenas com co-ocorrência anômala em nenhuma das nove modalidades.** O painel diz isso em voz alta, porque esse é o resultado.

**O gerador passou a otimizar, não só relatar.** Antes ele aceitava o primeiro sorteio que passasse nos filtros. Agora junta até 25 candidatos válidos e fica com o de menor co-ocorrência média. Todos têm exatamente a mesma probabilidade de sair — o critério só separa o que é menos trilhado por outros apostadores. Medido sobre 200 rodadas na Mega-Sena, o z médio dos pares cai de −0,85 para −1,23 (na escala não calibrada usada no teste). É ganho de rateio, nunca de chance.
