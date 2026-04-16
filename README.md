# Gerador de Assinaturas · CSOL

App estática (HTML + JS vanilla, sem dependências) que gera assinaturas de email em HTML a partir de um formulário.

## Testar localmente antes de publicar

Basta abrir `index.html` no browser — a app detecta automaticamente que está a correr de `file://` ou `localhost` e usa as imagens da pasta `./assets/` relativa. Não precisas de servidor.

**Atenção:** o HTML **gerado** em modo local aponta para `./assets/logo-csol.png` (caminho relativo), que só funciona enquanto testas. Antes de distribuir assinaturas reais, precisas de completar o setup no GitHub abaixo — aí o HTML gerado já apontará para URLs públicos do jsDelivr.

## Estrutura

```
csol-signatures/
├── index.html          ← a app
└── assets/             ← imagens servidas via jsDelivr
    ├── logo-csol.png
    ├── logo-csol-dark.png
    ├── separator.png
    ├── separator-dark.png
    ├── icon-linkedin.png
    ├── icon-instagram.png
    ├── icon-tiktok.png
    └── icon-website.png
```

## Setup em 5 passos

### 1. Criar repositório no GitHub

Cria um repositório **público** (pode ser privado mas jsDelivr só serve públicos gratuitamente). Sugestão de nome: `csol-email-signatures`.

### 2. Fazer upload dos ficheiros

Faz commit dos ficheiros mantendo a estrutura acima. Via terminal:

```bash
git init
git add .
git commit -m "Initial signature generator"
git branch -M main
git remote add origin https://github.com/TEU-USER/csol-email-signatures.git
git push -u origin main
```

### 3. Obter URL jsDelivr

Os assets ficam disponíveis em:

```
https://cdn.jsdelivr.net/gh/TEU-USER/csol-email-signatures@main/assets/logo-csol.png
```

Podes testar colando um destes URLs no browser — a imagem deve carregar.

### 4. Atualizar o `index.html`

Abre `index.html` e procura pela linha:

```javascript
const CDN_BASE_PRODUCTION = "https://cdn.jsdelivr.net/gh/USER/REPO@main/assets";
```

Substitui `USER/REPO` pelos valores reais. Por exemplo:

```javascript
const CDN_BASE_PRODUCTION = "https://cdn.jsdelivr.net/gh/csol-agency/csol-email-signatures@main/assets";
```

### 5. Substituir o telefone de Lisboa

No mesmo ficheiro, procura por `+351 21 XXX XX XX` e substitui pelo número real do escritório de Lisboa. Há **duas** ocorrências a atualizar:

```javascript
lisboa: {
  // ...
  phone: "+351 21 XXX XX XX",        // formato visual
  phoneDigits: "+35121XXXXXXX",      // formato link tel:
  // ...
}
```

## Deploy da app

A app é um único ficheiro HTML. Podes alojá-la em qualquer lado:

- **Vercel/Netlify/Cloudflare Pages**: liga o mesmo repositório, deploy automático.
- **GitHub Pages**: ativa na secção Settings do repositório.
- **Servidor próprio**: basta copiar `index.html` para uma pasta web.

## Atualizar a assinatura no futuro

Se quiseres mudar o design (cores, layout, fontes):
1. Edita a função `buildSignatureHTML()` dentro de `index.html`.
2. Faz commit.

**Importante sobre o cache do jsDelivr:** se mudares imagens, adiciona uma versão ao URL (ex.: `@main/assets/logo-csol.png?v=2`) para evitar cache agressivo. Em alternativa, usa tags Git: `@v1`, `@v2`.

## Como funciona o Dark Mode

A assinatura implementa suporte Dark Mode em três camadas, por ordem de fiabilidade:

**Camada 1 — Cores resilientes à inversão automática.** Texto em `#4a4a4a` (cinzento médio) em vez de preto puro, e labels em `#1a1a1a` em vez de `#000000`. Clientes que invertem automaticamente tendem a ajustar melhor estes tons.

**Camada 2 — Media query `prefers-color-scheme: dark`.** Funciona em Apple Mail (macOS e iOS), Thunderbird, e em parte dos webmails modernos. Altera as cores para versões claras (`#f5f5f5`, `#d8d8d8`) e alterna o logo e separador para as versões brancas (`logo-csol-dark.png`, `separator-dark.png`).

**Camada 3 — Atributo `[data-ogsc]`.** É o hook que o Outlook.com injecta no `<body>` quando o utilizador tem Dark Mode ativo. Duplicamos as regras da media query com este selector para cobrir o Outlook Web.

**O que ainda assim não funciona:**
- **Outlook Desktop (Windows)** ignora tudo e mantém cores claras — mas isto é intencional, porque o próprio Outlook não vira para Dark Mode (o utilizador vê a assinatura exatamente como quem a criou a desenhou).
- **Gmail** remove o `<style>` completamente em muitos contextos — cai-se de volta às cores inline resilientes da camada 1.

A app tem um toggle Light/Dark no preview para veres em tempo real como a assinatura aparece em cada modo.

## Notas técnicas sobre os assets

- **Logo e separador** foram reprocessados a partir dos originais para remover o fundo branco (agora transparente). Existem em duas versões: preta (`logo-csol.png`, `separator.png`) e branca (`logo-csol-dark.png`, `separator-dark.png`). A app alterna automaticamente consoante o modo, através de media queries.
- **Ícones sociais** mantêm fundo preto quadrado (intencional no design original) — funcionam bem em ambos os modos.
- **Dimensões 2x**: todas as imagens estão exportadas a 2x para ecrãs retina. O CSS reduz-as para tamanho display (logo 98×70, ícones 24×24).

## Limitações conhecidas

- A assinatura usa Arial como fonte principal (web-safe). Fontes como a Inter ou DM Sans **não funcionam** em Outlook.
- Dark Mode é melhor em Apple Mail do que em Outlook Web. Não há solução universal.
- Se a CSOL usar Microsoft Exchange com transport rules ou ferramentas como Exclaimer, a assinatura deve ser configurada nessas plataformas, não aqui.
