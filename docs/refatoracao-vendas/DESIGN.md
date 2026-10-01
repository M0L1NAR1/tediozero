# Tédio Zero — DESIGN.md (refatoração da página de vendas)

> Especificação de UX/UI/CRO para a nova página de vendas do e-book **Tédio Zero** (Quintal de Dentro).
> Este documento **não** contém copy final: todos os textos aparecem como *slots* `{{secao.campo}}` que o copywriter preenche em `docs/refatoracao-vendas/COPY.md`. Implementação só depois que DESIGN.md + COPY.md estiverem fechados.

---

## 0. Contexto: stack e estado atual

### 0.1 Stack detectada

| Item | Situação atual |
|---|---|
| Framework / build | **Nenhum.** Um único arquivo estático `src/index.html` (~730 linhas) com CSS e JS inline. Sem `package.json` na raiz, sem bundler, sem Tailwind. |
| Hospedagem | Vercel (`src/.vercel/project.json` → `prj_ncNecPrqeLczcRCvUlEbbpK7ncpC`). Produção: `https://tedio-zero-lp.vercel.app/`. |
| Checkout | Hotmart (`pay.hotmart.com/W107748719N?off=hsi91u8h&checkoutMode=10`), abrindo em nova aba. |
| Analytics | GA4 (`G-ECWV0X3TMD`) via gtag + Vercel Insights. Evento custom `cta_hotmart_click` com `location` lido de `data-cta-location`. Script que propaga UTMs para o link do checkout (`utm_content` = local do CTA). |
| Fontes | Google Fonts via `<link>`: **Lora** (500, 700, 500i) para títulos e **Poppins** (400, 500, 600, 700) para corpo. 7 variações carregadas, sem preload, sem self-host. |
| Paleta atual (CSS vars) | `--terracota #D9714A`, `--salvia #6E8C6B`, `--mostarda #D9A441`, `--azul #5B7C99`, `--creme #FAF3EA`, `--tinta #2E2A25`, `--tinta-suave #4A4238`, `--linha #E7D9C8`, `--cinza-suave #8a7c6c`. |
| Assets | 9 JPGs em `src/assets/` (`gal_1..6`, `hero_front`, `hero_back1`, `hero_back2`), todos **720×893 px**, 38–56 KB. `hero_front.jpg` é duplicata de `gal_1.jpg`. Sem WebP/AVIF, sem `width/height`, sem `loading="lazy"`. |
| Produto | `Tédio Zero - Guia Completo.pdf` (4,6 MB) e `Tédio Zero - Cartas na Manga.pdf` (0,7 MB) na raiz do repo — o segundo parece ser um **bônus** ainda não mencionado na página. |
| Repositório | Branch `main` **sem nenhum commit**. Raiz poluída com ~120 arquivos temporários `_*.json/.txt/.py/.js` de tentativas de deploy via MCP. Pasta `videos/tedio-zero/` com criativo HyperFrames (Reels 9:16) que já reaproveita a paleta e as fontes da página. |

### 0.2 Arquivos da página

- `src/index.html` — página inteira (head, CSS inline, 11 seções, FAQ JS, UTM/GA JS).
- `src/assets/*.jpg` — ilustrações das brincadeiras.
- `src/.vercel/project.json` — vínculo com o projeto Vercel.

### 0.3 Estrutura atual (ordem das seções)

1. Nav (marca + @handle) → 2. Hero → 3. "O que você já tem em casa" → 4. Problema ("Cinco da tarde…") → 5. Benefícios (7 cards) → 6. Origem → 7. Galeria (6 ilustrações) → 8. Como funciona (3 passos) → 9. "Por que R$9,90" (fundo terracota) → 10. Para quem é (4 objeções) → 11. Compra segura → 12. FAQ (6 itens) → 13. CTA final → 14. Rodapé.

**11 CTAs idênticos** ("Quero 30 brincadeiras para hoje"), todos com a mesma microcopy, um por seção.

### 0.4 Medições (Playwright, produção, 30/09/2026)

| Métrica | Mobile 390×844 | Desktop 1366×800 |
|---|---|---|
| Altura total da página | **10.142 px** (~12 telas) | ~6.900 px |
| Altura do hero | 1.303 px | 1 tela |
| Topo do CTA do hero | 719 px (botão com 59 px, quebrado em 2 linhas, encostado na dobra) | dentro da dobra |
| Topo da imagem do produto | **868 px — abaixo da dobra** | dentro da dobra |
| CTAs para o checkout | 11 | 11 |

---

## 1. Diagnóstico UX/CRO (priorizado por impacto)

Legenda de impacto: 🔴 alto · 🟠 médio · 🟡 baixo. Cada item cita o princípio/estudo que o sustenta.

### 🔴 P1. Mobile: o produto não aparece acima da dobra e o CTA fica no limite
No mobile (onde chega a maior parte do tráfego de Instagram/TikTok/Meta Ads), a ordem do grid coloca a ilustração **depois** do botão, a 868 px do topo. O visitante vê 3 linhas de H1, 2 parágrafos, 5 chips, preço e um botão de 2 linhas — sem ver o que está comprando. Hero de 1.303 px = 1,5 tela só de texto.
*Princípio:* "above the fold" continua concentrando ~57% do tempo de visualização (NN/g, *Scrolling and Attention*, 2018); Unbounce e CXL recomendam oferta + produto + CTA na primeira tela em LPs de low ticket. Mockup do produto no hero aumenta a compreensão da oferta (Baymard: "product imagery is the #1 information source").

### 🔴 P2. Zero prova social
Não há nenhum depoimento, número de compradores, avaliação, print de conversa, menção de imprensa ou embed do Instagram. Para um produto de R$9,90 comprado por impulso, a ausência de prova social é a maior barreira de confiança depois do preço.
*Princípio:* prova social junto ao CTA é um dos maiores lifts documentados em testes (CXL Institute; Cialdini). Baymard: 95% dos usuários confiam em reviews para decidir.

### 🔴 P3. CTA sem isolamento de cor e com contraste insuficiente
A cor do botão (`#D9714A`) é a **mesma** dos eyebrows, kickers, bullets, ícones "+" do FAQ e do fundo inteiro da seção "Por que R$9,90". O botão não se destaca. Além disso, branco sobre `#D9714A` tem contraste **3,28:1** — reprova WCAG AA para o texto de 17 px do botão (mínimo 4,5:1).
*Princípio:* efeito Von Restorff / isolamento — o elemento que difere do entorno é o mais lembrado e clicado; a cor do CTA deve ser exclusiva dele (CXL, *Button Color*). WCAG 2.1 SC 1.4.3.

### 🔴 P4. Sem CTA sticky no mobile e 11 CTAs iguais como compensação
A página tenta compensar a falta de um botão persistente repetindo o mesmo botão 11 vezes, com o mesmo texto e a mesma microcopy. Isso alonga a página, cria "banner blindness" e não ajuda quem decide no meio de uma seção.
*Princípio:* CTA sticky em mobile aumenta cliques sem reduzir leitura (Baymard, *Mobile Checkout*; Google *Mobile UX Playbook*: "keep CTAs visible"). Repetição idêntica reduz atenção por habituação (NN/g, *Banner Blindness Revisited*).

