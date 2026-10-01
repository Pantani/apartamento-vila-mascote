# Apartamento à venda – Vila Mascote | Condomínio Ville Dijon

Site estático do anúncio: **https://pantani.github.io/apartamento-vila-mascote/**

- 55 m² · 2 dormitórios (1 suíte) · 2 banheiros · 1 vaga + depósito privativo · sol da manhã e da tarde
- Av. Damasceno Vieira, 726 – Vila Mascote – São Paulo/SP
- R$ 459.000 (R$ 8.345/m²)
- Condomínio R$ 1.343/mês · IPTU R$ 188,91/ano (valor anual, conforme o carnê — não converter para mensal)

> Os valores acima têm fonte única no objeto `PROPERTY`, dentro do `index.html`. Ao alterar qualquer um deles, atualize o `PROPERTY` **e** as cópias estáticas listadas no comentário que o acompanha (`<head>`, JSON-LD, `sitemap.xml` e este README).

## Estrutura

- `index.html` — landing page única para compradores e corretores (HTML/CSS/JS puros, sem build)
- `fotos/` — fotos do imóvel e planta com áreas por ambiente
- `fotos-apartamento-vila-mascote.zip` — 37 fotos originais aprovadas para download pelos corretores
- `share-preview.jpg` — card 1200x630 de compartilhamento (WhatsApp, Facebook, X)
- `tools/share-preview.html` — gerador do card acima; ao mudar o preço, edite aqui e reexporte
- `sitemap.xml` / `robots.txt` — SEO
- `.nojekyll` — desativa o Jekyll no GitHub Pages

## Publicação

GitHub Pages: **Settings → Pages → Deploy from a branch → `main` / `(root)`**.

### Regerar o card de compartilhamento

O `share-preview.jpg` traz o preço embutido na imagem, então precisa ser refeito a cada mudança de valor:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu \
  --hide-scrollbars --force-device-scale-factor=1 --window-size=1200,630 \
  --screenshot=/tmp/preview.png tools/share-preview.html
sips -s format jpeg -s formatOptions 78 /tmp/preview.png --out share-preview.jpg
```

Depois de trocar a imagem, incremente o `?v=` nas 5 referências do `index.html` e na do `sitemap.xml` —
Facebook e WhatsApp cacheiam a prévia pela URL e não reconsultam o arquivo.

Depois de publicado, enviar `https://pantani.github.io/apartamento-vila-mascote/sitemap.xml` no [Google Search Console](https://search.google.com/search-console).
