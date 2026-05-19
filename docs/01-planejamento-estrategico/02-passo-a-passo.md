# Passo a Passo do Planejamento Estratégico

> Sequência de execução do planejamento estratégico da Azume. Define a ordem das atividades, os documentos que serão gerados em cada etapa, quem conduz, e o cronograma estimado. Criado em: 2026-04-15. Última nota editorial: 2026-05-19.

> **NOTA EDITORIAL — 2026-05-19**
>
> Parte das premissas deste plano mudou com a decisão estratégica tomada em mai/2026: **AI Apps** como feature central do Nexus (ver `00-contexto-atual.md` §3.2.1 e `tmp/ai_apps_apresentacao.md`).
>
> Impactos nas etapas previstas neste documento:
>
> - **Etapa 1 (SWOT):** continua válida, mas a SWOT precisa incluir AI Apps explicitamente entre as forças/oportunidades (não só "Nexus como plataforma de expansão" genérico). Ver `00-contexto-atual.md` §9 — "Nexus como plataforma de expansão — operacionalizado via AI Apps".
> - **Etapa 2 (Off-grid/Híbridos):** essa oportunidade **deixou de ser uma direção de produto isolada**. Virou um AI App específico dentro da Suite Azume CRM Smart (`00-contexto-atual.md` §10.1, `tmp/features.md` §1.1). Análise dedicada de TAM/SAM/SOM perde sentido. Substituir esta etapa por: análise de viabilidade técnica e de demanda do AI App de proposta off-grid/híbrido dentro da Suite — questão mais estreita.
> - **Etapa 3 (Solução Comercial IA):** virou a Suite Nexus Comercial (`00-contexto-atual.md` §10.2). A pergunta deixou de ser "vale fazer ou não?" (já está no roadmap) e passou a ser **"qual nicho atacar primeiro com a Suite Nexus Comercial?"**. A análise mantém valor mas a pergunta-chave mudou.
> - **Etapa 4 (Matriz de Decisão):** a decisão "Off-grid vs. Comercial IA" deixou de ser excludente — ambas estão no plano, em momentos diferentes. A matriz pode ser refeita para escolher entre **possíveis nichos da Suite Nexus Comercial** (energia/setor elétrico via E4.0 vs. outros).
> - **Etapa 5 (Direção Estratégica):** parte da decisão já foi tomada em mai/2026 (AI Apps como direção central, sequência de Suites). O documento `07-direcao-estrategica.md` original vira **registro formal dessa decisão** + escolha do primeiro nicho da Suite Nexus Comercial.
>
> **Recomendação:** antes de executar as etapas, revisar este documento em conjunto com `00-contexto-atual.md` §3.2.1, §9 e §10 atualizados, e com `tmp/ai_apps_apresentacao.md`. O fluxo abaixo permanece como referência histórica do que foi planejado em abril/2026.

## Visão geral

O planejamento segue 5 etapas sequenciais, com as etapas 2 e 3 podendo ser executadas em paralelo. Cada etapa gera um documento específico no diretório `docs/01-planejamento-estrategico/`.

```
Etapa 1: SWOT Azume
    |
    +---> Etapa 2: Análise Off-grid/Híbridos  --+
    |                                            |--> Etapa 4: Matriz de Decisão --> Etapa 5: Direção Estratégica
    +---> Etapa 3: Análise Solução Comercial  --+
```

## Documentos que serão gerados

| Etapa | Documento | Descrição |
|-------|-----------|-----------|
| 1 | `03-swot-azume.md` | SWOT do estado atual da Azume |
| 2 | `04-oportunidade-off-grid-hibridos.md` | Análise completa da oportunidade off-grid/híbridos |
| 3 | `05-oportunidade-solucao-comercial-ia.md` | Análise completa da oportunidade de solução comercial IA |
| 4 | `06-matriz-decisao.md` | Matriz comparativa com scoring ponderado |
| 5 | `07-direcao-estrategica.md` | Decisão final, racional e próximos passos |

**Estrutura final do diretório após conclusão:**

```
docs/01-planejamento-estrategico/
  00-contexto-atual.md
  01-metodologia.md
  02-passo-a-passo.md
  03-swot-azume.md
  04-oportunidade-off-grid-hibridos.md
  05-oportunidade-solucao-comercial-ia.md
  06-matriz-decisao.md
  07-direcao-estrategica.md
```

## Condução das etapas

