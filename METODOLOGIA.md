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

## 15. As demais modalidades — e um erro grave que elas revelaram (v10)

A versão 10 nasceu de um pedido simples: fazer para as outras modalidades o que já estava feito para a Mega-Sena e a Lotofácil. Fechar essas lacunas expôs um defeito no cálculo de prêmio que estava lá desde o começo.

### O erro

O arquivo de prêmios traz, por concurso, um array de faixas. O código encontrava a faixa certa fazendo `índice = dezenas marcadas − acertos`. Isso só funciona quando a modalidade tem faixas puramente por número de acertos — e cinco das nove não têm.

A Dupla-Sena tem oito faixas: as quatro do primeiro sorteio seguidas das quatro do segundo. Pela regra antiga, **2 acertos caíam no índice 4, que é a sena do segundo sorteio**. O backtest chegava a exibir 507.958% de retorno. A Timemania tem o Time do Coração como sexta faixa, então 2 acertos viravam Time do Coração. O Dia de Sorte tem o Mês da Sorte como quinta, então 3 acertos viravam Mês. A Lotomania guarda a faixa de zero acertos no fim do array, e 14 acertos caíam nela.

A correção foi trocar a aritmética por um mapa explícito de faixas por modalidade, conferido contra as contagens de ganhadores. O Dia de Sorte serve de exemplo do método de verificação: no concurso 1255, os 36.273 ganhadores da última faixa divididos por 1/12 dão cerca de 435 mil apostas; a quarta faixa prevê 435 mil × P(4 de 7) = 11.720 ganhadores, contra 11.781 observados.

A +Milionária ficou de fora. As dez faixas dela combinam acertos de dezenas com acertos de trevos, e não conseguimos confirmar a ordem conferindo contagens contra probabilidades. Preferimos a lacuna declarada a um número inventado com cara de medição.

### O segundo erro

O backtest gerava jogos do tamanho do **sorteio**, não do tamanho da **aposta**. Na Lotomania sorteiam-se 20 dezenas mas a aposta marca 50; na Timemania sorteiam-se 7 e a aposta marca 10. O simulador comprava apostas que não existem e cobrava por elas o preço da aposta real. Agora o tamanho vem da tabela oficial de apostas, e a distribuição de acertos bate com a hipergeométrica de cada jogo: Lotomania com moda em 10 acertos, Lotofácil em 9, Dia de Sorte em 2, Mega-Sena em 0.

### Uma rodada só engana

Corrigidos os dois erros, ficou visível o que já se suspeitava: o resultado de uma carteira de mil jogos é dominado por um punhado de prêmios de faixa alta, e trocar a semente muda tudo. Na Timemania o retorno do CRIVO variou de 3,8% a 58,4% entre oito sementes dos mesmos concursos. Mostrar um número só seria vender sorte como método.

O backtest agora roda oito sementes independentes e mostra média, faixa de variação e desvio das duas carteiras, além de comparar a diferença entre elas com o ruído da própria comparação. Na quase totalidade das modalidades a diferença cabe dentro do ruído — as duas carteiras são indistinguíveis, que é exatamente o previsto: o critério do CRIVO age no rateio, não na chance de acertar.

### Super Sete

Deixou de ser a modalidade excluída. Cada uma das sete colunas é um sorteio independente de um dígito de 0 a 9, então a unidade de análise mudou: a seleção do filtro cruzado passa a ser a célula (coluna, dígito), o qui-quadrado é calculado por coluna com 9 graus de liberdade, e a estratégia de "dígito quente" é medida em colunas certas por concurso, contra o esperado de 0,7.

Resultado sobre 877 concursos: qui-quadrado somado de 49,72 com 63 graus de liberdade, contra um valor crítico de 82,53 e um valor esperado por acaso de 63. Nenhuma coluna passa do limite individual de 16,92. Não há evidência de viés em nenhuma das sete.

O gerador do Super Sete traz um aviso que vale repetir: diferente do efeito aniversário na Mega-Sena, que **medimos** no rateio real, os critérios anti-popularidade do Super Sete são raciocínio por analogia com escolha humana de dígitos. A Caixa não publica a distribuição das apostas dessa modalidade. Está escrito na tela, ao lado de cada jogo gerado.

### Dupla-Sena, e o melhor teste que temos do nosso próprio método

Os dois sorteios agora podem ser vistos separados ou somados, e a escolha entra na barra de seleção como qualquer outro filtro. Os dois passam no qui-quadrado: 55,79 e 32,47, contra um crítico de 66,34 com 49 graus de liberdade.

Mas o valor real da Dupla-Sena é outro. Ela oferece um experimento natural que nenhuma outra modalidade oferece: pares de dezenas **dentro** do mesmo sorteio estão sob restrição sem reposição, enquanto pares **entre** os dois sorteios do mesmo concurso são independentes, porque são duas extrações separadas. A calibração descrita na seção 14 tem de devolver coisas diferentes nos dois casos.

Devolve. Dentro do mesmo sorteio: 0,8504, contra o teórico 250/294 = 0,8503. Entre os dois sorteios: 0,9994, contra o teórico 1,0000. E no Super Sete, onde as sete colunas são sorteios independentes, a escala entre colunas dá exatamente 1,0000.

É o resultado que valida a correção: a calibração está capturando a estrutura do sorteio, e não inventando um ajuste conveniente para fazer os números caírem onde a gente gostaria.

## 16. O break-even de 1,00 é alcançável? A varredura histórica (v11)

Pergunta do Hélio, e das boas: se o índice nunca chega perto de 1,00, talvez o limiar não seja realista. Fui medir.

### O método da varredura

Para cada sorteio da base em que o prêmio principal foi **realmente ganho**, o índice foi recalculado com o valor efetivamente pago — ganhadores × rateio — em vez de uma estimativa. Restringir aos sorteios com ganhador não é conveniência: é o que torna a medida exata. O bolo acumula até ser ganho, então o sorteio em que ele sai é justamente o pico da acumulação. Todo acúmulo passado terminou em alguém ganhando, então todos os picos históricos estão na amostra.

Faltavam duas peças. A primeira é o número de apostas, que define o fator de partilha; obtive estimando as apostas simples equivalentes a partir da contagem de ganhadores da faixa mais baixa, cuja probabilidade é conhecida por combinatória. A segunda é o preço da aposta na época, que veio de arrecadação ÷ apostas estimadas.

Essa reconstrução se valida sozinha, e é por isso que confio nela: o preço efetivo estimado para 2026 deu R$ 3,49 na Lotofácil contra R$ 3,50 oficiais, R$ 2,94 na Quina contra R$ 3,00, R$ 2,54 no Dia de Sorte contra R$ 2,50, R$ 3,54 na Timemania contra R$ 3,50. E a série reproduz os aumentos de preço da Caixa ao longo dos anos sem que ninguém tenha informado a data deles.

O RTP das faixas menores foi calculado em janela expansiva, usando só o passado de cada ponto — sem olhar o futuro.

### O resultado

**Em 4.635 sorteios com prêmio principal observado, o índice cruzou 1,00 uma única vez:** Lotomania, concurso 1741, 03/03/2017, índice 1,006, prêmio de R$ 26,4 milhões dividido entre 2 ganhadores.

O recorde da Mega-Sena é 0,851, na Mega da Virada de 2015, com prêmio de R$ 197 milhões. Nem a maior Virada da história chegou lá. O percentil 99 de cada modalidade fica entre 0,43 e 0,77.

**A conclusão é que o Hélio estava certo.** O 1,00 é um marco teórico correto e operacionalmente inútil. Dizer "índice menor que 1, não aposte" é matematicamente impecável e equivale a dizer "nunca aposte" — que é uma recomendação legítima, mas que não precisa de painel para ser dada.

### O que substitui o 1,00

O percentil histórico da própria modalidade. O painel passou a mostrar, ao lado do índice absoluto, onde ele se situa na distribuição daquele jogo. É a diferença entre "0,53, desfavorável" e "0,53, que é o percentil 99,8 da história da Lotofácil" — a segunda frase informa uma decisão, a primeira não.

O 1,00 continua marcado na régua, como referência do que seria o ponto de virada. Mas ele deixou de ser o veredito.

### Os sorteios especiais, e a tese do método aparecendo sozinha

Classificando os sorteios pela data, os especiais dominam o topo histórico: a Lotofácil da Independência é 1,2% da base e ocupa 13 das 20 melhores posições; a Quina de São João é 3,4% da base e ocupa 10 das 20.

Mas a mediana conta outra história, e é a mais interessante do capítulo:

| Sorteio especial | Ocorrências | Índice mediano | Sorteio comum | Ganho |
|---|---|---|---|---|
| Quina de São João | 18 | 0,532 | 0,262 | 2,03× |
| Mega da Virada | 22 | 0,352 | 0,248 | 1,42× |
| Timemania de Natal | 4 | 0,327 | 0,293 | 1,12× |
| Dia de Sorte da Primavera | 12 | 0,323 | 0,290 | 1,11× |
| Lotofácil da Independência | 35 | 0,355 | 0,347 | 1,02× |

