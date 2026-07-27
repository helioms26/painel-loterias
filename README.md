# Painel Loterias — projeto 13

Dashboard estatístico das 9 loterias da Caixa, com base oficial completa e metodologia própria (Método CRIVO).

## Como usar

**No celular ou em qualquer navegador — link público:**

### https://helioms26.github.io/painel-loterias/

**No computador, sem internet:** abra `painel-loterias-offline.html`. Todos os dados estão embutidos; funciona com dois cliques.

> A versão da pasta `dashboard/` **não funciona** abrindo o `index.html` direto do disco (`file://`) — o Chrome bloqueia a leitura dos arquivos de dados. Para uso local, use o arquivo único.

## Estrutura

```
13 - loteria/
├── painel-loterias-offline.html   ← arquivo único, abre com dois cliques
├── PAINEL LOTERIAS (abrir online).url
├── METODOLOGIA.md                 ← método CRIVO, testes e resultados
├── DESIGN-SYSTEM.md               ← tokens, componentes, princípios visuais
├── LEIA-ME.txt                    ← passo a passo de atualização
├── README.md
├── planilhas_caixa/               ← as 9 planilhas oficiais baixadas da Caixa
└── dashboard/
    ├── index.html                 ← versão para hospedar
    ├── tokens.css                 ← camada de tokens do design system
    ├── components.css             ← camada de componentes
    ├── styleguide.html            ← style guide navegável
    ├── atualizar_tudo.ps1         ← coleta + reparo + verificação + backup
    ├── data/                      ← base de dados
    └── data_backup/               ← backup único, refeito só após verificação
```

## Base de dados

Histórico oficial completo. As dezenas e datas vêm da API `servicebus2.caixa.gov.br/portaldeloterias`; os prêmios, ganhadores e arrecadação vêm das **nove planilhas oficiais** baixadas de `loterias.caixa.gov.br`, que cobrem todas as faixas desde o concurso 1.

| Jogo | Concursos | Último |
|---|---|---|
| Mega-Sena | 3.036 | 26/07/2026 |
| Lotofácil | 3.745 | 26/07/2026 |
| Quina | 7.075 | 26/07/2026 |
| Lotomania | 2.954 | 24/07/2026 |
| Dupla-Sena | 2.987 | 24/07/2026 |
| Timemania | 2.420 | 26/07/2026 |
| Dia de Sorte | 1.255 | 26/07/2026 |
| Super Sete | 877 | 24/07/2026 |
| +Milionária | 375 | 26/07/2026 |

**Preços oficiais conferidos em 26/07/2026:** Mega-Sena R$ 6,00 · Lotofácil R$ 3,50 · Quina R$ 3,00 · Lotomania R$ 3,00 · Dupla-Sena R$ 3,00 · Timemania R$ 3,50 · Dia de Sorte R$ 2,50 · Super Sete R$ 3,00 · +Milionária R$ 6,00. Duas correções vieram daí: Super Sete estava R$ 2,50 e Lotomania R$ 3,50 na configuração antiga.

**Horários oficiais** (cronograma da Caixa, agosto/2026): segunda a sexta às 21h, domingo às 11h, sem sorteio aos sábados. Apostas encerram uma hora antes. Como o campo `dataProximoConcurso` da API fica desatualizado, o painel deduz o calendário real pelos dias da semana dos concursos recentes.

**Integridade verificada.** 200 concursos da Lotofácil sorteados ao acaso foram reconferidos contra a API oficial: dos 118 que responderam, 118 conferiram exatamente. Uma varredura completa dos 25.718 concursos encontrou **um erro real** — o concurso 2373 da Dupla-Sena vinha do espelho comunitário com a dezena 34 duplicada e a 43 faltando. Foi corrigido pela API oficial. Hoje a base tem zero buracos, zero dezenas duplicadas, zero fora de faixa e zero concursos sem data.

## Para atualizar a base

Um comando só, no PowerShell dentro da pasta `dashboard`:

```powershell
.\atualizar_tudo.ps1
```

Ele executa nesta ordem: baixa os concursos novos dos 9 jogos → repara automaticamente qualquer concurso inconsistente → **verifica** a base inteira → **só então** refaz o backup único em `data_backup/` → regera o `bundle.json`.

Se a verificação falhar, o backup anterior é preservado e o log avisa. Log em `dashboard/data/atualizacao_log.txt`.

O painel online também tem o botão **Atualizar estatísticas**, que busca os concursos novos direto da API da Caixa, guarda no próprio navegador e roda a verificação de integridade ao final.

Uma tarefa agendada roda a rotina toda segunda-feira às 11h e envia o relatório com o índice CRIVO por modalidade.

## Backup

`dashboard/data_backup/` guarda **um único backup**, sobrescrito a cada atualização que passa na verificação. Para restaurar, copie tudo de `data_backup/` para `data/`.

