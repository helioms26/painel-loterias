# Continuar daqui

Leia este arquivo primeiro. Ele diz onde está a última versão de tudo — design e metodologia — e como retomar sem refazer nada. O estado exato da publicação está em `VERSAO.json`, gerado junto com cada versão.

Última versão: **15.0**, publicada em 27/07/2026.

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
