# Metodologia do Planejamento Estratégico

> Metodologias selecionadas para o planejamento estratégico da Azume. Define quais frameworks serão utilizados, por que cada um foi escolhido, e como serão aplicados. Criado em: 2026-04-15.

## Objetivo

Decidir **para onde direcionar a estratégia da Azume** diante de um mercado principal (GD solar) em contração. O planejamento deve produzir uma decisão informada sobre qual direção de expansão seguir, baseada em dados e análise estruturada — não em intuição.

**Fora de escopo nesta fase:** OKRs, metas, métricas de acompanhamento. Isso entra depois da decisão de direção.

## Metodologias selecionadas

### 1. SWOT (Forças, Fraquezas, Oportunidades, Ameaças)

**O que é:** framework de diagnóstico que mapeia fatores internos (forças e fraquezas) e externos (oportunidades e ameaças) de uma organização ou iniciativa.

**Por que:** é a base para qualquer decisão estratégica. Sem entender onde a Azume está forte, onde está vulnerável, o que o ambiente oferece e o que ameaça, qualquer decisão é cega.

**Como será aplicada:**

- **1 SWOT geral da Azume** — diagnóstico do estado atual da empresa (produtos, equipe, mercado, finanças, posição competitiva, E4.0). Baseada no documento de contexto (`00-contexto-atual.md`) e complementada com inputs dos diretores quando necessário.
- **1 SWOT por oportunidade de expansão** — análise específica de cada direção em avaliação. Hoje são 2:
  1. Expansão para sistemas off-grid e híbridos
  2. Solução comercial ponta-a-ponta powered by IA

**Produto:** a SWOT geral alimenta todas as análises subsequentes. As SWOTs por oportunidade são insumo para a matriz de decisão.

---

### 2. TAM/SAM/SOM (Dimensionamento de mercado)

**O que é:** framework para estimar o tamanho de um mercado em três níveis:

- **TAM (Total Addressable Market):** mercado total endereçável — o universo completo de receita possível se não houvesse concorrência.
- **SAM (Serviceable Addressable Market):** fatia do TAM que a empresa consegue atingir com seu modelo de negócio, distribuição e posicionamento.
- **SOM (Serviceable Obtainable Market):** fatia do SAM que a empresa pode realisticamente capturar no curto/médio prazo, dado recursos, concorrência e capacidade de execução.

**Por que:** sem dados de tamanho de mercado, qualquer decisão de investimento é chute. Precisamos saber se cada mercado-alvo é grande o suficiente para justificar a aposta — especialmente considerando que a Azume precisa de ticket médio alto (acima de R$ 12.000/ano) para viabilizar novos produtos.

**Como será aplicada:**

- **1 dimensionamento por oportunidade de expansão.**
- Fontes de pesquisa variam por oportunidade:
  - Off-grid/híbridos: ABSOLAR, EPE (Empresa de Pesquisa Energética), ANEEL, Greener, relatórios internacionais (IEA, BloombergNEF)
  - Solução comercial IA: pesquisas de mercado SaaS Brasil, relatórios de IA aplicada a vendas, dados de adoção de ferramentas comerciais por PMEs
- Quando dados precisos não existirem, usaremos proxies razoáveis com premissas explícitas.

**Produto:** estimativa de tamanho de mercado para cada oportunidade, com premissas documentadas. Alimenta a matriz de decisão.

---

### 3. Análise competitiva estruturada

**O que é:** mapeamento sistemático dos concorrentes em cada mercado potencial, classificados por tipo (direto, indireto, substituto).

**Por que:** identificar quem já atua em cada mercado, qual é o nível de competição, e se há espaço real para a Azume entrar. A Azume tem diretriz clara de **não competir com big techs nem construir soluções "óbvias/genéricas"** — a análise competitiva é o instrumento para validar se uma oportunidade cai ou não nessa armadilha.

**Como será aplicada:**

- **1 mapeamento por oportunidade de expansão.**
- Para cada concorrente relevante: o que oferece, qual o preço, qual o diferencial, qual o porte, qual a fraqueza explorável.
- Classificação em 3 categorias:
  - **Concorrentes diretos:** fazem exatamente o que a Azume faria no novo mercado.
  - **Concorrentes indiretos:** atendem a mesma dor com solução diferente.
  - **Substitutos:** alternativas que o cliente usa no lugar (planilhas, consultorias, fazer "na mão").
- Validação contra as 4 categorias de concorrência a evitar (seção 11 do contexto).

**Produto:** mapa competitivo por oportunidade. Alimenta a matriz de decisão.

---

### 4. Matriz de decisão ponderada

**O que é:** tabela comparativa que avalia cada oportunidade contra critérios pré-definidos, atribuindo notas e pesos para gerar um score final.

**Por que:** transforma análise qualitativa em comparação objetiva. Evita que a decisão seja tomada com base em preferência pessoal ou na ideia "mais empolgante" em vez da mais viável.