## O que o painel entrega

- **Visão geral** — último concurso, próximo sorteio com data, horário e contagem regressiva para encerrar as apostas, situação de acumulação, leitura de oportunidade e a tabela oficial de custo e probabilidade por quantidade de dezenas.
- **Onde apostar** (primeiro item da linha de modalidades) — compara as nove entre si pelo índice CRIVO, com o veredito apostar / momento bom / melhor opção / só se for jogar / não apostar, e o prêmio de gatilho de cada uma.
- **Minhas apostas** — registro das apostas reais. Cada uma ganha uma sombra aleatória do mesmo tamanho, e o painel confere as duas contra o resultado oficial. É a regra O do método em funcionamento.
- **Volante** — mapa de calor das dezenas por frequência ou atraso.
- **Mais & menos** — rankings com a data da última aparição, seletor de Top 10/20/30/todas, gráfico de dispersão (quando saiu × quantas vezes saiu, com o tamanho da bolha pela frequência recente) e maiores atrasos.
- **Conjuntos 2–6** — quais combinações mais e menos saíram juntas, e quantas nunca saíram.
- **Meu jogo** — escolha as dezenas e veja, ao vivo: histórico de cada uma, matriz de co-ocorrência dos seus números, índice de exclusividade, sugestões de troca de 1 a 3 dezenas, comparação de custo entre "1 jogo de 7" e "3 de 6", e análise dos seus números favoritos.
- **Gerador** — dois modos: por orçamento (monta o plano de compra que cabe no valor) ou por composição (você escolhe o tamanho do jogo). Cada jogo gerado vem com a explicação do porquê daquelas dezenas.
- **Receitas por combinações** (dentro de Meu jogo) — monta o jogo com blocos: 3 duplas que mais saíram, 2 trincas que menos saíram, e assim por diante. Para "menos saíram", busca blocos que **nunca** saíram juntos, não os que saíram uma vez.
- **Nosso método** — o método CRIVO e a tabela comparativa com todos os modelos existentes.
- **Evidências & testes** — os backtests walk-forward, o qui-quadrado com correção, o teste de estabilidade temporal, a prova brasileira da seleção consciente e o backtest do próprio método contra uma carteira aleatória de controle, com retorno financeiro real.
- **Evolução do método** — os gatilhos objetivos que dizem quando recalibrar, aperfeiçoar ou abandonar a metodologia, com base em revisão de literatura de 2024 a 2026.
- **Financeiro** — carga da planilha oficial da Caixa para análise de arrecadação e rateios.
- **Risco** — o que a matemática garante e as regras práticas.

## Design system

`dashboard/tokens.css` e `dashboard/components.css`, com `dashboard/styleguide.html` navegável nos dois temas. Três camadas — primitivas, semântica e componentes — com uma regra que sustenta tudo: nenhum componente escreve um hex ou um pixel.

