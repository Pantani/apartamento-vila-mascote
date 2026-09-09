---
name: property-data-integrity
description: Auditar e eliminar inconsistências de dados do imóvel (preço, área, condomínio, IPTU, R$/m²) num anúncio estático, e implementar a fonte única de verdade. Use SEMPRE que for alterar preço, metragem, condomínio, IPTU ou qualquer número do imóvel; ao encontrar valores divergentes entre README, HTML, JSON-LD ou JavaScript; ao refatorar valores hardcoded; ou ao verificar se o site está factualmente consistente. Também para "atualizar o preço", "corrigir os valores", "auditar os dados", "o site está inconsistente".
---

# Integridade de Dados do Imóvel

## Por que isso importa

Num anúncio imobiliário, uma divergência numérica não é um bug cosmético: é perda de credibilidade na exata etapa em que o comprador está decidindo confiar no vendedor. Corretores copiam o README, compradores leem o site e o Google lê o JSON-LD — se os três discordam, o anúncio parece descuidado ou desonesto.

A causa raiz é sempre a mesma: **o mesmo fato escrito em muitos lugares**. A correção não é acertar cada cópia; é reduzir o número de cópias a uma.

## Fato vs. derivado

A distinção que organiza tudo:

| Tipo | Exemplos | Regra |
|------|----------|-------|
| **Fato** | preço, área útil, condomínio, IPTU, endereço, nº de quartos/vagas | Editável em **um único lugar** |
| **Derivado** | R$/m², custo mensal total, entrada, valor financiado, parcela | **Nunca digitado.** Sempre calculado |

Todo derivado hardcoded é uma inconsistência futura já contratada. R$/m² é o caso clássico: muda toda vez que o preço muda, e é fácil esquecer.

## Método de auditoria

1. **Enumere as ocorrências** de cada fato e derivado em todo o repositório — HTML visível, `<title>`, meta description, OG/Twitter, JSON-LD, texto de FAQ, constantes JS, README, sitemap.
2. **Confirme com leitura direta.** Camadas de proxy de terminal podem alterar a saída de `grep`. Antes de reportar qualquer número, confirme com um comando não filtrado (`rtk proxy grep`, `rtk proxy sed -n`). Reportar um número errado numa auditoria de consistência destrói a confiança na própria auditoria.
3. **Classifique a severidade:** crítica (valor comercial divergente entre canais), média (derivado desatualizado), baixa (metadado de frescor).
4. **Não corrija durante a auditoria.** Auditar e escrever no mesmo passo faz perder a visão do todo.

## Padrão de fonte única

Declare um objeto único, calcule os derivados a partir dele e hidrate o DOM por atributos de dados:

```js
const PROPERTY = {
  price: 468000, areaM2: 55, condoFee: 1343, iptuMonthly: 49,
  bedrooms: 2, suites: 1, bathrooms: 2, parking: 1, storageM2: 1,
};
PROPERTY.pricePerM2   = PROPERTY.price / PROPERTY.areaM2;
PROPERTY.monthlyTotal = PROPERTY.condoFee + PROPERTY.iptuMonthly;
```

No HTML, cada ponto de exibição vira um alvo nomeado:

```html
<strong data-prop="price">R$ 468.000</strong>
<em data-prop="pricePerM2">R$ 8.509/m²</em>
```

### O HTML servido já deve estar correto

Deixar o HTML com um valor obsoleto "porque o JS corrige" falha em três frentes: o crawler não executa JS, o preview de compartilhamento tampouco, e o usuário vê um flash do valor errado.

Portanto o texto dentro de `data-prop` é o valor correto **escrito**, e a hidratação apenas o reafirma. A hidratação existe para impedir divergência futura, não para produzir o valor.

Consequência prática: ao mudar um fato, atualize o objeto **e** rode a rotina de sincronização do HTML. Um script que reescreve os nós `data-prop` a partir de `PROPERTY` torna isso um passo só.

## Formatação

Use uma função única de formatação (`pt-BR`, `BRL`, sem centavos para valores cheios). Formatação duplicada produz "R$ 8.509" e "R$ 8509" na mesma página.

## Checklist antes de declarar consistente

- [ ] Cada fato aparece editável em exatamente um lugar
- [ ] Nenhum derivado está hardcoded fora de um nó `data-prop` sincronizado
- [ ] HTML servido (sem JS) já mostra os valores corretos
- [ ] JSON-LD, meta tags e texto visível concordam
- [ ] README concorda com o site
- [ ] Valores verificados com leitura não filtrada
