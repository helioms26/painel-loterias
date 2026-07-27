# Continuar daqui

Leia este arquivo primeiro. Ele diz onde está a última versão de tudo — design e metodologia — e como retomar sem refazer nada. O estado exato da publicação está em `VERSAO.json`, gerado junto com cada versão.

Última versão: **9.0**, publicada em 26/07/2026 (noite de Brasília).

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
| Dados | `site/data/*.json` — 28 arquivos: sorteios, datas, prêmios e status |
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

O Super Sete fica de fora do ranking por dezena, do gerador e do backtest — ele sorteia um dígito por coluna, então frequência por dezena não significa nada ali. A Dupla-Sena tem os dois sorteios tratados em conjunto em algumas visões, o que não é errado mas também não é o ideal. E as telas 2 a 5 da v9 existem só no protótipo em HTML e no código, não no Figma, por causa da cota.

---

## Tarefa semanal ativa

`trig_01QG7R8GitNHNrH5Lvazpr6A`, segundas às 14:00 UTC — 11h de Brasília. Recalcula o índice CRIVO das nove modalidades e avisa se alguma acumulação passou a valer a pena. Se os preços das modalidades mudarem, o prompt dessa tarefa precisa ser atualizado junto: ela já carregou preço errado do Super Sete e da Lotomania uma vez.

---

## O aviso que não sai

Loteria não é investimento nem fonte de renda. O retorno esperado é negativo por lei. Nada neste projeto aumenta a chance de ganhar — o que ele faz é medir onde se perde menos e como dividir com menos gente no caso improvável de acerto. Se em algum momento o painel começar a sugerir outra coisa, o painel está errado.
