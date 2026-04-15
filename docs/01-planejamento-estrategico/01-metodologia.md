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

**O que é:** framework para estimar o tamanho de um mercado em três camadas progressivamente menores — como fatiar um bolo: primeiro você olha o bolo inteiro, depois identifica a parte que consegue alcançar, e por fim estima a fatia que de fato vai parar no seu prato.

- **TAM (Total Addressable Market — Mercado Total Endereçável):** o universo completo de receita possível se não houvesse limitação alguma — sem concorrência, sem restrição de vendas, sem limite geográfico. É o bolo inteiro.
- **SAM (Serviceable Addressable Market — Mercado Endereçável Acessível):** a fatia do TAM que a empresa *consegue atingir* dado seu modelo de negócio, tipo de produto, canais de distribuição e posicionamento. É a parte do bolo que está ao alcance da mão.
- **SOM (Serviceable Obtainable Market — Mercado Conquistável):** a fatia do SAM que a empresa pode *realisticamente capturar* num horizonte de 12-24 meses, considerando concorrência real, tamanho da equipe, capacidade de vendas e velocidade de crescimento. É a fatia que de fato vai parar no prato.

**Por que:** sem dados de tamanho de mercado, qualquer decisão de investimento é chute. Precisamos saber se cada mercado-alvo é grande o suficiente para justificar a aposta — especialmente considerando que a Azume precisa de ticket médio alto (acima de R$ 12.000/ano) para viabilizar novos produtos.

#### Exemplo concreto (Azume CRM — mercado atual)

Para tornar tangível, veja como seria o raciocínio para o CRM solar que já vendemos:

| Nível | Pergunta | Exemplo |
|-------|----------|---------|
| **TAM** | "Se TODAS as integradoras de GD solar do Brasil usassem nosso CRM, quanto seria?" | Ex: 30.000 integradoras × R$ 1.160/ano (ticket médio atual) = R$ 34,8M/ano |
| **SAM** | "Dessas, quantas têm perfil para usar nosso produto?" | Filtrar: só empresas ativas que usam software, com porte compatível. Ex: 10.000 integradoras × R$ 1.160/ano = R$ 11,6M/ano |
| **SOM** | "Dessas, quantas vamos conseguir converter nos próximos 12-24 meses?" | Considerar: capacidade do time comercial, concorrência, canal da E4.0, churn. Ex: 500 novos clientes × R$ 1.160/ano = R$ 580K/ano |

**Atenção:** os números acima são ilustrativos — o objetivo é mostrar a lógica do funil, não dimensionar o mercado atual do CRM.

#### Duas abordagens de cálculo

Existem dois caminhos para chegar nos números. O ideal é usar os dois e comparar.

**Top-down (de cima para baixo):**
- Parte de um número macro — geralmente de relatórios setoriais, associações de classe, órgãos do governo.
- Aplica filtros sucessivos (percentuais) para reduzir até o endereçável.
- **Vantagem:** rápido, usa dados de fontes reconhecidas.
- **Risco:** se o número macro estiver errado ou desatualizado, tudo que vem depois herda o erro. Tende a gerar estimativas infladas.

**Bottom-up (de baixo para cima):**
- Parte de dados granulares: conta o número de empresas-alvo por segmento e multiplica pelo ticket médio.
- **Vantagem:** mais realista, porque força você a pensar empresa por empresa.
- **Risco:** mais trabalhoso e depende de conseguir dados granulares.

**Recomendação:** usar ambas as abordagens e comparar os resultados. Se divergirem por mais de 2-3x, alguma premissa está errada — as causas mais comuns são: (1) o dado macro do top-down inclui segmentos fora do mercado-alvo; (2) o bottom-up conta empresas que não existem mais; (3) o ticket médio estimado difere entre as abordagens. Investigar antes de seguir.

#### Quando dados precisos não existem (uso de proxies)

Raramente vamos encontrar um relatório que diga exatamente "o mercado de software para integradores off-grid no Brasil vale R$ X". Nesses casos, usamos **proxies** — dados indiretos que permitem uma estimativa razoável:

