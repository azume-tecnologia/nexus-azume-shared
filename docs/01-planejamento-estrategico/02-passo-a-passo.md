# Passo a Passo do Planejamento Estratégico

> Sequência de execução do planejamento estratégico da Azume. Define a ordem das atividades, os documentos que serão gerados em cada etapa, quem conduz, e o cronograma estimado. Criado em: 2026-04-15.

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
| 1 | Consultor (IA) + Paulo | Revisão e validação. Victor e Thúlio complementam se necessário. |
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

- **Forças** — o que a Azume faz bem e que a diferencia (ex: liderança no CRM solar, canal E4.0, operação enxuta no azul, Nexus como plataforma de expansão)
- **Fraquezas** — limitações internas (ex: 1 dev, ticket médio baixo, dependência de mercado único, afinidade tecnológica baixa dos clientes)
- **Oportunidades** — fatores externos favoráveis (ex: tendência global de armazenamento de energia, IA generativa como habilitador, autoridade da Bárbara/ABSOLAR)
- **Ameaças** — fatores externos desfavoráveis (ex: contração do mercado de GD, regulação desfavorável, distribuidores "atravessando" integradores)
- **Implicações estratégicas** — o que a SWOT implica para a decisão de direção

**Dependências:** nenhuma.

**Estimativa de tempo:** 1-2 dias.

---

### Etapa 2: Análise da oportunidade — Off-grid/Híbridos

**Objetivo:** avaliar a viabilidade e atratividade da expansão do Azume CRM/Nexus para sistemas off-grid e híbridos com armazenamento de energia.

**Conteúdo do documento `04-oportunidade-off-grid-hibridos.md`:**

1. **Descrição da oportunidade** — o que seria feito, para quem, como se conecta aos produtos atuais
2. **SWOT específica** — forças, fraquezas, oportunidades e ameaças desta direção
3. **TAM/SAM/SOM** — dimensionamento do mercado de off-grid/híbridos no Brasil
   - Fontes: ABSOLAR, EPE, ANEEL, Greener, relatórios internacionais (IEA, BloombergNEF)
   - Estimativa de número de integradores atuando com off-grid/híbridos
   - Estimativa de volume de projetos e tendência
   - Projeção de mercado endereçável para software
4. **Mapeamento de concorrentes**
   - Concorrentes diretos (softwares de dimensionamento off-grid/híbrido)
   - Concorrentes indiretos (ferramentas de engenharia genéricas, calculadoras de fabricantes)
   - Substitutos (planilhas, cálculo manual, consultorias)
   - Validação contra as 4 categorias de concorrência a evitar
5. **Análise de risco**
   - Premissas críticas (listar e avaliar probabilidade de cada uma)
   - Cenário de fracasso — o que acontece se der errado? Qual a perda?
   - Aposta mínima viável — qual o menor investimento para testar a tese?
6. **Scoring preliminar** — avaliação contra os 9 critérios de scoring da matriz de decisão (ver `01-metodologia.md`)

**Dependências:** etapa 1 (SWOT geral como contexto).

**Estimativa de tempo:** 1-2 semanas (pesquisa de mercado e concorrência demanda tempo).

---

### Etapa 3: Análise da oportunidade — Solução comercial ponta-a-ponta powered by IA

**Objetivo:** avaliar a viabilidade e atratividade de uma solução comercial ponta-a-ponta com IA generativa.

**Conteúdo do documento `05-oportunidade-solucao-comercial-ia.md`:**

1. **Descrição da oportunidade** — o que seria feito, para quem, como se conecta aos produtos atuais, qual o diferencial de nicho (se houver)
2. **SWOT específica** — forças, fraquezas, oportunidades e ameaças desta direção
3. **TAM/SAM/SOM** — dimensionamento do mercado de automação comercial com IA no Brasil
   - Fontes: pesquisas de mercado SaaS Brasil, relatórios de IA aplicada a vendas, dados de adoção de ferramentas comerciais por PMEs
   - Segmentação por porte de empresa e setor
   - Diferenciação entre mercado genérico (a evitar) e oportunidades nichadas
4. **Mapeamento de concorrentes**
   - Concorrentes diretos (SaaS de automação comercial com IA)
   - Concorrentes indiretos (CRMs com funcionalidades de IA, ferramentas de outbound)
   - Substitutos (equipe comercial humana, processos manuais)
   - Validação contra as 4 categorias de concorrência a evitar — **atenção especial aqui**, pois esta oportunidade tem risco alto de cair na categoria "solução óbvia powered by IA"
5. **Análise de risco**
   - Premissas críticas (listar e avaliar probabilidade de cada uma)
   - Cenário de fracasso — o que acontece se der errado? Qual a perda?
   - Aposta mínima viável — qual o menor investimento para testar a tese?
6. **Scoring preliminar** — avaliação contra os 9 critérios de scoring da matriz de decisão (ver `01-metodologia.md`)

**Dependências:** etapa 1 (SWOT geral como contexto).

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

**Estimativa de tempo:** 2-3 dias (inclui reunião de alinhamento de pesos com diretoria).

---

### Etapa 5: Direção estratégica

**Objetivo:** registrar a decisão final da diretoria, com racional, riscos aceitos e próximos passos concretos.

**Conteúdo do documento `07-direcao-estrategica.md`:**

1. **Decisão** — qual direção foi escolhida e por que
2. **Racional** — resumo dos dados e análises que sustentam a decisão
3. **Riscos aceitos** — quais riscos a diretoria está conscientemente assumindo
4. **Premissas críticas** — o que precisa ser verdade para a decisão funcionar
5. **Próximos passos** — ações concretas com responsável e prazo
   - Quem faz o que
   - Quando
   - Qual é o primeiro marco (milestone) para validar se a direção está funcionando
6. **Critérios de abandono** — em que condições a diretoria deve reconsiderar a decisão

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

- O item mais demorado é a pesquisa de mercado (TAM/SAM/SOM) nas etapas 2 e 3. Se os dados forem difíceis de encontrar, pode estender o cronograma.
- A etapa 4 exige participação ativa dos 3 diretores para definir pesos e discutir trade-offs.
- Se novas oportunidades surgirem durante o processo, basta adicionar uma nova etapa com o mesmo template de análise e incluí-la na matriz de decisão.
- Informações novas descobertas durante as pesquisas que sejam relevantes para o contexto geral devem ser incorporadas ao `00-contexto-atual.md`.

## Próximo passo

Iniciar a **Etapa 1: SWOT da Azume** com base no documento de contexto atual.
