# Continuar daqui

Leia este arquivo primeiro. Ele diz onde está a última versão de tudo — design e metodologia — e como retomar sem refazer nada. O estado exato da publicação está em `VERSAO.json`, gerado junto com cada versão.

Última versão: **19.10**, 03/08/2026.

---

## O que é este projeto

Um painel de estatística das nove loterias da Caixa que responde uma pergunta só: **se você já decidiu gastar, onde o mesmo dinheiro rende mais hoje.** A tese central é que o sorteio é imprevisível e o comportamento da multidão não é — então o método não tenta acertar a bola, tenta melhorar o rateio.

O método se chama CRIVO: Custo-teto, Rateio, Independência, Valor esperado, Observação auditável. Está descrito por inteiro em `METODOLOGIA.md`, que é o documento de referência. `README.md` é a porta de entrada. `DESIGN-SYSTEM.md` descreve as regras visuais.

---

## Onde cada coisa mora

Tudo em `C:\Users\hmsue\OneDrive\IA_meus projetos\13 - loteria`.

| O quê | Caminho |
|---|---|
| Repositório publicado | `repo-github/` (clone de github.com/helioms26/painel-loterias) |
| Painel publicado, arquivo único | `repo-github/index.html` |
| Cópia offline idêntica | `painel-loterias-offline.html` |
| Fonte do painel | `site/index.html` — sem dados embutidos |
| Dados | `site/data/*.json` — 29 arquivos: sorteios, datas, prêmios, status e a varredura histórica do CRIVO |
| Design system | `site/tokens.css`, `site/components.css`, `site/styleguide.html` |
| Protótipo da v9 | `prototipo-v9.html` — navegável, serve de referência de interação |
| Planilhas oficiais baixadas | `planilhas_caixa/` |
| Script de atualização | `dashboard/atualizar_tudo.ps1` |
| Empacotador | `bundle.py` |
| Varredura histórica do CRIVO | `gerar_historico_crivo.py` |

A distinção que mais confunde: `site/index.html` é a **fonte** e busca os dados por `fetch`; `repo-github/index.html` é o **produto**, com `window.EMBEDDED_DATA` embutido. Nunca edite o segundo à mão — ele é gerado.

---

## Como publicar

Este é o caminho que funciona. O upload pelo site do GitHub falhou duas vezes seguidas com arquivo de 4 MB; não insista nele.

```powershell
$repo = "C:\Users\hmsue\OneDrive\IA_meus projetos\13 - loteria\repo-github"
git -C $repo add -A
git -C $repo commit -m "descrição da mudança"
git -C $repo push origin main
```

As credenciais já estão no Gerenciador de Credenciais do Windows via Git Credential Manager, então o push não pede senha. O GitHub Pages leva de 40 a 90 segundos para servir a versão nova.

Para conferir se pegou mesmo — e não a versão em cache:

```powershell
$c = (Invoke-WebRequest ('https://helioms26.github.io/painel-loterias/?v=' + (Get-Random)) -UseBasicParsing).Content
"minhas apostas: " + $c.IndexOf('Minhas apostas')
"filtro cruzado: " + $c.IndexOf('PILHA_SEL')
```

Índice `-1` significa que não subiu.

---

## Como atualizar os dados

O ponto que já causou erro: **não confie no endpoint `/api/{modalidade}`**, que devolve o "último" concurso. Ele atrasa horas depois do sorteio e já fez a atualização retornar "nenhum concurso novo" com seis concursos pendentes. O script correto sonda para a frente a partir do último que temos localmente — `ultimoLocal+1`, `+2`, … — e só para depois de duas respostas vazias seguidas.

Rode `dashboard/atualizar_tudo.ps1`, que faz coleta, reparo, verificação de integridade, backup único e empacotamento, nessa ordem. O backup só é gravado depois que a verificação passa.

Para reempacotar sem coletar nada, `python bundle.py caminho/de/saida.html`.

Horários oficiais em vigor (documento "Sorteio e Apuração das Loterias Federais", agosto/2026): segunda a sexta às 21h, domingo às 11h, sem sorteio aos sábados desde 19/07/2026. As apostas encerram **uma hora antes** do sorteio. A data que a API devolve para o próximo concurso fica velha; o painel infere o calendário a partir dos dias da semana observados nos últimos dez dias e só usa a data da API quando ela está no futuro.

---

## Design

O sistema tem três camadas — primitivas, semântica e componentes — e a regra que sustenta tudo é que **um componente nunca escreve um hex ou um pixel**. Se precisou, falta um token. `DESIGN-SYSTEM.md` tem o resto.

Antes de mexer em qualquer cor, rode o validador nos dois modos. Julgar paleta no olho é exatamente o erro que este projeto critica em outro contexto.

O arquivo no Figma é <https://www.figma.com/design/A9hm8FcF7ctUhve0KR2aM7>, com as páginas "Cor", "Tipografia e espaço" e "Painel v9 · telas".

**Limitação que vai atrapalhar:** o plano do Figma é Starter com assento View, o que dá **6 chamadas de MCP por mês** — não por dia nem por hora. A cota de julho/2026 acabou com a tela 1 da v9 montada e as outras quatro por fazer. Renova no dia 1º de cada mês. Para trabalhar de verdade no Figma seria preciso um plano Professional com assento Full ou Dev, que dá 200 por dia. Enquanto isso, o protótipo em HTML (`prototipo-v9.html`) cumpre melhor o papel, porque as interações dá para clicar de verdade.

