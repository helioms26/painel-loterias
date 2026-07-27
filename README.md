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
├── METODOLOGIA.md                 ← método CRIVO, testes e resultados
├── README.md
└── dashboard/
    ├── index.html                 ← versão para hospedar
    ├── update_base.ps1            ← atualiza os resultados de todos os jogos
    ├── fetch_dates.ps1            ← coleta as datas dos concursos
    ├── fetch_premios.ps1          ← coleta ganhadores, rateio e arrecadação
    ├── verify.ps1                 ← reconfere a base contra a API oficial
    └── data/
        ├── megasena.json ... maismilionaria.json   ← dezenas por concurso
        ├── dates_<jogo>.json                       ← data de cada concurso
        ├── premios_<jogo>.json                     ← ganhadores, rateio, arrecadação
        └── status.json                             ← situação atual de cada jogo
```

## Base de dados

Histórico oficial completo, coletado da API `servicebus2.caixa.gov.br/portaldeloterias`.

| Jogo | Concursos | Último |
|---|---|---|
| Mega-Sena | 3.035 | 23/07/2026 |
| Lotofácil | 3.744 | 24/07/2026 |
| Quina | 7.074 | 24/07/2026 |
| Lotomania | 2.954 | 24/07/2026 |
| Dupla-Sena | 2.987 | 24/07/2026 |
| Timemania | 2.419 | 23/07/2026 |
| Dia de Sorte | 1.254 | 24/07/2026 |
| Super Sete | 877 | 24/07/2026 |
| +Milionária | 374 | 22/07/2026 |

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

- **Visão geral** — último concurso, situação de acumulação, prêmio estimado, e a leitura de oportunidade (prêmio ÷ ponto de equilíbrio).
- **Volante** — mapa de calor das dezenas por frequência ou atraso.
- **Mais & menos** — rankings com a data da última aparição, seletor de Top 10/20/30/todas, gráfico de dispersão (quando saiu × quantas vezes saiu, com o tamanho da bolha pela frequência recente) e maiores atrasos.
- **Conjuntos 2–6** — quais combinações mais e menos saíram juntas, e quantas nunca saíram.
- **Meu jogo** — escolha as dezenas e veja, ao vivo: histórico de cada uma, matriz de co-ocorrência dos seus números, índice de exclusividade, sugestões de troca de 1 a 3 dezenas, comparação de custo entre "1 jogo de 7" e "3 de 6", e análise dos seus números favoritos.
- **Gerador** — jogos anti-popularidade.
- **Receitas por combinações** (dentro de Meu jogo) — monta o jogo com blocos: 3 duplas que mais saíram, 2 trincas que menos saíram, e assim por diante. Para "menos saíram", busca blocos que **nunca** saíram juntos, não os que saíram uma vez.
- **Nosso método** — o método CRIVO e a tabela comparativa com todos os modelos existentes.
- **Evidências & testes** — os backtests walk-forward, o qui-quadrado com correção, o teste de estabilidade temporal e a prova brasileira da seleção consciente.
- **Evolução do método** — os gatilhos objetivos que dizem quando recalibrar, aperfeiçoar ou abandonar a metodologia, com base em revisão de literatura de 2024 a 2026.
- **Financeiro** — carga da planilha oficial da Caixa para análise de arrecadação e rateios.
- **Risco** — o que a matemática garante e as regras práticas.

## O achado principal

Medimos, com dado oficial da Caixa, que sorteios da Mega-Sena com muitas dezenas ≤ 31 produzem quase o **dobro de ganhadores** por real arrecadado — e prêmio médio da quadra **81% menor**. É a prova direta de que os brasileiros concentram apostas em datas de aniversário, e a única vantagem real, mensurável e legítima disponível: fugir desse padrão não aumenta sua chance de ganhar, mas aumenta muito o quanto você levaria se ganhasse.

Detalhes completos em `METODOLOGIA.md`.

## Aviso

Loteria não é investimento. O retorno esperado é negativo por lei (43,79% da arrecadação volta em prêmios, dos quais ainda se retém 30% de IR). Este projeto serve para decidir melhor um gasto de entretenimento e para não cair em quem vende método.