### 🔴 P5. Ancoragem de preço fraca e sem stack de valor
"De R$69 por R$9,90" aparece pequeno (16 px riscado) no hero e como parágrafo na seção "Por que R$9,90". Não existe um bloco de oferta com: itens inclusos + valor individual + total riscado + preço final grande + garantia + selos de pagamento + CTA. O bônus (*Cartas na Manga*, PDF já existente no repo) não é sequer mencionado.
*Princípio:* ancoragem (Tversky & Kahneman) funciona melhor quando o ancorado é visual e decomposto (CXL, *Pricing Page Best Practices*). Bônus com valor percebido justificam o "de/por".

### 🟠 P6. Ordem narrativa dispersa e seções redundantes
Sequência atual: hero → "o que tem em casa" → problema → benefícios → origem → galeria → passos → preço → para quem → segurança → FAQ. O problema vem depois de uma seção de materiais; o mecanismo (por que funciona) está misturado em "origem"; "o que você já tem em casa" e "materiais que já estão em casa" (card de benefício) dizem a mesma coisa; a seção de passos ("Como funciona") é sobre o checkout, não sobre o produto. Não há seção "não é para você".
*Princípio:* hierarquia narrativa PAS/AIDA e leitura em padrão F/Z (NN/g, *F-Shaped Pattern*): problema → agitação → solução/mecanismo → prova → oferta → risco zero → FAQ → CTA final.

### 🟠 P7. Performance de carregamento (LCP/CLS) não otimizada
- Google Fonts em `<link>` (2 requisições encadeadas, 7 variações, ~150 KB) sem `preload` — FOIT/FOUT e atraso no LCP do H1.
- 9 imagens JPG sem `width/height` (CLS), sem `loading="lazy"` (todas carregam no primeiro byte), sem AVIF/WebP, sem `fetchpriority="high"` na imagem do hero, sem `srcset`.
- Hero visual usa 3 imagens sobrepostas com transforms + `will-change` permanentes (custo de compositing em celulares fracos).
- `scroll-behavior: smooth` global sem respeitar `prefers-reduced-motion`.
*Princípio:* Google/Chrome UX: LCP ≤ 2,5 s; cada 100 ms de atraso reduz conversão (Deloitte *Milliseconds Make Millions*: −8% em conversão por +0,1 s em retail). CLS > 0,1 gera cliques errados.