---

## O que a v9 mudou

Uma seleção única — modalidade, janela e dezenas — compartilhada por todas as abas, numa barra fixa no topo, com desfazer e refazer que cobrem qualquer passo. A régua do gatilho abrindo a aba Onde apostar. E um painel de co-ocorrência que mostra quais dezenas saem junto com a selecionada mais e menos do que o esperado.

Esse último trouxe junto a correção de um erro nosso, e vale saber dela: o esperado sob independência não pode ser `freq(i) × freq(j) ÷ total`, porque essa fórmula supõe sorteio com reposição. Sem a correção, **todos** os pares apareceriam do lado negativo e qualquer combinação pareceria pouco trilhada — um artefato de cálculo vendido como descoberta, que é justamente o que este projeto critica nos outros. A escala agora é calibrada nos próprios dados e reencontrou o valor teórico com quatro casas decimais. Seção 14 de `METODOLOGIA.md`.

Com a correção, a conclusão é limpa: **não há nenhuma dupla de dezenas com co-ocorrência anômala em nenhuma das nove modalidades.** O painel diz isso em voz alta, porque esse é o resultado.

## O que a v10 mudou

Super Sete e Dupla-Sena saíram da segunda classe. O primeiro ganhou análise por coluna, seleção cruzada por (coluna, dígito) e a aba Evidências inteira; a segunda ganhou os dois sorteios separáveis.

Fechar essas lacunas expôs dois erros sérios que já estavam lá. O índice da faixa de prêmio era calculado como `marcadas − acertos`, o que atropela as faixas extras: na Dupla-Sena 2 acertos caíam na sena do segundo sorteio e o backtest chegava a mostrar 507.958% de retorno; na Timemania 2 acertos viravam Time do Coração; no Dia de Sorte 3 acertos viravam Mês da Sorte; na Lotomania 14 acertos caíam na faixa de zero acertos. Agora existe um mapa explícito por modalidade, conferido contra as contagens de ganhadores. E o backtest gerava apostas do tamanho do sorteio em vez do tamanho da aposta real — 20 dezenas na Lotomania, que marca 50; 7 na Timemania, que marca 10.

Se você for mexer no cálculo de prêmio, o mapa se chama FAIXAS e o comentário acima dele explica cada caso. Não volte para a aritmética.

A Dupla-Sena virou o melhor teste do nosso próprio método: pares dentro de um sorteio estão sob restrição sem reposição, pares entre os dois sorteios são independentes. A calibração devolve 0,8504 no primeiro caso (teórico 0,8503) e 0,9994 no segundo. No Super Sete, entre colunas, dá 1,0000.

## O que a v11 mudou, e por que importa mais que o resto

O Hélio perguntou se o break-even de 1,00 é realista. Fui medir na base inteira: para cada sorteio em que o prêmio principal foi realmente ganho, recalculei o índice com o valor efetivamente pago.

**Em 4.635 sorteios, o índice cruzou 1,00 uma única vez** — Lotomania, concurso 1741, 03/03/2017. Nem a maior Mega da Virada da história (R$ 197 milhões, 2015) passou de 0,851.

Consequência de produto: o 1,00 deixou de ser o veredito. Continua marcado na régua como referência teórica, mas o sinal operacional agora é o **percentil histórico da própria modalidade**. "0,53" não diz nada; "0,53, que é o percentil 99,8 da história da Lotofácil" decide.

A varredura vive em `site/data/historico_crivo.json`, gerado offline. Se a base crescer muito, vale regerar — o script está descrito na seção 16 de `METODOLOGIA.md`.

A reconstrução do preço histórico se valida sozinha (Lotofácil estimada em R$ 3,49 contra R$ 3,50 oficial, e a série reproduz os aumentos da Caixa sem que ninguém informe as datas) — se algum dia ela parar de bater, é sinal de que a base ou a fórmula quebrou.

## O que a v12 corrigiu — leia antes de mexer no imposto

O painel descontava 30% de IR do prêmio. **Estava errado: o valor divulgado pela Caixa já vem líquido.** O Hélio informou, e eu verifiquei contra a base inteira antes de mudar.

O teste: a retenção de 30% só incide sobre prêmio acima de R$ 1.903,98. Reconstruindo o bruto de cada faixa que passa desse piso (dividindo por 0,70) e deixando as menores como estão, o total tem de bater a fatia legal de 43,35% da arrecadação. Bate em sete modalidades com estruturas de faixa diferentes, todas dentro de 0,7 ponto percentual — Mega-Sena +0,21pp, Quina +0,11pp, Dupla-Sena +0,07pp. Sem o ajuste do piso, o erro ia de −2 a −3,6pp em todas.

O teste está reproduzido em comentário no código, logo acima da constante `IR_LOTERIA`, que hoje vale **0**. Se a fatia legal ou o piso de retenção mudarem, é esse teste que precisa ser refeito. Não volte a multiplicar por 0,70 sem refazê-lo.

Consequências: todos os índices subiram ~43%, o prêmio de gatilho caiu 30%, e os cruzamentos históricos de 1,00 passaram de 1 para 5 em 4.635 sorteios — quatro deles em sorteios especiais (Mega da Virada 2015, Quina de São João 2016 e 2018, Timemania de Natal 2025). O `historico_crivo.json` foi regerado sem o fator.