A Lotofácil da Independência tem o prêmio mais chamativo da lista e o menor ganho real: 2%. O prêmio gigante atrai uma enxurrada de apostas — o fator de partilha nesses sorteios chega a λ ≈ 44, ou seja, quarenta e quatro ganhadores esperados — e a multidão come quase toda a vantagem. A Quina de São João é a exceção porque o prêmio cresce mais do que o público.

É a tese do CRIVO se confirmando sem que a gente tenha forçado nada: **o que decide não é o tamanho do prêmio, é quanta gente está disputando ele com você.**

### A ressalva que não consegui fechar

O painel desconta 30% de imposto de renda do prêmio antes de calcular o retorno esperado. Duas reportagens afirmam que o valor divulgado pela Caixa **já é líquido**; a página oficial da Mega-Sena diz que "o prêmio bruto corresponde a 43,79% da arrecadação" e não esclarece em que momento a retenção acontece.

Testei contra os dados. Somando tudo que foi pago e dividindo pela arrecadação, a Mega-Sena dá 32,9%, a Lotofácil 38,6%, o Dia de Sorte 35,5%. Se os valores fossem brutos, esperaríamos algo perto de 43,35%; se fossem líquidos, perto de 30,3%. Nenhuma das duas hipóteses fecha de forma limpa, e a diferença entre modalidades sugere que as reservas dos sorteios especiais desviam frações diferentes da arrecadação em cada jogo — o que impede a conta de fechar por esse caminho.

Se o valor divulgado já for líquido, todos os índices sobem cerca de 43% e os cruzamentos de 1,00 passam de 1 para 5 em 4.635 sorteios: Mega da Virada 2015 (1,155), Quina de São João 2016 (1,012) e 2018 (1,022), Timemania de Natal 2025 (1,014) e a Lotomania de 2017 (1,355).

**Continua raro nas duas leituras, e a conclusão prática não muda** — mas o número exato depende disso, e prefiro registrar a dúvida a escolher a hipótese que deixa o resultado mais bonito.

## 17. O imposto: a dúvida da seção 16, resolvida — e o erro que ela escondia (v12)

Na seção anterior registrei uma dúvida em aberto: o painel descontava 30% de IR do prêmio, mas eu não tinha conseguido confirmar se o valor divulgado pela Caixa já vinha líquido. O Hélio informou que vem. Fui verificar antes de mudar, e o teste fechou de um jeito que não deixa dúvida.

### A verificação

Se o rateio publicado já é líquido, então para reconstruir o valor bruto basta dividir por 0,70 — mas só as faixas que efetivamente sofrem retenção. A retenção de 30% incide sobre prêmios acima de R$ 1.903,98; abaixo disso não há desconto. Então: grosso modo, reconstruir o bruto significa dividir por 0,70 toda faixa cujo prêmio por ganhador passe desse piso, e deixar as demais como estão.

Feito isso, o total reconstruído tem de bater a fatia legal da arrecadação destinada a prêmios: 43,35% nos prognósticos numéricos. Resultado sobre a base inteira:

| Modalidade | Bruto reconstruído | Desvio da fatia legal |
|---|---|---|
| Mega-Sena | 43,56% | +0,21pp |
| Quina | 43,46% | +0,11pp |
| Dupla-Sena | 43,42% | +0,07pp |
| Lotofácil | 43,16% | −0,19pp |
| Lotomania | 43,16% | −0,19pp |
| Dia de Sorte | 42,92% | −0,43pp |
| Super Sete | 42,69% | −0,66pp |

Sete modalidades, com estruturas de faixa completamente diferentes — três, quatro, cinco, sete, oito faixas —, todas caindo dentro de sete décimos de ponto percentual do mesmo alvo. Sem o ajuste do piso de retenção, o erro ia de −2 a −3,6 pontos em todas. Um acaso não se comporta assim.

Foi também o que explicou por que, na seção 16, nenhuma das duas hipóteses simples fechava: eu estava testando "tudo bruto" contra "tudo líquido", quando a realidade é mista — as faixas grandes vêm líquidas, as pequenas não sofrem retenção.

### A consequência

O rateio publicado é o que chega na mão do apostador. Então o índice passa a usá-lo como está, sem nenhum fator de imposto. Na prática:

- todos os índices sobem cerca de 43%. A Lotofácil de hoje foi de 0,53 para **0,63**; a Mega-Sena de 0,30 para **0,38**;
- o prêmio de gatilho cai 30%. Na Lotofácil, de R$ 26,4 milhões para **R$ 18,5 milhões**; na Mega-Sena, de R$ 400 para **R$ 280 milhões**;
- os cruzamentos históricos de 1,00 passam de 1 para **5 em 4.635 sorteios**.

### Os cinco cruzamentos, e o que eles têm em comum

| Modalidade | Concurso | Data | Índice | Prêmio | |
|---|---|---|---|---|---|
| Lotomania | 1741 | 03/03/2017 | 1,355 | R$ 26,4 mi | sorteio comum |
| Mega-Sena | 1772 | 22/12/2015 | 1,155 | R$ 197,4 mi | Mega da Virada |
| Quina | 4706 | 23/06/2018 | 1,022 | R$ 125,1 mi | Quina de São João |
| Timemania | 2336 | 27/12/2025 | 1,014 | R$ 73,2 mi | Timemania de Natal |
| Quina | 4114 | 24/06/2016 | 1,012 | R$ 143,1 mi | Quina de São João |

**Quatro dos cinco são sorteios especiais.** O break-even existe e é alcançável — mas praticamente só no calendário, uma ou duas vezes por ano, e nem toda edição chega lá. Cinco em 4.635 é pouco mais de um décimo de um por cento.

Isso não contradiz a seção 16, apenas refina a conclusão. O 1,00 continua sem servir de veredito diário: dizer "índice menor que 1, não aposte" segue equivalendo a "quase nunca aposte". O percentil histórico da própria modalidade continua sendo o sinal que separa um dia comum de um dia raro. O que mudou é que agora sabemos exatamente onde fica a fronteira, e que ela tem endereço no calendário.

### Registro de responsabilidade

A informação de que o valor divulgado já é líquido veio do Hélio, não de uma fonte documental que eu tenha conseguido citar. O que eu fiz foi testá-la contra a base inteira, e ela passou com folga. Fica registrado assim: **afirmação do usuário, confirmada por verificação independente nos dados**, não uma leitura de norma. Se algum dia a fatia legal mudar ou o piso de retenção for corrigido, este teste é o que precisa ser refeito — está reproduzido em comentário no código, logo acima da constante `IR_LOTERIA`.

## 18. Conjuntos, ordenação, desdobramento e repetições (v13)

Quatro pedidos do Hélio numa rodada só, mais um erro de base que apareceu no caminho.

### O que faltava nos conjuntos

O painel de "Mais sorteados juntos" mostrava só a contagem. Agora mostra, para cada conjunto, **quando saiu pela última vez**, **há quantos concursos** e **qual foi o maior intervalo entre duas aparições**. Calculado de forma incremental — um objeto por conjunto observado, sem guardar a lista de concursos, que na Lotofácil passaria de um milhão de entradas.

Para a quadra 04-18-21-38 da Mega-Sena, por exemplo: quatro aparições, a última no concurso 1783 de 23/01/2016, há 1.253 concursos, com maior intervalo de 890 concursos entre duas delas.

Mas o número que dá sentido a tudo isso é outro, e ele também entrou: **quantos conjuntos o acaso sozinho faria repetir tanto assim.** Existem 487.635 quadras possíveis na Mega-Sena, cada uma aparecendo em média 0,093 vez em 3.036 concursos. O acaso produziria cerca de 1,43 quadras com quatro ou mais aparições; observamos 5. A probabilidade de uma base honesta produzir 5 ou mais é de **1,6%**.

Esse 1,6% foi validado por simulação: 250 universos honestos de 3.036 sorteios da Mega-Sena deram média de 1,49 quadras com 4+ aparições (a fórmula de Poisson previa 1,43) e produziram 5 ou mais em 1,6% das vezes — exatamente o que a fórmula de segundo nível calcula. A simulação também desfez uma suposição minha: eu esperava que a sobreposição entre as quinze quadras de um mesmo sorteio inflasse a cauda, e ela praticamente não infla.

É um resultado no limite alto, e o painel diz isso sem drama. Vale lembrar que o teste foi escolhido **depois** de olhar o painel, que é exatamente o tipo de teste post-hoc que fabrica eventos de 1,6%.

### Ordenação do mapa de calor