| Etapa | Quem conduz | Participação da diretoria |
|-------|-------------|---------------------------|
| 1 | Consultor (IA) + Paulo | Revisão e validação pelos 3 diretores. Victor e Thúlio devem validar especialmente forças/fraquezas operacionais (comercial, CS, marketing) e ameaças de mercado. |
| 2 | Consultor (IA) + Paulo | Revisão e validação. Pesquisa de mercado pode demandar inputs dos 3 diretores. |
| 3 | Consultor (IA) + Paulo | Revisão e validação. Mesma lógica da etapa 2. |
| 4 | Consultor (IA) + 3 diretores | **Participação ativa dos 3 diretores.** Definição de pesos e discussão de trade-offs. |
| 5 | 3 diretores + Consultor (IA) | **Decisão da diretoria.** Consultor facilita e documenta. |

---

## Detalhamento por etapa

### Etapa 1: SWOT da Azume (contexto atual)

**Objetivo:** diagnosticar o estado atual da empresa — onde está forte, onde está vulnerável, o que o ambiente oferece e o que ameaça.

**Fonte principal:** `00-contexto-atual.md`

**Conteúdo do documento `03-swot-azume.md`:**

- **Forças** — o que a Azume faz bem e que a diferencia (ex: liderança no CRM solar, canal E4.0, operação enxuta no azul, Nexus + framework AI Apps como plataforma de expansão multi-tenant — ver `00-contexto-atual.md` §3.2.1)
- **Fraquezas** — limitações internas (ex: 1 dev, ticket médio baixo, dependência de mercado único, afinidade tecnológica baixa dos clientes)
- **Oportunidades** — fatores externos favoráveis (ex: tendência global de armazenamento de energia, IA generativa como habilitador, autoridade da Bárbara/ABSOLAR)
- **Ameaças** — fatores externos desfavoráveis (ex: contração do mercado de GD, regulação desfavorável, distribuidores "atravessando" integradores)
- **Implicações estratégicas** — o que a SWOT implica para a decisão de direção

**Dependências:** nenhuma.

**Critério de saída:** SWOT revisada e validada pelos 3 diretores. Nenhum quadrante (forças, fraquezas, oportunidades, ameaças) pode ter menos de 3 itens — se tiver, indica análise superficial.

**Estimativa de tempo:** 1-2 dias.

---

### Etapa 2: Análise da oportunidade — Off-grid/Híbridos

> **AVISO 2026-05-19:** ver Nota Editorial no topo deste documento. Off-grid/Híbridos deixou de ser direção de produto isolada — virou um AI App específico dentro da Suite Azume CRM Smart (`00-contexto-atual.md` §10.1). O conteúdo abaixo permanece como referência histórica; antes de executar esta etapa, reposicioná-la como **análise de viabilidade do AI App de proposta off-grid/híbrido** (questão mais estreita) e descartar a parte de TAM/SAM/SOM dedicada.

**Objetivo:** avaliar a viabilidade e atratividade da expansão do Azume CRM/Nexus para sistemas off-grid e híbridos com armazenamento de energia.

**Conteúdo do documento `04-oportunidade-off-grid-hibridos.md`:**

