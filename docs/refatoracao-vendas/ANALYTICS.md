# Tédio Zero · Auditoria de analytics e plano GA4

> Estado auditado: `src/index.html` (idêntico ao que está em produção em https://tedio-zero-lp.vercel.app/, conferido em 30/09/2026).
> Nenhum código foi alterado. As linhas citadas são do arquivo atual.

---

## 1. O que existe hoje

### 1.1 Ferramentas carregadas

| Ferramenta | ID | Onde é carregada | Observação |
|---|---|---|---|
| **Google Analytics 4 (gtag.js)** | **`G-ECWV0X3TMD`** | `src/index.html:4-12` (no `<head>`, primeiro script da página) | `gtag('config', 'G-ECWV0X3TMD')` na linha 11, sem parâmetros extras |
| `dataLayer` | — | `src/index.html:7-8` | Criado só pelo snippet do gtag. **Não há GTM** (nenhum `GTM-XXXX`) |
| **Vercel Web Analytics** | — | `src/index.html:721-724` | Fila `window.va` (linha 722) + `/_vercel/insights/script.js` (linha 724). Nenhum evento customizado `va('event')` |
| **Meta Pixel** | — | **não existe** | ⚠️ Nenhum `fbq`, embora o tráfego venha de Meta Ads |
| Google Tag Manager | — | **não existe** | — |
| Plataforma de checkout | Hotmart, produto `W107748719N`, oferta `off=hsi91u8h`, `checkoutMode=10` | 11 links `<a href="https://pay.hotmart.com/...">` | Todos abrem em nova aba (`target="_blank"`) |

### 1.2 Eventos disparados hoje

| Evento | Tipo | Parâmetros | Gatilho / elemento | Arquivo:linha |
|---|---|---|---|---|
| `page_view` | Automático (gtag `config`) | padrão GA4 (`page_location`, `page_title`, `page_referrer`…) | Carregamento da página | `src/index.html:11` |
| `session_start`, `first_visit`, `user_engagement` | Automáticos GA4 | padrão | Início de sessão / engajamento | `src/index.html:11` (implícito) |
| `scroll`, `click` (saída), etc. | **Medição otimizada** (depende da configuração da propriedade) | `percent_scrolled: 90`; `link_url`, `link_domain`, `outbound: true` | 90% de rolagem; clique em link para outro domínio (pay.hotmart.com) | Não está no código. [VERIFICAR em Admin → Fluxos de dados → Medição otimizada] |
| **`cta_hotmart_click`** | Customizado | `location` (valor de `data-cta-location` ou `'unknown'`), `product: 'tedio_zero'`, `price: 9.9`, `currency: 'BRL'` | Clique em qualquer `a[href*="pay.hotmart.com"]` (11 botões) | `src/index.html:704-717` (chamada `gtag('event', …)` na linha 709) |
| (FAQ) | — | — | Abrir/fechar pergunta **não gera evento** | `src/index.html:651-672` (só UI) |

**Valores de `location` hoje** (atributo `data-cta-location` nos botões):

| `data-cta-location` | Texto do botão | Linha |
|---|---|---|
| `hero` | Quero 30 brincadeiras para hoje | 323 |
| `problem` | idem | 366 |
| `benefits` | idem | 430 |
| `origin` | idem | 447 |
| `gallery` | idem | 486 |
| `steps` | idem | 521 |
| `price` | idem | 541 |
| `whom` | idem | 572 |
| `trust` | idem | 588 |
| `faq` | idem | 626 |
| `final` | idem | 638 |

### 1.3 Rastreamento de UTMs no link da Hotmart (`src/index.html:674-719`)

O script reescreve o `href` de cada botão quando a página carrega:

- Copia `utm_source`, `utm_medium` e `utm_campaign` da URL da página. **Se não houver, força `utm_source=meta`, `utm_medium=paid`, `utm_campaign=tedio_zero`** (linhas 676-680 e 691-694).
- `utm_content` = `utm_content` da URL, senão o valor de `data-cta-location` (linhas 695-697).
- `utm_term` é repassado se existir (linhas 698-700).

### 1.4 Problemas encontrados

| # | Problema | Impacto | Correção proposta |
|---|---|---|---|
| 1 | **Não há Meta Pixel / API de Conversões** | O Meta Ads não otimiza para compra, e a atribuição da campanha fica cega | Instalar o Pixel (PageView, ViewContent, InitiateCheckout) + integração Pixel/CAPI da Hotmart para Purchase (seção 5) |
| 2 | **Tráfego sem UTM vira `meta / paid`** | Acessos diretos, orgânicos do Instagram e compartilhamentos são contados como mídia paga no relatório da Hotmart, o que infla o ROAS | Só usar o fallback `meta/paid` quando houver `fbclid` na URL. Caso contrário, deixar `utm_source=direct` ou não adicionar nada |
| 3 | `utm_content` sobrescreve o criativo | Quando o anúncio manda `utm_content=<criativo>`, o CTA clicado se perde (e vice-versa) | Criativo em `utm_content`, CTA no parâmetro `sck` da Hotmart |
| 4 | Não usa `src`/`sck` da Hotmart | Os parâmetros nativos da Hotmart aparecem nos relatórios de vendas e no webhook, e hoje estão vazios | Adicionar `src=<utm_source>` e `sck=<cta_location>` (seção 6) |
| 5 | Nome do evento `cta_hotmart_click` é específico da plataforma e o parâmetro `location` é genérico | Fica difícil trocar de checkout, e `location` pode confundir com dimensões de geografia | Novo padrão `cta_click` + `cta_location` (seção 2), mantendo o evento antigo por 30 dias |
| 6 | Usa `price` em vez de `value` | O GA4 só soma receita e valor de evento com o parâmetro `value` | Usar `value` + `currency` + `items[]` |
| 7 | Não há `begin_checkout` / `purchase` | Não há funil de e-commerce nem receita no GA4 | Seção 2 + integração Hotmart (seção 6) |
| 8 | Não há eventos de FAQ, de rolagem parcial ou de visualização de seção | Não dá para saber onde as pessoas desistem nem quais objeções pesam | `faq_open`, `scroll_depth`, `section_view` |
| 9 | Não há versão de página nos eventos | A comparação antes/depois da refatoração depende só de datas | Parâmetro global `page_version` |
| 10 | `pay.hotmart.com` não deve aparecer como referência | Quem volta do checkout abre uma nova sessão com origem "hotmart" | Incluir em "Referências indesejadas" (passo 4) |

---

## 2. Esquema de nomenclatura proposto

### 2.1 Convenções

- **Eventos:** `snake_case`, verbo no fim ou evento recomendado do GA4 (`view_item`, `begin_checkout`, `purchase`). Sem nome de fornecedor no evento (nada de `hotmart_` ou `meta_`).
- **Parâmetros:** `snake_case`, prefixados pelo objeto (`cta_*`, `section_*`, `faq_*`).
- **Valores:** `snake_case`, sem acento e em inglês curto, iguais aos `id` das seções.
- **No HTML:** todo CTA recebe atributos de dados. O JavaScript lê só esses atributos, sem depender de texto nem de classe CSS.

```html
<section id="offer" data-section="offer" data-section-index="9">
  ...
  <a class="btn"
     href="https://pay.hotmart.com/W107748719N?off=hsi91u8h&checkoutMode=10"
     target="_blank" rel="noopener"
     data-cta
     data-cta-location="offer"
     data-cta-id="offer_primary"
     data-cta-variant="price_in_button">Garantir meu acesso por R$9,90</a>
</section>
```

| Atributo | Obrigatório | Exemplo | Vira o parâmetro |
|---|---|---|---|
| `data-cta` | sim (seletor) | — | — |
| `data-cta-location` | sim | `hero`, `offer`, `sticky` | `cta_location` |
| `data-cta-id` | sim | `hero_primary`, `offer_primary`, `sticky_bar` | `cta_id` |
| `data-cta-variant` | opcional (testes) | `price_in_button` | `cta_variant` |
| `data-section` / `data-section-index` | sim nas `<section>` | `problem` / `3` | `section_id` / `section_index` |

### 2.2 Valores de `cta_location` (novos e mapeamento dos antigos)

| Novo `cta_location` | Seção | Antigo `location` |
|---|---|---|
| `topbar` | Barra de topo | — |
| `hero` | Hero | `hero` |
| `problem` | Problema | `problem` |
| `inside` | O que tem dentro | `benefits` |
| `gallery` | Galeria | `gallery` |
| `testimonials` | Depoimentos | — |
| `whom` | Para quem é / não é | `whom` |
| `offer` | Oferta + stack | `price` |
| `guarantee` | Garantia | `trust` |
| `how` | Como funciona | `steps` |
| `author` | Quem criou | `origin` |
| `faq` | FAQ | `faq` |
| `final` | CTA final | `final` |
| `sticky` | Barra fixa mobile | — |

### 2.3 Catálogo de eventos (depois da refatoração)

| Evento | Status | Quando dispara | Parâmetros | Evento-chave? |
|---|---|---|---|---|
| `page_view` | manter (automático) | Carregamento | padrão + `page_version`, `ab_variant` (via `gtag('set')`) | não |
| `view_item` | **criar** | 1× no carregamento | `currency: 'BRL'`, `value: 9.9`, `items: [ITEM]` | não |
| `section_view` | **criar** | 1× por seção, ao ficar ≥ 50% visível | `section_id`, `section_index` | não |
| `view_promotion` | **criar** (opcional) | 1× quando o bloco `offer` fica visível | `promotion_id: 'oferta_990'`, `promotion_name: 'Tédio Zero R$9,90'`, `creative_slot: 'offer'`, `items: [ITEM]` | não |
| `scroll_depth` | **criar** | 1× em cada marco de 25/50/75/90% | `percent_scrolled` (25, 50, 75, 90) | não |
| `cta_click` | **criar** (substitui `cta_hotmart_click`) | Clique em qualquer `[data-cta]` | `cta_location`, `cta_id`, `cta_text`, `cta_variant`, `section_index`, `link_domain` | opcional (secundário) |
| `begin_checkout` | **criar** | No mesmo clique que leva ao checkout | `currency`, `value: 9.9`, `items: [ITEM]`, `cta_location` | **sim** |
| `select_promotion` | **criar** (opcional) | Clique no CTA do bloco `offer` ou na barra `sticky` | `promotion_id`, `creative_slot` = `cta_location`, `items` | não |
| `faq_open` | **criar** | Abrir uma pergunta (não dispara ao fechar) | `faq_index`, `faq_question` (texto curto, até 100 caracteres) | não |
| `gallery_interact` | **criar** (opcional) | Hover/toque na pilha do hero ou clique numa imagem da galeria | `section_id`, `image_name` | não |
| `cta_hotmart_click` | **manter por 30 dias** e depois remover | Mesmo gatilho de `cta_click` | os atuais (`location`, `product`, `price`, `currency`) | não |
| `add_payment_info` | pela Hotmart (se houver integração GA4) | Dentro do checkout | padrão | não |
| `purchase` | **criar** (via Hotmart ou Measurement Protocol) | Compra aprovada | `transaction_id` (código da transação Hotmart), `value`, `currency`, `items`, `coupon` | **sim** |
| `refund` | **criar** (via webhook) | Reembolso/estorno | `transaction_id`, `value`, `currency` | não |

**Objeto `ITEM` padrão:**

```js
{
  item_id: 'tedio_zero_guia',
  item_name: 'Tédio Zero - Guia Completo',
  item_brand: 'Quintal de Dentro',
  item_category: 'ebook',
  price: 9.9,
  quantity: 1
}
```

Se o "Cartas na Manga" virar order bump, ele entra no `purchase` como segundo item (`item_id: 'tedio_zero_cartas'`).

### 2.4 Proposta de script (para a implementação, ainda não aplicado)

**No `<head>`**, entre `gtag('js', …)` e `gtag('config', …)` (linhas 9-11), para que o `page_view` automático também leve a versão e a variante:

```html
<script>
  gtag('set', {
    page_version: 'v2_2026_10',
    ab_variant: document.documentElement.dataset.abVariant || 'control'
  });
</script>
```

**No fim do `<body>`**, substituindo os scripts das linhas 674-719, com FAQ, rolagem e seções. Não depende de biblioteca.

```html
<script>
(function () {
  var ITEM = { item_id: 'tedio_zero_guia', item_name: 'Tédio Zero - Guia Completo',
               item_brand: 'Quintal de Dentro', item_category: 'ebook', price: 9.9, quantity: 1 };
  var qs = new URLSearchParams(location.search);

  function track(name, params) {
    try { if (typeof gtag === 'function') gtag('event', name, params || {}); } catch (e) {}
  }

  track('view_item', { currency: 'BRL', value: 9.9, items: [ITEM] });

  function decorate(link) {
    var url;
    try { url = new URL(link.href); } catch (e) { return; }
    var isMeta = qs.has('fbclid');
    var src = qs.get('utm_source') || (isMeta ? 'meta' : 'direct');
    var set = function (k, v) { if (v && !url.searchParams.has(k)) url.searchParams.set(k, v); };
    set('utm_source', src);
    set('utm_medium', qs.get('utm_medium') || (isMeta ? 'paid' : 'none'));
    set('utm_campaign', qs.get('utm_campaign') || 'tedio_zero');
    set('utm_content', qs.get('utm_content'));
    set('utm_term', qs.get('utm_term'));
    set('src', src);
    set('sck', link.dataset.ctaLocation || 'unknown');
    link.href = url.toString();
  }

  document.querySelectorAll('[data-cta]').forEach(function (link) {
    decorate(link);
    link.addEventListener('click', function () {
      var loc = link.dataset.ctaLocation || 'unknown';
      var section = link.closest('[data-section]');
      track('cta_click', {
        cta_location: loc,
        cta_id: link.dataset.ctaId || loc,
        cta_text: (link.textContent || '').trim().slice(0, 100),
        cta_variant: link.dataset.ctaVariant || 'default',
        section_index: section ? Number(section.dataset.sectionIndex) : null,
        link_domain: 'pay.hotmart.com'
      });
      track('begin_checkout', { currency: 'BRL', value: 9.9, items: [ITEM], cta_location: loc });
      if (loc === 'offer' || loc === 'sticky') {
        track('select_promotion', { promotion_id: 'oferta_990', creative_slot: loc, items: [ITEM] });
      }
      track('cta_hotmart_click', { location: loc, product: 'tedio_zero', price: 9.9, currency: 'BRL' });
    });
  });

  document.querySelectorAll('.faq-q').forEach(function (btn, i) {
    btn.addEventListener('click', function () {
      if (btn.closest('.faq-item').classList.contains('open')) {
        track('faq_open', { faq_index: i + 1, faq_question: btn.textContent.replace('+', '').trim().slice(0, 100) });
      }
    });
  });

  var marks = [25, 50, 75, 90], sent = {};
  window.addEventListener('scroll', function () {
    var h = document.documentElement;
    var pct = (h.scrollTop + window.innerHeight) / h.scrollHeight * 100;
    marks.forEach(function (m) {
      if (pct >= m && !sent[m]) { sent[m] = true; track('scroll_depth', { percent_scrolled: m }); }
    });
  }, { passive: true });

  if ('IntersectionObserver' in window) {
    var io = new IntersectionObserver(function (entries) {
      entries.forEach(function (en) {
        if (!en.isIntersecting) return;
        var s = en.target;
        track('section_view', { section_id: s.dataset.section, section_index: Number(s.dataset.sectionIndex) });
        if (s.dataset.section === 'offer') {
          track('view_promotion', { promotion_id: 'oferta_990', promotion_name: 'Tédio Zero R$9,90',
                                    creative_slot: 'offer', items: [ITEM] });
        }
        io.unobserve(s);
      });
    }, { threshold: 0.5 });
    document.querySelectorAll('[data-section]').forEach(function (s) { io.observe(s); });
  }
})();
</script>
```

Notas:
- O listener do FAQ precisa ser registrado **depois** do script que alterna a classe `open` (linhas 651-672), para ler o estado já atualizado.
- O fallback `utm_medium=none` para tráfego sem UTM e sem `fbclid` evita inflar o "meta/paid". [DECIDIR se prefere não adicionar UTM nenhuma nesses casos.]
- Seções muito altas (ex.: FAQ aberto no mobile) podem nunca chegar a 50% visíveis. Se isso acontecer nos testes, usar `threshold: 0.25` ou `rootMargin`.

---

## 3. Meta Pixel (fora do GA4, mas crítico)

O tráfego pago vem do Meta, e sem pixel a campanha não otimiza para compra.

| Evento Meta | Quando | Onde |
|---|---|---|
| `PageView` | Carregamento | Página (base code do pixel) |
| `ViewContent` | Carregamento | Página, `{content_ids:['tedio_zero_guia'], content_type:'product', value:9.9, currency:'BRL'}` |
| `InitiateCheckout` | Clique em `[data-cta]` | Página, junto com `begin_checkout` |
| `Purchase` | Compra aprovada | **Hotmart**: Ferramentas → Pixel de rastreamento → Meta, com o mesmo Pixel ID + API de Conversões (se disponível na conta) |

[PREENCHER Pixel ID]. Usar o mesmo `event_id` em pixel e CAPI para deduplicar, quando a Hotmart permitir.

---

## 4. Passo a passo no GA4 (depois da refatoração)

### Antes do deploy

**Passo 1. Registrar a linha de base (antes/depois).**
Em **Explorar → Formato livre**, com o período dos últimos 28 dias antes do deploy, anotar:
- Usuários, sessões e taxa de engajamento da página;
- Contagem de `cta_hotmart_click` e **taxa de clique** = usuários com `cta_hotmart_click` ÷ usuários;
- Cliques por `location` (se o parâmetro já estiver registrado; senão, só o total);
- Vendas e receita do mesmo período no painel da Hotmart (fonte oficial de receita hoje);
- Divisão por dispositivo (mobile/desktop).

Salvar em uma planilha. Este é o "antes".

**Passo 2. Retenção de dados.**
**Admin → Coleta e retenção de dados → Retenção de dados → 14 meses.** O padrão de 2 meses apaga dados das explorações e impede a comparação antes/depois.

**Passo 3. Revisar a medição otimizada.**
**Admin → Fluxos de dados → [fluxo web] → Medição otimizada:**
- Manter **Visualizações de página** e **Cliques de saída**;
- **Rolagem:** pode manter (o `scroll` de 90% continua existindo em paralelo ao `scroll_depth`) ou desligar para evitar duplicidade no marco de 90%;
- Desligar **Interações de formulário** (não há formulário e isso gera ruído).

**Passo 4. Domínios e referências.**
**Admin → Fluxos de dados → [fluxo] → Configurar tag → Mostrar mais:**
- **Configurar seus domínios:** adicionar `tedio-zero-lp.vercel.app`, [PREENCHER domínio próprio, se houver] e `pay.hotmart.com` (só faz efeito se o checkout da Hotmart carregar o mesmo ID GA4, ver passo 10);
- **Listar referências indesejadas:** `pay.hotmart.com`, `hotmart.com`, `consumer.hotmart.com`.

**Passo 5. Registrar as dimensões personalizadas.**
**Admin → Definições personalizadas → Criar dimensão personalizada** (escopo **Evento**). As dimensões **não são retroativas**, então é preciso criá-las **antes** do deploy.

| Nome da dimensão | Parâmetro | Escopo |
|---|---|---|
| CTA · Local | `cta_location` | Evento |
| CTA · ID | `cta_id` | Evento |
| CTA · Texto | `cta_text` | Evento |
| CTA · Variante | `cta_variant` | Evento |
| Seção · ID | `section_id` | Evento |
| FAQ · Pergunta | `faq_question` | Evento |
| Versão da página | `page_version` | Evento |
| Variante A/B | `ab_variant` | Evento (ou Usuário, se a variante for fixa por usuário) |
| CTA · Local (legado) | `location` | Evento (para ler o `cta_hotmart_click` antigo) |

Métricas personalizadas (opcional): `section_index` e `faq_index` como métricas "Padrão", se quiser médias.

`percent_scrolled` já existe como dimensão padrão ("Porcentagem rolada"). Confirmar em Definições personalizadas; se não aparecer para o evento customizado, registrar como `scroll_depth · %`.

### No deploy

**Passo 6. Anotação da data.**
Em **Relatórios**, usar o recurso de **anotações** do GA4 (ícone no gráfico de séries temporais) com o texto "Deploy refatoração página de vendas v2", na data e hora do deploy. Como reforço, o parâmetro `page_version` separa os dados mesmo em dias de transição.

**Passo 7. Validar no DebugView.**
1. Abrir a página com `?debug_mode=1` **e** a extensão *Google Analytics Debugger* ativa, ou pelo [Tag Assistant](https://tagassistant.google.com/) conectado à URL de produção;
2. **Admin → DebugView** e checar, na ordem:
   - `page_view` com `page_version = v2_2026_10`;
   - `view_item` com `items` e `value`;
   - rolar a página: `section_view` (uma vez por seção) e `scroll_depth` 25/50/75/90;
   - abrir 2 perguntas do FAQ: `faq_open` com `faq_question`;
   - clicar num CTA de cada tipo (hero, offer, sticky): `cta_click` + `begin_checkout` (+ `select_promotion` no offer/sticky) + `cta_hotmart_click`;
   - no link aberto, confirmar `src`, `sck`, `utm_*` na URL da Hotmart;
3. Testar no celular real (não só no emulador), com e sem `fbclid` na URL, e confirmar os UTMs;
4. Conferir em **Relatórios → Tempo real** nos 30 minutos seguintes ao deploy.

### Depois do deploy

**Passo 8. Marcar os eventos-chave.**
**Admin → Eventos-chave** (ou **Eventos → marcar como evento-chave**):
- `begin_checkout`: **evento-chave** (conversão de página, disponível desde o dia 1);
- `purchase`: **evento-chave** (assim que a integração Hotmart estiver ativa);
- **Não** marcar `cta_click` e `begin_checkout` ao mesmo tempo como eventos-chave para importação no Google Ads, porque isso conta o mesmo clique duas vezes. `cta_click` fica como métrica de análise.
- O evento só aparece na lista de eventos depois de ser recebido pela primeira vez; se preferir, criar antes em **Eventos-chave → Novo evento-chave** digitando o nome.

**Passo 9. Funil de exploração.**
**Explorar → Exploração de funil**, com o nome "Funil página de vendas v2":

| Etapa | Condição |
|---|---|
| 1. Visitou | `page_view` e `page_version = v2_2026_10` |
| 2. Viu o problema | `section_view` com `section_id = problem` |
| 3. Viu o que tem dentro | `section_view` com `section_id = inside` |
| 4. Viu a oferta | `section_view` com `section_id = offer` (ou `view_promotion`) |
| 5. Clicou no CTA | `cta_click` |
| 6. Compra | `purchase` |

- Funil **aberto** (usuários podem entrar em qualquer etapa, porque há CTA no hero);
- **Divisão:** `cta_location`, `Categoria do dispositivo`, `Origem/mídia da sessão`, `ab_variant`;
- Mostrar o tempo decorrido entre as etapas.

Explorações complementares:
- **Formato livre "CTA por seção":** linhas = `cta_location`, valores = contagem de eventos `cta_click` e usuários. Mostra quais dobras realmente convertem;
- **Formato livre "Objeções":** linhas = `faq_question`, valor = contagem de `faq_open`. As perguntas mais abertas devem subir de posição no FAQ ou virar copy nas seções anteriores;
- **Profundidade de rolagem:** linhas = `percent_scrolled`, valor = usuários. Mostra onde está o maior abandono.

**Passo 10. Integração com a Hotmart (compra no GA4).**
Duas opções, que podem ser combinadas:

**A) Integração nativa de pixel da Hotmart** [VALIDAR as opções disponíveis na conta]
1. Hotmart → **Produtos → Tédio Zero → Ferramentas/Configurações → Pixel de rastreamento** (ou "Integrações de rastreamento");
2. Adicionar **Google Analytics 4** com `G-ECWV0X3TMD`;
3. Adicionar **Meta Pixel** com [PREENCHER Pixel ID] (seção 3);
4. Salvar e fazer uma compra real de teste (e depois pedir o reembolso) para validar no DebugView/Tempo real;
5. Limitação: o checkout roda em outro domínio, então a sessão pode ser separada da sessão da página. O passo 4 (domínios) ajuda se a Hotmart repassar o parâmetro `_gl`. Se a receita não se ligar à origem, usar a opção B.

**B) Webhook da Hotmart → GA4 Measurement Protocol (mais confiável)**
1. Na página, passar o `client_id` do GA4 para a Hotmart (ex.: `gtag('get', 'G-ECWV0X3TMD', 'client_id', cb)` e incluir num parâmetro de rastreamento aceito pela Hotmart, como `xcod` ou dentro do `sck`) [VALIDAR qual parâmetro volta no payload do webhook];
2. **GA4 → Admin → Fluxos de dados → [fluxo] → Chaves secretas da API do Measurement Protocol → Criar** (guardar como variável de ambiente na Vercel, nunca no HTML);
3. Criar uma função serverless (ex.: `/api/hotmart-webhook` na Vercel) que:
   - valida o token do webhook da Hotmart (`hottok`);
   - no status **aprovado**, envia `purchase` com `client_id`, `transaction_id` = código da transação Hotmart, `value`, `currency: 'BRL'`, `items` (incluindo o order bump, se houver);
   - no status **reembolsado/estornado**, envia `refund` com o mesmo `transaction_id`;