### 🟠 P8. Acessibilidade e contraste reprovando em vários pontos
Medidos (contraste vs fundo real):
- Eyebrows terracota sobre creme: **2,97:1** (reprova para 12 px maiúsculo).
- `--cinza-suave` (#8a7c6c) sobre creme: **3,68:1** — usado na microcopy de 12,5 px sob os CTAs, legendas da galeria, @handle.
- Texto legal do rodapé (#B0A18C): **2,29:1**.
- Tag "Pagamento único" (branco sobre sálvia): **3,73:1** em 11,5 px.
- Número dos passos (branco sobre mostarda): **2,25:1**.
- Perguntas "Para quem é" (azul sobre creme): **3,98:1**.
- FAQ: botões sem `aria-expanded`/`aria-controls`; o container `.stack` do hero tem `tabindex="0"` sem ser interativo (ruído para leitor de tela).
- Checkout em `target="_blank"` sem aviso — em mobile, nova aba quebra o fluxo e o botão "voltar".
*Princípio:* WCAG 2.1 AA (1.4.3, 4.1.2, 2.4.4). Baymard: abrir o checkout em nova aba prejudica a percepção de continuidade.

### 🟡 P9. Sem selos de confiança visuais perto do botão
A confiança está só em texto ("pagamento único · acesso imediato · 7 dias de garantia"). Não há ícones de Pix/cartão, cadeado, logo Hotmart, selo de garantia. A seção "Sem risco pra você" é um card genérico sem selo.
*Princípio:* Baymard — selos de segurança próximos ao campo/botão de pagamento aumentam a percepção de segurança mesmo quando não alteram a segurança real; CXL: "trust badges near CTA".

### 🟡 P10. Metadados e identidade incompletos
Sem favicon, sem `og:image`/`og:title` (o link compartilhado no WhatsApp/Instagram aparece sem imagem), sem `canonical`, sem `theme-color`. Nome do produto ("Tédio Zero") não aparece visualmente no hero — só "Quintal de Dentro" na nav.

### 🟡 P11. Botão quebra em duas linhas no mobile
"Quero 30 brincadeiras para hoje" em 17 px com padding 32 px não cabe em 350 px. Botão de 2 linhas parece erro e reduz área de leitura instantânea. Limite: ~22 caracteres para 1 linha em 100% da largura, ou reduzir para 16 px.

### 🟡 P12. Galeria estática e sem contexto de "página real"
As 6 ilustrações são bonitas, mas mostradas como cards soltos; não comunicam "isto é uma página do PDF" (idade, tempo, materiais, passo a passo). Prévia real de página (ou mockup do PDF aberto no celular) converte mais do que arte isolada (Baymard, "show the product in context").

---

## 2. Princípios de projeto (o que guia todas as decisões)

1. **Mobile-first, de verdade.** Layout, tipografia e ordem do DOM são desenhados para 360–430 px e só depois expandidos. Desktop é a exceção.
2. **Primeira tela fecha a venda para quem já quer.** Produto (mockup), promessa, preço ancorado e CTA — tudo visível sem rolar em um iPhone 14 (390×844) e em um Android médio (360×780).
3. **Um botão, uma cor.** A cor `--cta` aparece **somente** em botões de conversão. Nada mais na página usa essa cor.
4. **Prova onde há dúvida.** Prova social e selos aparecem *ao lado* de cada decisão (hero, preço, CTA final), não isolados em uma seção.
5. **Escaneável em 8 segundos.** Cada seção tem: eyebrow (contexto) → título (uma ideia) → 1–3 linhas → lista/visual. Nada com mais de 3 linhas de parágrafo no mobile.
6. **Ritmo visual alternado.** Fundos alternam creme / branco / tinta para criar "capítulos" e reduzir monotonia (padrão Z entre seções).
7. **Rápido antes de bonito.** Nada que empurre o LCP: sem vídeo autoplay, sem lib de animação, sem carrossel JS no hero.
8. **Acessível por padrão.** Contraste AA em todo texto, foco visível, alvos de toque ≥ 48 px, `prefers-reduced-motion`, semântica correta.

---

## 3. Design system

### 3.1 Direção estética

"**Cozinha às cinco da tarde**": quente, artesanal, acolhedor — sem parecer "template de infoproduto". Mantemos a identidade já reconhecida (creme, terracota, sálvia, Lora + Poppins), que também está no criativo em vídeo, e adicionamos:
- uma cor **exclusiva e mais profunda** para o CTA (para contraste e isolamento),
- um **fundo "tinta"** para o bloco de oferta (âncora visual da página),
- **pílulas com contorno** (herança do frame.md do vídeo) como assinatura de componente,
- textura leve de papel/grão no creme (CSS, sem imagem) para tirar o aspecto "flat de template".

### 3.2 Cores (tokens)

Todos os valores abaixo foram checados com a fórmula de contraste WCAG 2.1.

```css
:root {
  /* Superfícies */
  --bg:            #FAF3EA;  /* creme — fundo base */
  --bg-alt:        #FFFFFF;  /* seções alternadas / cards */
  --bg-ink:        #2E2A25;  /* bloco de oferta, CTA final */
  --bg-ink-soft:   #3A342E;  /* cards dentro de fundo ink */
  --line:          #E7D9C8;  /* bordas suaves */
  --line-strong:   #2E2A25;  /* contorno 2px das pílulas */

  /* Texto */
  --ink:           #2E2A25;  /* títulos e corpo — 12,9:1 sobre creme */
  --ink-2:         #5C5146;  /* texto secundário — 7,0:1 sobre creme */
  --ink-3:         #6B5E52;  /* microcopy/legendas — 5,7:1 sobre creme (AA em 12 px+) */
  --on-ink:        #FAF3EA;  /* texto sobre --bg-ink — 12,9:1 */
  --on-ink-2:      rgba(250,243,234,0.78);

  /* Marca (decorativa — NUNCA em botões de conversão) */
  --brand-terracota: #D9714A; /* ilustrações, glows, detalhes grandes (≥ 24 px) */
  --brand-terracota-soft: #F2B8A0; /* fundo de pílula com texto --ink — 8,2:1 */
  --brand-salvia:  #6E8C6B;  /* decorativa; para texto usar --salvia-text */
  --salvia-text:   #3E5A3C;  /* texto/ícone verde sobre creme — ≥ 7:1 */
  --brand-mostarda:#D9A441;  /* destaques, estrelas de avaliação, marcadores */
  --brand-mostarda-soft: #F3DFAE;

  /* CTA — cor EXCLUSIVA dos botões de conversão */
  --cta:           #B4451C;  /* "brasa" — branco sobre ela: 5,51:1 (AA normal) */
  --cta-hover:     #9A3412;  /* 7,31:1 */
  --cta-active:    #7F2B0F;
  --cta-text:      #FFFFFF;
  --cta-ring:      rgba(180,69,28,0.35); /* foco/halo */

  /* Semânticas */
  --success:       #2F6B4C;  /* garantia, check — branco sobre: 6,3:1 */
  --success-soft:  #DDEBE1;
  --warning-soft:  #FBE9D7;  /* aviso de escassez/limite, se houver */
}
```

**Justificativas**
- `--cta #B4451C` em vez do terracota atual: sobe o contraste de 3,28:1 para 5,51:1 (AA), continua na família cromática da marca e vira a cor **mais saturada e escura** da página → isolamento (Von Restorff). Todos os usos decorativos de terracota migram para `--brand-*` com regras: só em elementos grandes (ilustração, glow, faixa) ou com texto `--ink` sobre o tom *soft*.
- `--ink-2` e `--ink-3` substituem `--cinza-suave` (3,68:1) e o cinza do rodapé (2,29:1), que reprovavam AA.
- `--bg-ink` para o bloco de preço: o único bloco escuro da página, no ponto da decisão → cria a âncora visual do padrão Z e faz o botão claro sobre fundo escuro "saltar".
- Eyebrows deixam de ser terracota (2,97:1) e passam a `--salvia-text` ou `--ink-2` em caixa alta com tracking.

**Regra de ouro:** se um elemento não é um botão de compra, ele **não pode** usar `--cta`, `--cta-hover` ou `--cta-active`. Lint visual antes de publicar: buscar `#B4451C` no CSS e confirmar que só aparece em `.btn-cta`, `.sticky-cta` e `:focus-visible`.

### 3.3 Tipografia

Famílias mantidas (identidade + consistência com o vídeo): **Lora** (display) e **Poppins** (corpo). Reduzir para **3 arquivos woff2 self-hosted e subsetados (latin + latin-ext)**: Lora 700, Poppins 400, Poppins 600. Itálico via `font-synthesis: none` desativado e uso pontual de `<em>` só no H1 se a copy pedir (aí incluir Lora 700 italic — 4º arquivo, opcional).

Escala fluida (`clamp`) — mínimo mobile / máximo desktop:

| Token | Fonte | Tamanho (clamp) | Peso | Line-height | Uso |
|---|---|---|---|---|---|
| `--fs-display` | Lora | `clamp(2rem, 5.5vw + 1rem, 3.25rem)` → 32–52 px | 700 | 1.1 | H1 do hero (máx. 3 linhas no mobile) |
| `--fs-h2` | Lora | `clamp(1.625rem, 3vw + 1rem, 2.5rem)` → 26–40 px | 700 | 1.15 | Títulos de seção |
| `--fs-h3` | Lora | `clamp(1.25rem, 1vw + 1rem, 1.5rem)` → 20–24 px | 700 | 1.25 | Cards, bônus, itens de valor |
| `--fs-price` | Lora | `clamp(3rem, 8vw, 4.5rem)` → 48–72 px | 700 | 1 | Preço final |
| `--fs-lead` | Poppins | `clamp(1.0625rem, 0.4vw + 1rem, 1.25rem)` → 17–20 px | 400 | 1.5 | Subtítulo do hero, leads |
| `--fs-body` | Poppins | `1rem` (16 px fixo) | 400 | 1.6 | Corpo |
| `--fs-small` | Poppins | `0.875rem` (14 px) | 400/600 | 1.5 | Microcopy sob botão, legendas |
| `--fs-eyebrow` | Poppins | `0.75rem` (12 px) | 600 | 1.2 | Eyebrow, tracking `0.14em`, uppercase |
| `--fs-btn` | Poppins | `1.0625rem` (17 px) mobile / `1.125rem` (18 px) desktop | 600 | 1.2 | Texto de botão |

**Justificativas:** corpo em 16 px é o piso recomendado para leitura no mobile (Google Material / WCAG 1.4.4); nada abaixo de 12 px em lugar nenhum, e 12 px só em eyebrow com peso 600 e contraste ≥ 7:1. `clamp()` elimina saltos de breakpoint e mantém o H1 em ≤ 3 linhas de 390 a 1440 px. Largura de medida: parágrafos com `max-width: 60ch` (NN/g: 50–75 caracteres por linha).

### 3.4 Espaçamento, grid e container

```css
--space-1: 4px;  --space-2: 8px;  --space-3: 12px; --space-4: 16px;
--space-5: 24px; --space-6: 32px; --space-7: 48px; --space-8: 64px; --space-9: 96px;

--section-y: clamp(56px, 8vw, 96px);      /* padding vertical de seção */
--section-y-tight: clamp(40px, 6vw, 64px); /* seções curtas (prova social, faixa) */
--container: 1120px;                        /* largura máx. */
--container-narrow: 720px;                  /* texto corrido, FAQ */
--gutter: clamp(20px, 4vw, 32px);           /* padding lateral */
```

Breakpoints (min-width): `sm 480` · `md 768` · `lg 1024` · `xl 1280`. Hero vira 2 colunas em `lg`; grids de cards em 2 colunas em `md` e 3 em `lg`. Preferir **container queries** (`@container`) nos cards para que se adaptem em qualquer coluna.

*Justificativa:* espaçamento em escala de 4/8 px cria ritmo consistente; padding vertical ≥ 56 px no mobile separa "capítulos" e reduz densidade percebida (Unbounce: páginas com respiro convertem melhor que densas).

### 3.5 Raios

```css
--r-pill: 9999px;   /* botões, chips, tags */
--r-card: 20px;     /* cards */
--r-card-lg: 28px;  /* bloco de oferta, CTA final */
--r-media: 16px;    /* imagens */
--r-sm: 10px;       /* inputs, selos pequenos */
```

Botões sempre em pílula (continuidade com a marca e com o criativo em vídeo); cards em 20 px para não parecerem "bolha".

### 3.6 Sombras e elevação

```css
--shadow-card: 0 1px 2px rgba(46,42,37,.06), 0 8px 24px -12px rgba(46,42,37,.18);
--shadow-float: 0 24px 48px -20px rgba(46,42,37,.35);         /* mockup do e-book */
--shadow-cta:   0 12px 28px -10px rgba(180,69,28,.55);        /* halo do botão */
--shadow-cta-hover: 0 16px 32px -10px rgba(180,69,28,.6);
--shadow-sticky: 0 -8px 24px -12px rgba(46,42,37,.28);        /* barra sticky */
--outline-pill:  2px solid var(--line-strong);                /* pílulas "capsule" */
```

Duas famílias apenas: neutra (cards) e colorida (só CTA). O halo colorido reforça o isolamento do botão sem aumentar contraste de texto.

### 3.7 Iconografia e textura

- Ícones: **Lucide** (inline SVG, `stroke-width: 2`, 20–24 px), cor `--ink` ou `--salvia-text`. Nunca emoji (renderização inconsistente entre Android/iOS).
- Textura: `body::before` com grão via `radial-gradient` repetido a 3–4% de opacidade — CSS puro, zero requisição.
- Glows: 1–2 `radial-gradient` em terracota/mostarda a 8–12% no hero e no CTA final (herança do frame.md).

### 3.8 Componentes base

| Componente | Especificação |
|---|---|
| **Pill (chip)** | `--r-pill`, padding `6px 12px`, `--fs-small` 600, fundo `--bg-alt` ou `--brand-*-soft`, borda `1px --line` (ou `--outline-pill` na variante "capsule"), texto `--ink`. |
| **Card** | fundo `--bg-alt`, borda `1px --line`, `--r-card`, padding `20–24px`, `--shadow-card`. |
| **Card ink** | fundo `--bg-ink-soft`, borda `1px rgba(250,243,234,.12)`, texto `--on-ink`. |
| **Eyebrow** | `--fs-eyebrow`, uppercase, tracking `.14em`, cor `--salvia-text`, margem inferior `--space-3`. |
| **Section header** | eyebrow + h2 + lead (`--fs-lead`, `--ink-2`, `max-width 60ch`). Centralizado em seções de largura total; alinhado à esquerda em seções narrativas. |
| **Check list** | ícone `check` em círculo 24 px `--success` + texto `--fs-body`. |
| **Badge de garantia** | círculo/selo 72–96 px, borda dupla `--success`, texto "7 DIAS" em Lora 700 + "GARANTIA" em eyebrow. SVG inline. |
| **Selos de pagamento** | linha de ícones monocromáticos (`--ink-3`) 20 px: cadeado + "Pix" + "Cartão" + "Boleto" + logotipo Hotmart em cinza. Sempre abaixo do CTA principal do bloco de oferta e no CTA final. |
| **Estrelas** | 5 estrelas `--brand-mostarda` 16–18 px, com `aria-label="Avaliação 5 de 5"`. |
| **Accordion (FAQ)** | `<details>/<summary>` nativos, `summary` com 56 px de altura mínima, ícone `plus` rotacionando 45°, `name="faq"` para abrir um por vez, conteúdo com `--ink-2`. |

---

## 4. Wireframe textual — seção por seção

Convenções: **[M]** layout mobile (< 1024 px) · **[D]** desktop (≥ 1024 px) · `{{slot}}` = copy em COPY.md · `data-cta-location` indicado em cada botão. Fundos alternam para criar ritmo: creme → branco → creme → … com o bloco de oferta e o CTA final em `--bg-ink`.

### 4.0 Header (fixo, mínimo)
- Altura 56 px [M] / 64 px [D]. Fundo `--bg` com `backdrop-filter` leve ao rolar.
- Esquerda: marca "Quintal de Dentro" (Lora italic) + wordmark pequena "Tédio Zero". Direita [D]: CTA compacto `btn-cta btn-sm` "{{header.cta}}" `data-cta-location="header"` que **só aparece após o hero sair da tela** (mesmo observer do sticky mobile). No [M] a direita fica vazia ou mostra apenas "@quintal.de.dentro".
- *Por quê:* NN/g — header enxuto não compete com o hero; CTA de header só depois do hero evita duplicar o botão na primeira tela.

### 4.1 Hero — `section#hero` (fundo creme + glow)
Objetivo: em ≤ 5 s o visitante entende **o que é, para quem, quanto custa e onde clica**.

**[M] (ordem do DOM = ordem visual):**
1. Pill de contexto (`--brand-terracota-soft`): "{{hero.kicker}}" (ex.: guia digital · 2 a 4 anos).
2. **H1** `{{hero.headline}}` — máx. 8–10 palavras / 3 linhas em 390 px.
3. Sub `{{hero.sub}}` — 1–2 linhas, `--fs-lead`, `--ink-2`.
4. **Mockup do e-book** (ver §9 assets): imagem única com aspecto 4:3, `max-height: 36vh`, `fetchpriority="high"`, `width/height` definidos. É o **elemento LCP**.
5. Linha de preço: `De {{oferta.preco_ancora}}` riscado (`--ink-3`, 16 px) + `{{oferta.preco}}` (Lora 40 px) + pill "{{oferta.tag}}" (ex.: pagamento único).
6. **CTA primário** full-width 56 px `data-cta-location="hero"` — texto `{{cta.primario}}` (≤ 22 caracteres para 1 linha).
7. Microcopy 14 px com 3 ícones em linha: `{{cta.micro.1}}` · `{{cta.micro.2}}` · `{{cta.micro.3}}` (ex.: acesso imediato · 7 dias de garantia · Pix ou cartão).
8. Mini prova social (só se houver dado real — ver §9): avatares 3× 28 px sobrepostos + "★★★★★ {{prova.hero}}".
9. Link secundário `{{cta.secundario}}` (ex.: "ver o que tem dentro ↓") → `#conteudo`, `data-cta-location="hero-secondary"`.

Meta de dobra: base do CTA ≤ 720 px em 390×844 e ≤ 640 px em 360×740 (mockup encolhe via `clamp(180px, 36vh, 320px)`). Remove-se o segundo parágrafo e os 5 chips do hero atual (vão para "O que você recebe").

**[D]:** grid 2 colunas `1.05fr 0.95fr`, gap 64 px, alinhado ao centro. Esquerda: itens 1–3, 5–9. Direita: mockup grande (até 520 px) com leve rotação 3D (`perspective`) e sombra `--shadow-float`; ao lado, 2 "cards flutuantes" pequenos com prova (ex.: pill "30 brincadeiras" e pill de avaliação) posicionados sobre o mockup.
- *Por quê:* padrão Z — olho vai do H1 (topo esq.) ao mockup (topo dir.), descendo ao preço/CTA (base esq.). CTA e prova próximos (CXL). Mockup 3D comunica "produto real" (Baymard).

### 4.2 Faixa de prova social — `section#prova` (fundo branco, `--section-y-tight`)
- [M] linha rolável horizontal (scroll-snap, sem JS) de 3–5 **cards curtos**: avatar 40 px + nome + 1 frase `{{prova.itens[n]}}` + estrelas. [D] 3–4 colunas.
- Acima dos cards, uma linha de números **reais**: "{{prova.numero_1}}" · "{{prova.numero_2}}" (ex.: famílias atendidas, brincadeiras, avaliação média). **Sem dado real, esta linha não existe** — nunca inventar.
- Alternativa quando não houver depoimentos ainda: 3 prints de DM (Instagram/WhatsApp) em mockup de celular; ou substituir por "faixa de logos" de onde a marca já apareceu. Se nada existir, esta seção é **omitida no lançamento** e adicionada depois — não colocar placeholders visíveis.
- *Por quê:* prova social imediatamente após a promessa reduz ceticismo antes da leitura da dor (Cialdini; CXL *social proof placement tests*).

### 4.3 Dor / problema — `section#problema` (fundo creme)
- Eyebrow `{{problema.eyebrow}}` · H2 `{{problema.titulo}}` (gancho das 17h) · sem lead.
- Lista de **3–4 cenas** em cards com ícone à esquerda (Lucide: `clock`, `smartphone`, `utensils`, `search`), 1 frase cada `{{problema.itens[n]}}`. [M] empilhados; [D] grid 2×2.
- Frase-ponte em itálico Lora, borda esquerda `--brand-salvia`: `{{problema.ponte}}` ("não é falta de amor…").
- Sem CTA aqui (leitura ainda em fase de identificação — pedir venda cedo demais no meio da dor soa agressivo; o sticky já cobre quem quiser comprar).

### 4.4 Mecanismo / solução — `section#solucao` (fundo branco)
- Eyebrow `{{solucao.eyebrow}}` · H2 `{{solucao.titulo}}` · lead `{{solucao.lead}}`.
- **3 passos do mecanismo do produto** (não do checkout): "abre → escolhe pelo filtro (idade/tempo/material) → brinca". Cards numerados com número em Lora sobre círculo `--brand-mostarda-soft` com texto `--ink` (contraste ok).
- Ao lado [D] / abaixo [M]: **imagem de página real do PDF** (`gal_*` ou nova captura) anotada com 3 callouts (pills com contorno): "idade indicada", "tempo de preparo", "materiais". Isso resolve P12.
- Faixa de materiais (herda "o que você já tem em casa"): pills "papelão · copos · fita · bolinhas · papel" em uma linha.
- *Por quê:* mostrar o mecanismo transforma promessa em plausibilidade ("unique mechanism"); callouts em página real ensinam a ler o produto (Baymard, *in-context imagery*).

### 4.5 Conteúdo do e-book — `section#conteudo` (fundo creme)
- Eyebrow · H2 `{{conteudo.titulo}}` · lead `{{conteudo.lead}}`.
- **Grid dos 4 eixos** (coordenação, concentração, imaginação, aprendizagem): cards com ícone, título `{{conteudo.eixos[n].titulo}}`, 1 linha `{{conteudo.eixos[n].desc}}`, contagem de brincadeiras `{{conteudo.eixos[n].qtd}}` como pill. [M] 2×2 compacto; [D] 4 colunas.
- **Galeria** de 6 ilustrações (`gal_1..6`, convertidas para AVIF/WebP, `loading="lazy"`), com legenda `{{conteudo.galeria[n]}}`. [M] carrossel horizontal com scroll-snap e indicador de posição; [D] 3 colunas. Cada imagem com moldura "página" (borda 1 px + sombra) para ler como PDF.
- Lista curta "o que você recebe" (herda os 7 benefícios) reduzida a **5 checks** `{{conteudo.checks[n]}}`.
- **CTA primário** `data-cta-location="content"` + microcopy padrão.
- *Por quê:* seção longa mas escaneável; primeiro CTA depois do hero vem quando o visitante já viu o produto por dentro (Unbounce: CTA após "valor demonstrado").

### 4.6 Bônus — `section#bonus` (fundo branco)
- Eyebrow `{{bonus.eyebrow}}` · H2 `{{bonus.titulo}}`.
- Card horizontal em destaque: mockup do bônus **"Cartas na Manga"** (PDF já existe; precisa de mockup — §9) à esquerda, à direita título `{{bonus.itens[0].titulo}}`, descrição `{{bonus.itens[0].desc}}` (2 linhas), pill "Valor: {{bonus.itens[0].valor}}" riscado + pill `--success-soft` "Grátis hoje".
- Se houver mais bônus, repetir o card (máx. 3).
- *Por quê:* bônus com valor explícito alimenta a ancoragem do bloco de oferta seguinte (CXL *value stacking*).

### 4.7 Stack de valor + preço — `section#oferta` (fundo `--bg-ink`, `--r-card-lg` interno, glow terracota 10%)
O **centro visual da página**. Um único card grande centralizado (max-width 560 px), fundo `--bg-ink-soft`, borda sutil.
1. Eyebrow `--on-ink-2` `{{oferta.eyebrow}}`.
2. Mockup pequeno do e-book + bônus (imagem única, lazy).
3. Lista "o que está incluso" com valor à direita:
   - `{{oferta.itens[0]}}` ………… `{{oferta.itens[0].valor}}`
   - `{{oferta.itens[1]}}` (bônus) … `{{oferta.itens[1].valor}}`
   - `{{oferta.itens[2]}}` (acesso vitalício / atualizações) … `{{oferta.itens[2].valor}}`
   - Linha "Valor total" riscado `{{oferta.total}}` em `--on-ink-2`.
4. **Preço final** `{{oferta.preco}}` em `--fs-price` Lora, `--on-ink`, com "{{oferta.tag}}" (pagamento único) e, se aplicável, "{{oferta.parcelamento}}" (ex.: "ou 2× de…") em `--fs-small`.
5. **CTA primário** full-width 60 px `data-cta-location="pricing"` — este é o botão mais importante da página (halo `--shadow-cta`, único com `animation: pulse-ring` sutil, ver §6).
6. Microcopy 14 px `--on-ink-2` + **linha de selos de pagamento** (cadeado, Pix, cartão, boleto, Hotmart) em `--on-ink-2`.
7. **Selo de garantia** (SVG 80 px) com texto `{{garantia.curta}}` ao lado — dentro do card, logo abaixo dos selos.
- *Por quê:* ancoragem decomposta (Kahneman) + preço grande (saliência) + selos e garantia no mesmo bloco do botão (Baymard) + fundo escuro único que atrai o olho (isolamento também do bloco, não só do botão).

### 4.8 Garantia — `section#garantia` (fundo creme, curta)
- Card horizontal: selo de garantia 96 px à esquerda; à direita H3 `{{garantia.titulo}}` + 2 linhas `{{garantia.texto}}` (CDC 7 dias + "pede reembolso pela Hotmart, sem perguntas").
- Sem CTA (o bloco anterior acabou de ter um).
- *Por quê:* reversão de risco explícita e visual reduz ansiedade pós-preço.

### 4.9 Depoimentos — `section#depoimentos` (fundo branco)
- Eyebrow · H2 `{{depoimentos.titulo}}`.
- [M] carrossel scroll-snap; [D] grid 3 colunas (masonry simples com `columns: 3`). Cards: estrelas, aspas em Lora, texto `{{depoimentos.itens[n].texto}}` (máx. 3 linhas visíveis + "ler mais" nativo via `<details>` se longo), avatar 40 px + `{{depoimentos.itens[n].nome}}` + contexto ("mãe do Theo, 3 anos"). Prints reais de DM têm prioridade sobre texto digitado (percepção de autenticidade).
- **CTA primário** `data-cta-location="testimonials"` ao final.
- Se não houver depoimentos no lançamento: **omitir a seção** e manter apenas a faixa 4.2 quando esta tiver dado real.

### 4.10 Para quem é / não é — `section#para-quem` (fundo creme)
- H2 `{{paraquem.titulo}}`.
- Duas colunas [D] / empilhado [M]: card "É para você se…" (checks `--success`) com `{{paraquem.sim[n]}}` e card "Não é para você se…" (ícone `x` em `--ink-3`) com `{{paraquem.nao[n]}}`. 3–4 itens cada.
- *Por quê:* auto-qualificação aumenta confiança e reduz reembolso; a coluna "não é" gera credibilidade (CXL: honestidade percebida).

### 4.11 FAQ — `section#faq` (fundo branco, `--container-narrow`)
- H2 `{{faq.titulo}}`.
- 6–8 itens `<details name="faq">` com `<summary>` ≥ 56 px, ícone `plus`. Primeiro item pode vir aberto (objeção nº 1: formato/entrega).
- Marcação `FAQPage` em JSON-LD (SEO).
- **CTA primário** `data-cta-location="faq"` abaixo, com microcopy.

### 4.12 CTA final — `section#final` (fundo `--bg-ink`, glow, `--r-card-lg`)
- H2 `--on-ink` `{{final.titulo}}` · 1 linha `{{final.sub}}` com preço ancorado repetido ("De X por Y").
- CTA primário 60 px `data-cta-location="final"` + microcopy + selos de pagamento + mini selo de garantia.
- *Por quê:* último ponto de decisão para quem leu tudo; repetir preço evita rolar de volta.

### 4.13 Rodapé — `footer`
- Marca, @handle (link), e-mail de suporte `{{rodape.suporte}}`, links: Termos · Privacidade · "Reembolso em 7 dias" (âncora para FAQ).
- Disclaimer educativo em `--ink-3` 12 px (contraste 5,7:1 — corrige P8).
- Selo "Compra processada pela Hotmart".

### 4.14 Barra sticky mobile — `div#sticky-cta` (ver §5.3)

**Contagem final de CTAs para checkout:** header (D), hero, content, pricing, testimonials, faq, final + sticky (M) = **7 fixos + 1 sticky** (vs 11 hoje). Distribuídos onde a decisão acontece, não em toda seção.

---

## 5. Especificação dos componentes de CTA

### 5.1 `btn-cta` (primário)

```
Tamanho:     altura 56 px [M] / 60 px [D] no bloco de oferta e final; 52 px nos demais.
             Largura 100% no mobile (máx. 420 px), auto no desktop (min-width 280 px).
Toque:       área ≥ 48×48 px sempre (WCAG 2.5.5 / Material). Espaço ≥ 8 px de outros alvos.
Tipografia:  Poppins 600, 17 px [M] / 18 px [D], letter-spacing 0, 1 linha (≤ 22 caracteres).
Forma:       border-radius --r-pill; padding 0 28px; ícone opcional 20 px à direita (arrow-right).
Cor:         fundo --cta, texto --cta-text (5,51:1). Sombra --shadow-cta.
```

Estados:
| Estado | Estilo |
|---|---|
| default | `--cta`, `--shadow-cta` |
| hover (pointer: fine) | `--cta-hover`, `translateY(-1px)`, `--shadow-cta-hover`, ícone desloca 2 px |
| active / pressed | `--cta-active`, `translateY(0)`, sombra reduzida; `transform: scale(.98)` em touch |
| focus-visible | `outline: 3px solid var(--cta)`, `outline-offset: 3px`, mais `box-shadow 0 0 0 6px var(--cta-ring)` |
| loading (opcional, ao clicar) | texto vira "Abrindo pagamento…" + spinner 18 px; `aria-busy="true"`; evita duplo clique |
| disabled | não existe nesta página (CTA nunca desabilita) |

Microcopy obrigatória **imediatamente abaixo** (gap 10 px), 14 px, `--ink-3` (ou `--on-ink-2` sobre ink), com ícones 16 px inline: `{{cta.micro.1}}` · `{{cta.micro.2}}` · `{{cta.micro.3}}`. Quebra em 2 linhas centralizadas em ≤ 360 px.

Abaixo da microcopy nos CTAs `pricing` e `final`: linha de selos (cadeado + Pix + cartão + boleto + Hotmart), 20 px, monocromáticos, `aria-hidden` com texto alternativo "Pagamento seguro via Hotmart: Pix, cartão e boleto" em `.sr-only`.

Comportamento do link: `href` do checkout com UTMs (script existente mantido), **sem `target="_blank"`** (abrir na mesma aba — Baymard/NN/g: nova aba quebra fluxo e "voltar" no mobile). `rel="noopener"` fica desnecessário; manter `data-cta-location` e adicionar `data-cta-variant="primary"`.

### 5.2 `btn-secondary` (âncora interna / baixa intenção)
- Texto `--ink` 600, 16 px, fundo transparente, borda `1.5px --ink`, altura 48 px, pílula. Hover: fundo `--bg-alt`.
- Usado só no hero ("ver o que tem dentro ↓") e em eventual "voltar ao preço" no FAQ. Nunca com a cor `--cta`.
- `data-cta-location="hero-secondary"`, `data-cta-variant="secondary"`, `data-cta-type="scroll"`.

### 5.3 `sticky-cta` (barra fixa inferior — mobile e tablet < 1024 px)

```
Posição:  position: fixed; bottom: 0; inset-inline: 0; z-index: 50;
          padding: 10px var(--gutter) calc(10px + env(safe-area-inset-bottom));
Fundo:    --bg-alt (branco) com borda superior 1px --line e --shadow-sticky.
Layout:   grid 2 colunas: [preço] [botão]
          - Esquerda: "De {{oferta.preco_ancora}}" riscado 12 px --ink-3 em cima; "{{oferta.preco}}" Lora 22 px --ink embaixo; abaixo "7 dias de garantia" 11 px (opcional, só se couber sem quebrar).
          - Direita: btn-cta 52 px, texto curto {{cta.sticky}} (≤ 16 caracteres, ex.: "Quero o guia"), largura mínima 160 px.
Altura:   64 px + safe-area. `body { padding-bottom: 80px }` enquanto visível para não cobrir o rodapé.
```

Comportamento:
- **Aparece** quando o CTA do hero (`[data-cta-location="hero"]`) sai completamente da viewport pelo topo (IntersectionObserver, `threshold: 0`) — **não** por tempo, não por % de scroll.
- **Esconde** quando o `section#oferta` ou `section#final` estiverem ≥ 40% visíveis (evita botão duplicado na tela) e quando o teclado virtual estiver aberto (não há inputs, então irrelevante aqui).
- Entrada: `transform: translateY(100%) → 0` em 240 ms `ease-out`; saída inversa. Com `prefers-reduced-motion: reduce`, só toggle de `visibility`.
- Dispara evento `sticky_cta_shown` uma vez por sessão.
- `data-cta-location="sticky-mobile"`, `data-cta-variant="sticky"`.
- No desktop (≥ 1024 px) a barra não existe; o CTA compacto do header (§4.0, `data-cta-location="header"`) cumpre a função com o mesmo observer.

*Por quê:* Baymard e Google recomendam manter a ação primária alcançável com o polegar; disparar pela saída do CTA do hero (e não por tempo) evita duplicidade na primeira tela e respeita a leitura.

### 5.4 Regras de texto de botão (para o copywriter)
- Primeira pessoa + benefício ("Quero…", "Garantir…"), ≤ 22 caracteres no primário, ≤ 16 no sticky.
- Não usar "Comprar" isolado nem "Clique aqui". Não usar CAIXA ALTA no botão (legibilidade).
- Um só texto principal para todos os CTAs primários (consistência), com variação permitida apenas no `final` (fechamento) e no `sticky` (curto).

---

## 6. Animações e microinterações (permitidas)

Regras gerais: **CSS puro**, sem GSAP/Framer, sem animação no elemento LCP, tudo desligado com `@media (prefers-reduced-motion: reduce)`.

| Onde | O quê | Duração/easing | Observação |
|---|---|---|---|
| Hero (após load) | Fade-in + `translateY(8px→0)` em kicker, H1, sub, preço e CTA com stagger de 60 ms | 400 ms `cubic-bezier(.2,.7,.2,1)` | Só texto; **o mockup não anima** (é o LCP) |
| Mockup [D] | Leve `rotateY(-6deg)` estático + parallax de 4–6 px no `mousemove` | — | Opcional; nunca no mobile |
| Seções abaixo da dobra | Reveal `opacity 0→1` + `translateY(12px)` via `animation-timeline: view()` (CSS scroll-driven) com fallback para sem animação | 500 ms | Zero JS; navegadores sem suporte mostram estático |
| Botão `pricing` | `pulse-ring`: halo `box-shadow` expandindo a cada 3 s (1 ciclo de 1,2 s) | `ease-out`, 3 repetições no máximo | Somente no botão do bloco de oferta; para após interação |
| Hover em cards | `translateY(-2px)` + sombra um passo acima | 150 ms | Apenas `pointer: fine` |
| FAQ | Rotação do `plus` 45°; conteúdo com `grid-template-rows: 0fr→1fr` | 250 ms | `<details>` nativo com `interpolate-size: allow-keywords` onde suportado |
| Sticky | slide-up (ver §5.3) | 240 ms | — |
| Carrosséis [M] | `scroll-snap-type: x mandatory`, sem autoplay | — | Indicador de posição via `:has()`/`scroll-timeline` ou simples pontos estáticos |

Proibido: autoplay de vídeo no hero, contadores falsos, "pessoas comprando agora" fictício, confete, parallax pesado, bibliotecas de animação.

---

## 7. Performance (metas e receita)

**Metas (mobile, 4G lento simulado):** LCP ≤ 2,0 s · CLS ≤ 0,05 · INP ≤ 200 ms · peso total da primeira tela ≤ 350 KB · HTML+CSS inline ≤ 40 KB gzip · Lighthouse mobile ≥ 95.

### 7.1 Imagens
- Converter tudo para **AVIF + WebP** com fallback JPG via `<picture>`; qualidade 60–70. Meta: mockup do hero ≤ 60 KB em 800 px de largura; galeria ≤ 35 KB cada.
- `srcset` com 480/800/1200 px e `sizes` correto; `width` e `height` **sempre** declarados (CLS).
- Hero: `<img fetchpriority="high" decoding="async">` + `<link rel="preload" as="image" imagesrcset imagesizes>` no `<head>`. **Uma** imagem no hero (não três sobrepostas).
- Abaixo da dobra: `loading="lazy"` + `decoding="async"`. Galeria e mockups de bônus/oferta lazy.
- Remover `hero_front.jpg` (duplicata de `gal_1.jpg`).
- Avatares de depoimentos: 80×80 WebP ≤ 4 KB cada; ou iniciais em círculo colorido se não houver foto.

### 7.2 Fontes
- Self-host em `/assets/fonts/` (woff2, subset `latin`+`latin-ext`): `lora-700.woff2`, `poppins-400.woff2`, `poppins-600.woff2` (≈ 20–28 KB cada). Remover Google Fonts (`<link>` + 2 preconnects).
- `@font-face { font-display: swap; }` + `<link rel="preload" as="font" type="font/woff2" crossorigin>` só para Lora 700 e Poppins 400 (as que aparecem acima da dobra).
- Fallbacks com métrica ajustada (`size-adjust`, `ascent-override`) — Georgia para Lora, Arial para Poppins — para CLS ≈ 0 na troca.

### 7.3 CSS e JS
- Manter **CSS inline** no `<head>` (página única; evita 1 round-trip). Alvo ≤ 25 KB min. Sem framework, sem Tailwind runtime (o ganho de DX não compensa numa página só; se a equipe preferir Tailwind, usar build com purge e inline do resultado).
- JS mínimo (≤ 4 KB), `defer`, no fim do body: observer do sticky/header, UTM propagation (existente), eventos GA. Sem jQuery, sem Swiper.
- GA4 e Vercel Insights carregados com `defer`/`async` após o `load`, ou via Partytown se algum dia houver mais scripts. Manter `gtag` de saída.
- `content-visibility: auto` + `contain-intrinsic-size` nas seções abaixo da dobra.

### 7.4 HTML / meta
- `<meta name="theme-color" content="#FAF3EA">`, `favicon.svg` + `apple-touch-icon.png`, `og:title/description/image` (1200×630 com mockup e preço), `twitter:card`, `canonical`, JSON-LD `Product` (com `offers`) e `FAQPage`.
- `<html lang="pt-BR">` mantido; `scroll-behavior: smooth` apenas dentro de `@media (prefers-reduced-motion: no-preference)`.

---

## 8. Acessibilidade (WCAG 2.1 AA) — checklist de implementação

- [ ] Todo texto ≥ 4,5:1 (tokens de §3.2 garantem); texto sobre imagem só com overlay.
- [ ] Um único `<h1>`; hierarquia `h2` por seção, `h3` em cards.
- [ ] Botões de checkout são `<a>` com texto explícito; ícones decorativos `aria-hidden="true"`.
- [ ] FAQ com `<details>/<summary>` (semântica nativa, teclado gratuito). Se usar botões custom: `aria-expanded` + `aria-controls`.
- [ ] `:focus-visible` visível em todos os interativos (anel `--cta` 3 px); nunca `outline: none` sem substituto.
- [ ] Alvos ≥ 48×48 px; espaçamento ≥ 8 px.
- [ ] `alt` descritivo nas ilustrações ("Criança de 3 anos pescando bolinhas com colher em uma tigela — brincadeira de coordenação"); mockups com `alt` do produto; decorativos `alt=""`.
- [ ] Carrosséis com `role="region"` + `aria-label` e itens acessíveis por teclado (scroll-snap nativo mantém foco/tab).
- [ ] Sticky bar não cobre conteúdo focado: `scroll-padding-bottom: 88px` no `html` enquanto visível.
- [ ] `prefers-reduced-motion` respeitado (§6). `prefers-contrast: more` aumenta espessura de bordas.
- [ ] Remover `tabindex="0"` do container de imagens do hero.
- [ ] Zoom até 200% sem perda de conteúdo (nenhum `max-height` fixo em texto; `clamp` em tudo).

---

## 9. Assets necessários (o que falta produzir)

| # | Asset | Uso | Especificação | Status |
|---|---|---|---|---|
| A1 | **Mockup 3D do e-book** (capa do *Guia Completo* em formato "livro/tablet inclinado" ou "celular + páginas soltas") | Hero (LCP), oferta, OG image | PNG/AVIF com fundo transparente ou creme, 1600×1200 (4:3), sombra própria suave. Capa extraída da página 1 do `Guia Completo.pdf`. | **Falta** |
| A2 | **Mockup do bônus "Cartas na Manga"** | Seção bônus, stack de valor | Mesmo estilo de A1, 1200×900. Capa da página 1 do PDF do bônus. | **Falta** |
| A3 | **Página real do PDF com callouts** | Mecanismo/solução | Captura em alta de 1 página interna (ex.: Pescaria de Bolinhas) 1200×1500, sem marca d'água; callouts são pills em HTML, não na imagem. | **Falta** (as `gal_*` são só ilustrações) |
| A4 | **Depoimentos** (3 a 9) | Faixa de prova + seção depoimentos | Prints reais de DM/WhatsApp (com autorização, nome/foto borrados se pedido) ou texto + nome + idade da criança + foto/avatar. | **Falta** — não existe nenhum |
| A5 | **Números de prova** (compradores, avaliação média, seguidores) | Hero, faixa de prova | Só se reais e verificáveis (Hotmart, Instagram). | **Falta / confirmar** |
| A6 | **Foto/avatar da autora** (Quintal de Dentro) | Seção mecanismo ou garantia ("quem fez") | 400×400 WebP. Opcional, mas humaniza. | **Falta** |
| A7 | **Selos**: cadeado, Pix, cartão, boleto, Hotmart, garantia 7 dias | Sob CTAs `pricing` e `final` | SVG inline monocromático; garantia como SVG desenhado no design system. Logos Pix/Hotmart: usar versões oficiais em cinza. | **Falta** (produzir/baixar) |
| A8 | **Favicon + apple-touch-icon + og:image 1200×630** | `<head>` | SVG + PNG 180; OG com mockup A1, título e preço. | **Falta** |
| A9 | Ícones Lucide (clock, smartphone, utensils, search, check, x, shield-check, lock, arrow-right, plus, star, download, infinity) | Toda a página | Inline SVG, 24 px. | Disponível (open source) |
| A10 | Ilustrações existentes `gal_1..6`, `hero_back1/2` | Galeria, cards de eixos | Reconverter para AVIF/WebP em 480/800 px; descartar `hero_front.jpg` (duplicata). | Existe (otimizar) |
| A11 | Fontes self-hosted (Lora 700, Poppins 400/600 woff2 subset) | Global | Gerar via google-webfonts-helper ou `glyphhanger`. | **Falta** |

---

## 10. Rastreamento (`data-cta-location` e eventos)

Manter o script atual de UTM (`utm_content` = `data-cta-location`) e o evento `cta_hotmart_click`. Padronizar valores em kebab-case:

| Botão | `data-cta-location` | `data-cta-variant` |
|---|---|---|
| CTA compacto do header (desktop, pós-hero) | `header` | `primary` |
| CTA do hero | `hero` | `primary` |
| Link "ver o que tem dentro" | `hero-secondary` | `secondary` (+ `data-cta-type="scroll"`) |
| CTA após conteúdo do e-book | `content` | `primary` |
| CTA do bloco de oferta | `pricing` | `primary` |
| CTA após depoimentos | `testimonials` | `primary` |
| CTA após FAQ | `faq` | `primary` |
| CTA final | `final` | `primary` |
| Barra sticky mobile | `sticky-mobile` | `sticky` |

Eventos GA4 adicionais (todos ≤ 1 por sessão salvo indicado):
- `cta_hotmart_click` { location, variant, price: 9.9, currency: 'BRL' } — existente, adicionar `variant`.
- `sticky_cta_shown` — quando a barra aparece pela primeira vez.
- `pricing_view` — `section#oferta` ≥ 50% visível.
- `scroll_depth` { percent: 25|50|75|100 } — via IntersectionObserver em marcadores, não em `scroll` listener.
- `faq_open` { question } — por item (útil para o copywriter descobrir objeções).
- `secondary_scroll_click` — clique na âncora do hero.

Com isso o funil fica: view → pricing_view → cta_click (por local) → checkout (Hotmart) e dá para saber qual local converte por sessão, quanto do tráfego chega ao preço e se o sticky adiciona cliques incrementais.

---

## 11. Slots de copy que COPY.md precisa fornecer

Resumo dos `{{slots}}` referenciados acima, para alinhamento com o copywriter:

`header.cta` · `hero.kicker` · `hero.headline` · `hero.sub` · `oferta.preco_ancora` · `oferta.preco` · `oferta.tag` · `oferta.parcelamento?` · `cta.primario` (≤ 22 chars) · `cta.sticky` (≤ 16 chars) · `cta.secundario` · `cta.micro.1..3` · `prova.hero?` · `prova.numero_1..2?` · `prova.itens[]` · `problema.eyebrow/titulo/itens[3-4]/ponte` · `solucao.eyebrow/titulo/lead/passos[3]/callouts[3]/materiais[]` · `conteudo.titulo/lead/eixos[4]{titulo,desc,qtd}/galeria[6]/checks[5]` · `bonus.eyebrow/titulo/itens[]{titulo,desc,valor}` · `oferta.eyebrow/itens[]{nome,valor}/total` · `garantia.curta/titulo/texto` · `depoimentos.titulo/itens[]{texto,nome,contexto}` · `paraquem.titulo/sim[]/nao[]` · `faq.titulo/itens[6-8]{q,a}` · `final.titulo/sub` · `rodape.suporte/legal`.

Restrições de tamanho já indicadas nos slots (H1 ≤ 10 palavras; botão ≤ 22 caracteres; sticky ≤ 16; cards ≤ 2 linhas de 40 caracteres no mobile).

---

## 12. Plano de implementação e testes sugeridos

1. Produzir assets A1, A2, A7, A8, A11 (bloqueadores de layout); coletar A4/A5 (podem entrar na v1.1).
2. Implementar a página nova em `src/index.html` (mesma stack estática) seguindo §3–§8; manter scripts de UTM/GA.
3. QA: Lighthouse mobile ≥ 95; checar dobra em 360×740, 390×844, 430×932; axe DevTools sem violações AA; testar sticky em iOS Safari (safe-area) e Android Chrome.
4. Limpar a raiz do repo (arquivos `_*`), fazer o primeiro commit e configurar deploy contínuo na Vercel.
5. Testes A/B prioritários (em ordem): (a) texto do CTA; (b) sticky on/off; (c) preço grande no hero vs só no bloco de oferta; (d) headline dor vs headline benefício. KPI primário: taxa de clique para checkout por sessão; secundário: `pricing_view` / sessões.