1. **Descrição da oportunidade** — o que seria feito, para quem, como se conecta aos produtos atuais
2. **SWOT específica** — forças, fraquezas, oportunidades e ameaças desta direção
3. **TAM/SAM/SOM** — dimensionamento do mercado de off-grid/híbridos no Brasil (ver definições e conceitos em `01-metodologia.md`, seção 2)

   **Fontes prioritárias:** ABSOLAR, EPE, ANEEL, Greener, relatórios internacionais (IEA, BloombergNEF)

   **Passo a passo para conduzir o dimensionamento:**

   **Passo 1 — Definir o mercado em uma frase.** Escrever com clareza: "nosso mercado é [tipo de empresa] que [faz X] no [país/região]". Ex: "integradoras de energia solar que instalam sistemas off-grid e/ou híbridos com armazenamento de energia no Brasil". Se não conseguir definir em uma frase, o escopo está vago demais — refinar antes de continuar.

   **Passo 2 — Calcular o TAM (abordagem top-down).**
   - Buscar nas fontes acima: quantas empresas integradoras atuam (ou poderiam atuar) com off-grid/híbridos no Brasil?
   - Qual o volume total de projetos off-grid/híbridos instalados por ano? Qual a tendência (crescimento ou contração)?
   - Multiplicar o número total de empresas-alvo pelo ticket médio anual estimado do produto.
   - Registrar: fonte de cada dado, data de publicação, e se o número é dado real ou estimativa.

   **Passo 3 — Calcular o TAM (abordagem bottom-up) para validação cruzada.**
   - Listar os segmentos dentro desse mercado (ex: off-grid rural, off-grid industrial, híbridos residenciais, minigeração com armazenamento).
   - Estimar número de empresas por segmento.
   - Multiplicar por ticket médio estimado para cada segmento.
   - Somar tudo. Comparar com o top-down — se a diferença for maior que 2-3x, alguma premissa está errada em um dos dois cálculos. Causas comuns de divergência: (1) o dado macro do top-down inclui segmentos que não fazem parte do mercado-alvo; (2) o bottom-up está contando empresas que não existem mais ou duplicando segmentos; (3) o ticket médio estimado está muito diferente entre as abordagens. Cheque essas três hipóteses antes de seguir.

   **Passo 4 — Filtrar para o SAM.**
   - Do TAM, remover quem está fora do alcance real do produto:
     - Empresas de porte incompatível (muito grandes para nosso produto ou muito pequenas para pagar)
     - Empresas que não usam software (operam 100% manual/planilha e não vão mudar)
     - Regiões ou segmentos com barreiras de entrada específicas
   - Para cada filtro, registrar o percentual estimado de remoção e a justificativa.

   **Passo 5 — Estimar o SOM (horizonte: 12-24 meses).**
   - Do SAM, aplicar restrições de capacidade de execução da Azume:
     - Quantos clientes o time comercial consegue abordar por mês?
     - Qual a taxa de conversão realista? (benchmark SaaS B2B: 2-5% de leads viram clientes, mas varia muito — venda consultiva com ticket alto tende a converter menos leads porém com valor maior; canal de influencer/indicação tende a converter mais que outbound frio)
     - Qual a velocidade de ramp-up? (meses 1-6 vs. meses 6-18)
     - Quanto da base já está com concorrentes estabelecidos?
   - **Exemplo concreto da conta:** se o time comercial consegue abordar 100 leads/mês via E4.0 e outbound, com taxa de conversão de 3%, isso resulta em ~3 novos clientes/mês ou ~36/ano. Com ticket de R$ 12.000/ano, o SOM seria ~R$ 432K/ano em receita recorrente nova.
   - O SOM é o número que vai para o business case — deve ser conservador, não otimista.

   **Passo 6 — Documentar premissas.**
   - Cada número deve ter: fonte, data, e nível de confiança:
     - **Dado real:** número publicado por fonte confiável (ex: relatório ABSOLAR 2025)
     - **Estimativa informada:** cálculo baseado em dados reais com ajustes (ex: "X% do total de integradores segundo proporção de projetos off-grid na ANEEL")
     - **Proxy:** dado indireto usado na falta de dado direto (ex: "baseado em proporções do mercado norte-americano, ajustado por maturidade")
   - Listar as premissas críticas separadamente — se qualquer uma delas cair, o dimensionamento muda significativamente.

   **Passo 7 — Teste de sanidade.**
   - O SOM justifica o investimento necessário? (SOM × margem estimada > custo de desenvolvimento + operação)
   - O SOM comporta o ticket médio alvo de R$ 12.000+/ano? Ou o mercado é grande em volume mas com ticket muito baixo?
   - O TAM tem tendência de crescimento ou contração? Um mercado em contração exige captura mais rápida.
   - **Análise de sensibilidade:** e se as premissas principais estiverem erradas? Testar pelo menos: "e se o TAM for metade do estimado?" e "e se a taxa de conversão for 1% em vez de 3%?". Se a conclusão muda completamente, o dimensionamento depende demais de premissas frágeis — sinalizar isso.

4. **Mapeamento de concorrentes**
   - Concorrentes diretos (softwares de dimensionamento off-grid/híbrido)
   - Concorrentes indiretos (ferramentas de engenharia genéricas, calculadoras de fabricantes)
   - Substitutos (planilhas, cálculo manual, consultorias)
   - Validação contra as 4 categorias de concorrência a evitar
5. **Análise de risco**
   - Premissas críticas (listar e avaliar probabilidade de cada uma)
   - Cenário de fracasso — o que acontece se der errado? Qual a perda?
   - Aposta mínima viável — qual o menor investimento para testar a tese?