O volante em ordem de volante é bonito e péssimo para responder "quais saíram mais". Agora dá para ordenar por dezena crescente ou decrescente, por frequência e por atraso, nos dois sentidos. Quando a ordem sai do padrão, o painel avisa que a grade deixou de representar o papel que se preenche na lotérica — e aproveita para lembrar que estar no topo da lista não dá vantagem nenhuma no próximo sorteio.

A aba Volante foi absorvida por Mais & menos. Eram duas abas olhando os mesmos números com controles duplicados. Rotas antigas continuam funcionando: qualquer estado que aponte para `volante` é redirecionado em silêncio.

### Marcar mais uma dezena melhora o índice?

Não melhora, e a razão é exata. Marcar n dezenas equivale a C(n, pick) apostas simples, e a Caixa paga cada combinação vencedora separadamente. Valor esperado e custo são multiplicados pelo **mesmo** fator, e o índice, que é a razão entre os dois, não se move um centésimo. Na Mega-Sena, o custo por unidade de chance é R$ 300.383.160 marcando 6, 7 ou 8 dezenas — idêntico até o último real.

O que muda, muda para pior se o critério for levar algum prêmio. Um desdobramento de 7 dezenas na Mega-Sena paga alguma coisa 1 vez em 1.015. Os mesmos R$ 42 em 7 apostas mínimas independentes pagam 1 em 329 — **três vezes mais frequente**. As sete combinações do desdobramento compartilham seis dezenas entre si, então acertam juntas ou erram juntas. Mesmo valor esperado, risco mais concentrado.

A única razão real para desdobrar é operacional: gastar mais de uma vez só, preenchendo um volante em vez de vários.

### Alguma sena já saiu repetida?

Na Mega-Sena, **nunca** — e isso não significa nada. São 50.063.860 combinações possíveis para 3.036 sorteios. O número esperado de repetições não é sorteios ÷ combinações, é **pares** de sorteios ÷ combinações: 3.036 × 3.035 ÷ 2 ÷ 50.063.860 = 0,09. Não achar nada era o resultado previsto.

Onde o universo é menor, repetições aparecem e também são normais:

| Modalidade | Sorteios | Combinações | Repetições | Esperado |
|---|---|---|---|---|
| Quina | 7.075 | 24.040.016 | 3 | 1,04 |
| Dupla-Sena | 5.974 | 15.890.700 | 2 | 1,12 |
| Lotofácil | 3.745 | 3.268.760 | 0 | 2,14 |
| Mega-Sena | 3.036 | 50.063.860 | 0 | 0,09 |

Todas as repetições da Quina e da Dupla-Sena foram conferidas uma a uma na API oficial da Caixa e são reais.

Para a Mega-Sena ter 50% de chance de exibir alguma repetição, seriam necessários cerca de 8.300 sorteios — mais de vinte anos no ritmo atual, e isso só para chegar ao "cara ou coroa".

### O erro de base que essa pergunta revelou

A varredura de repetições acusou algo impossível: na Dupla-Sena, os concursos 2009 e 2019 tinham **os dois sorteios idênticos**. A chance disso por acaso é da ordem de 1 em 30 milhões. Conferido na API oficial: o concurso 2019 é `08 19 29 39 40 41` e `03 08 30 33 37 42`, mas a nossa base guardava uma cópia do 2009. Registro corrigido.

É a segunda corrupção encontrada na Dupla-Sena — a primeira foi o concurso 2373, achado por outra varredura. Fica a lição de método: **a busca por coincidências impossíveis é um detector de erro de dados**, não só um exercício estatístico. Uma varredura de "registro inteiro idêntico a outro" passou a fazer parte da verificação, e hoje as nove modalidades passam limpas.

### Um erro de processo, para constar

Ao rodar a regressão desta rodada descobri que o script apontava para `painel_v9.html` desde a versão 10 — os comandos de substituição do caminho nunca casaram, e eu não conferi. As verificações que reportei para a v11 e a v12 rodaram na v9. Refeitas na v13: 198 combinações (9 modalidades × 11 abas × 2 temas), zero erro de console, zero estouro no iPhone. O script de verificação agora imprime qual arquivo está testando.

## 19. Bandeiras de exclusividade, e a intuição que aponta para o lado errado (v14)

Pedido do Hélio: uma bandeira em "Meu jogo" que busque combinações que nunca saíram, ou que mais saíram, e que fujam do comportamento do brasileiro. A pergunta veio acompanhada de "faz sentido ter isso?", e a resposta honesta é: **um terço faz muito sentido, um terço é inútil e um terço aponta para o lado contrário do que a intuição sugere.**

### O que é inútil: "nunca saiu"

Na Mega-Sena existem 50.063.860 combinações e só 3.036 já foram sorteadas. **99,9939% nunca saíram.** Um filtro que elimina seis milésimos de por cento do universo não escolhe nada — ele só produz a sensação de ter escolhido. O mesmo vale para todas as modalidades: a menor taxa é da Lotofácil, com 99,8854%.

### O que aponta para o lado errado: "as que mais saíram"

Buscar as dezenas mais sorteadas não muda a chance — ela é idêntica para qualquer combinação. Mas muda o rateio, e **para pior**, porque é exatamente o que faz a parcela da multidão que joga estatística.

Isso deu para medir, com a mesma máquina que mediu o efeito das datas: para cada concurso, quantas das dezenas sorteadas estavam no terço mais frequente **até o concurso anterior**, cruzado com o número de ganhadores da faixa baixa por real arrecadado.

O resultado bruto na Mega-Sena foi 1,15× mais ganhadores nos sorteios ricos em dezenas quentes. Mas parte disso era o efeito das datas vazando, já que as duas coisas se correlacionam. Controlando por faixa de quantidade de datas, o efeito cai para cerca de **1,07×**.

A medida limpa vem da Lotofácil, e ela é elegante: como todas as 25 dezenas são ≤ 31, **não existe efeito de data para confundir**. Lá o efeito das quentes é de **1,12×**, isolado.

Conclusão: existe gente jogando "números quentes", o efeito é real e mensurável, mas é de uma ordem de grandeza menor do que o de quem joga aniversário — 1,07 a 1,12 contra 1,96. A bandeira certa é para **evitar** as quentes, não para procurá-las.

### O que faz muito sentido: fugir de datas

Já era o achado central do projeto, e continua sendo o mais forte: sorteios com muitas dezenas ≤ 31 produziram **1,96×** mais ganhadores por real arrecadado na Mega-Sena e **1,45×** na Quina.

### O que foi construído

Um bloco de bandeiras em "Meu jogo", cada uma declarando a própria força:

| Bandeira | Status |
|---|---|
| Fugir de dezenas ≤ 31 | efeito medido: 1,96× na Mega-Sena, 1,45× na Quina |
| Evitar dezenas muito sorteadas | efeito medido: 1,07× na Mega-Sena controlada, 1,12× na Lotofácil isolada |
| Evitar combinações já sorteadas | sem efeito medido — raciocínio de que jogo premiado circula |
| Evitar sequências longas | sem efeito medido |
| Espalhar pelo volante | sem efeito medido |

E um botão, **Completar fugindo da multidão**, que preenche as dezenas que faltam respeitando as bandeiras ativas e **preservando o que o usuário já marcou**. A busca é por amostragem com rejeição, até 3.000 tentativas, devolvendo a melhor encontrada caso nenhuma passe em tudo — melhor entregar a menos ruim do que travar a interface.

Verificado sobre 32 gerações na Mega-Sena: média de 1,66 dezenas ≤ 31 contra 3,10 esperadas por acaso, nenhuma geração passando de 2, e sequência máxima de 2.

A separação entre "medido" e "raciocínio" fica na tela, ao lado de cada bandeira. É a mesma regra usada nos critérios do Super Sete: quando não há medição, o painel diz que não há.

## 20. A coluna que prova a igualdade (v15)

Pedido: mostrar, na aba Onde apostar, qual seria o índice CRIVO ao marcar uma dezena a mais em cada modalidade. A tabela ganhou duas colunas de índice, **mínima** e **+1 dezena**, e elas exibem exatamente o mesmo número.

A igualdade é a resposta, e a coluna existe para torná-la visível em vez de argumentável. Marcar n+1 dezenas equivale a comprar C(n+1, pick) apostas simples, e a Caixa paga cada combinação vencedora separadamente. Por linearidade da esperança — cada combinação do desdobramento é, ela própria, uma aposta simples válida — o retorno esperado total é exatamente C(n+1, pick) vezes o de uma aposta simples. O custo sobe pelo mesmo fator. A razão entre os dois não se move.

O que a coluna deixa visível é o preço embaixo de cada número:

