# apartamento-vila-mascote

Landing page estática do anúncio de venda do apartamento na Vila Mascote (Condomínio Ville Dijon).
HTML/CSS/JS puros num único `index.html`, sem build step, publicada via GitHub Pages.

## Harness: Anúncio Imobiliário

**Objetivo:** manter o anúncio factualmente consistente em todos os canais e maximizar a conversão de visitante em visita agendada.

**Trigger:** para qualquer trabalho sobre o anúncio — corrigir dados, atualizar preço, melhorar conversão, revisar SEO, auditar consistência — use a skill `listing-harness`. Dúvidas pontuais e leitura podem ser respondidas direto.

**Regra permanente:** preço, área, condomínio e IPTU têm **fonte única de verdade** (objeto `PROPERTY` no `index.html`). Nunca edite esses valores em mais de um lugar, e nunca escreva um valor derivado (R$/m², total mensal) à mão.

**Histórico de mudanças:**
| Data | Mudança | Alvo | Motivo |
|------|---------|------|--------|
| 2026-09-09 | Construção inicial do harness (6 agentes, 5 skills) | completo | Auditoria encontrou preço divergente entre README e site, derivados obsoletos no simulador e seção comparativa sem números |
| 2026-09-09 | IPTU corrigido para R$ 188,91/ano; nota sobre re-spawn no lugar de SendMessage | skills/listing-harness, CLAUDE.md | Proprietário informou IPTU real (site publicava ~3x maior); SendMessage indisponível na sessão |