6. **Scoring preliminar (rascunho)** — proposta de notas contra os 9 critérios de scoring da matriz de decisão (ver `01-metodologia.md`). **Atenção:** este scoring é rascunho — o scoring oficial é o da etapa 4, com pesos definidos pela diretoria. As notas preliminares servem para antecipar discussões, não para ancorar a decisão.

**Dependências:** etapa 1 (SWOT geral como contexto).

**Critério de saída:** TAM/SAM/SOM com premissas documentadas (mesmo que baseado em proxies), mapeamento competitivo com pelo menos 3 concorrentes avaliados por categoria, e SWOT específica com implicações claras.

**Estimativa de tempo:** 1-2 semanas (pesquisa de mercado e concorrência demanda tempo).

---

### Etapa 3: Análise da oportunidade — Solução comercial ponta-a-ponta powered by IA

> **AVISO 2026-05-19:** ver Nota Editorial no topo deste documento. A Solução Comercial IA virou a **Suite Nexus Comercial** (`00-contexto-atual.md` §10.2), já incluída no roadmap pós-AI Apps. A pergunta deixou de ser "fazer ou não?" e passou a ser **"qual nicho atacar primeiro com a Suite?"**. Reposicionar esta etapa como análise comparativa entre 2-3 nichos-candidato (energia/setor elétrico via E4.0 vs. outros) antes de executá-la.

**Objetivo:** avaliar a viabilidade e atratividade de uma solução comercial ponta-a-ponta com IA generativa.

**Conteúdo do documento `05-oportunidade-solucao-comercial-ia.md`:**

1. **Descrição da oportunidade** — o que seria feito, para quem, como se conecta aos produtos atuais, qual o diferencial de nicho (se houver)
2. **SWOT específica** — forças, fraquezas, oportunidades e ameaças desta direção
3. **TAM/SAM/SOM** — dimensionamento do mercado de automação comercial com IA no Brasil (ver definições e conceitos em `01-metodologia.md`, seção 2)

   **Fontes prioritárias:** pesquisas de mercado SaaS Brasil, relatórios de IA aplicada a vendas, dados de adoção de ferramentas comerciais por PMEs

   **Passo a passo para conduzir o dimensionamento:**

   **Passo 1 — Definir o mercado em uma frase.** Ex: "PMEs brasileiras do setor [X] que precisam de automação do processo comercial ponta-a-ponta". **Atenção especial aqui:** se a definição ficar genérica demais ("empresas que vendem coisas"), o TAM vai ser enorme e inútil. A definição precisa incluir o nicho — qual setor, qual porte, qual dor específica.

   **Passo 2 — Calcular o TAM (abordagem top-down).**
   - Buscar nas fontes acima: qual o tamanho do mercado de SaaS de automação comercial/vendas no Brasil?
   - Segmentar por porte de empresa e por setor.
   - Diferenciar entre o mercado genérico (CRMs, ferramentas de outbound — que devemos evitar) e oportunidades nichadas.
   - Multiplicar o número total de empresas-alvo pelo ticket médio anual estimado.
   - Registrar: fonte de cada dado, data de publicação, e se o número é dado real ou estimativa.

   **Passo 3 — Calcular o TAM (abordagem bottom-up) para validação cruzada.**
   - Listar os segmentos-alvo (ex: por setor, por porte, por tipo de venda — consultiva vs. transacional).
   - Estimar número de empresas por segmento.
   - Multiplicar por ticket médio estimado para cada segmento.
   - Somar tudo. Comparar com o top-down — se a diferença for maior que 2-3x, alguma premissa está errada em um dos dois cálculos. Causas comuns de divergência: (1) o dado macro do top-down inclui segmentos que não fazem parte do mercado-alvo; (2) o bottom-up está contando empresas que não existem mais ou duplicando segmentos; (3) o ticket médio estimado está muito diferente entre as abordagens. Cheque essas três hipóteses antes de seguir.

   **Passo 4 — Filtrar para o SAM.**
   - Do TAM, remover quem está fora do alcance real do produto:
     - Empresas de porte incompatível
     - Setores onde a solução não se aplica ou onde já há dominância de big techs (validar contra as 4 categorias de concorrência a evitar — seção 11 do contexto)
     - Empresas que não investem em software comercial
   - Para cada filtro, registrar o percentual estimado de remoção e a justificativa.

   **Passo 5 — Estimar o SOM (horizonte: 12-24 meses).**
   - Do SAM, aplicar restrições de capacidade de execução da Azume:
     - Quantos clientes o time comercial consegue abordar por mês?
     - Qual a taxa de conversão realista? (benchmark SaaS B2B: 2-5% de leads viram clientes, mas varia muito — venda consultiva com ticket alto tende a converter menos leads porém com valor maior; canal de influencer/indicação tende a converter mais que outbound frio)
     - Qual a velocidade de ramp-up? (meses 1-6 vs. meses 6-18)
     - Quanto da base já está com concorrentes estabelecidos?
   - **Exemplo concreto da conta:** se o time comercial consegue abordar 80 leads/mês, com taxa de conversão de 4%, isso resulta em ~3 novos clientes/mês ou ~38/ano. Com ticket de R$ 15.000/ano, o SOM seria ~R$ 570K/ano em receita recorrente nova.
   - O SOM é o número que vai para o business case — deve ser conservador, não otimista.

   **Passo 6 — Documentar premissas.**
   - Cada número deve ter: fonte, data, e nível de confiança:
     - **Dado real:** número publicado por fonte confiável
     - **Estimativa informada:** cálculo baseado em dados reais com ajustes
     - **Proxy:** dado indireto usado na falta de dado direto
   - Listar as premissas críticas separadamente — se qualquer uma delas cair, o dimensionamento muda significativamente.

   **Passo 7 — Teste de sanidade.**
   - O SOM justifica o investimento necessário? (SOM × margem estimada > custo de desenvolvimento + operação)
   - O SOM comporta o ticket médio alvo de R$ 12.000+/ano?
   - O TAM tem tendência de crescimento ou contração?
   - **Análise de sensibilidade:** e se as premissas principais estiverem erradas? Testar pelo menos: "e se o TAM for metade do estimado?" e "e se a taxa de conversão for 1% em vez de 4%?". Se a conclusão muda completamente, o dimensionamento depende demais de premissas frágeis — sinalizar isso.
   - **Validação extra para esta oportunidade:** o mercado identificado é realmente nichado, ou estamos dimensionando um mercado genérico onde competiríamos com Salesforce, HubSpot, Pipedrive? Se o TAM parecer enorme, provavelmente estamos olhando para o mercado errado.