| Modalidade | Aposta mínima | Índice | Com +1 | Índice |
|---|---|---|---|---|
| Lotofácil | R$ 3,50 | 0,632 | 16 dezenas — R$ 56,00 | 0,632 |
| Quina | R$ 3,00 | 0,469 | 6 dezenas — R$ 18,00 | 0,469 |
| Mega-Sena | R$ 6,00 | 0,376 | 7 dezenas — R$ 42,00 | 0,376 |
| Super Sete | R$ 3,00 | 0,331 | 8 dígitos — R$ 6,00 | 0,331 |
| Dia de Sorte | R$ 2,50 | 0,287 | 8 dezenas — R$ 20,00 | 0,287 |

Sete vezes o preço na Mega-Sena, o mesmo índice. A verificação foi feita recalculando o índice do desdobramento do zero — retorno esperado total dividido por custo total — e comparando com o da aposta mínima: iguais até a sexta casa decimal em todas as modalidades onde o desdobramento existe.

Lotomania e Timemania aparecem com travessão: nelas a aposta tem tamanho fixo, 50 e 10 dezenas, e não há o que desdobrar. O Super Sete tem tratamento próprio — "uma a mais" ali significa marcar dois dígitos em uma das sete colunas, o que dobra o custo pelo produto das escolhas, e o índice também não muda.

Esta seção e a 18 dizem a mesma coisa por dois caminhos diferentes, o que é proposital: uma pela aritmética do custo por unidade de chance, outra pela tabela lado a lado. **Não existe desconto por volume na sorte.**

## 21. Revisitando a metodologia: a falha que o índice escondia (v16)

O Hélio perguntou se não devíamos revisitar a metodologia. Fui auditar, e encontrei um erro estrutural — não de código, de modelagem — que estava lá desde a primeira versão do índice.

### A falha

O índice CRIVO usava o **prêmio de agora** com o **volume de apostas típico**. A mediana da arrecadação dos últimos 60 concursos entrava no cálculo de λ, e λ define o fator de partilha.

O problema é que o público responde ao prêmio. Quando a acumulação cresce, mais gente aposta, λ sobe, o fator de partilha desaba e o prêmio se divide entre mais gente. Ou seja: **o índice era mais otimista exatamente na situação que ele existe para avaliar.**

Pior: o "prêmio de gatilho" é, por definição, uma acumulação enorme. Ele era calculado segurando λ fixo, o que o tornava sistematicamente baixo demais.

Isso também explicava uma inconsistência que eu não tinha percebido: a varredura histórica da seção 16 usava o número de apostas **real** de cada sorteio, estimado pela faixa mais baixa. O índice ao vivo usava o volume típico. Os dois nunca falaram a mesma língua.

### Medindo, e evitando a armadilha da causalidade reversa

O primeiro instinto é regredir o volume contra o prêmio. Fiz isso, e o ajuste log-log dá expoentes de 0,18 a 0,44 — o volume cresce mais ou menos com a raiz quadrada do prêmio, com R² entre 0,23 e 0,56.

Mas esse número está inflado, e por um motivo que invalidaria a correção se eu ignorasse: **o prêmio cresce porque as pessoas apostam.** Parte da arrecadação de cada concurso alimenta o bolo. Correlação mecânica, não resposta do público.

O instrumento limpo é a **sequência de concursos acumulados** até o sorteio anterior. Ela é determinada inteiramente antes do sorteio atual, então não pode ser causada pelo volume de agora. Se o volume sobe com a sequência, a causalidade prêmio → público existe de fato.

Sobe, e de forma monótona:

| Concursos acumulados | Mega-Sena | Quina | Dia de Sorte |
|---|---|---|---|
| 0 (saiu no anterior) | 0,67× | 0,71× | 0,78× |
| 1 | 0,82× | 0,80× | 0,86× |
| 2 a 3 | 0,95× | 0,94× | 1,02× |
| 4 a 6 | 1,22× | 1,16× | 1,37× |
| 7 ou mais | 1,93× | 1,53× | 1,86× |

Valores relativos à mediana de cada modalidade. Na Lotofácil, onde a acumulação é rara, o salto é ainda mais rápido: 0,91× com prêmio saindo todo concurso, 1,94× com duas ou três acumulações.

### A correção

λ passou a ser multiplicado pela curva medida, indexada pela sequência de acumulação corrente. A curva é recalibrada nos próprios dados a cada carga do painel, então melhora sozinha conforme a base cresce.

O efeito nos números de hoje:

| Modalidade | Acumulados | Público | Índice antes | Índice agora |
|---|---|---|---|---|
| Lotofácil | 2 | 1,96× | 0,632 | **0,487** |
| Mega-Sena | 7 | 1,93× | 0,376 | **0,360** |
| Quina | 11 | 1,53× | 0,469 | **0,458** |
| Dupla-Sena | 9 | 1,24× | 0,221 | 0,220 |

A Lotofácil perde 22,9% do índice e o percentil histórico dela cai de 99,8 para 98,9. Continua na frente, mas a distância para a Quina encolheu de 0,163 para 0,029 — de folga confortável para empate técnico.

O prêmio de gatilho da Lotofácil subiu de R$ 17,6 milhões para R$ 31,7 milhões. Faz sentido: para o índice bater 1,00 seria preciso um prêmio que, ele próprio, atrairia uma multidão maior.

### Por que a Lotofácil é a mais castigada

Porque o λ dela já é alto. Com λ ≈ 2,8, o fator de partilha está no regime em que se comporta como 1/λ, então qualquer aumento de público bate proporcionalmente. Nas modalidades com λ bem abaixo de 1 — Mega-Sena, Quina, Lotomania — o fator de partilha está perto de 1 e mal se move.

É a tese do método aparecendo mais uma vez, agora contra nós mesmos: **o que decide não é o tamanho do prêmio, é quanta gente está disputando ele com você** — inclusive quando essa gente aparece *por causa* do prêmio.

### O que continua de pé, e o que continua frágil

De pé: o efeito das datas (1,96× medido), o efeito das quentes (1,12× medido na Lotofácil isolada), o fator de partilha como eixo central, o percentil histórico como veredito operacional, e a disciplina de separar medido de raciocínio.

Frágil, e declarado: o R² da elasticidade é baixo, entre 0,23 e 0,56 — a relação é ruidosa e a curva é uma mediana por faixa, não um modelo fino. A +Milionária não tem base suficiente para calibrar e fica com multiplicador 1. E o backtest do próprio método continua indistinguível do acaso na maioria das modalidades, o que é o resultado previsto e permanece o dado mais importante do projeto: **o CRIVO escolhe onde e quando gastar, não escolhe números que ganham.**

## 22. Matriz de decisão e o problema do número único (v17)

Três pedidos que se resolvem juntos: saber se apostar o mínimo ou uma dezena a mais faz diferença, ter indicadores separados em vez de um só, e ter uma matriz consolidada abrindo a aba Onde apostar.

### A resposta sobre a dezena a mais

Marcar uma dezena a mais **não melhora nenhum dos três eixos**, e piora um.

O índice não se move, porque custo e retorno esperado sobem pelo mesmo fator. A chance do prêmio principal **por real gasto** também não se move, pelo mesmo motivo. E a frequência de retorno piora: na Mega-Sena, um desdobramento de 7 dezenas leva algum prêmio 1 vez em 1.015, enquanto os mesmos R$ 42 em sete apostas mínimas separadas levam 1 em 329.

Para o mesmo dinheiro, desdobrar é **igual em dois eixos e pior no terceiro**. A vantagem real é de outra natureza: preencher um volante em vez de sete, e receber em várias faixas de uma vez quando acerta.

### Por que um número único escondia a decisão

O índice CRIVO soma duas coisas de naturezas diferentes: o troco das faixas pequenas, que é quase devolução do próprio dinheiro, e o prêmio principal, que é o motivo pelo qual quase todo mundo joga. Separando os dois, o ranking muda:

| Modalidade | Índice total | Troco | Prêmio grande | % que é prêmio |
|---|---|---|---|---|
| Lotofácil | 0,487 | 0,284 | 0,203 | 42% |
| Quina | 0,458 | 0,174 | **0,284** | 62% |
| Mega-Sena | 0,360 | 0,135 | 0,224 | 62% |

A Lotofácil lidera o índice total e cai para **terceira** no eixo do prêmio grande. Ela devolve muito nas faixas pequenas, o que é excelente para quem quer jogar muitas vezes com o mesmo dinheiro e irrelevante para quem quer o prêmio. A Mega-Sena é o contrário: 62% do índice dela é prêmio principal, com apenas 1 chance em 2.298 de levar qualquer coisa.

Nenhuma das duas é melhor. São apostas em coisas diferentes.

### A matriz

Abrindo a aba Onde apostar, uma tabela com as nove modalidades e quatro indicadores, cada um normalizado de 0 a 100 entre as modalidades da rodada:

- **Negócio** — retorno esperado por real
- **Prêmio grande** — parcela desse retorno que vem da faixa máxima
- **Frequência** — probabilidade de a aposta mínima levar qualquer prêmio
- **Raridade** — percentil do índice de hoje na história daquela modalidade