- **Extrapolar de mercados adjacentes:** "se o mercado on-grid tem X integradores, e off-grid representa Y% dos projetos segundo a ANEEL, então há aproximadamente Z integradores atuando com off-grid."
- **Usar dados internacionais ajustados:** "nos EUA, X% dos integradores solares trabalham com armazenamento de energia. Ajustando pelo estágio de maturidade do mercado brasileiro..."
- **Triangular fontes:** usar 2-3 fontes diferentes e ver se convergem para uma faixa similar.

**Regra:** todo proxy deve ter a premissa documentada explicitamente. "Assumimos que..." é obrigatório. Sem isso, o número parece mais confiável do que é.

#### Armadilhas comuns

Erros típicos de quem faz dimensionamento de mercado pela primeira vez:

1. **TAM inflado ("o mercado de energia solar no mundo vale US$ 300 bilhões")** — um TAM enorme e genérico não ajuda em nada. O TAM precisa ser específico ao produto que estamos considerando, não ao setor inteiro.
2. **Confundir TAM com SAM** — dizer "nosso mercado é de R$ 50M" quando na verdade só R$ 5M é acessível com nosso modelo de negócio.
3. **SOM otimista demais** — ignorar que existe concorrência, que vendas demoram, que churn acontece. O SOM deve ser conservador.
4. **Não documentar premissas** — sem saber de onde veio cada número, é impossível questionar ou atualizar a análise depois.
5. **Usar dados desatualizados sem sinalizar** — um relatório de 2022 pode estar completamente defasado em 2026. Sempre registrar a data do dado.
6. **Não fazer análise de sensibilidade** — se o dimensionamento inteiro depende de uma premissa (ex: "o mercado off-grid vai triplicar em 3 anos"), testar o que acontece se essa premissa for metade do estimado. Se a conclusão muda radicalmente, o dimensionamento é frágil demais para basear uma decisão.

**Como será aplicada:**

- **1 dimensionamento por oportunidade de expansão.**
- Fontes de pesquisa variam por oportunidade:
  - Off-grid/híbridos: ABSOLAR, EPE (Empresa de Pesquisa Energética), ANEEL, Greener, relatórios internacionais (IEA, BloombergNEF)
  - Solução comercial IA: pesquisas de mercado SaaS Brasil, relatórios de IA aplicada a vendas, dados de adoção de ferramentas comerciais por PMEs
- Quando dados precisos não existirem, usaremos proxies razoáveis com premissas explícitas (ver seção acima).
- O passo a passo operacional de como conduzir o dimensionamento está no documento `02-passo-a-passo.md`, dentro das etapas 2 e 3.

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

- Cada critério recebe um peso de 1 a 5 (definido pela diretoria, onde 5 = máxima importância) e cada oportunidade recebe uma nota por critério.
- Score final = soma ponderada.
- **Limiar mínimo:** se todas as oportunidades ficarem com score abaixo de 50% do máximo possível, a diretoria deve considerar que nenhuma é suficientemente atrativa e acionar o plano de contingência descrito na etapa 5 do passo a passo (`02-passo-a-passo.md`).
- **A matriz não decide sozinha** — é insumo para a discussão da diretoria. O documento de direção estratégica registra a decisão final.

**Nota sobre timing de mercado:** janela de oportunidade, maturidade do mercado e momento regulatório são fatores relevantes, mas difíceis de pontuar numa escala 1-5 com rigor. Serão avaliados na **análise qualitativa** da etapa 4 (ver `02-passo-a-passo.md`), não como critério de scoring.

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

Os 6 critérios base para avaliar um novo nicho estão definidos na subseção **"Critérios para avaliar um novo nicho"** da **seção 9 (Direção estratégica)** do documento de contexto (`00-contexto-atual.md`). As 4 categorias de concorrência a evitar estão na **seção 11** do mesmo documento. A matriz de decisão (metodologia 4) incorpora ambos, reorganizados em filtros (pré-requisitos) e critérios de scoring (graduáveis).
