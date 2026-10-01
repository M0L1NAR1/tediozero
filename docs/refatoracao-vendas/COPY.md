# Tédio Zero · Nova copy da página de vendas

> Documento de copy para a refatoração de `src/index.html` (produção: https://tedio-zero-lp.vercel.app/).
> O código ainda não foi alterado. Este arquivo deve ser usado junto com a especificação de design e com `ANALYTICS.md`.

---

## 0. Diagnóstico da página atual (o que muda e por quê)

### Fatos do produto (fonte: PDFs e `src/index.html`)

| Item | Valor real |
|---|---|
| Produto | **Tédio Zero · Guia Completo** (PDF, 24 páginas), da marca Quintal de Dentro (@quintal.de.dentro) |
| Conteúdo | 30 brincadeiras em 4 grupos: Coordenação Motora (8), Concentração e Foco (7), Imaginação e Faz de Conta (8), Aprendizagem e Cognição (7) |
| Cada brincadeira traz | Idade, tempo de preparo, lista de materiais, "como brincar" em passos numerados e **"por que funciona"** |
| Extras dentro do guia | Página "Seu kit de casa" (checklist de materiais), 3 regras de uso, alertas de segurança (grãos pequenos, tinta) |
| Tempo de preparo | De 1 a 15 min. **7 brincadeiras ficam prontas em até 3 min** e **17 em até 5 min** |
| Faixa etária | **16 das 30 servem a partir dos 2 anos**; as outras 14 são de 3 a 4 anos |
| Segundo PDF existente | **Tédio Zero · Cartas na Manga** (8 páginas): 30 cartas para imprimir, recortar e sortear, versão resumida das mesmas 30 brincadeiras. **A página atual não cita esse material.** [CONFIRMAR se já é entregue ao comprador] |
| Preço | De R$69,00 por **R$9,90**, pagamento único |
| Checkout | Hotmart: `https://pay.hotmart.com/W107748719N?off=hsi91u8h&checkoutMode=10` (11 botões, todos com o mesmo link) |
| Garantia | 7 dias (a página atual apresenta como "conforme o CDC") |
| Entrega | PDF por e-mail e na tela de confirmação da compra |
| Tráfego | Meta Ads (Reels/Stories 9:16). O criativo usa o gancho **"17h / cinco da tarde"** |

### Problemas de conversão na página atual

1. **A headline não conversa com o anúncio.** O criativo abre com "cinco da tarde, a tela já passou do combinado", mas a página abre com "Menos telas. Mais brincadeiras. Mais momentos juntos.", que é genérica. A dor mais forte só aparece na 3ª dobra. Quebra de *message match* (Unbounce/Oli Gardner): quem clica num anúncio espera ver a mesma promessa na primeira tela.
2. **Não há nenhuma prova social.** Nem depoimento, nem número de compradores, nem print. Para tráfego frio e produto de uma marca pouco conhecida, esse é o maior buraco da página.
3. **A garantia está enfraquecida.** Dizer que a garantia existe "conforme o Código de Defesa do Consumidor" passa a ideia de uma obrigação legal, não de confiança no produto. Falta reversão de risco ativa.
4. **O ativo mais forte do guia não é vendido.** O "por que funciona" de cada brincadeira (desenvolvimento motor, foco, linguagem) não aparece em lugar nenhum da página. É ele que tira o produto de "lista de ideias" e o transforma em "estímulo com propósito", o que justifica o gasto e acalma a culpa de quem compra.
5. **Não existe stack de valor.** O preço aparece solto ("De R$69 por R$9,90") sem mostrar o que compõe o valor. O bônus Cartas na Manga, que é um ótimo item de stack ou de order bump, está parado.
6. **São 11 CTAs idênticos**, todos com "Quero 30 brincadeiras para hoje" e o mesmo microcopy. A repetição é boa, mas o texto não acompanha o contexto de cada seção.
7. **A seção "Para quem é" é, na verdade, quebra de objeções.** Não há "para quem não é", que qualifica o público e aumenta a credibilidade.
8. **A seção de origem não tem rosto nem nome.** "Quintal de Dentro" sem uma pessoa por trás gera pouca confiança.
9. **O FAQ é curto** (6 perguntas) e não responde às objeções de compra: forma de pagamento, se é assinatura, se o e-mail não chegar, se é seguro para 2 anos, se serve para mais de um filho.

---

## 1. Estrutura proposta (ordem das seções)

A ordem segue **PASTOR** (Problem, Amplify, Story/Solution, Transformation/Testimony, Offer, Response), com a oferta e o CTA também visíveis logo na primeira tela. Em low ticket com tráfego frio no celular, a decisão costuma acontecer cedo: segundo a NN/g, ~57% do tempo de visualização fica na primeira tela e ~74% nas duas primeiras.

| # | Seção | `id` / `data-cta-location` | Papel no funil |
|---|---|---|---|
| 0 | Barra de topo | `topbar` | Reforço da oferta, sem CTA grande |
| 1 | Hero | `hero` | Atenção + promessa + oferta + CTA |
| 2 | Faixa de prova | `proof_strip` | Credibilidade imediata |
| 3 | Problema (17h) | `problem` | Identificação e amplificação da dor |
| 4 | Virada / mecanismo | `mechanism` | Solução: "não falta amor, falta cardápio" |
| 5 | O que tem dentro (fascinação) | `inside` | Desejo por especificidade |
| 6 | Espie por dentro (galeria) | `gallery` | Prova visual do produto |
| 7 | Depoimentos | `testimonials` | Prova social (Transformation/Testimony) |
| 8 | Para quem é / para quem não é | `whom` | Qualificação + identificação |
| 9 | Oferta + stack de valor | `offer` | Ancoragem e fechamento |
| 10 | Garantia | `guarantee` | Reversão de risco |
| 11 | Como funciona (3 passos) | `how` | Redução de atrito pós-clique |
| 12 | Quem criou | `author` | Autoridade e simpatia |
| 13 | FAQ | `faq` | Quebra de objeções |
| 14 | CTA final | `final` | Última decisão (future pacing) |
| 15 | Rodapé | `footer` | Confiança legal / suporte |
| + | Barra fixa no mobile | `sticky` | CTA sempre ao alcance do polegar |

**Regras de CTA:**
- Há um CTA a cada dobra, sempre com o mesmo destino e o mesmo estilo visual. O texto varia pelo contexto, mas mantém o padrão "Quero…" (1ª pessoa).
- Todo botão leva microcopy de redução de atrito logo abaixo.
- Todo botão recebe `data-cta-location` (ver `ANALYTICS.md`).

---

## 2. Microcopy padrão abaixo dos botões

Usar em todos os CTAs, salvo quando a seção indicar outro texto:

> **Pagamento único de R$9,90 · Acesso imediato no e-mail · 7 dias de garantia**

Versão curta (barra fixa no mobile):

> **R$9,90 · acesso na hora · garantia de 7 dias**

Selo opcional sob o CTA do hero e da oferta:

> 🔒 Compra segura pela Hotmart · Pix ou cartão [CONFIRMAR formas de pagamento ativas na oferta]

**Justificativa:** a Baymard mostra que custo inesperado e desconfiança no pagamento estão entre as principais causas de abandono de checkout (a taxa média documentada de abandono de carrinho é de ~70%). Informar o valor final, a forma de entrega e a garantia *antes* do clique elimina essas surpresas. Citar a Hotmart transfere a confiança de uma plataforma conhecida para uma marca pequena.

---

## 3. Headline: 3 variações para teste A/B

Controle (atual): *"Menos telas. Mais brincadeiras. Mais momentos juntos."*

| Variante | Headline | Ângulo | Hipótese |
|---|---|---|---|
| **A (recomendada para abrir)** | **Cinco da tarde, a tela já passou do combinado e ainda faltam duas horas pro banho?** Aqui estão 30 brincadeiras prontas com o que você já tem em casa. | Dor + horário (igual ao anúncio) | O *message match* com o criativo de 17h reduz a rejeição e aumenta o clique no CTA do hero |
| **B** | **30 brincadeiras para crianças de 2 a 4 anos que ficam prontas em minutos,** com papelão, copo e fita que já estão na sua casa. | Especificidade + rapidez | Números e concretude ("30", "2 a 4", "minutos") passam credibilidade e são escaneáveis (NN/g recomenda usar numerais) |
| **C** | **Menos tela, sem culpa e sem Pinterest:** abra o guia, escolha uma brincadeira e comece em minutos. | Alívio emocional / identidade | Foca na culpa e no cansaço de quem cuida, que é o motor emocional da compra |

**Como testar:** uma variável por vez (só a headline), divisão 50/50, parâmetro `ab_variant` enviado ao GA4. Rodar por pelo menos 14 dias ou até atingir a amostra calculada (ver `ANALYTICS.md`, passo 12). Com o volume típico de low ticket, testar primeiro **A contra o controle** e só depois o vencedor contra B e C.

---

## 4. Copy seção por seção

### 0. Barra de topo · `topbar`

**Objetivo:** reafirmar a oferta em uma linha para quem chega do anúncio, sem roubar a atenção do hero.

**Texto (escolher um):**
- Opção honesta, sem prazo: **"Guia digital em PDF · 30 brincadeiras · R$9,90 com acesso na hora"**
- Só se houver prazo real: **"Preço de lançamento R$9,90 até [PREENCHER data real]"**

**Justificativa:** escassez e urgência aumentam a conversão (Cialdini), mas **só se forem verdadeiras**. Um contador falso que reinicia é publicidade enganosa (CDC, art. 37) e destrói a confiança quando é percebido. Na dúvida, use a opção sem prazo.

---

### 1. Hero · `hero`

**Objetivo:** em até 5 segundos, dizer o que é, para quem é, qual problema resolve, quanto custa e o que fazer em seguida.

**Kicker (acima da headline):**
> GUIA DIGITAL EM PDF · PARA CRIANÇAS DE 2 A 4 ANOS

**Headline:** variante A (ver seção 3)
> **Cinco da tarde, a tela já passou do combinado e ainda faltam duas horas pro banho?**

**Subheadline (mecanismo):**
> O **Tédio Zero** é um cardápio de **30 brincadeiras prontas**, cada uma com idade, tempo de preparo e passo a passo. Você abre no celular, escolhe em segundos e começa com papelão, copo e fita que já estão em casa. **17 delas ficam prontas em até 5 minutos.**

**Chips de oferta (substituem os atuais):**
> `30 brincadeiras` · `Prontas em 1 a 15 min` · `Materiais de casa` · `PDF no celular` · `Seu para sempre`

**Mini prova social (logo abaixo dos chips, antes do preço):**
> ★★★★★ "[PREENCHER frase curta de depoimento real, até 12 palavras]" · **[PREENCHER nome], mãe de [PREENCHER idade]**
> *Se ainda não houver depoimento:* "Criado pelo @quintal.de.dentro · [PREENCHER nº real de seguidores ou de compradores]"

**Bloco de preço:**
> ~~De R$69,00~~ **R$9,90** · `Pagamento único`
> *menos de R$0,34 por brincadeira*

**CTA:**
> **[ Quero as 30 brincadeiras por R$9,90 ]**

**Microcopy:**
> Pagamento único · Acesso imediato no e-mail · 7 dias de garantia
> 🔒 Compra segura pela Hotmart

**Visual:** manter a pilha de páginas ilustradas; se o design permitir, mostrar também um mockup de celular com o PDF aberto (reforça "abre no celular agora").

**Justificativa:**
- **Message match** com o anúncio de 17h (Unbounce): o visitante reconhece na hora que está no lugar certo.
- A headline em forma de **pergunta de identificação** ativa o "isso sou eu" (a etapa P do PAS) já na primeira linha.
- A subheadline entrega o **mecanismo** ("cardápio com idade e tempo de preparo"), que responde "por que isso funciona quando o Pinterest não funciona?".
- **Reenquadramento de preço por unidade** ("menos de R$0,34 por brincadeira"; R$9,90 ÷ 30 = R$0,33): efeito *pennies-a-day* (Gourville, 1998, *Journal of Consumer Research*). Um valor apresentado em unidades pequenas parece menor.
- A NN/g aponta que a maior parte da atenção fica na primeira tela, então a oferta completa precisa estar ali.
- **CTA em 1ª pessoa ("Quero…")**: testes publicados pela ContentVerve (Michael Aagaard) mostram ganho ao trocar "seu/sua" por "meu/minha" no botão. Colocar o preço no botão elimina a surpresa no checkout.

> ⚠️ **[CONFIRMAR] A âncora "De R$69,00" precisa ser verdadeira.** O produto deve ter sido vendido de fato a R$69 por um período razoável. Se não foi, trocar por uma ancoragem de valor ("valor de referência do pacote: R$[soma do stack]") ou comparativa ("menos que uma hora de brinquedoteca"). Preço "de/por" fictício é prática abusiva (CDC e Procon).

---

### 2. Faixa de prova · `proof_strip`

**Objetivo:** dar credibilidade logo depois do hero, antes de o visitante rolar a página.

**Formato:** uma faixa horizontal com 3 ou 4 itens.

> **[PREENCHER nº real] famílias já baixaram** · **★ [PREENCHER nota real]/5** · **24 páginas ilustradas** · **Garantia de 7 dias**

*Se ainda não houver números reais, usar apenas os fatos do produto:*
> **30 brincadeiras** · **4 áreas de desenvolvimento** · **24 páginas ilustradas** · **Prontas em 1 a 15 min**

**Justificativa:** prova social logo no topo reduz a incerteza de quem ainda não conhece a marca (Cialdini). Não inventar número: um contador falso prejudica a confiança e cria risco jurídico.

---

### 3. Problema · `problem`

**Objetivo:** fazer o visitante se reconhecer na cena e amplificar o custo de continuar como está (P + A do PAS).

**Eyebrow:** VOCÊ JÁ VIVEU ISSO?

**Headline:**
> **Cinco da tarde, a casa vira zona de guerra**

**Corpo:**
> O jantar não se faz sozinho. Seu filho também não brinca sozinho. E aquele "só mais um vídeo" já virou o terceiro.

**Bullets de dor (manter os atuais, que são bons, com pequenos ajustes):**
- A tela já passou do combinado e ainda faltam duas horas até o banho.
- O "não tenho ideia" bate justo na hora em que você mais precisa de vinte minutos de paz.
- Pinterest, YouTube e grupo de mães têm ideia demais, e nenhuma pronta pra usar agora.
- Você até tenta inventar algo, mas em cinco minutos ele já largou e voltou a pedir o celular.

**Amplificação (novo):**
> E aí vem a parte que ninguém fala: a culpa. A sensação de estar "terceirizando" seu filho pra uma tela, todo santo dia, só porque não sobrou energia pra pensar em outra coisa.

**Virada (fechamento da seção):**
> Não é falta de amor, nem de esforço. **É falta de uma ideia pronta na hora certa.**

**CTA:**
> **[ Quero ter o que fazer hoje às 17h ]**

**Microcopy:** padrão.

**Justificativa:** o PAS (Problem, Agitate, Solve) é a estrutura de resposta direta mais usada para produtos de dor. A cena específica (horário, jantar, banho) funciona melhor do que uma dor abstrata porque gera identificação. A culpa é a emoção que realmente move a compra nesse nicho; nomeá-la mostra empatia. A frase final **tira a culpa da pessoa** e a coloca na falta de ferramenta, o que abre espaço para a solução sem ofender.

---

### 4. Virada / mecanismo · `mechanism`

**Objetivo:** apresentar o Tédio Zero como uma ferramenta nova, e não como "mais uma lista de atividades".

**Eyebrow:** A DIFERENÇA

**Headline:**
> **Você não precisa de mais ideias. Precisa de um cardápio pronto.**

**Corpo:**
> Na internet sobra inspiração, mas falta o que importa às 17h: saber **o que dá pra fazer agora, com o que tem em casa, na idade do seu filho**.
>
> O Tédio Zero organiza isso por você. Cada brincadeira mostra logo no topo **a idade indicada e o tempo de preparo**, e as 30 estão separadas em 4 áreas: coordenação, concentração, imaginação e aprendizagem. É só abrir, bater o olho e escolher.

**Mini comparativo (3 linhas, formato "Antes / Com o Tédio Zero"):**

| Sem o guia | Com o Tédio Zero |
|---|---|
| 20 minutos rolando o Pinterest | Escolhe em segundos, pela idade e pelo tempo de preparo |
| Ideia que pede material que você não tem | Papelão, copo, fita, tampinha e bolinha |
| Brincadeira "só pra passar o tempo" | Cada uma explica **por que funciona** para o desenvolvimento |

**CTA:** não precisa (a próxima seção já é de desejo). Se o design pedir, usar o padrão da seção 5.

**Justificativa:** esse é o "mecanismo único" da escola de resposta direta (Eugene Schwartz, *Breakthrough Advertising*). Num mercado saturado de ideias grátis, é preciso explicar **por que este produto resolve o que o grátis não resolve**. A tabela antes/depois torna o contraste escaneável: a NN/g mostra que, na web, a maioria das pessoas escaneia em vez de ler.

---

### 5. O que tem dentro · `inside`

**Objetivo:** criar desejo com especificidade. Bullets de fascinação mostram o benefício e deixam uma curiosidade no ar.

**Eyebrow:** O QUE VOCÊ RECEBE

**Headline:**
> **30 brincadeiras prontas, e cada uma com um porquê**

**Subheadline:**
> Um PDF de 24 páginas ilustradas para abrir no celular ou imprimir. Veja um pouco do que tem lá dentro:

**Bullets de fascinação (usar todos ou selecionar de 6 a 8):**
- **A brincadeira de 2 minutos de preparo** que treina a mesma "pinça" dos dedos que, mais tarde, vai segurar o lápis *(Pescaria de Bolinhas)*
- **7 brincadeiras que ficam prontas em até 3 minutos**, para quando a paciência já acabou
- **O jogo de 1 minuto** que planta uma das primeiras bases do raciocínio lógico-matemático, usando só tampinhas *(Sequência de Cores)*
- **A garrafa que você monta em 10 minutos** e deixa por perto para os momentos de birra ou cansaço *(Garrafa Sensorial Calmante)*
- **A brincadeira de apagar a luz** que também serve pra acalmar antes de dormir *(Teatro de Sombras)*
- **Como uma caixa de papelão vira nave espacial, casa de bonecos ou labirinto**, e três tardes resolvidas
- **Por que deixar seu filho "mudar as regras" é um bom sinal**, e não bagunça
- **O truque da caixa de sapato** que, segundo o próprio guia, resolve quase todo o trabalho de preparação
- **16 brincadeiras que já servem a partir dos 2 anos**, com alertas de segurança onde é preciso (grãos pequenos, tinta)

**Grid de 4 áreas (cards):**

| Área | Nº | O que estimula | Exemplos |
|---|---|---|---|
| Coordenação Motora | 8 | Mãos firmes e movimentos precisos | Boliche de Garrafa PET, Alvo Certeiro, Labirinto na Caixa |
| Concentração e Foco | 7 | Terminar o que começou, no próprio ritmo | Caça ao Tesouro Sensorial, Jogo da Memória com Copos |
| Imaginação e Faz de Conta | 8 | Linguagem, narrativa e autoconfiança | Fantoche de Meia, Restaurante de Mentirinha, Capa de Super-Herói |
| Aprendizagem e Cognição | 7 | Contar, comparar, reconhecer formas | Cartaz dos Números, Sombra e Forma, Torre por Tamanho |

**Checklist "em cada brincadeira você encontra":**
> ✓ Idade indicada ✓ Tempo de preparo ✓ Lista exata de materiais ✓ Passo a passo numerado ✓ **Por que funciona**

**CTA:**
> **[ Quero o guia completo agora ]**

**Microcopy:** padrão.

**Justificativa:** bullets de fascinação (técnica de Mel Martin e Gary Bencivenga) aumentam o desejo porque combinam **benefício concreto com curiosidade** ("qual é o truque?"). Os números reais (7, 16, 17, 30) aumentam a credibilidade. O "por que funciona" responde ao medo de "comprar só passatempo" e transforma a compra em investimento no desenvolvimento do filho, o que reduz a culpa pelo gasto.

---

### 6. Espie por dentro · `gallery`

**Objetivo:** provar visualmente que o produto existe, é bonito e é bem-feito.

**Eyebrow:** POR DENTRO DO GUIA

**Headline:**
> **Espie algumas páginas**

**Subheadline:**
> Cada brincadeira vem ilustrada, com idade, tempo de preparo e o passo a passo completo.

**Legendas das imagens (formato: área · nome · preparo):**
- Coordenação · Pescaria de Bolinhas · **2 min**
- Coordenação · Alvo Certeiro · **10 min**
- Concentração · Sequência de Cores · **1 min**
- Concentração · Quebra-Cabeça Caseiro · **10 min**
- Imaginação · Restaurante de Mentirinha · **10 min**
- Imaginação · Capa de Super-Herói · **5 min**

**Nota para o design:** se possível, incluir 1 imagem de **página inteira do PDF** (com "Materiais / Como brincar / Por que funciona" visíveis) além das ilustrações. Isso mostra a profundidade do conteúdo, e não só a arte.

**CTA:**
> **[ Quero as 30 brincadeiras por R$9,90 ]**

**Justificativa:** num produto digital, a galeria substitui o "pegar na mão". Mostrar o tempo de preparo nas legendas reforça o mecanismo a cada imagem.

---

### 7. Depoimentos · `testimonials`

**Objetivo:** prova social de pessoas parecidas com o visitante (o T de PASTOR, *Testimony*).

**Eyebrow:** QUEM JÁ USOU

**Headline:**
> **O que acontece quando a próxima tarde difícil chega**

**Estrutura de cada depoimento (3 a 6 cards):**
> "[PREENCHER depoimento real, de preferência com a situação, a brincadeira usada e o resultado]"
> **[PREENCHER nome]**, mãe/pai de [PREENCHER] ([PREENCHER idade da criança]) · [PREENCHER cidade] [PREENCHER foto ou print, com autorização]

**Modelo do que pedir às compradoras (para coletar depoimentos bons):**
1. Qual era a situação antes? (ex.: horário, tela, cansaço)
2. Qual brincadeira você fez primeiro?
3. O que aconteceu? (quanto tempo a criança ficou entretida, a reação dela)
4. Posso usar seu nome, a idade do seu filho e um print?

**Se ainda não houver depoimentos:** **não publicar esta seção** e não inventar. Alternativas honestas até lá:
- prints de comentários/DMs reais do @quintal.de.dentro sobre as brincadeiras (com autorização);
- seção "Feito com mães de verdade" apenas se houver beta readers reais [PREENCHER].

**CTA:**
> **[ Quero testar em casa hoje ]**

**Justificativa:** prova social é o gatilho mais forte para tráfego frio (Cialdini), e funciona melhor quando o depoente é parecido com o leitor (mãe de criança da mesma idade). A estrutura situação → ação → resultado deixa o depoimento concreto e crível. Depoimento inventado é propaganda enganosa e expõe a marca a denúncias no Meta Ads e no Procon.

---

### 8. Para quem é / para quem não é · `whom`

**Objetivo:** gerar identificação e qualificar o público. Dizer para quem **não** é aumenta a credibilidade do resto.

**Eyebrow:** É PRA VOCÊ?

**Headline:**
> **Feito pra quem tem pouco tempo, não pouco amor**

**Coluna "É pra você se…" ✓**
- Seu filho tem entre 2 e 4 anos e a tela está ocupando mais espaço do que você gostaria.
- Você quer uma ideia pronta, e não mais uma pesquisa no Pinterest.
- Não quer gastar com brinquedo novo: papelão, copo e fita já resolvem.
- "Não sou criativa" é uma frase que você já disse. (Não precisa ser: o guia pensa por você.)
- Seu filho enjoa rápido, então 30 opções diferentes ajudam.

**Coluna "Não é pra você se…" ✗**
- Você procura um curso, um método pedagógico ou acompanhamento profissional. Isto é um guia prático de brincadeiras.
- Você quer um produto físico ou um kit com material. O Tédio Zero é um PDF digital.
- Seu filho já passou dos 5 anos. A maioria das brincadeiras vai parecer fácil demais.
- Você espera que a criança brinque sozinha o tempo todo. Várias brincadeiras pedem um adulto por perto, principalmente com os menores.

**CTA:**
> **[ É pra mim, quero o Tédio Zero ]**

**Justificativa:** a lista "para quem não é" funciona como *qualificação negativa*: ao admitir limites, a página fica mais honesta, e o resto das promessas ganha credibilidade. Também reduz reembolsos, porque quem compraria errado se autoexclui. Os itens de "É pra você" absorvem as objeções que hoje estão nos cards atuais ("Não sou criativa", "Meu filho enjoa rápido").

---

### 9. Oferta + stack de valor · `offer`

**Objetivo:** mostrar tudo o que a pessoa recebe, ancorar o valor e fechar com o preço. Esta é a seção mais importante para a decisão.

**Eyebrow:** SUA OFERTA

**Headline:**
> **Tudo o que você recebe hoje**

**Stack de valor (card, com um item por linha):**

| Item | Descrição | Valor de referência |
|---|---|---|
| 📘 **Tédio Zero · Guia Completo** | 30 brincadeiras em 24 páginas ilustradas, com idade, preparo, materiais, passo a passo e "por que funciona" | R$[PREENCHER valor de referência real] |
| 🧰 **Checklist "Seu kit de casa"** | A lista de materiais para deixar tudo numa caixa e brincar sem preparação | incluso |
| 🃏 **BÔNUS · Cartas na Manga** [CONFIRMAR se é entregue] | 30 cartas para imprimir, recortar e sortear na hora do "não tenho o que fazer" | R$[PREENCHER] |
| ♾️ **Acesso vitalício** | Sem mensalidade e sem prazo. Serve de novo quando seu filho crescer | incluso |

**Total de referência:** ~~R$[PREENCHER soma, ex.: 69,00]~~
**Hoje:** **R$9,90** · pagamento único
*Menos de R$0,34 por brincadeira. Menos que uma hora de brinquedoteca, e fica com você.*

**CTA:**
> **[ Garantir meu acesso por R$9,90 ]**

**Microcopy:**
> Pagamento único · Acesso imediato no e-mail · 7 dias de garantia
> 🔒 Compra segura pela Hotmart · [CONFIRMAR: Pix, cartão e boleto]

**Justificativa:**
- O **stack de valor** (Russell Brunson, Alex Hormozi) faz o visitante comparar o preço com um "pacote" e não com zero. Quanto mais itens concretos, maior o valor percebido.
- **Ancoragem** (Tversky e Kahneman): o valor de referência maior vem antes do preço final.
- **Preço terminado em ,90** (*charm pricing*) e reenquadramento por unidade (Gourville).
- A comparação com a brinquedoteca é mantida da página atual porque é uma ótima âncora concreta.

**Sobre o "Cartas na Manga" — escolher UMA estratégia:**

| Opção | Como | Quando usar |
|---|---|---|
| **1. Bônus na página** (recomendada se já é entregue hoje) | Entra no stack como bônus | Aumenta a conversão (valor percebido), e o ticket fica igual |
| **2. Order bump no checkout Hotmart** | "Adicione as Cartas na Manga por + R$[PREENCHER, sugestão R$4,90–7,90]" | Aumenta o ticket médio. Faz sentido quando o custo por venda no Meta está apertado |

**Copy do order bump (se escolher a opção 2):**
> **✅ SIM, quero as Cartas na Manga por só + R$[PREENCHER]**
> 30 cartas prontas para imprimir e deixar numa caixinha. Na próxima vez que bater o "não tenho o que fazer", seu filho sorteia uma carta e a brincadeira já está ali, sem abrir o celular. *(Só aparece nesta tela.)*

> ⚠️ Não usar o mesmo material como "bônus grátis" na página e como order bump pago. Isso gera reclamação e reembolso.

---

### 10. Garantia · `guarantee`

**Objetivo:** remover o risco da decisão. Transformar a garantia de "obrigação legal" em "confiança no produto".

**Visual:** selo "7 DIAS · GARANTIA INCONDICIONAL".

**Headline:**
> **Teste por 7 dias. Se não servir pra vocês, o dinheiro volta.**

**Corpo:**
> Baixe o guia, escolha uma brincadeira e experimente hoje à tarde. Se em 7 dias você achar que o Tédio Zero não ajudou na sua rotina, é só pedir o reembolso pela própria Hotmart. Você recebe **100% do valor**, sem perguntas e sem burocracia.
>
> O risco é todo nosso. Você só tem uma tarde mais tranquila a ganhar.

**CTA:**
> **[ Quero testar sem risco ]**

**Microcopy:** padrão.

**Justificativa:** a reversão de risco é essencial no low ticket, porque a principal barreira não é o preço, é o medo de "cair em mais um PDF ruim". Em vez de citar o CDC, a garantia é apresentada como **decisão da marca** ("o risco é todo nosso"), o que transmite confiança. A aversão à perda (Kahneman e Tversky) é neutralizada quando o comprador sente que não pode perder nada.

> ⚠️ **[CONFIRMAR]** que o prazo de garantia configurado no produto Hotmart é de 7 dias e que o reembolso é incondicional (padrão da plataforma). Não prometer prazo maior do que o configurado.

---

### 11. Como funciona · `how`

**Objetivo:** mostrar que comprar e acessar é fácil e rápido (reduz o atrito imaginado depois do clique).

**Eyebrow:** COMO FUNCIONA

**Headline:**
> **Da compra à primeira brincadeira em menos de 10 minutos**

**Passos:**
1. **Compre com segurança.** Pela Hotmart, em menos de 2 minutos.
2. **Receba na hora.** O PDF chega no seu e-mail e já aparece na tela de confirmação.
3. **Escolha e brinque.** Abra no celular, bata o olho no tempo de preparo e comece. Tem brincadeira pronta em 1 minuto.

**CTA:**
> **[ Quero começar agora ]**

**Justificativa:** mostrar os passos até o resultado reduz a incerteza sobre "o que acontece depois de pagar" (heurística de visibilidade do status do sistema, NN/g). A promessa de "menos de 10 minutos" se apoia em fatos: compra em 2 min, acesso imediato e brincadeiras com 1 a 3 min de preparo.

---

### 12. Quem criou · `author`

**Objetivo:** dar um rosto e uma história à marca (autoridade e simpatia).

**Eyebrow:** QUEM ESTÁ POR TRÁS

**Headline:**
> **Criado para a correria de verdade**

**Corpo (manter a essência do texto atual, com uma pessoa assinando):**
> Oi, eu sou a **[PREENCHER nome]**, do **@quintal.de.dentro**. [PREENCHER 1 frase real: ex.: "mãe do/da [nome], de [idade]" e/ou formação relevante, se houver.]
>
> Toda mãe que já ficou travada no meio da cozinha, com a criança puxando a barra da calça e nenhuma ideia na cabeça, conhece esse momento. Não é sobre não saber brincar. É sobre estar cansada demais pra inventar do zero, todo santo dia, com o jantar ainda por fazer.
>
> O Tédio Zero é a lista que eu queria ter tido: pronta para a rotina real, com o que já está na gaveta, e explicando por que cada brincadeira faz bem pro seu filho.

**Assinatura:**
> *[PREENCHER nome]* · Quintal de Dentro
> [PREENCHER foto real da autora]

**Justificativa:** pessoas confiam em pessoas, não em logos. Uma foto real e um nome aumentam a credibilidade, e a história pessoal gera simpatia e reciprocidade (Cialdini). A seção fica depois da oferta porque, a essa altura, quem ainda está em dúvida quer saber "quem está me vendendo isso".

> ⚠️ Não atribuir formação (pedagoga, psicóloga etc.) que a autora não tenha.

---

### 13. FAQ · `faq`

**Objetivo:** responder às objeções que ainda impedem o clique. Cada resposta termina reforçando um benefício.

**Eyebrow:** PERGUNTAS FREQUENTES

**Headline:**
> **Ainda com dúvida? A gente responde**

**Perguntas (ordenadas pela força da objeção):**

1. **É uma assinatura? Vou pagar todo mês?**
   Não. É um pagamento único de R$9,90. O guia é seu para sempre, sem mensalidade.

2. **Como e quando eu recebo?**
   Na hora. Assim que o pagamento é aprovado, o PDF chega no seu e-mail e também aparece na tela de confirmação da compra. [CONFIRMAR: no Pix a aprovação é imediata; no boleto leva até [PREENCHER] dias úteis.]

3. **E se o e-mail não chegar?**
   Confira a caixa de spam e a aba "Promoções". Se não estiver lá, você também pode acessar pela área de compras da Hotmart com o mesmo e-mail usado na compra, ou falar com a gente em [PREENCHER e-mail/WhatsApp de suporte].

4. **Quais formas de pagamento são aceitas?**
   [CONFIRMAR na oferta Hotmart: Pix, cartão de crédito e boleto.]

5. **É um produto físico?**
   Não. É um guia digital em PDF. Você abre no celular, tablet ou computador, ou imprime se preferir folhear no papel.

6. **Preciso comprar algum material especial?**
   Não. As 30 brincadeiras usam coisas que a maioria das casas já tem: papelão, copo plástico, fita adesiva, tampinhas, bolinha de ping-pong. O guia traz uma checklist "Seu kit de casa" para você separar tudo numa caixa.

7. **Quanto tempo leva para preparar cada brincadeira?**
   De 1 a 15 minutos, e o tempo vem indicado em cada página. 17 das 30 ficam prontas em até 5 minutos.

8. **Serve para o meu filho de 2 anos? É seguro?**
   Sim. 16 brincadeiras servem a partir dos 2 anos, e cada página indica a idade. Onde há algum cuidado (grãos pequenos, tinta), o guia avisa. Como em toda brincadeira com crianças pequenas, é importante ter um adulto por perto.

9. **E se meu filho tiver menos de 2 ou mais de 4 anos?**
   Muitas brincadeiras também funcionam com crianças de 1 ano, com supervisão, e de 5 anos, com um pouco mais de desafio. A faixa de cada atividade é uma referência.

10. **Posso usar com mais de um filho?**
    Pode. Várias brincadeiras funcionam ainda melhor com irmãos ou primos juntos.

11. **E se eu não gostar?**
    Você tem 7 dias de garantia. É só pedir o reembolso pela Hotmart e você recebe 100% do valor, sem perguntas.

**CTA:**
> **[ Quero as 30 brincadeiras por R$9,90 ]**

**Justificativa:** o FAQ é a última barreira antes da decisão. As perguntas 1 a 4 atacam o **medo transacional** (assinatura, entrega, pagamento), que a Baymard identifica como causa de abandono. As perguntas 7 e 8 reforçam o mecanismo e a segurança, que são as maiores preocupações de quem compra para uma criança pequena. O acordeão mantém a página curta no mobile, e o evento `faq_open` (ver `ANALYTICS.md`) mostra quais objeções mais aparecem, o que pode orientar a próxima rodada de copy.

---

### 14. CTA final · `final`

**Objetivo:** última chance de decisão, com *future pacing* (fazer a pessoa imaginar o depois) e urgência natural.

**Headline:**
> **Hoje às 17h pode ser diferente**

**Corpo:**
> Imagine: bate o "não tenho o que fazer", você abre o celular, mas dessa vez é pra escolher uma brincadeira. Dois minutos depois, seu filho está pescando bolinhas na tigela, e você finalmente consegue terminar o jantar.
>
> Você pode continuar improvisando todo dia, ou guardar 30 brincadeiras prontas na manga por menos de R$10.

**Bloco de preço:**
> ~~De R$69,00~~ **R$9,90** · pagamento único · acesso imediato

**CTA:**
> **[ Quero o Tédio Zero agora ]**

**Microcopy:**
> 7 dias de garantia · Se não servir, você recebe 100% de volta

**Justificativa:** o *future pacing* (técnica da PNL usada em copy de resposta direta) faz o leitor viver o resultado antes de comprar. A "urgência" aqui é **real e honesta**: a próxima tarde difícil é hoje. A escolha dupla ("continuar improvisando ou…") enquadra a compra como a opção óbvia sem pressionar de forma falsa.

---

### 15. Rodapé · `footer`

**Conteúdo:**
> **Quintal de Dentro** · @quintal.de.dentro
> Suporte: [PREENCHER e-mail] · [PREENCHER CNPJ/CPF do produtor, se exigido]
> [Termos de uso] · [Política de privacidade] [PREENCHER links]
>
> Este material é educativo e não substitui a orientação de pediatras, psicólogos ou outros profissionais de saúde e educação infantil. © 2026 Quintal de Dentro. Todos os direitos reservados.

**Justificativa:** um contato de suporte visível aumenta a confiança (sinaliza que há alguém do outro lado). A política de privacidade é exigida pelas políticas de anúncio do Meta e pela LGPD quando há pixel e analytics na página.

---

### + Barra fixa no mobile · `sticky`

**Comportamento:** aparece depois que o CTA do hero sai da tela e some quando o CTA final aparece.

**Texto:**
> **R$9,90** · acesso na hora **[ Quero agora ]**

**Justificativa:** no celular, manter o CTA sempre ao alcance do polegar reduz o esforço de voltar ao botão. A barra deve ser discreta (uma linha, altura ≤ 64px) para não atrapalhar a leitura nem parecer intrusiva.

---

## 5. Resumo dos CTAs

| Seção | `data-cta-location` | Texto do botão | Microcopy |
|---|---|---|---|
| Hero | `hero` | Quero as 30 brincadeiras por R$9,90 | Pagamento único · Acesso imediato no e-mail · 7 dias de garantia + 🔒 Hotmart |
| Problema | `problem` | Quero ter o que fazer hoje às 17h | padrão |
| O que tem dentro | `inside` | Quero o guia completo agora | padrão |
| Galeria | `gallery` | Quero as 30 brincadeiras por R$9,90 | padrão |
| Depoimentos | `testimonials` | Quero testar em casa hoje | padrão |
| Para quem é | `whom` | É pra mim, quero o Tédio Zero | padrão |
| Oferta | `offer` | Garantir meu acesso por R$9,90 | padrão + 🔒 Hotmart + formas de pagamento |
| Garantia | `guarantee` | Quero testar sem risco | padrão |
| Como funciona | `how` | Quero começar agora | padrão |
| FAQ | `faq` | Quero as 30 brincadeiras por R$9,90 | padrão |
| CTA final | `final` | Quero o Tédio Zero agora | 7 dias de garantia · Se não servir, você recebe 100% de volta |
| Barra fixa (mobile) | `sticky` | Quero agora | R$9,90 · acesso na hora |

**Teste A/B de CTA (depois do teste de headline):** "Quero as 30 brincadeiras por R$9,90" (com preço) contra "Quero 30 brincadeiras para hoje" (atual, sem preço).

---

## 6. SEO e compartilhamento (meta tags)

- **`<title>`:** Tédio Zero · 30 brincadeiras prontas para crianças de 2 a 4 anos | R$9,90
- **`meta description`:** 30 brincadeiras com o que você já tem em casa, prontas em 1 a 15 minutos, com idade, passo a passo e por que cada uma ajuda no desenvolvimento. PDF com acesso imediato por R$9,90.
- **Open Graph (`og:title`):** Cinco da tarde e a tela já passou do combinado? 30 brincadeiras prontas por R$9,90
- **`og:image`:** [PREENCHER: mockup 1200×630 com a capa e o preço]

---

## 7. Pendências antes de publicar

| Marcação | O que falta |
|---|---|
| [PREENCHER] | Depoimentos reais (texto, nome, idade da criança, print, autorização) |
| [PREENCHER] | Número real de compradores/famílias e nota média (se houver) |
| [PREENCHER] | Nome, foto e 1 frase real sobre a autora |
| [PREENCHER] | E-mail/WhatsApp de suporte, links de termos e privacidade |
| [PREENCHER] | Valores de referência do stack e preço do order bump |
| [CONFIRMAR] | Se "Cartas na Manga" já é entregue (bônus) ou será order bump |
| [CONFIRMAR] | Se a âncora "De R$69,00" é um preço praticado de verdade |
| [CONFIRMAR] | Formas de pagamento ativas na oferta `hsi91u8h` e prazo da garantia na Hotmart |
| [CONFIRMAR] | Prazo real de qualquer preço promocional (só usar urgência com data verdadeira) |