## O que a v13 mudou

A aba Volante deixou de existir — o mapa de calor mora dentro de Mais & menos, agora ordenável por dezena, frequência ou atraso. Estados antigos que apontem para `volante` são redirecionados em silêncio dentro do `refresh()`.

Conjuntos passaram a mostrar última aparição, maior intervalo e a probabilidade de o acaso produzir aquela repetição. Dois painéis novos: um responde se marcar mais dezenas melhora o índice (não melhora — a resposta e a aritmética estão no comentário acima de `painelMaisDezenas`), outro responde se algum sorteio já se repetiu (`painelRepeticoes`).

**Corrupção de base corrigida:** o concurso 2019 da Dupla-Sena era uma cópia byte a byte do 2009. Foi achado pela varredura de repetições e conferido na API oficial. É a segunda corrupção nessa modalidade — a primeira foi o 2373.

Disso saiu uma verificação nova que vale manter: **procurar registro inteiro idêntico a outro**. Coincidência impossível é detector de erro de dados, não curiosidade estatística. Hoje as nove modalidades passam limpas.

## Um erro de processo que não pode se repetir

A regressão desta rodada revelou que o script de teste apontava para `painel_v9.html` desde a v10 — os `sed` que trocavam o caminho nunca casaram, e eu não conferi. As verificações reportadas para a v11 e a v12 rodaram na versão errada.

**Antes de confiar em qualquer regressão, confirme qual arquivo ela está abrindo.** O script agora imprime o alvo. Refeito na v13: 198 combinações (9 modalidades × 11 abas × 2 temas), zero erro, zero estouro no iPhone.

## O que a v14 mudou

Bandeiras de exclusividade em "Meu jogo" (`painelBandeiras`) e um botão que completa o jogo fugindo dos padrões da multidão, preservando o que já foi marcado.

A regra que sustenta o painel inteiro aparece aqui de forma explícita: **cada bandeira declara se o efeito é medido ou é raciocínio.** Fugir de datas tem 1,96× medido na Mega-Sena. Evitar dezenas quentes tem efeito medido também, mas dez vezes menor — e a medida limpa vem da Lotofácil, onde todas as dezenas são ≤ 31 e portanto não existe efeito de data para confundir: 1,12×. As outras três bandeiras não têm medição e dizem isso na tela.

Duas ideias intuitivas ficaram de fora, com o motivo escrito: "nunca saiu" não filtra nada (99,9939% das combinações da Mega-Sena nunca saíram) e "as que mais saíram" piora o rateio, porque é o que faz quem joga estatística.

Se for acrescentar bandeira nova, mantenha a disciplina: ou tem número medido ao lado, ou está escrito que é raciocínio.

## O que a v15 mudou

A tabela de Onde apostar tem duas colunas de índice — mínima e +1 dezena — mostrando o mesmo número. Não é bug: `comMaisUmaDezena` existe para tornar a igualdade visível, com o custo embaixo de cada valor. Sete vezes o preço na Mega-Sena, o mesmo 0,376.

Se alguém "consertar" essa duplicação achando que é erro, terá removido justamente o que a coluna prova.

## O que a v16 mudou — revisão de metodologia, leia antes de mexer no índice

Auditoria encontrou uma falha estrutural no índice CRIVO: ele usava o **prêmio de agora** com o **volume de apostas típico**. Como o público responde ao prêmio, o índice era otimista exatamente nas acumulações grandes que ele existe para avaliar. O prêmio de gatilho sofria ainda mais, já que é por definição uma acumulação enorme.

A correção usa a **sequência de concursos acumulados** como preditor do público — não o prêmio. Isso é deliberado: a sequência é determinada antes do sorteio, então não pode ser causada pelo volume de agora. Regredir contra o prêmio dá efeito maior, mas parte é mecânica, porque o prêmio cresce *porque* as pessoas apostam. Se for reescrever `multiplicadorVolume`, não troque o preditor sem refazer esse raciocínio.

A curva se recalibra nos dados a cada carga (`curvaVolume`), então melhora sozinha conforme a base cresce.

Efeito: Lotofácil caiu de 0,632 para 0,487 (−22,9%) e o gatilho dela subiu de R$ 17,6 mi para R$ 31,7 mi. A distância para a Quina encolheu de 0,163 para 0,029.

Benefício lateral que vale saber: o índice ao vivo e a varredura histórica agora usam a mesma noção de volume. Antes a varredura usava o número real de apostas de cada sorteio e o índice ao vivo usava a mediana — nunca tinham falado a mesma língua.

Fragilidade declarada: o R² da elasticidade fica entre 0,23 e 0,56. A relação é ruidosa e a curva é mediana por faixa, não modelo fino. A +Milionária não tem base para calibrar e fica com multiplicador 1.

## O que a v17 mudou

A aba Onde apostar abre com a **matriz de decisão** (`matrizDecisao`): nove modalidades, quatro indicadores normalizados de 0 a 100 e um índice final ponderado.

A regra que não pode ser quebrada nessa tela: **os quatro indicadores são medidos, os pesos são preferência.** Por isso os perfis são escolhidos pelo usuário e os pesos aparecem escritos. Se alguém "simplificar" escondendo os pesos, terá transformado a ferramenta em oráculo — que é o que o painel existe para desmontar em quem vende método.