O sistema também está no Figma: **[Painel Loterias — Design System](https://www.figma.com/design/A9hm8FcF7ctUhve0KR2aM7)**, com 54 variáveis de cor (coleções Claro e Escuro), 12 de espaçamento, 6 de raio, 9 estilos de texto e 3 de elevação. Cada variável carrega o nome da variável CSS correspondente no code syntax, então a ponte entre Figma e código é direta.

## O achado principal

Medimos, com dado oficial da Caixa, que sorteios da Mega-Sena com muitas dezenas ≤ 31 produzem quase o **dobro de ganhadores** por real arrecadado — e prêmio médio da quadra **81% menor**. É a prova direta de que os brasileiros concentram apostas em datas de aniversário, e a única vantagem real, mensurável e legítima disponível: fugir desse padrão não aumenta sua chance de ganhar, mas aumenta muito o quanto você levaria se ganhasse.

Detalhes completos em `METODOLOGIA.md`.

## Aviso

Loteria não é investimento. O retorno esperado é negativo por lei (43,79% da arrecadação volta em prêmios, dos quais ainda se retém 30% de IR). Este projeto serve para decidir melhor um gasto de entretenimento e para não cair em quem vende método.

## Novidades da v9

Seleção única e compartilhada entre todas as abas, com desfazer e refazer; régua do gatilho abrindo a aba Onde apostar; painel de co-ocorrência que mostra quem sai junto com quem — e, principalmente, uma **correção no nosso próprio cálculo**: o esperado sob independência ignorava que o sorteio é sem reposição, o que empurrava todos os pares para o lado negativo. Corrigido por calibração empírica, o resultado é que nenhuma dupla de dezenas tem co-ocorrência anômala em nenhuma das nove modalidades. Detalhes na seção 14 de `METODOLOGIA.md`.

## Novidades da v10

Super Sete e Dupla-Sena deixaram de ser modalidades de segunda classe — o primeiro ganhou análise por coluna, filtro cruzado por (coluna, dígito) e a aba Evidências completa; a segunda ganhou os dois sorteios separados. Fechar essas lacunas expôs **dois erros graves no cálculo de prêmio** que afetavam cinco modalidades: o índice de faixa era calculado por aritmética e atropelava as faixas extras (o backtest da Dupla-Sena chegava a mostrar 507.958% de retorno), e o simulador gerava apostas do tamanho do sorteio em vez do tamanho da aposta real. Os dois estão corrigidos e documentados na seção 15 de `METODOLOGIA.md`. O backtest agora roda oito sementes e mostra a dispersão, porque uma rodada só engana.

## Novidades da v11

Varredura histórica do índice CRIVO sobre os 4.635 sorteios da base em que o prêmio principal foi realmente ganho. Resultado: **o índice cruzou o break-even de 1,00 uma única vez em toda a história** — Lotomania, março de 2017. Nem a maior Mega da Virada chegou lá. O painel deixou de tratar o 1,00 como veredito e passou a mostrar o **percentil histórico da própria modalidade**, que é o que distingue um dia comum de um dia raro. Detalhes e a ressalva sobre o imposto na seção 16 de `METODOLOGIA.md`.

## Correção da v12

O painel descontava 30% de IR do prêmio — errado, porque o valor divulgado pela Caixa **já vem líquido**. Verificado contra a base inteira: reconstruindo o bruto das faixas acima do piso de retenção de R$ 1.903,98, o total reproduz a fatia legal de 43,35% da arrecadação com erro menor que 0,7 ponto percentual em sete modalidades independentes. Consequência: todos os índices sobem ~43%, o prêmio de gatilho cai 30%, e os cruzamentos históricos do break-even passam de 1 para 5 em 4.635 sorteios — quatro deles em sorteios especiais. Seção 17 de `METODOLOGIA.md`.

## Novidades da v13

Conjuntos passaram a mostrar quando saíram pela última vez e o maior intervalo entre aparições, com a probabilidade de o acaso produzir aquela repetição. O mapa de calor virou ordenável (por dezena, frequência ou atraso) e absorveu a aba Volante. Um painel novo responde se marcar mais dezenas melhora o índice — não melhora, e piora a frequência de prêmios. Outro responde se algum sorteio já se repetiu: na Mega-Sena nunca, o que era o previsto; na Quina e na Dupla-Sena sim, também previsto. Essa última varredura achou e corrigiu uma corrupção no concurso 2019 da Dupla-Sena. Seção 18 de `METODOLOGIA.md`.

## Novidades da v14

Bandeiras de exclusividade em "Meu jogo", com um botão que completa o jogo fugindo dos padrões que a multidão joga — preservando as dezenas que você já escolheu. Cada bandeira declara a própria força: fugir de datas tem efeito medido de 1,96× na Mega-Sena; evitar dezenas quentes tem efeito medido, mas dez vezes menor (1,12× na Lotofácil, onde não há efeito de data para confundir); as demais são raciocínio declarado, sem medição. E o painel explica por que duas ideias intuitivas não entraram: "nunca saiu" não filtra nada (99,99% das combinações nunca saíram) e "as que mais saíram" piora o rateio em vez de melhorar. Seção 19 de `METODOLOGIA.md`.

## Novidades da v15

A tabela de Onde apostar ganhou duas colunas de índice — aposta mínima e com uma dezena a mais — e elas mostram o mesmo número de propósito. Sete vezes o preço na Mega-Sena, o mesmo 0,376. Verificado recalculando o índice do desdobramento do zero: iguais até a sexta casa decimal. Seção 20 de `METODOLOGIA.md`.

## Revisão da v16

Auditoria da metodologia encontrou uma falha estrutural: o índice usava o prêmio atual com o volume de apostas típico, sendo otimista exatamente nas acumulações grandes que ele existe para avaliar. Corrigido usando a sequência de concursos acumulados como preditor do público — um instrumento causalmente limpo, já que é determinado antes do sorteio. O índice da Lotofácil caiu 22,9% e o prêmio de gatilho dela quase dobrou. Seção 21 de `METODOLOGIA.md`.

## Novidades da v17

A aba Onde apostar abre com uma **matriz de decisão**: nove modalidades, quatro indicadores medidos (negócio, prêmio grande, frequência, raridade) e um índice final ponderado. Os pesos são escolhidos por você entre quatro perfis e aparecem na tela — trocar de perfil troca o primeiro colocado, e isso é intencional. A decomposição mostra algo que o índice único escondia: a Lotofácil lidera no total mas cai para terceira quando se olha só o prêmio grande, onde a Quina passa na frente. Tabelas ordenáveis por clique. Seção 22 de `METODOLOGIA.md`.