E um **índice final**, que é a média ponderada desses quatro.

### O aviso que acompanha o índice final

Os quatro indicadores são medidos. O **peso** de cada um é preferência, não medição. Por isso o painel oferece quatro perfis — prêmio grande, ver algo voltar, equilibrado, só o retorno por real — e mostra os pesos na tela.

Trocar de perfil troca o primeiro colocado, e isso é o comportamento correto:

| Perfil | 1º | 2º | 3º |
|---|---|---|---|
| Equilibrado | Lotofácil 90,0 | Quina 79,9 | Mega-Sena 50,5 |
| Quero o prêmio grande | **Quina 95,5** | Lotofácil 80,0 | Mega-Sena 67,8 |
| Quero ver algo voltar | Lotofácil 96,7 | Quina 62,4 | Super Sete 36,8 |
| Só o retorno por real | Lotofácil 100,0 | Quina 89,0 | Mega-Sena 52,3 |

**Um número final com pesos escondidos seria opinião disfarçada de resultado** — exatamente o que este painel existe para desmontar em quem vende método. Deixar os pesos visíveis e ajustáveis é o que separa uma ferramenta de decisão de um oráculo.

E nenhum perfil torna a aposta favorável: mesmo o primeiro colocado tem retorno esperado negativo.

### Tabelas ordenáveis

Toda tabela grande do painel passou a ordenar por clique no cabeçalho — um clique crescente, dois decrescente, três devolve à ordem original, que costuma ser o ranking escolhido de propósito. O leitor de valores entende "R$", "1 em", "%" e "×", além da pontuação brasileira.

## 23. Dois bugs que os meus testes não pegavam (v18)

### O primeiro: botões vivos que não respondiam

Os botões de perfil da matriz de decisão apareciam na tela, tinham aparência de botão, e não faziam nada. Nenhum erro no console.

A causa é uma armadilha clássica do DOM: `elemento.innerHTML += html` **reserializa e reconstrói todo o conteúdo do elemento**, o que descarta silenciosamente qualquer listener já ligado nos filhos. Na matriz de decisão os botões eram criados com `onclick` e a tabela era montada depois com `innerHTML +=` — apagando o clique deles sem deixar rastro.

A correção foi introduzir `htmlBloco(html)`, que devolve o HTML dentro de um contêiner próprio em vez de somar no pai. O comentário acima da função explica o motivo, porque este é o tipo de bug que volta.

**Mas o mais grave foi por que meus testes não pegaram.** Eu vinha testando a troca de perfil assim: `state.perfilAposta = 'premio'; await refresh()`. Isso exercita a lógica e nunca toca no botão. Testei o cálculo, não a interface.

Agora existe um teste que varre **todo botão de toda aba de todas as modalidades** e verifica se ele ainda tem um `onclick` ligado, mais um teste funcional que clica de verdade nos quatro perfis e confere que o primeiro colocado muda. Foi assim que a correção ficou provada: com "quero o prêmio grande" a Quina assume a primeira posição, exatamente como a decomposição previa.

### O segundo: o limite de conjuntos estava chutado

Na Lotofácil, a aba Conjuntos parava em 3 dezenas. O limite era uma constante escrita à mão por modalidade, e estava errado nos dois sentidos.

Medi o custo real no navegador:

| Modalidade | k=3 | k=4 | k=5 | k=6 |
|---|---|---|---|---|
| Lotofácil | 272ms | 1,0s | 3,0s | 6,4s |
| Lotomania | 1,2s | 17s | inviável | inviável |
| Timemania | 56ms | 79ms | 45ms | 18ms |
| Dia de Sorte | 9ms | 14ms | 9ms | 4ms |

O custo tem duas pernas, e é a segunda que decide: número de operações, que é C(dezenas sorteadas, k) × sorteios; e número de chaves distintas no mapa. A Lotomania com k=4 gera 3,9 milhões de chaves e por isso leva 17 segundos, enquanto k=3 gera 162 mil e leva 1,2 segundo. Já a Timemania sorteia só 7 dezenas, então k=6 custa 18 milissegundos — e estava travada em 3 sem motivo.

O limite passou a ser calculado, com orçamento de 15 milhões de operações e 1,5 milhão de chaves:

| Modalidade | Limite antes | Limite agora |
|---|---|---|
| Lotofácil | 3 | **5** |
| Lotomania | 2 | **3** |
| Timemania | 3 | **6** |
| Dia de Sorte | 4 | **6** |
| Dupla-Sena | 4 | **6** |
| +Milionária | 4 | **6** |
| Mega-Sena | 6 | 6 |

E o seletor avisa quando a opção é lenta: "5 dezenas · leva ~3s". Melhor informar do que travar sem explicação.

### A lição de processo

Os dois bugs têm a mesma raiz: eu testei o que era fácil testar. A lógica, por chamada direta; o desempenho, por suposição. Nenhum dos dois foi testado como o usuário encontra — clicando e esperando.

O terceiro caso da mesma família apareceu na v13, quando o script de regressão passou seis versões apontando para o arquivo errado. Três ocorrências é padrão, não azar.

---

## 24. Popularidade medida, não suposta (v19) — e a regra que estava com o sinal trocado

Até a v18 o "índice de exclusividade" era um conjunto de regras que eu escrevi por
raciocínio: penalizava sequências consecutivas, paridade desequilibrada, dígitos finais
repetidos, concentração no volante. Nenhuma delas tinha sido medida. Ao medir, uma
estava **invertida** — e era a de maior peso.

### Desenho

O mesmo experimento natural do efeito das datas. O sorteio é exógeno: ninguém escolhe o
que sai. Então, comparando concursos que tiveram o padrão X com os que não tiveram,
quantos ganhadores da faixa máxima apareceram **por combinação jogada**, isolamos a
preferência da multidão.

O volume de combinações jogadas é estimado pela **faixa mais baixa de prêmio** (milhares
de ganhadores por concurso), não pela arrecadação: o preço da aposta mudou várias vezes
ao longo da série e contaminaria a comparação. Como controle, refiz tudo com a
arrecadação restrita aos 900 concursos mais recentes — mesma direção, magnitude parecida.

**Viés conhecido, e ele é conservador.** A faixa baixa também sobe quando o sorteio é
popular. Isso normaliza parte do efeito para dentro do denominador, então as razões
abaixo são **piso**, não teto.

### Resultado — Lotofácil (3.411 concursos)

| padrão no sorteio | ganhadores vs. esperado | IC95 |
|---|---|---|
| maior sequência ≤ 3 | **2,21×** | 1,97 – 2,53 |
| sequência 4 | 0,95× | — |
| sequência 5 | 0,83× | — |
| sequência ≥ 6 | 0,80× | 0,76 – 0,87 |
| soma 186–210 | 1,15× | — |
| soma > 210 | 0,79× | 0,74 – 0,85 |
| alguma linha inteira do volante | 0,72× | 0,66 – 0,78 |
| alguma coluna inteira do volante | 1,21× | 1,10 – 1,35 |
| paridade 7/8 pares | 0,96× | 0,89 – 1,05 — **sem efeito** |

**Leitura: a multidão foge de dezenas coladas.** Quem joga sequência longa divide com
menos gente. A v18 penalizava exatamente isso.

### Resultado — Mega-Sena (3.036 concursos)

| padrão | razão | IC95 |
|---|---|---|
| todas as 6 ≤ 31 (datas) | 2,05× | 1,40 – 2,77 |
| nenhuma consecutiva | 1,41× | 1,18 – 1,70 |
| 2+ consecutivas | 0,71× | 0,58 – 0,85 |
| 3+ mesmo dígito final | 0,75× | 0,54 – 0,98 (fraco) |
| paridade 3/3 | sem efeito | 0,98 – 1,61 |
| soma no miolo | sem efeito | 0,78 – 1,19 |

O efeito das consecutivas aparece **de forma independente** em duas modalidades.

### Resultado — Quina (2.991 concursos)

Nada. Coladas 0,99× vs 1,02×; dígito final e soma sem efeito. Medido e nulo — a Quina
fica sem regras, e o painel diz isso em vez de inventar.

### Teto de honestidade

Os fatores foram medidos um a um. Multiplicá-los supõe independência. Testei **uma**
célula conjunta — sequência ≥6 **e** soma >210 — e ela mediu 0,72× enquanto o produto
previa 0,64×: o modelo multiplicativo **exagera**. Por isso o painel aplica um piso de
**0,70×**, a melhor célula conjunta efetivamente medida, e mostra na tela quando o piso
entrou em ação.

### Desdobramento dilui a vantagem