4. **Mapeamento de concorrentes**
   - Concorrentes diretos (SaaS de automação comercial com IA)
   - Concorrentes indiretos (CRMs com funcionalidades de IA, ferramentas de outbound)
   - Substitutos (equipe comercial humana, processos manuais)
   - Validação contra as 4 categorias de concorrência a evitar — **atenção especial aqui**, pois esta oportunidade tem risco alto de cair na categoria "solução óbvia powered by IA"
5. **Análise de risco**
   - Premissas críticas (listar e avaliar probabilidade de cada uma)
   - Cenário de fracasso — o que acontece se der errado? Qual a perda?
   - Aposta mínima viável — qual o menor investimento para testar a tese?
6. **Scoring preliminar (rascunho)** — proposta de notas contra os 9 critérios de scoring da matriz de decisão (ver `01-metodologia.md`). **Atenção:** este scoring é rascunho — o scoring oficial é o da etapa 4, com pesos definidos pela diretoria. As notas preliminares servem para antecipar discussões, não para ancorar a decisão.

**Dependências:** etapa 1 (SWOT geral como contexto).

**Critério de saída:** mesmos critérios da etapa 2 — TAM/SAM/SOM com premissas documentadas, mapeamento competitivo com pelo menos 3 concorrentes por categoria, SWOT específica com implicações claras. Atenção especial à validação contra as 4 categorias de concorrência a evitar (seção 11 do contexto).

**Estimativa de tempo:** 1-2 semanas (em paralelo com etapa 2).

---

### Etapa 4: Matriz de decisão comparativa

**Objetivo:** consolidar todas as análises em uma comparação lado a lado com scoring transparente.

**Pré-atividade:** reunião dos 3 diretores para definir o peso de cada critério (escala de 1 a 5). Sem alinhamento sobre o que importa mais, a matriz perde valor.

**Conteúdo do documento `06-matriz-decisao.md`:**