O índice também foi decomposto em troco das faixas menores e prêmio principal, e a decomposição muda o ranking: a Lotofácil lidera o total e cai para terceira no eixo do prêmio grande, atrás da Quina. Não existe "melhor" aqui — existe o que a pessoa quer.

Toda tabela grande virou ordenável por clique (`tabelaOrdenavel`), com terceiro clique voltando à ordem original.

## O que a v18 corrigiu — duas armadilhas que valem memorizar

**Nunca use `elemento.innerHTML += html` num painel que tenha algo clicável.** Isso reserializa o conteúdo inteiro e descarta silenciosamente todos os listeners já ligados nos filhos, sem erro no console. Foi o que matou os botões de perfil da matriz de decisão. Use `htmlBloco(html)`, que devolve o HTML num contêiner próprio.

**O limite de tamanho dos conjuntos agora é calculado, não escrito à mão** (`comboMaxViavel`). O custo tem duas pernas: operações e chaves distintas no mapa — e é a segunda que decide. A Lotomania com k=4 gera 3,9 milhões de chaves e leva 17 segundos; a Timemania com k=6 gera 17 mil e leva 18ms. Se for mexer no orçamento, meça antes.

## Teste o que o usuário faz, não o que é fácil testar

Três bugs desta rodada de versões tiveram a mesma raiz. O script de regressão apontou para o arquivo errado por seis versões porque eu nunca conferi o alvo. Os botões de perfil ficaram mortos porque eu trocava o perfil por `state.perfilAposta = ...` em vez de clicar. O limite de conjuntos ficou apertado porque eu supus o custo em vez de medir.

Os testes agora fazem as três coisas: imprimem qual arquivo estão abrindo, clicam em todo botão de toda aba verificando se tem `onclick` ligado, e medem tempo em vez de estimar. Mantenha assim.

---

## Antes de publicar qualquer versão

Regressão automatizada com Playwright, que é o que pegou os erros que importaram até agora:

- as 9 modalidades × 12 abas × 2 temas renderizando sem erro de console e sem painel vazio — são 216 combinações;
- iPhone 13 (390 px) sem estouro horizontal em nenhuma aba;
- filtro cruzado: clicar numa dezena do volante, ver o chip aparecer na barra, ir para Mais & menos e encontrar o painel de co-ocorrência;
- desfazer e refazer restaurando a seleção;
- gerador produzindo jogos sem dezena repetida, com a composição batendo com o orçamento.

O backup só depois que tudo isso passa.

---

## Lacunas conhecidas

A +Milionária fica de fora do backtest do método: as dez faixas dela combinam acertos de dezenas com acertos de trevos e não conseguimos confirmar a ordem conferindo contagens de ganhadores contra probabilidades. Sem essa confirmação, qualquer retorno seria número inventado com cara de medição.

Os critérios anti-popularidade do Super Sete são raciocínio por analogia com escolha humana de dígitos, não medição — a Caixa não publica a distribuição das apostas dessa modalidade. O efeito aniversário da Mega-Sena, esse sim, foi medido no rateio real. A diferença está escrita na tela.

As telas 2 a 5 do redesenho existem no código e no protótipo em HTML, não no Figma, por causa da cota mensal.

---

## Tarefa semanal ativa

`trig_01QG7R8GitNHNrH5Lvazpr6A`, segundas às 14:00 UTC — 11h de Brasília. Recalcula o índice CRIVO das nove modalidades e avisa se alguma acumulação passou a valer a pena. Se os preços das modalidades mudarem, o prompt dessa tarefa precisa ser atualizado junto: ela já carregou preço errado do Super Sete e da Lotomania uma vez.

---

## O aviso que não sai

Loteria não é investimento nem fonte de renda. O retorno esperado é negativo por lei. Nada neste projeto aumenta a chance de ganhar — o que ele faz é medir onde se perde menos e como dividir com menos gente no caso improvável de acerto. Se em algum momento o painel começar a sugerir outra coisa, o painel está errado.


## O que a v19 mudou — a rodada em que o método mediu a si mesmo

Esta é a rodada mais importante desde a v12. **Uma regra do painel estava com o sinal invertido, e era a de maior peso.**

Até a v18, o "índice de exclusividade" era um conjunto de regras que eu tinha escrito por raciocínio: penalizava sequências consecutivas, paridade desequilibrada, dígitos finais repetidos, concentração no volante. Nenhuma delas tinha sido medida. Ao aplicar o mesmo experimento natural do efeito das datas — comparar, entre concursos já sorteados, quantos ganhadores da faixa máxima apareceram **por combinação jogada** — o resultado foi:

| Lotofácil (3.411 concursos) | fator | IC95 |
|---|---|---|
| maior sequência ≤ 3 | **2,21×** | 1,97 – 2,53 |
| sequência ≥ 6 | 0,80× | 0,76 – 0,87 |
| soma 186–210 | 1,15× | — |
| soma > 210 | 0,79× | 0,74 – 0,85 |
| linha inteira do volante | 0,72× | 0,66 – 0,78 |
| coluna inteira do volante | 1,21× | 1,10 – 1,35 |
| paridade 7/8 pares | 0,96× | **sem efeito** |