Num jogo com dezenas extras, quem ganha é **um** dos subconjuntos de `pick` dezenas, e
cada um tem perfil próprio. O fator honesto é a média sobre eles. Exemplo real: um
conjunto de 16 dezenas com sequência de 6 e soma 223 tem fator 0,70× **como conjunto**,
mas seus 16 desdobramentos variam de 0,70× a 2,20× e a média é **1,02×** — a vantagem
some. Com o mesmo dinheiro, 16 jogos de 15 escolhidos um a um seguram o fator em 0,70×
e ainda premiam com muito mais frequência (83% dos concursos contra 22%), porque não
erram todos juntos.

### 24.1 Dois defeitos que a medição expôs no gerador

**O gerador não gerava.** O filtro "evitar 3+ dezenas consecutivas" era rejeição pura:
sorteia, testa, descarta. Num jogo de 16 dezenas em 25, quase nenhum sorteio tem
sequência máxima ≤ 2 — a rejeição estourava o limite de 900 tentativas e o jogo era
descartado **em silêncio**. Pedir 1 jogo devolvia 0; pedir 3 devolvia 1. Não havia
erro no console, e nenhum teste meu contava os jogos gerados contra os pedidos.

Conserto: nada de filtro por palpite. O gerador junta até 300 candidatos livres e
escolhe o de menor **fator de rateio medido**; se por qualquer motivo não houver
candidato, sorteia um sem filtro e entrega assim mesmo. O rodapé agora mostra
"N de M jogos gerados" — se faltar, aparece.

**O filtro estava do lado errado.** Ver 24: sequências coladas são anti-multidão,
não pró. O gerador da v18 empurrava para o perfil mais disputado, e o rótulo
"POUCO DISPUTADO" confirmava a escolha errada. O rótulo agora carrega o fator
medido, e diz "SEM MEDIÇÃO" onde não há medida.

**Abas "Meu jogo" e "Gerador" fundidas.** Eram a mesma tarefa em dois lugares: quem
gerava um jogo tinha de redigitar as dezenas para ver a análise. Agora é uma aba com
dois modos e um botão "Analisar este jogo" em cada jogo gerado.

### 24.2 Menu de uma opção só

Em "Meus números favoritos", o seletor "Dezenas por jogo" vinha de um intervalo que eu
inventei (`pick` até `pick+4`, com teto arbitrário de 15). Na Lotofácil, cujo `pick` já
é 15, isso colapsava para **uma única opção** — um menu suspenso que não oferece escolha
nenhuma. Agora os tamanhos vêm dos que a Caixa realmente aceita (Lotofácil 15 a 20,
Mega-Sena 6 a 20), com o preço de cada um no rótulo; onde só existe um tamanho
(Lotomania, Timemania), aparece texto em vez de menu. O card de custo também estava
errado para tamanhos acima do mínimo: multiplicava pelo preço da aposta simples em vez
do preço do jogo daquele tamanho.

O ranking desse painel passou a ser ordenado pelo **fator de rateio medido**, e o título
diz que é o único dos três que importa — os de desempenho histórico ficam como
contraprova de que a diferença entre eles é ruído.

### 24.3 O ordenador de tabelas ignorava colunas em "1 em N"

`numDaCelula` extraía o primeiro número da célula. Numa coluna escrita como razão —
"1 em 9", "1 em 2.298" — ele devolvia sempre **1**, então a ordenação era estável e nada
se movia: parecia que o clique no cabeçalho não funcionava. Agora "1 em N" é lido pelo
denominador, e menor = melhor. Células sem dado ("—") vão para o fim nos dois sentidos,
em vez de serem tratadas como o menor valor possível.

O defeito valia para **todas** as tabelas ordenáveis do painel, não só a dos três eixos.

### 24.4 Primeiro resultado real registrado — concurso 3746

Sorteio de 27/07/2026: `01 03 04 06 07 08 09 10 14 15 17 18 21 22 24`. Acumulou.

O sorteio caiu num perfil que o método classifica como **pouco disputado**: maior
sequência 5 (fator 0,83×) e soma 179, abaixo de 185 (0,90×). A previsão, portanto, era
de *menos* ganhadores que o esperado. Com R$ 43.366.662,50 arrecadados, ou 12.390.475
combinações de 15 a R$ 3,50, o esperado por faixa e o observado foram:

| faixa | esperado | observado | obs/esp |
|---|---|---|---|
| 11 acertos | 1.086.551 | 977.773 | 0,90 |
| 12 acertos | 206.963 | 181.576 | 0,88 |
| 13 acertos | 17.910 | 15.510 | 0,87 |
| 14 acertos | 569 | 464 | 0,82 |
| 15 acertos | 3,79 | **0** | — |

O gradiente é monótono: **quanto mais parecida com o sorteio, mais a faixa fica abaixo
do esperado.** É a assinatura exata de um sorteio impopular — a preferência da multidão
morde mais forte justamente onde a aposta se aproxima do resultado.

**Isto é um concurso, não uma validação.** A direção bate com o efeito medido na seção
24, mas um único sorteio não distingue efeito de ruído, e o zero na faixa de 15 tem
probabilidade nada extraordinária (~2% sob Poisson, mais que isso com superdispersão).
Registrado aqui porque é o primeiro ponto fora da amostra que gerou os coeficientes —
e porque um registro que só guardasse os concursos favoráveis não valeria nada.

### 24.5 O histórico de apostas sumia ao trocar de versão

O registro mora no `localStorage`. Em páginas abertas por `file://`, o Chrome mantém um
baú **por arquivo**: `painel-loterias-offline_7.html` e `_8.html` não enxergam o mesmo
armazenamento. Baixar a versão nova parecia apagar o histórico; ele continuava preso no
arquivo anterior. Três defesas, nesta ordem:

1. **Semente versionada** em `site/data/apostas.json`, embutida pelo `bundle.py` como
   qualquer outro dado — entra automaticamente em todo arquivo novo.
2. **Exportar / importar JSON**, para o histórico ser um arquivo do usuário.
3. **Mesclagem por chave estável** (`jogo|concurso|dezenas`), nunca sobrescrita: o que
   já está no navegador tem prioridade sobre a semente. A versão anterior deduplicava
   por `id`, que é regerado — importar duas vezes duplicava tudo.

A semente **não** traz a sombra aleatória pronta. A sombra nasce no instante do registro
e se perde junto com o armazenamento; inventar números seria fabricar dado. Ela é
regerada de forma determinística a partir da chave da aposta e marcada como regerada.

Junto saiu outro defeito silencioso: o placar filtrava por `DB[x.jogo]` e ignorava, sem
avisar, as apostas de modalidades ainda não carregadas — o total apostado saía menor que
o real. Agora a aba carrega toda modalidade presente no histórico, e avisa se alguma
ficar de fora.

### 24.6 Repetir a mesma aposta — e por que o desdobramento do Hélio é melhor que o nosso

Repetir dezenas ou trocá-las **não muda nada** na chance: o sorteio não tem memória, e
toda combinação de 15 vale 1 em 3.268.760 sempre. A pergunta só tem conteúdo no eixo do
rateio, e aí ela tem uma resposta clara.

O jogo registrado — `01 02 03 04 05 10 11 14 16 19 20 21 22 23 24 25` — mede **0,70×
em todos os 16 desdobramentos**: média 0,70, mínimo 0,70, máximo 0,70. O conjunto que
eu tinha proposto (`02 04 06 07 08 10 13 14 15 16 17 18 21 23 24 25`) mede 0,70× como
conjunto, mas seus desdobramentos variam de 0,70× a **2,20×**, com média 1,02×.

A causa é estrutural. O jogo do Hélio tem **duas linhas inteiras do volante** (01–05 e
21–25, fator medido 0,72×) e duas sequências longas separadas por um miolo esparso.
Tirar qualquer uma das 16 dezenas não consegue derrubar a maior sequência abaixo de 4
nem quebrar as duas linhas ao mesmo tempo. O meu tinha um único bloco de seis (13–18):
tirar o 15 ou o 16 parte esse bloco em 13-14 e 17-18, a maior sequência cai para 3 e o
subconjunto despenca no perfil mais jogado do país.

**A regra geral que sai daqui: um desdobramento vale o que valem os seus PIORES
subconjuntos, não o conjunto inteiro.** O que se procura não é um conjunto bom, é um
conjunto cujo subconjunto mais fraco ainda seja bom — e isso favorece estruturas
redundantes, com mais de um padrão anti-multidão, para que nenhuma remoção isolada
destrua todos.

Efeito no índice para o concurso 3747 (prêmio estimado R$ 15.000.000): 0,625 no perfil
médio contra **0,747** no perfil 0,70× — R$ 41,83 de retorno esperado sobre R$ 56 em vez
de R$ 35,01.

Os botões `↻` e `↻+` no histórico repetem uma aposta e procuram dezenas de perfil melhor.
O `↻+` **só troca se conseguir melhora estrita**; empate não é melhora, e trocar por um
fator idêntico jogaria fora a estrutura escolhida sem ganho nenhum. Em modalidade sem
medição ele se recusa a trocar e diz por quê.