4. Hotmart → **Ferramentas → Webhook (Postback)** → apontar para a URL da função, com os eventos de compra aprovada e reembolso;
5. Validar primeiro no servidor de validação `https://www.google-analytics.com/debug/mp/collect`;
6. Se as opções A e B estiverem ativas ao mesmo tempo, o mesmo `transaction_id` permite ao GA4 deduplicar o `purchase`. Mesmo assim, preferir **uma única fonte** de `purchase`.

**Passo 11. Padrão de UTMs para os anúncios.**
Parâmetros de URL no nível do anúncio no Meta Ads (usando as macros dinâmicas):

```
utm_source=meta&utm_medium=paid_social&utm_campaign={{campaign.name}}&utm_content={{ad.name}}&utm_term={{adset.name}}
```

- Nomear os anúncios com o gancho e o formato, ex.: `hook17h_reels_30s_v1`;
- Bio do Instagram: `utm_source=instagram&utm_medium=bio&utm_campaign=tedio_zero`;
- Stories orgânicos: `utm_source=instagram&utm_medium=stories&utm_campaign=tedio_zero&utm_content=<data>`;
- O CTA clicado vai no `sck` (a página adiciona sozinha), então `utm_content` fica livre para o criativo;
- No GA4, usar **Aquisição de tráfego** com a dimensão **Conteúdo manual da sessão** para comparar os criativos.