Na Mega-Sena, independentemente: todas ≤31 → 2,05×; nenhuma consecutiva → 1,41×; paridade e soma sem efeito. Na Quina, medido e **nulo em tudo** — e a Quina ficou sem regra nenhuma, porque inventar é pior que não ter.

**A multidão foge de dezenas coladas.** O painel empurrava o usuário exatamente para o perfil mais disputado, com um selo verde de "POUCO DISPUTADO" por cima.

Detalhes, desenho da medição e o viés conhecido (conservador) na seção 24 de `METODOLOGIA.md`.

### Regras que a v19 estabeleceu

1. **Onde não há medição, não há regra.** O painel diz "SEM MEDIÇÃO" em vez de dar nota.
2. **Teto de honestidade.** Os fatores foram medidos um a um; multiplicá-los supõe independência. A única célula conjunta testada (sequência ≥6 **e** soma >210) mediu 0,72× contra 0,64× previsto pelo produto — o modelo multiplicativo exagera. Piso de 0,70×, e o painel avisa quando ele entra em ação.
3. **Desdobramento dilui.** Num jogo com dezenas extras quem ganha é **um** subconjunto, e cada um tem perfil próprio. O fator honesto é a média sobre eles. Caso real: conjunto de 16 com sequência de 6 e soma 223 dá 0,70× como conjunto, mas seus 16 desdobramentos variam de 0,70× a 2,20× e a média é 1,02× — a vantagem some.

### Três defeitos que a medição expôs

**O gerador não gerava.** "Evitar 3+ dezenas consecutivas" era rejeição pura, e num jogo de 16 dezenas em 25 quase nenhum sorteio passa. A rejeição estourava as 900 tentativas e o jogo era descartado **em silêncio**: pedir 1 devolvia 0, pedir 3 devolvia 1. Nenhum erro no console, e nenhum teste contava gerados contra pedidos. Hoje o gerador junta até 300 candidatos livres, escolhe pelo fator medido, e o rodapé mostra "N de M".

**O ordenador ignorava colunas em "1 em N".** `numDaCelula` pegava o primeiro número da célula e devolvia sempre 1 para "1 em 9" / "1 em 2.298" — a ordenação era estável e nada se movia. Valia para **todas** as tabelas ordenáveis, inclusive as que já tinham sido "verificadas". Agora "1 em N" ordena pelo denominador, e "—" vai para o fim nos dois sentidos.

**Menu de uma opção só.** Em "Meus números favoritos", o seletor de tamanho vinha de um intervalo inventado (`pick`..`pick+4`, teto 15), que na Lotofácil colapsava para um item. Agora vem dos tamanhos que a Caixa aceita, com preço no rótulo. O card de custo também multiplicava tudo pelo preço da aposta simples.

### Produto

- Abas **"Meu jogo"** e **"Gerador de jogos"** fundidas numa só, com dois modos e um botão "Analisar este jogo" em cada jogo gerado. Estados antigos apontando para `gerador` são redirecionados no `refresh()`, como já se fazia com `volante`.
- **Botão copiar** em toda parte: dezenas, desdobramento linha a linha, jogo com a análise, jogos gerados, tabelas de combinações. Com queda para `execCommand` porque `navigator.clipboard` não existe em `file://`.
- Tabela dos três eixos ordenável por coluna.

### Lição de processo, de novo

Na v18 o defeito foi teste que não clicava nos botões. Na v19 foi teste que não contava quantos jogos o gerador devolveu, e regra de produto que nunca tinha sido medida embora o projeto inteiro se apresente como método de medição. **Quando o painel afirma alguma coisa sobre comportamento humano, essa afirmação precisa de um número e de um intervalo de confiança, ou de um rótulo dizendo que não tem.**

### Aposta real registrada

Em 27/07/2026 o Hélio fez duas apostas na Lotofácil para o concurso 3746 (prêmio estimado R$ 9 milhões, índice CRIVO 0,487, percentil 98,9 da história da modalidade, gatilho R$ 31,7 milhões). Os jogos sugeridos e a comparação entre desdobramento e jogos separados estão em `jogos-3746.md`. **Quando sair o resultado, vale conferir — não para validar o método, que não prevê sorteio nenhum, mas porque um registro honesto de apostas reais é o único jeito de o painel não virar autoelogio.**


## O que a v19.1 mudou

### O histórico de apostas sumia ao trocar de versão

O registro mora no `localStorage`. Em páginas abertas por `file://` o Chrome guarda um baú **por arquivo**: `painel-loterias-offline_7.html` e `_8.html` não enxergam o mesmo armazenamento. Baixar a versão nova parecia apagar o histórico — ele continuava preso no arquivo anterior.

Três defesas, nesta ordem, e a ordem importa:

1. **Semente versionada** em `site/data/apostas.json`. O `bundle.py` embute como qualquer outro dado, então toda versão nova já nasce com o histórico dentro.
2. **Exportar / importar JSON** no topo da aba, para o histórico ser um arquivo do usuário e não um detalhe do navegador.
3. **Mesclagem por chave estável** (`jogo|concurso|dezenas`), nunca sobrescrita — o que já está no navegador tem prioridade sobre a semente.

A versão anterior tinha exportar/importar escondido no rodapé da aba e deduplicava por `id`, que é regerado a cada mesclagem: importar o mesmo arquivo duas vezes duplicava tudo. O painel antigo foi removido para não haver dois widgets concorrentes.

