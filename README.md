# Painel Loterias — projeto 13

Dashboard estatístico das 9 loterias da Caixa, com base oficial completa e metodologia própria (Método CRIVO).

## Como usar

**Opção 1 — arquivo único, dois cliques (mais simples)**
Abra `painel-loterias-offline.html`. Todos os dados estão embutidos; funciona sem internet e sem servidor, em qualquer navegador de computador.

**Opção 2 — versão para hospedar (GitHub Pages, Netlify, Vercel)**
A pasta `dashboard/` contém `index.html` + `data/`. Suba a pasta inteira em qualquer hospedagem estática e você terá uma URL pública que abre no iPhone e em qualquer navegador.

> Atenção: a versão da pasta `dashboard/` **não funciona** abrindo o `index.html` direto do disco (`file://`) — o Chrome bloqueia a leitura dos arquivos de dados por segurança. Para uso local, use o arquivo único da Opção 1.

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

**Integridade verificada:** 200 concursos da Lotofácil sorteados ao acaso foram reconferidos contra a API oficial. Dos 118 que responderam, 118 conferiram exatamente — zero divergências.

## Para atualizar a base

No PowerShell, dentro da pasta `dashboard`:

```powershell
.\update_base.ps1      # resultados novos de todos os jogos
.\fetch_dates.ps1      # datas dos concursos
.\fetch_premios.ps1    # ganhadores, rateio e arrecadação
```

Depois de atualizar, regenere o arquivo único (o script de build está descrito em METODOLOGIA.md; na prática é embutir os JSONs de `data/` numa tag `<script>window.EMBEDDED_DATA={...}</script>` antes do script principal do `index.html`).

## O que o painel entrega

- **Visão geral** — último concurso, situação de acumulação, prêmio estimado, e a leitura de oportunidade (prêmio ÷ ponto de equilíbrio).
- **Volante** — mapa de calor das dezenas por frequência ou atraso.
- **Mais & menos** — rankings e maiores atrasos.
- **Conjuntos 2–6** — quais combinações mais e menos saíram juntas, e quantas nunca saíram.
- **Meu jogo** — escolha as dezenas e veja, ao vivo: histórico de cada uma, matriz de co-ocorrência dos seus números, índice de exclusividade, sugestões de troca de 1 a 3 dezenas, comparação de custo entre "1 jogo de 7" e "3 de 6", e análise dos seus números favoritos.
- **Gerador** — jogos anti-popularidade.
- **Nosso método** — o método CRIVO e a tabela comparativa com todos os modelos existentes.
- **Evidências & testes** — os backtests walk-forward, o qui-quadrado com correção, o teste de estabilidade temporal e a prova brasileira da seleção consciente.
- **Financeiro** — carga da planilha oficial da Caixa para análise de arrecadação e rateios.
- **Risco** — o que a matemática garante e as regras práticas.

## O achado principal

Medimos, com dado oficial da Caixa, que sorteios da Mega-Sena com muitas dezenas ≤ 31 produzem quase o **dobro de ganhadores** por real arrecadado — e prêmio médio da quadra **81% menor**. É a prova direta de que os brasileiros concentram apostas em datas de aniversário, e a única vantagem real, mensurável e legítima disponível: fugir desse padrão não aumenta sua chance de ganhar, mas aumenta muito o quanto você levaria se ganhasse.

Detalhes completos em `METODOLOGIA.md`.

## Aviso

Loteria não é investimento. O retorno esperado é negativo por lei (43,79% da arrecadação volta em prêmios, dos quais ainda se retém 30% de IR). Este projeto serve para decidir melhor um gasto de entretenimento e para não cair em quem vende método.
