---
name: listing-harness
description: Orquestra o time completo do anúncio do apartamento Vila Mascote — auditoria factual, comparáveis de mercado, SEO/schema, copy de conversão, implementação e QA. Use SEMPRE que a tarefa envolver melhorar o anúncio, corrigir inconsistências, atualizar preço ou dados, reforçar conversão, revisar SEO, ou responder pedidos como "melhorar o site", "resolver as pendências", "atualizar o anúncio", "rodar de novo", "reexecutar", "refazer só a parte de X", "aplicar as correções", "revalidar os comparáveis", "o preço mudou". Pedidos simples de leitura ou dúvidas pontuais podem ser respondidos direto, sem o time.
---

# Harness do Anúncio — Orquestrador

**Modo de execução: híbrido.** Especialistas trabalham em paralelo produzindo *especificações* (somente leitura); a escrita no `index.html` é serializada num único agente. Essa é a decisão arquitetural central: todo o site vive num arquivo de ~1.600 linhas, e agentes escrevendo em paralelo o corromperiam.

Todos os agentes são invocados com `model: "opus"`.

## Phase 0 — Contexto

Antes de tudo, verifique `_workspace/`:

| Estado | Modo |
|--------|------|
| `_workspace/` não existe | **Execução inicial** — todas as fases |
| Existe + pedido de correção pontual | **Reexecução parcial** — só os agentes afetados + QA |
| Existe + fatos mudaram (preço, condomínio) | **Nova execução** — mover para `_workspace_prev/`, recomeçar |

Sempre reporte ao usuário qual modo foi escolhido antes de iniciar.

## Phase 1 — Levantamento (paralelo, somente leitura)

Dois subagentes em background, simultâneos — não têm dependência entre si:

- `property-data-auditor` → `_workspace/01_auditor_inconsistencias.md`, `_workspace/01_auditor_modelo_dados.md`
- `market-analyst` → `_workspace/02_market_comparaveis.md`

**Portão:** se o auditor marcar algum item como `DECISÃO DO PROPRIETÁRIO` (ex.: qual preço é o correto), **pare e pergunte ao usuário**. Adivinhar um valor comercial é o pior erro possível deste harness.

## Phase 2 — Especificação (paralelo, somente leitura)

Depende da Phase 1. Dois subagentes simultâneos:

- `seo-schema-engineer` → `_workspace/03_seo_spec.md`
- `conversion-copywriter` → `_workspace/04_copy_spec.md`

O copywriter só pode usar afirmações da seção "Afirmações seguras para publicar" do analista.

## Phase 3 — Implementação (sequencial, escritor único)

`landing-implementer` aplica tudo, **nesta ordem**:

1. Fonte única de verdade (objeto `PROPERTY` + hidratação `data-*`)
2. Correções factuais (README, sitemap, valores obsoletos)
3. Spec de SEO
4. Spec de copy

A ordem importa: aplicar copy antes da refatoração multiplica os pontos hardcoded a deduplicar depois.

Saída: `_workspace/05_implementer_changelog.md`.

## Phase 4 — QA (subagente)

`qa-verifier` confere contra as **specs originais**, não contra o changelog, e produz `_workspace/06_qa_report.md`.

Falhas voltam à Phase 3 (uma rodada de correção). Falhas persistentes são reportadas ao usuário, nunca silenciadas.

## Phase 5 — Entrega

Resumo ao usuário: o que mudou, o que foi verificado com evidência, o que ficou pendente e por quê. Nunca declare pronto o que o QA marcou como `NÃO VERIFICADO`.

Publicação (`git push`) só com autorização explícita — o push publica ao vivo, sem staging.

## Continuação de agentes

`SendMessage` pode estar desabilitado nesta sessão. **Não projete o fluxo contando com continuar um agente já encerrado.**

Para revisar o trabalho de um especialista, dispare um agente novo do mesmo tipo e passe como insumo (a) o arquivo de spec que ele mesmo produziu, (b) a correção a aplicar e (c) a lista explícita do que deve permanecer inalterado. Peça que ele reescreva o arquivo inteiro e acrescente no topo uma seção `## Revisão N — o que mudou`.

Sem o item (c), a revisão tende a descartar decisões já tomadas.

## Fatos do imóvel já decididos pelo proprietário

Estes valores foram confirmados diretamente pelo proprietário e **prevalecem sobre qualquer inferência de portal ou do próprio site**:

| Fato | Valor | Observação |
|------|-------|-----------|
| Preço | R$ 468.000 | Escolha do proprietário, ciente de que a análise recomendava faixa maior |
| IPTU | **R$ 188,91/ano** | O site publicava ≈ R$ 49/mês (~3x maior). Publicar **apenas em valor anual** |
| Condomínio | R$ 1.343/mês | Mesmo patamar do prédio |
| Depósito | "aproximadamente 1 m²" no texto corrido | 1,07 m² apenas na tabela técnica da planta |
| Comparativo | Faixa agregada | Nunca identificar unidade vizinha individualmente |

## Fluxo de dados

Arquivo, em `_workspace/`, nomeado `{fase}_{agente}_{artefato}.md`. Intermediários são preservados para auditoria. Retornos dos subagentes carregam só o resumo; o conteúdo vai para arquivo.

## Erros

| Situação | Ação |
|----------|------|
| Agente falha | 1 retentativa; persistindo, siga sem ele e **registre a lacuna no relatório final** |
| Specs conflitantes | Pare e pergunte — não escolha silenciosamente |
| Portal de mercado bloqueado | Fonte alternativa; após 2 falhas, registre como dado ausente |
| Preview indisponível | QA estático; itens de navegador ficam `NÃO VERIFICADO` |
| Edição quebra a página | Reverter só aquela edição e registrar |

## Decisões que são do usuário, não do harness

- **Valor de venda.** O harness pode recomendar faixa com evidência; escolher é do proprietário.
- **Publicar ao vivo.**
- **Divulgar comparáveis nominalmente** (citar vizinhos concorrentes).

## Cenários de teste

**Normal:** "resolver as pendências do anúncio" → Phase 0 detecta execução inicial → levantamento paralelo → specs → implementação → QA aprova → resumo com evidências.

**Erro:** portal de mercado retorna 403 nas duas tentativas → analista registra dado ausente → copywriter bloqueia o bloco de rentabilidade por falta de dado → implementador não escreve esse bloco → QA marca como `NÃO VERIFICADO` → resumo informa a lacuna ao usuário.

**Parcial:** "só atualizar o preço para X" → Phase 0 detecta `_workspace/` existente → pula levantamento e specs → implementador altera `PROPERTY.price` (1 linha, graças à fonte única) → QA verifica consistência em todos os canais.