**A semente não traz sombra pronta, e isso é deliberado.** A sombra aleatória nasce no instante do registro e se perde junto com o armazenamento. Inventar números seria fabricar dado num painel cujo argumento inteiro é não fabricar dado. Ela é regerada de forma determinística a partir da chave da aposta, e marcada com `sombraRegerada` para ninguém confundir com a original.

**Ao adicionar uma aposta nova, atualize `site/data/apostas.json`.** É ele que sobrevive à troca de arquivo.

### Placar escondia apostas

O placar filtrava por `DB[x.jogo]` e ignorava, sem avisar, as apostas de modalidades ainda não carregadas em memória — o total apostado saía menor que o real. Agora a aba carrega toda modalidade presente no histórico e avisa se alguma ficar de fora.

### Primeiro resultado real

Lotofácil 3746, 27/07/2026: `01 03 04 06 07 08 09 10 14 15 17 18 21 22 24`, acumulou. A aposta do Hélio (16 dezenas, R$ 56) fez 8 acertos, sem prêmio; a sombra fez 10, também sem prêmio. A Quina 7076 fez 0 de 5.

O sorteio caiu num perfil que o método classifica como pouco disputado — sequência máxima 5, soma 179 — e as quatro faixas premiadas vieram **todas abaixo do esperado, num gradiente monótono**: 0,90 na faixa de 11 acertos, 0,88 na de 12, 0,87 na de 13, 0,82 na de 14, e zero na de 15 contra 3,79 esperados. É a assinatura de um sorteio impopular. Seção 24.4 de `METODOLOGIA.md`, **com o aviso de que um concurso não valida nada** — está registrado porque é o primeiro ponto fora da amostra que gerou os coeficientes, e porque registro que só guarda o que dá certo não vale nada.

### Base de dados

Atualizada até 27/07/2026: Lotofácil 3746, Quina 7076, Dupla-Sena 2988, Lotomania 2955, Dia de Sorte 1256, Super Sete 878. Mega-Sena, Timemania e +Milionária não tiveram sorteio nesse dia. Integridade conferida: nenhum concurso faltando, tamanhos de sorteio consistentes nas nove modalidades.

**Atenção para quem for reconstruir na máquina do Hélio:** em 28/07/2026 a pasta `site/data/` local tinha só 3 dos 29 arquivos. O painel de arquivo único não sofre com isso porque os dados estão embutidos, mas `site/index.html` não roda sem eles. Se for editar a fonte, confira a pasta antes.


## O que a v19.2 mudou

Botões `↻` e `↻+` em cada linha do histórico. O primeiro repete as mesmas dezenas no próximo concurso da modalidade; o segundo mantém tamanho e custo e procura dezenas novas com perfil de rateio melhor.

Duas recusas deliberadas no `↻+`: ele **só troca com melhora estrita** — empate não é melhora, e trocar por fator idêntico jogaria fora a estrutura escolhida sem ganho nenhum; e em modalidade sem medição de popularidade ele **se recusa a trocar** e explica que seria sorteio no escuro.

A análise que motivou tudo isso está na seção 24.6 de `METODOLOGIA.md`, e ela vale como regra geral: **um desdobramento vale o que valem os seus piores subconjuntos, não o conjunto inteiro.** O que se procura não é um conjunto bom, é um conjunto cujo subconjunto mais fraco ainda seja bom — o que favorece estruturas redundantes, com mais de um padrão anti-multidão, para que nenhuma remoção isolada destrua todos de uma vez.


## O que a v19.3 mudou

Base até 28/07/2026 (Lotofácil 3747, Mega-Sena 3037, Quina 7077, Timemania 2421, Dia de Sorte 1257), integridade conferida nas nove.

E um painel novo que talvez seja o mais importante do projeto: o **diário fora da amostra**, na aba Evidências. Os coeficientes de popularidade foram medidos numa base que terminava na Lotofácil 3745 e na Mega-Sena 3036; tudo depois é teste cego. O painel recalcula sozinho a cada atualização, registra acerto e erro com o mesmo destaque, e ninguém consegue escolher quais concursos entram.

Os dois primeiros da Lotofácil vieram na direção prevista com gradientes monótonos e opostos entre si — 3746 previsto 0,54× e observado caindo de 0,90 a 0,00 conforme a faixa sobe; 3747 previsto 1,72× e observado subindo de 0,99 a 1,25. A Mega-Sena 3037 **errou**: previsto 1,41×, observado 0,59 na faixa de 5. Placar 2 de 3, que não é evidência de nada — está tudo escrito na seção 24.7 de `METODOLOGIA.md`, inclusive o cálculo de que acertar 2 de 3 por acaso tem probabilidade 3/8.

Detalhe que vale preservar: quando nenhum padrão medido aparece no sorteio, o fator fica em 1,00 e **não há previsão**. Esse concurso entra marcado e fora da conta. Contar fator 1,00 como acerto seria inflar o placar de graça; contar como erro, o contrário. A Quina não tem diário porque foi medida e deu nulo — sem coeficiente não há o que testar.


## O que a v19.4 mudou

**"Minhas apostas" mudou de lugar.** Saiu da fila de abas e virou um chip ao lado de "Onde apostar". O motivo não é estético: as abas trocam de conteúdo conforme a modalidade selecionada, e o registro não é uma visão *de* uma modalidade — é a carteira inteira, atravessando as nove. A fila de abas some quando ele está aberto, como já acontecia com "Onde apostar", e clicar numa modalidade sai do registro para a Visão geral.