**Como será aplicada:**

- **1 matriz comparativa** com todas as oportunidades lado a lado.

#### Pré-requisitos (filtros passa/não-passa)

Critérios binários que toda oportunidade **deve** atender para ser considerada. Se não atender, é eliminada antes do scoring:

| # | Filtro | Descrição |
|---|--------|-----------|
| 1 | B2B | Mantém perfil B2B? |
| 2 | SaaS como oferta principal | Modelo de receita recorrente por assinatura? |

Ambas as oportunidades hoje em avaliação atendem esses filtros. Caso surjam novas oportunidades, devem passar por aqui primeiro.

#### Critérios de scoring (notas de 1 a 5, com peso)

Critérios graduáveis que permitem comparação numérica entre as oportunidades.

**Escala de notas (convenção: nota maior = sempre mais favorável):**

| Nota | Significado |
|------|-------------|
| 1 | Muito desfavorável |
| 2 | Desfavorável |
| 3 | Neutro / inconclusivo |
| 4 | Favorável |
| 5 | Muito favorável |

**Atenção à inversão semântica:** critérios como "risco" e "intensidade competitiva" devem ser pontuados na lógica da oportunidade, não na lógica do fator. Exemplo: uma oportunidade de risco muito alto e irreversível recebe nota 1 (muito desfavorável), não 5. Competição altíssima recebe nota 1, não 5.

| # | Critério | Descrição |
|---|----------|-----------|
| 1 | Estratégia de distribuição | Existem canais/influencers no nicho? A E4.0 ajuda? Ou precisa construir canal do zero? |
| 2 | IA como diferencial real | Usa IA para valor genuíno e mensurável, não "IA por IA"? O diferencial é defensável? |
| 3 | Nichagem | A solução é específica o suficiente para evitar competição direta com big techs e soluções genéricas? |
| 4 | Viabilidade com recursos limitados | Construção, operação, marketing e vendas cabíveis na estrutura atual (~10 pessoas, 1 dev)? |
| 5 | Tamanho de mercado | TAM/SAM/SOM grande o suficiente para justificar a aposta? |
| 6 | Intensidade competitiva | Quantos concorrentes já atuam? Há barreira de entrada? Existe espaço real? |
| 7 | Sinergia com ativos existentes | Aproveita CRM, Nexus, base de clientes, conhecimento de domínio e/ou canal da E4.0? |
| 8 | Risco e reversibilidade | Se der errado, qual a perda (tempo, dinheiro, foco)? É reversível? |
| 9 | Potencial de ticket médio | Atinge o alvo de R$ 12.000+/ano? O público tem capacidade de pagar? |

**Diferença entre "nichagem" (3) e "intensidade competitiva" (6):** nichagem avalia se a proposta de valor é específica o suficiente para não cair em categoria genérica; intensidade competitiva avalia quantos players já existem no nicho específico e qual o nível de competição real, mesmo que nichado.

- Cada critério recebe um peso (definido pela diretoria) e cada oportunidade recebe uma nota por critério.
- Score final = soma ponderada.
- **A matriz não decide sozinha** — é insumo para a discussão da diretoria. O documento de direção estratégica registra a decisão final.

**Produto:** ranking das oportunidades com scoring transparente. Inclui recomendação de sequenciamento (pode ser "A primeiro, depois B" ao invés de "A ou B").

---

## Metodologias reservadas para fases posteriores

As metodologias abaixo **não serão aplicadas agora**, mas estão no radar para etapas futuras do planejamento:

| Metodologia | Quando entra | Por que não agora |
|-------------|-------------|-------------------|
| **Porter (5 Forças)** | Após escolha de direção | Análise profunda do mercado escolhido. Aplicar em todos os mercados potenciais agora seria esforço desproporcional ao ganho. |
| **Business Model Canvas** | Após escolha de direção | Detalhar modelo de negócio só faz sentido quando a direção já está definida. |
| **JTBD (Jobs to be Done)** | Fase de validação/product-market fit | Útil para entender o que o cliente "contrata" o produto para fazer, mas entra na validação, não na decisão de direção. |
| **OKRs** | Após definição de direção e metas | Explicitamente fora de escopo nesta fase. |
| **Unit Economics (CAC, LTV, etc.)** | Após primeiros pilotos/dados reais | Sem dados de operação no novo mercado, as estimativas seriam muito especulativas. |

## Critérios de avaliação — referência

Os 6 critérios base para avaliar um novo nicho estão definidos na **seção 9 do documento de contexto** (`00-contexto-atual.md`). As 4 categorias de concorrência a evitar estão na **seção 11** do mesmo documento. A matriz de decisão (metodologia 4) incorpora ambos, reorganizados em filtros (pré-requisitos) e critérios de scoring (graduáveis).