**Passo 12. Teste A/B de headline.**
- O Google Optimize foi descontinuado (setembro de 2023). A divisão precisa ser feita na própria página (JavaScript com sorteio 50/50 guardado em `localStorage`, ou Edge Middleware na Vercel);
- Gravar a variante em `document.documentElement.dataset.abVariant` **antes** do snippet do gtag, para que `ab_variant` vá em todos os eventos;
- Métrica principal: taxa de `begin_checkout` por usuário. Métrica final: `purchase` (ou vendas da Hotmart por `sck`/variante);
- Calcular a amostra antes de começar (ex.: calculadora de Evan Miller). Com uma taxa de clique base de ~10% e o objetivo de detectar +20% relativo, são necessários **~3.600 usuários por variante**. Só encerrar depois de atingir a amostra **e** de pelo menos 14 dias corridos (dois ciclos semanais);
- Testar uma variável por vez: primeiro a headline, depois o texto do CTA, depois a ordem das seções.

**Passo 13. Comparação antes/depois.**
Depois de 28 dias com a v2, repetir as medições do passo 1 com o mesmo intervalo de dias da semana:
- Taxa de clique no CTA (`cta_hotmart_click` antes ↔ `cta_click` depois; o evento legado continua disparando durante a transição e serve para validar que as contagens batem);
- Taxa de conversão página → venda (vendas Hotmart ÷ usuários GA4);
- Receita por visitante (RPV) = receita ÷ usuários, que também captura o efeito do order bump;
- Separar por dispositivo e por origem, porque mudanças no mix de tráfego do Meta podem mascarar o resultado;
- Referência externa: o *Conversion Benchmark Report* da Unbounce aponta uma mediana de ~6,6% de conversão em landing pages. Serve de orientação, mas a comparação que vale é **v1 contra v2 da própria página**.