**Gráfico do histórico**, com dois modos pela mesma disciplina da aba "Onde apostar" — perguntas diferentes, gráficos diferentes. *Por modalidade*: barras horizontais de gasto e prêmio, escala do zero, valor escrito em cada barra. *No tempo*: saldo acumulado aposta a aposta, com a sombra aleatória tracejada.

**O detalhe que vale guardar** é o de escala. Um prêmio de faixa alta é milhares de vezes o custo de uma aposta. No teste com R$ 2,1 milhão contra R$ 59,50 gastos, o eixo compartilhado achatou todas as barras de gasto a menos de um pixel — a comparação entre modalidades, razão de ser do gráfico, sumiu. A saída **não** foi escala logarítmica, que em dinheiro engana quem lê rápido: o gráfico detecta o caso (prêmio máximo acima de 4× o gasto máximo), dá um eixo para cada série e **diz na figura** que fez isso, com a razão entre os máximos escrita. Enquanto os valores forem comparáveis, o eixo continua sendo um só. Seção 24.8 de `METODOLOGIA.md`.


## O que a v19.5 mudou

Base até **31/07/2026**: Lotofácil 3750, Quina 7080, Mega-Sena 3038, Lotomania 2957, Dia de Sorte 1260, Super Sete 880, Timemania 2422, +Milionária 376. Integridade conferida nas nove — nenhum concurso faltando, tamanhos consistentes, datas e prêmios presentes, nenhuma repetição exata do último sorteio.

**Uma lacuna declarada:** a Dupla-Sena 2989 (29/07) entrou, mas o **2990 (31/07) não foi publicado** em nenhuma das duas fontes consultadas no momento da coleta — o mirror devolveu 404 e o endpoint oficial devolveu 500. A base fica em 2989 e o concurso precisa ser buscado na próxima atualização. Não inventamos a linha.

**Coleta pela nuvem.** A ponte com a máquina caiu e o PowerShell ficou indisponível, então os concursos vieram por requisição direta às APIs a partir do ambiente do Cowork, um concurso por vez, com a mesma varredura para a frente de sempre. Ficou registrado que o endpoint `/latest` do mirror estava **onze dias atrasado** na Dupla-Sena (devolvia o 2985, de 20/07) — a advertência de nunca confiar nele continua valendo, e agora tem um caso novo.

**Diário fora da amostra: a Lotofácil está em 5 de 5.** Sob uma moeda honesta isso sai com probabilidade 1/32. Antes de comemorar, três ressalvas que estão escritas na seção 24.9 de `METODOLOGIA.md` e que precisam viajar junto com o número: quatro dos cinco sorteios caíram no mesmo perfil, então não são cinco testes independentes; a faixa de 15 acertos não acompanhou em dois deles, e é por isso que o placar usa a mediana das faixas; e a magnitude prevista (2,20×) é o dobro da observada (~1,15×) — a direção acerta, o tamanho não.


## O que a v19.6 mudou

**A lacuna da v19.5 fechou.** A Dupla-Sena 2990, de 31/07, apareceu nas fontes e entrou. Base completa até 31/07/2026 nas nove modalidades, sem concurso faltando.

**A varredura histórica virou script.** Até agora o `historico_crivo.json` tinha sido gerado uma vez, num ambiente que não existe mais, e a seção 16 descrevia o procedimento em prosa. Uma varredura que ninguém consegue rodar de novo é uma afirmação, não uma medição. Agora é `gerar_historico_crivo.py`, na raiz do projeto — rode depois de cada coleta grande.

A reescrita foi **validada contra o arquivo antigo antes de substituí-lo** e reproduz todos os máximos históricos: Mega-Sena 1,1549 (contra 1,1548 guardado, arredondamento na quarta casa), Lotofácil 0,6930, Quina 1,0219, Lotomania 1,3553, Timemania 1,0135, Dia de Sorte 0,8316, Dupla-Sena 0,7130, Super Sete 0,5892. Se um dia essa reprodução parar de bater, é sinal de que a base ou a fórmula quebrou.

Base atual: **4.642 sorteios com prêmio principal observado, 5 cruzaram 1,00** — quatro deles em sorteios especiais.

**Dois detalhes que vale não repetir.** A aferição do preço reconstruído — aquela que valida a varredura sozinha — precisa de janela por **data**, não por contagem. "Últimos 60 sorteios com ganhador" atravessa cinco anos no Super Sete e acusa 12% de erro sem nada estar errado; "de 2025 em diante" atravessa o último aumento de preço na Lotofácil. Doze meses corridos resolve os dois, e onde a amostra fica pequena demais (Super Sete, n=4) isso é dito em vez de virar alarme falso. E o `_meta` do arquivo passou a ser **calculado**: antes trazia "4.635 sorteios" escrito à mão, número que o painel exibe em texto corrido e que teria envelhecido em silêncio a cada coleta.


## O que a v19.7 mudou — bolão

Bolão virou modalidade de registro, e ele quebra três coisas do modelo antigo de uma vez: são **vários jogos no mesmo bilhete**, o que você paga (a cota) **não é** o que o bilhete custa, e o prêmio que chega até você é uma **fração** do que o bilhete ganhou.

