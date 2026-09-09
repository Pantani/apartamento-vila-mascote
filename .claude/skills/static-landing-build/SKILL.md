---
name: static-landing-build
description: Editar e verificar com segurança a landing page estática deste projeto (HTML/CSS/JS puros num único index.html, sem build step, publicada no GitHub Pages). Use SEMPRE que for alterar index.html, adicionar seção, mexer em CSS/JS inline, rodar o preview local, verificar a página no navegador, ou responder "rodar o site", "testar a página", "adicionar uma seção", "verificar se funcionou".
---

# Build da Landing Estática

## O que este projeto é

Um único `index.html` (~1.600 linhas) com CSS e JS inline, sem dependências, sem bundler, sem framework, publicado direto pelo GitHub Pages. `.nojekyll` desativa o processamento do Jekyll.

Essa simplicidade é uma escolha, não um atraso: zero build significa zero divergência entre o que está no repositório e o que está no ar. **Não introduza framework, bundler, CDN ou etapa de build.** Uma dependência externa aqui trocaria uma página que sempre carrega por uma que depende de terceiros.

## A restrição que governa as edições

Tudo mora num arquivo só. Duas edições simultâneas se atropelam e o arquivo é grande demais para reescrever inteiro com segurança.

Portanto:

- **Um único escritor por vez.** Trabalho paralelo produz *especificações*; a escrita é serializada.
- **Edições cirúrgicas**, com âncoras únicas de contexto — nunca reescrita completa do arquivo.
- **Verifique após cada mudança**, não ao final de um lote. Descobrir tarde qual das doze edições quebrou a página custa mais que verificar doze vezes.

## Ordem de trabalho

Refatoração estrutural (fonte única de verdade) vem **antes** de copy e SEO. Aplicar texto novo sobre valores hardcoded multiplica o trabalho de deduplicação depois.

## Preview e verificação

O preview local está configurado em `.claude/launch.json` sob o nome `site` (`python3 -m http.server` na porta 8749). Suba-o pelas ferramentas de preview do harness, nunca por um comando de shell em background.

Verifique nesta ordem:

1. **HTML servido, sem JS** — leitura direta do arquivo ou `curl`. É o que o crawler e o preview de compartilhamento enxergam. Valores corretos precisam já estar aqui.
2. **DOM renderizado, com JS** — confirme que a hidratação não alterou nenhum valor que já estava certo. Se alterou, um dos dois lados está errado.
3. **Console e rede** — sem erros.
4. **Responsivo** — 375px e desktop, sem overflow horizontal. Blocos largos (tabelas) rolam dentro do próprio contêiner.
5. **Acessibilidade** — foco visível, navegação por teclado, `alt` descritivo, contraste, `prefers-reduced-motion` respeitado.

## Padrões a preservar

- HTML semântico e âncoras estáveis (`#valores`, `#comparativo`, …): são usadas na navegação e podem estar em links compartilhados.
- `loading="lazy"` + `width`/`height` explícitos nas imagens (evita layout shift).
- `.webp` com fallback e variantes `-640` para telas pequenas.
- Animações de revelação sob `prefers-reduced-motion`.

## Publicação

`git push` para `main` publica. Não há staging: **o que passa no QA é o que vai ao ar.** Atualize `sitemap.xml` (`lastmod`) quando o conteúdo mudar de fato.