### 24.7 Diário fora da amostra — os dois primeiros concursos cegos

Os coeficientes da seção 24 foram medidos numa base que terminava na Lotofácil 3745 e na
Mega-Sena 3036. Tudo depois disso é teste cego: o número já estava escrito quando a bola
saiu. O painel agora mantém esse diário sozinho, na aba Evidências — recalcula a cada
atualização da base, e ninguém escolhe quais concursos entram.

**Lotofácil 3746** (27/07), `01 03 04 06 07 08 09 10 14 15 17 18 21 22 24`: maior
sequência 5, soma 179, linha 06–10 inteira. Fator previsto **0,54×** — pouco disputado.

**Lotofácil 3747** (28/07), `01 02 04 05 06 08 09 11 12 13 18 19 21 24 25`: maior
sequência 3, soma 178. Fator previsto **1,72×** — muito disputado.

| faixa | 3746 (previsto 0,54×) | 3747 (previsto 1,72×) |
|---|---|---|
| 11 acertos | 0,90 | 0,99 |
| 12 acertos | 0,88 | 1,03 |
| 13 acertos | 0,87 | 1,04 |
| 14 acertos | 0,82 | 1,04 |
| 15 acertos | 0,00 (0 de 3,79 esperados) | **1,25** (6 de 4,81) |

Dois sorteios seguidos, perfis opostos, **gradientes opostos e monótonos**: no impopular
a razão cai conforme a faixa sobe; no popular, sobe. É a forma que a teoria prevê —
a preferência da multidão morde mais forte justamente onde a aposta se aproxima do
resultado — e não é uma forma que ruído produza com facilidade.

**Mega-Sena 3037** (28/07), `02 11 22 30 51 54`: nenhuma consecutiva, fator previsto
**1,41×**. Observado 0,80 na faixa de 4 acertos e 0,59 na de 5. **Errou.** Fica
registrado com o mesmo destaque dos acertos, porque é para isso que o diário existe.

**Placar: 2 de 3.** Isso não é evidência de nada. Com três concursos, acertar dois por
acaso tem probabilidade 3/8. O valor do quadro hoje é existir, crescer sozinho e
registrar os erros — um teste de sinal só começa a dizer algo perto de vinte concursos.

Detalhe de honestidade no código: quando nenhum padrão medido aparece no sorteio, o
fator fica colado em 1,00 e **não há previsão**. Esses concursos entram na tabela
marcados e ficam fora da conta; contar fator 1,00 como acerto ou erro seria fraude de
placar nos dois sentidos. A Quina, medida e nula em tudo, não tem diário — sem
coeficiente não há o que testar.

### 24.8 O histórico em gráfico, e a escala que mentiria

"Minhas apostas" saiu da fila de abas e virou um chip ao lado de "Onde apostar". O
motivo não é estético: as abas trocam de conteúdo conforme a modalidade selecionada, e
o registro não é uma visão *de* uma modalidade — é a carteira inteira, atravessando as
nove. Ficar entre elas dava a impressão errada de que o histórico era por jogo.

O painel novo tem dois gráficos, pela mesma disciplina da aba "Onde apostar": perguntas
diferentes, gráficos diferentes.

**Por modalidade** responde "onde meu dinheiro foi e o que voltou" — comparação entre
categorias, barra horizontal, escala começando em zero, valor escrito em cada barra.

**No tempo** responde "como estou indo" — série acumulada, com a sombra aleatória
tracejada por cima, para a comparação ser visual em vez de aritmética.

**O problema de escala que aparece assim que houver um prêmio.** Um prêmio de faixa alta
é milhares de vezes o custo de uma aposta. No teste com um prêmio de R$ 2,1 milhão contra
R$ 59,50 gastos, o eixo compartilhado achatou **todas** as barras de gasto a menos de um
pixel: a comparação entre modalidades, que é a razão de o gráfico existir, desapareceu.

A saída não é escala logarítmica — em dinheiro ela engana quem lê rápido. O gráfico
detecta a situação (prêmio máximo acima de 4× o gasto máximo), dá **um eixo para cada
série** e diz na própria figura que fez isso, com a razão entre os dois máximos escrita.
Enquanto os valores forem comparáveis, o eixo continua sendo um só, que é a leitura mais
honesta. A comparação entre gasto e prêmio fica na coluna Saldo da tabela, onde é
aritmética e não visual.

Detalhe menor, mesmo princípio: o rótulo na ponta da linha do gráfico no tempo estourava
o `viewBox` quando o valor era grande. Virou legenda no topo, que não depende do
comprimento do texto.

### 24.9 Diário fora da amostra — cinco concursos de Lotofácil

Base atualizada até 31/07/2026. O diário da Lotofácil agora tem cinco entradas, todas
posteriores ao corte da medição (3745):

| concurso | perfil do sorteio | previsto | mediana observada | direção |
|---|---|---|---|---|
| 3746 · 27/07 | seq 5, soma 179, linha cheia | 0,54× | 0,866 | ✓ menos |
| 3747 · 28/07 | seq 3, soma 178 | 1,72× | 1,035 | ✓ mais |
| 3748 · 29/07 | seq 3, soma miolo | 2,20× | 1,179 | ✓ mais |
| 3749 · 30/07 | seq 3, soma miolo | 2,20× | 1,169 | ✓ mais |
| 3750 · 31/07 | seq 3, soma miolo | 2,20× | 1,014 | ✓ mais |

**Cinco de cinco.** Sob uma moeda honesta isso sai com probabilidade 1/32, ou 3,1% —
baixo o bastante para ser interessante, alto o bastante para não ser conclusão. E há
três ressalvas que precisam vir junto:

Primeiro, **quatro dos cinco sorteios caíram no mesmo perfil** (sequência curta, soma no
miolo). Não são cinco testes independentes da regra inteira; são um teste repetido da
mesma célula, mais um da célula oposta. A Lotofácil vinha sorteando conjuntos espalhados
quatro dias seguidos, o que é só acaso.

Segundo, **a faixa de 15 acertos não acompanhou** em 3749 (0,59) e 3750 (0,82), enquanto
as faixas de 11 a 14 ficaram todas acima de 1. É a faixa mais ruidosa — meia dúzia de
ganhadores esperados, um a mais ou a menos vira 20% de diferença. Por isso o placar usa
a **mediana** das faixas e não a faixa máxima.

Terceiro, **a magnitude não bate**. Previsto 2,20×, observado perto de 1,15×. A direção
acerta, o tamanho não — coerente com o viés conservador declarado na seção 24, mas
também compatível com um coeficiente simplesmente exagerado. Só mais concursos separam
as duas explicações.

**Mega-Sena continua em 0 de 1.** O 3038 (30/07) não acionou nenhuma regra medida —
fator 1,00, **sem previsão** — e entrou marcado, fora da conta, como manda a regra.

### 24.10 A varredura histórica virou script (v19.6)

Até aqui, o `historico_crivo.json` tinha sido gerado uma vez, num ambiente que não
existe mais, e a seção 16 descrevia o procedimento em prosa. Isso significa que a
varredura **não era reproduzível** — e uma varredura que ninguém consegue rodar de novo
é uma afirmação, não uma medição. Agora ela é `gerar_historico_crivo.py`, no repositório.

A reescrita foi validada contra o arquivo antigo antes de substituí-lo, e reproduz todos
os números: Mega-Sena 435 sorteios e máximo 1,1549 (contra 1,1548 guardado — diferença de
arredondamento na quarta casa), Lotofácil máximo 0,6930, Quina 1,0219, Lotomania 1,3553,
Timemania 1,0135, Dia de Sorte 0,8316, Dupla-Sena 0,7130, Super Sete 0,5892. As contagens
subiram só onde a base cresceu.

Com a base até 31/07/2026: **4.642 sorteios com prêmio principal observado, 5 cruzaram
1,00.** Quatro deles em sorteios especiais.

**A aferição do preço reconstruído ganhou uma janela por data**, e o caminho até lá vale
registrar porque errei duas vezes:

- *"Últimos 60 sorteios com ganhador"* — no Super Sete isso pega cinco anos, porque ele
  só tem 33 sorteios com ganhador na base inteira, e mistura a época em que a aposta
  custava R$ 2,50. Acusava 12% de erro sem nada estar errado.
- *"De 2025 em diante"* — na Lotofácil isso pega 367 sorteios e atravessa o último
  aumento de preço, empurrando o estimado para R$ 3,32 contra R$ 3,50.
- *Doze meses corridos* — resolve os dois. Mega-Sena 1,1% de erro, Lotofácil 0,7%,
  Dia de Sorte 0,3%. O Super Sete fica com quatro pontos e é marcado como **amostra
  pequena** em vez de FORA, porque com n=4 a mediana não sustenta veredito.