**Passo 14. Limpeza (30 dias depois do deploy).**
- Remover `cta_hotmart_click` do código;
- Manter a dimensão "CTA · Local (legado)" para consulta histórica;
- Revisar as dimensões não usadas (limite de 50 por propriedade).

**Passo 15. Privacidade (LGPD).**
- Publicar a política de privacidade citando GA4, Meta Pixel e Vercel Analytics (também é exigência das políticas de anúncio do Meta);
- Para o público brasileiro, um aviso de cookies simples costuma ser suficiente. O *Consent Mode v2* só é obrigatório para usuários do Espaço Econômico Europeu, mas pode ser ativado depois sem mudar o esquema de eventos.

---

## 5. Checklist de implementação (resumo)

- [ ] Todas as `<section>` com `id`, `data-section` e `data-section-index`
- [ ] Todos os CTAs com `data-cta`, `data-cta-location` e `data-cta-id`
- [ ] Script de tracking da seção 2.4 no lugar das linhas 674-719
- [ ] Fallback de UTM corrigido (sem forçar `meta/paid` quando não há `fbclid`)
- [ ] `src` e `sck` nos links da Hotmart
- [ ] Meta Pixel instalado (PageView, ViewContent, InitiateCheckout)
- [ ] Pixel GA4/Meta ou webhook configurado na Hotmart para `purchase`
- [ ] Dimensões personalizadas criadas **antes** do deploy
- [ ] Retenção de dados em 14 meses
- [ ] Referências indesejadas configuradas
- [ ] Validação completa no DebugView (desktop e celular)
- [ ] Anotação criada na data do deploy
- [ ] `begin_checkout` e `purchase` marcados como eventos-chave
- [ ] Funil de exploração salvo