1. **Validação de pré-requisitos** — confirmar que todas as oportunidades atendem os filtros B2B e SaaS
2. **Pesos definidos pela diretoria** — registro dos pesos e racional por trás de cada um
3. **Matriz de scoring** — tabela com notas (1-5) para cada oportunidade em cada critério, multiplicadas pelo peso

   | Critério | Peso | Off-grid/Híbridos | Solução Comercial IA |
   |----------|------|-------------------|----------------------|
   | Distribuição | ? | ? | ? |
   | IA como diferencial | ? | ? | ? |
   | Nichagem | ? | ? | ? |
   | Viabilidade | ? | ? | ? |
   | Tamanho de mercado | ? | ? | ? |
   | Intensidade competitiva | ? | ? | ? |
   | Sinergia com ativos | ? | ? | ? |
   | Risco e reversibilidade | ? | ? | ? |
   | Potencial de ticket médio | ? | ? | ? |
   | **Score total** | | **?** | **?** |

4. **Análise qualitativa** — o que os números não capturam (timing de mercado, motivação dos sócios, intuição informada)
5. **Recomendação de sequenciamento** — não necessariamente "A ou B"; pode ser "A primeiro, depois B se A funcionar"
6. **Trade-offs explícitos** — o que se ganha e o que se perde com cada escolha

**Dependências:** etapas 2 e 3.

**Critério de saída:** pesos definidos e registrados com racional, scores preenchidos com justificativa por nota, e análise qualitativa documentada. Os 3 diretores devem ter participado da definição de pesos.

**Estimativa de tempo:** 2-3 dias (inclui reunião de alinhamento de pesos com diretoria).

---

### Etapa 5: Direção estratégica

**Objetivo:** registrar a decisão final da diretoria, com racional, riscos aceitos e próximos passos concretos.

**Conteúdo do documento `07-direcao-estrategica.md`:**

1. **Decisão** — qual direção foi escolhida e por que
2. **Racional** — resumo dos dados e análises que sustentam a decisão
3. **Riscos aceitos** — quais riscos a diretoria está conscientemente assumindo
4. **Premissas críticas** — o que precisa ser verdade para a decisão funcionar
5. **Estimativa de investimento e payback** — quantificação do investimento necessário (tempo do Paulo, contratações, infra, marketing), retorno esperado, prazo de payback, e comparação com a alternativa de não investir (manter operação atual e distribuir lucro)
6. **Próximos passos** — ações concretas com responsável e prazo
   - Quem faz o que
   - Quando
   - Qual é o primeiro marco (milestone) para validar se a direção está funcionando
7. **Critérios de abandono** — em que condições a diretoria deve reconsiderar a decisão

**Contingência: se nenhuma oportunidade for viável.** Se a análise revelar que ambas as oportunidades são ruins (score muito baixo, risco inaceitável, mercado insuficiente), o documento deve registrar essa conclusão e definir os próximos passos alternativos: brainstorming de novas direções, aprofundamento do mercado atual (GD solar), ou revisão dos critérios de avaliação.

**Dependências:** etapa 4 + decisão da diretoria.

**Estimativa de tempo:** 1-2 dias.

---

## Cronograma estimado (~1 mês)

| Semana | Atividade |
|--------|-----------|
| Semana 1 (15-18/abr) | Etapa 1: SWOT da Azume |
| Semanas 2-3 (21/abr - 02/mai) | Etapas 2 e 3 em paralelo: análise das oportunidades (inclui pesquisa de mercado) |
| Semana 4 (05-09/mai) | Etapa 4: matriz de decisão + reunião de alinhamento com diretoria |
| Semana 5 (12-16/mai) | Etapa 5: documento de direção estratégica |

**Observações:**

- **Este cronograma é referência, não compromisso.** Qualidade dos dados e das análises prevalece sobre prazo. Decisão estratégica mal fundamentada por pressa é pior que decisão atrasada.
- O item mais demorado é a pesquisa de mercado (TAM/SAM/SOM) nas etapas 2 e 3. Se os dados forem difíceis de encontrar, pode estender o cronograma.
- A etapa 4 exige participação ativa dos 3 diretores para definir pesos e discutir trade-offs.
- Se novas oportunidades surgirem durante o processo, basta adicionar uma nova etapa com o mesmo template de análise e incluí-la na matriz de decisão.
- Informações novas descobertas durante as pesquisas que sejam relevantes para o contexto geral devem ser incorporadas ao `00-contexto-atual.md`.

## Próximo passo

Iniciar a **Etapa 1: SWOT da Azume** com base no documento de contexto atual.