Modelar como "várias apostas simples separadas" erraria nos três — perderia o vínculo entre os jogos, inflaria o gasto em até doze vezes e pagaria o prêmio integral a quem tem uma cota. Então a aposta passou a ter uma **lista** de jogos (`jogos`), e o bolão carrega `valorCota`, `totalCotas` e `cotas`. `custo` é o seu desembolso, `custoTotal` é o bilhete, participação é `cotas ÷ totalCotas`.

A **sombra** de um bolão tem o mesmo formato: um jogo aleatório para cada jogo do bilhete. Sombra de um jogo só compararia coisas diferentes e daria vantagem ao bolão de graça.

**A conferência que ninguém faz na lotérica** está no formulário: cotas × valor da cota contra o custo real do bilhete. Se sobrar, é taxa de serviço — pode ser combinada, mas sai do seu bolso e agora aparece com o percentual.

O `↻+` **se recusa a agir num bolão**. As dezenas foram escolhidas por quem montou o bilhete; trocá-las criaria um registro que não corresponde ao que foi comprado.

**E apareceu um lado faltando na medição.** A regra das consecutivas da Mega-Sena só codificava o lado popular — "nenhuma colada = 1,41×". O jogo COM coladas caía no neutro 1,00, quando a medição diz **0,71×**. Metade do efeito estava escondida, justamente a metade que favorece quem joga colado. Agora é faixa com os dois lados, como já era na Lotofácil. Vale a lição geral: **quando uma medição tem dois lados, os dois entram no modelo** — codificar só um transforma o neutro em referência errada.


## O que a v19.8 mudou

Base até **02/08/2026**: Mega-Sena 3039, Lotofácil 3751, Quina 7081, Timemania 2423, Dia de Sorte 1261. `historico_crivo.json` regerado — 4.644 sorteios avaliados, 5 cruzaram 1,00.

**Uma lacuna declarada:** a +Milionária **377 (02/08) ficou de fora**. Nenhuma das duas fontes devolveu os trevos, e o formato guardado exige seis dezenas mais dois trevos. Inventar trevo não é opção. Enquanto isso, o card dela mostra a data do sorteio já passado, que é o comportamento honesto de `janelaAposta`.

**O diário fora da amostra está em 6 de 9,** e as duas novidades importam mais que o placar.

A Lotofácil **errou pela primeira vez**, no 3751: previsto 1,09×, observado 0,962. Repare em *como* é o erro — 1,09 é o fator mais próximo de 1,00 de toda a série, e 0,962 também. Modelo e realidade discordaram em cima de uma linha que quase não separa nada. O teste de sinal conta como erro cheio, o que está certo para não inflar o placar, mas quem lê a tabela precisa ver que este ponto pesa menos que os extremos.

E a Mega-Sena **3038 virou acerto** — não porque o sorteio mudou, mas porque a correção da v19.7 deu ao lado "tem coladas" o fator medido 0,71× em vez do neutro 1,00. O sorteio tinha 38 e 39 coladas, previa menos ganhadores, e saiu 0,536. Era um acerto escondido pela metade faltante da medição. O 3039 errou.

**As apostas do Hélio no 3039** (dois bolões, R$ 26,60): o sorteio saiu `14 16 21 39 53 58` e os sete jogos fizeram **1 acerto cada**. Sem prêmio.


## O que a v19.9 mudou

**O arquivo-fonte agora se explica.** Abrir `site/index.html` solto na pasta de downloads mostrava só `Erro ao carregar dados: Failed to fetch` — um erro técnico repassado cru, que não diz a causa nem a saída. Agora ele diz que é a fonte e não o painel pronto, que os dados moram em `data/` ao lado, e manda abrir o `painel-loterias-offline.html`. Também avisa que nem com a pasta no lugar a fonte roda por `file://`, porque o navegador bloqueia o fetch — precisa de servidor local.

**Como as entregas passam a ser feitas.** No chat vão **só dois itens**: o `painel-loterias-offline.html` e o link publicado. Todo o resto — documentação, fonte, pacote de dados — vai para `versoes/vXX/` dentro da pasta do projeto, e as versões antigas são apagadas mantendo apenas **as duas últimas**. Quem retomar o projeto encontra a versão corrente na raiz e uma anterior para comparar, sem uma pilha de arquivos soltos em Downloads.


## O que a v19.10 mudou

**A matriz de decisão ganhou a coluna do prêmio.** Ela decide onde apostar e não mostrava o valor — o número que a pessoa carrega na cabeça quando entra na lotérica estava fora justamente da tabela feita para decidir. Fica ao lado da modalidade, antes das quatro notas, porque é fato e não avaliação, com a data do próximo sorteio embaixo e o maior prêmio da rodada destacado. Ordena como as demais colunas.

Ela serve também de contraprova visual da tese. Quando a primeira colocada **não** é a de maior prêmio, a nota escreve isso e diz de quem é o maior. Quando coincidem — como hoje, Mega-Sena com R$ 135 milhões liderando também o índice final — a nota diz que coincidiu e que **isso acontece às vezes e não é a regra**. Comentar só o caso que favorece a própria tese seria seleção de evidência na cara do usuário.

**A lacuna da v19.8 fechou em um dia:** os trevos da +Milionária 377 apareceram no mirror (`4` e `6`) e o concurso entrou. As nove modalidades estão completas até 02/08/2026.