E o `_meta` do arquivo passou a ser **calculado**. Antes trazia "4.635 sorteios" escrito
à mão, número que o painel exibe em texto corrido — ele teria envelhecido em silêncio a
cada coleta, e ninguém perceberia.

### 24.11 Bolão, e as três coisas que ele quebra no modelo (v19.7)

Até aqui uma aposta tinha **um** conjunto de dezenas, o custo registrado era o preço do
bilhete e o prêmio conferido era o prêmio inteiro. Bolão quebra os três de uma vez:

1. são **vários jogos no mesmo bilhete**, e eles não são apostas independentes — quem
   escolheu foi quem montou;
2. o que você paga (**a cota**) não é o que o bilhete custa;
3. o prêmio que chega até você é uma **fração** do que o bilhete ganhou.

Modelar bolão como "várias apostas simples separadas" erraria nos três: perderia o
vínculo entre os jogos, inflaria o gasto registrado em até doze vezes e pagaria o prêmio
integral a quem tem uma cota. Por isso a aposta passou a ter uma **lista** de jogos, e o
bolão carrega `valorCota`, `totalCotas` e `cotas`. `custo` é o seu desembolso,
`custoTotal` é o bilhete inteiro, e a participação é `cotas ÷ totalCotas`.

A **sombra aleatória** de um bolão tem o mesmo formato: um jogo sorteado para cada jogo
do bilhete, do mesmo tamanho. Uma sombra de um jogo só compararia coisas diferentes e
favoreceria o bolão de graça.

**A conferência que ninguém faz na lotérica.** O painel multiplica cotas × valor da cota
e compara com o custo real do bilhete. Se sobrar, a diferença é taxa de serviço — pode
ser perfeitamente combinada, mas sai do seu bolso e agora aparece escrita, com o
percentual. Nos dois bolões registrados em 01/08/2026 a conta fechou exata: 12 × R$ 14,00
= R$ 168,00 = 4 jogos de 7 dezenas; 10 × R$ 12,60 = R$ 126,00 = 3 jogos de 7 dezenas.
Sem taxa embutida.

**Um lado da medição estava faltando.** Ao analisar esses bolões apareceu que a regra das
consecutivas da Mega-Sena só codificava o lado popular: "nenhuma colada = 1,41×". O jogo
COM coladas caía no neutro 1,00, quando a medição diz **0,71×**. O painel escondia metade
do efeito — justamente a metade que favorece quem joga colado. Agora a regra é uma faixa
com os dois lados, como já era na Lotofácil.

O botão de trocar dezenas (`↻+`) **se recusa a agir num bolão**: as dezenas foram
escolhidas por quem montou o bilhete, e trocá-las produziria um registro que não
corresponde ao que foi comprado.

### 24.12 Diário fora da amostra — a Lotofácil errou pela primeira vez

Base até 02/08/2026. O diário passou a ter nove entradas.

**Lotofácil 3751** (02/08), `01 02 04 07 08 09 11 12 16 17 18 19 21 23 24`: maior
sequência 4, soma 192. Fator previsto **1,09×** — levemente disputado. Mediana observada
**0,962**. **Errou.** Placar da Lotofácil: **5 de 6**.

Vale reparar em como o erro é: o fator previsto 1,09 é o mais próximo de 1,00 de toda a
série, e a mediana observada 0,962 também. O modelo e a realidade discordaram do lado de
uma linha que quase não separa nada. Um teste de sinal trata isso como erro cheio, o que
está correto para não inflar o placar — mas quem lê a tabela deve ver que este ponto pesa
menos que os extremos.

**Mega-Sena 3038 e 3039.** O 3038 mudou de "sem previsão" para **acerto** — não porque o
sorteio mudou, mas porque a correção da v19.7 deu ao lado "tem coladas" o fator medido
0,71× em vez do neutro 1,00. O sorteio tinha 38 e 39 coladas, previa menos ganhadores, e
saiu 0,536. **É um acerto que estava escondido pela metade faltante da medição.**

O 3039 (02/08), `14 16 21 39 53 58`, sem nenhuma consecutiva, previa 1,41× — e saiu
0,804. **Errou.** Mega-Sena: **1 de 3**.

Somando as duas modalidades: **6 de 9**. Continua sem valor estatístico — 6 de 9 por
acaso tem probabilidade 25% — e continua registrando os erros com o mesmo destaque.

### 24.13 A coluna que faltava na matriz de decisão (v19.10)

A matriz decide onde apostar e não mostrava **o prêmio**. É o número que a pessoa carrega
na cabeça quando entra na lotérica, e ele estava fora justamente da tabela feita para
decidir. Agora fica ao lado da modalidade — antes das quatro notas, porque é fato e não
avaliação — com a data do próximo sorteio embaixo e o maior prêmio da rodada destacado.
A coluna ordena como as outras.

Ela também funciona como contraprova visual da tese do método. Quando a primeira colocada
**não** é a de maior prêmio, o painel escreve isso na nota e diz de quem é o maior prêmio.
Quando as duas coincidem — como em 03/08/2026, com a Mega-Sena em R$ 135 milhões — ele
diz que coincidiu, e que **isso acontece às vezes e não é a regra**. Um painel que só
comentasse o caso favorável à própria tese estaria fazendo seleção de evidência na cara
do usuário.

**+Milionária 377 entrou.** A lacuna declarada na v19.8 durou um dia: os trevos `4` e `6`
apareceram no mirror e o concurso foi incorporado. As nove modalidades estão fechadas até
02/08/2026, sem concurso faltando.

## 25. Um bolão premiado expôs um erro que pagava um sexto do prêmio (v19.11)

O Hélio mandou o comprovante de um bolão da Mega-Sena 3042: 10 jogos de 8 dezenas,
bilhete de R$ 1.680, 80 cotas de R$ 21,00, uma cota comprada. O sorteio saiu
`02 05 10 35 40 53` e o **jogo 6** — `05 06 22 35 37 40 53 59` — fez **4 acertos**.

Ao conferir, o painel pagou **uma** quadra. O certo são **seis**.

### A conta que estava errada

Um jogo de n dezenas contém C(n, pick) apostas simples, e elas caem em faixas
**diferentes** ao mesmo tempo. O código fazia:

```js
bilhetes = C(acertos, pick)      // errado
```

Isso só funciona quando `acertos = pick`. Para 4 acertos numa aposta de 6, dá
`C(4,6) = 0`, o `Math.max(1, …)` transformava em 1, e o painel pagava uma quadra
solitária. A conta correta é combinatória elementar: para **h** acertos em **n**
marcadas, o número de combinações de `pick` com **exatamente j** acertos é

```
C(h, j) × C(n − h, pick − j)
```

somado sobre todas as faixas premiadas. No caso do jogo 6: `C(4,4) × C(4,2) = 6`
quadras, R$ 706,56 cada, **R$ 4.239,36** no bilhete em vez de R$ 706,56. O painel
pagava um sexto.

O erro cresce com o tamanho do jogo. Um jogo de 8 dezenas com 5 acertos leva
`C(5,5)×C(3,1) = 3` quinas **e** `C(5,4)×C(3,2) = 15` quadras — o código antigo pagava
uma quina e ignorava as quinze quadras.

**Por que ninguém tinha visto.** Todos os registros anteriores eram apostas simples ou
jogos com dezenas extras que não premiaram. O caminho do bug só é percorrido quando um
jogo grande ganha, e isso levou três semanas para acontecer. Testar o caso premiado
exigia inventar um resultado — foi o primeiro prêmio real que expôs a falha.

### A tarifa de serviço, que também estava fora da conta

O comprovante traz três valores: cota R$ 21,00, **tarifa de serviço R$ 7,35**, total
R$ 28,35. A tarifa é exatamente 35% da cota — o teto que a Caixa permite — e **não compra
combinação nenhuma**. Ela é 25,9% de tudo que o apostador desembolsou, e reduz o retorno
esperado na mesma proporção.

O modelo de bolão da v19.7 não tinha esse campo: o custo registrado seria R$ 21,00 e o
painel diria que não havia taxa embutida, porque 80 × 21 fecha com o bilhete. E fecha
mesmo — a tarifa é cobrada **por fora**, e é justamente por isso que passava despercebida.
Agora há um campo próprio, o custo registrado é o desembolso real, e a prévia escreve
quanto por cento do seu dinheiro nunca chega às urnas.

### O resultado

Bilhete: R$ 4.239,36. Cota de 1/80: **R$ 52,99**. Custo: R$ 28,35. **Saldo +R$ 24,64.**
A sombra aleatória do mesmo bolão fez no máximo 3 acertos e não levou nada — a primeira
vez em cinco apostas registradas que as duas carteiras se separam.

**Isso não valida nada.** Cinco apostas é ruído puro, e um prêmio de faixa baixa numa
carteira pequena vira o placar sozinho — exatamente o que o próprio painel avisa na caixa
acima do quadro. O que vale registrar é o conserto, não a quadra.
