# Design System — Painel Loterias

Direção: **sóbrio e analítico**. Julho de 2026.
Arquivos: `dashboard/tokens.css`, `dashboard/components.css`, `dashboard/styleguide.html`.

---

## Princípios

Quatro regras que decidem qualquer dúvida de design neste projeto. Quando duas soluções parecerem igualmente boas, é isto que desempata.

**1. O número é o herói; a interface recua.** Grade, eixos e bordas ficam no tom mais discreto que ainda funcione. Se um elemento de interface disputa atenção com o dado, ele está errado.

**2. Cor tem função, nunca decoração.** Cada cor faz um trabalho: identidade de série, magnitude, polaridade ou estado. Cor que não faz trabalho vira ruído — e cor nunca carrega significado sozinha.

**3. Honestidade visual.** Nada de eixo truncado, escala dupla ou destaque que sugira padrão onde há ruído. O painel existe para desmentir método de palpite; ele não pode mentir com pixel.

**4. Todo estado é declarado.** Repouso, hover, foco, ativo, desabilitado, carregando, vazio e erro. Estado esquecido é bug, não detalhe.

---

## Arquitetura em três camadas

| Camada | Arquivo | Papel |
|---|---|---|
| Primitivas | `tokens.css` §1 | Escalas cruas: espaço, tipo, raio, movimento. Nunca usadas direto no componente. |
| Semântica | `tokens.css` §2–3 | O que a cor *significa*: superfície, tinta, série, estado. Trocar de tema troca só esta camada. |
| Componentes | `components.css` | Card, botão, chip, aba, bola, barra, tabela, aviso. Só consomem tokens — nenhum valor cru. |

A regra que sustenta tudo: **um componente nunca escreve um hex ou um pixel**. Se precisou, falta um token.

---

## Cor

Paleta validada por script nos dois modos: banda de luminosidade, piso de croma, separação para daltonismo (protanopia, deuteranopia, tritanopia) e contraste contra a superfície. **Não altere um hex sem revalidar.**

Resultado da validação: todos os testes passam no claro e no escuro. No claro, três cores da série ficam abaixo de 3:1 contra a superfície — por isso a regra de alívio se aplica a elas (rótulo direto visível ou tabela alternativa).

### Os quatro trabalhos da cor

| Trabalho | Onde | Regra |
|---|---|---|
| **Identidade** (categórica) | séries de gráfico, chips | Ordem fixa, nunca ciclada. A cor segue a entidade, nunca a posição no ranking. A partir da nona série, agrupe em "outros" — nunca gere cor nova. |
| **Magnitude** (sequencial) | volante, mapa de calor | Uma cor só, claro→escuro. Azul para frequência, laranja para atraso. Nunca arco-íris. |
| **Polaridade** (divergente) | não usado hoje | Duas cores opostas com cinza neutro no meio. |
| **Estado** | avisos, selos | Reservadas. Nunca reutilizar como série. Sempre com ícone e rótulo. |

### Superfícies

Três planos de profundidade — `page`, `raised`, `overlay` — mais um recuado (`sunken`) para trilhos e campos. A elevação é curta e de baixa opacidade: a luz vem de cima e a sombra só separa planos, nunca decora.

### Modo escuro

Não é inversão automática. É um conjunto próprio de degraus, escolhido para a superfície escura e validado separadamente. As rampas sequenciais **invertem** (do escuro para o claro), e por isso o texto dentro de uma célula de mapa de calor precisa contrastar com o *swatch*, não com a página — o painel calcula isso e inverte junto com o tema.

---

## Tipografia

Uma família só, a sans do sistema — carrega instantaneamente e parece nativa em cada aparelho. Nove degraus, cada um com um trabalho definido.

| Token | Tamanho | Uso |
|---|---|---|
| `--text-3xl` | 38px | número herói |
| `--text-2xl` | 26px | número de card |
| `--text-xl` | 20px | título de seção |
| `--text-lg` | 17px | título de painel |
| `--text-md` | 15px | corpo em destaque |
| `--text-base` | 14px | corpo, leitura longa |
| `--text-sm` | 12,5px | nota, texto auxiliar |
| `--text-xs` | 11,5px | legenda, metadado |
| `--text-2xs` | 10,5px | rótulo de eixo |

