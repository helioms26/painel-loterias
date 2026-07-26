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

## 7. Aviso

Aposta de loteria não é investimento nem fonte de renda. O retorno esperado é estruturalmente negativo por lei. Este projeto serve para tomar decisões informadas sobre um gasto de entretenimento, e para não ser enganado por quem vende método. Se o jogo deixar de ser diversão, procure o programa Jogo Responsável da Caixa.