**Algarismos tabulares** em coluna de tabela, rótulo de eixo e bola — para os dígitos alinharem. **Nunca** no número herói: ali o espaçamento proporcional é mais bonito e não há nada com que alinhar.

---

## Espaçamento

Base de 4px com escala **não-linear** — cresce como a percepção, não como a régua. Doze degraus de 2px a 80px. Só estes valores; nada de número solto.

Os mais usados: `--space-4` (8px) dentro de elementos pequenos, `--space-5` (12px) entre irmãos, `--space-6` (16px) dentro de painel, `--space-8` (24px) entre blocos, `--space-9` (32px) entre seções.

Detalhe que importa em gráfico: `--space-1` (2px) é o vão de superfície entre preenchimentos adjacentes — barras empilhadas e vizinhas — e a espessura do anel de separação em marcas sobrepostas.

---

## Forma e elevação

Objeto maior pede raio maior, senão parece quadrado.

`--radius-xs` 4px ponta de barra · `--radius-sm` 6px chip e célula · `--radius-md` 8px botão e campo · `--radius-lg` 12px painel e card · `--radius-xl` 16px modal · `--radius-full` bola.

Três níveis de elevação: painel em repouso, card em hover, modal e dica flutuante.

---

## Movimento

Rápido o bastante para não atrapalhar, lento o bastante para ser lido. **Movimento comunica causa e efeito — se não explica nada, remova.**

| Token | Duração | Uso |
|---|---|---|
| `--motion-instant` | 90ms | feedback de clique |
| `--motion-fast` | 140ms | hover, mudança de cor |
| `--motion-base` | 200ms | entrada de conteúdo |
| `--motion-slow` | 320ms | crescimento de barra |

Quatro curvas: `standard` para o caso geral, `enter` e `exit` para coisas que aparecem e somem, e `spring` — com leve passar do ponto — reservada ao feedback tátil, como a célula do volante que cresce sob o cursor.

Tudo respeita `prefers-reduced-motion`. Quem pediu menos movimento recebe transições instantâneas, não animações mais lentas.

---

## Estados de interação

Todo componente interativo declara os cinco: **repouso, hover, foco, ativo, desabilitado**. Além deles, toda tela declara **carregando, vazio e erro** — sem isso a interface parece quebrada.

O foco é visível e consistente: anel de 2px na cor da marca, com 2px de afastamento. **Nunca remover sem substituir** — é o que permite navegar por teclado.

Hover muda superfície e borda, nunca só a cor do texto. Ativo desce 1px, o que dá a sensação física de apertar.

---

## Acessibilidade

- Identidade nunca depende só de cor: legenda presente sempre que houver duas séries ou mais, rótulo direto em até quatro.
- Selo de estado sempre com ícone e rótulo — cor é reforço, não mensagem.
- Aviso identificado pela **borda de 3px à esquerda**, que sobrevive à impressão e ao modo de alto contraste.
- Texto veste tinta neutra, nunca a cor da série; a marca colorida ao lado é que carrega a identidade.
- Onde a cor fica abaixo de 3:1 contra a superfície, a regra de alívio obriga rótulo visível ou tabela alternativa.

---

## Como usar

No painel os dois arquivos estão embutidos, para o HTML continuar sendo um arquivo único. Para reutilizar em outro projeto:

```html
<link rel="stylesheet" href="tokens.css">
<link rel="stylesheet" href="components.css">
```

Abra `styleguide.html` para ver todos os tokens e componentes ao vivo, nos dois temas, com o movimento demonstrável no clique.

## Antes de mudar qualquer cor

Rode o validador com a paleta nova, nos dois modos, contra as superfícies reais:

```
node validate_palette.js "<hex,hex,…>" --mode light --surface "#fcfcfb"
node validate_palette.js "<hex,hex,…>" --mode dark  --surface "#1a1a19"
```

Se algum teste falhar, corrija antes de subir. Julgar paleta no olho é exatamente o erro que este projeto critica em outro contexto.
