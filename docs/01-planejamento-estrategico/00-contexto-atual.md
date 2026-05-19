# Contexto Atual — Azume

> Documento de contexto consolidado sobre a Azume, seus produtos, sua equipe, seu mercado e sua direção estratégica. Fonte única de verdade para planejamento. Última atualização: 2026-05-19.

## 1. A empresa

Azume - AZUME TECNOLOGIA LTDA - é uma empresa brasileira que desenvolve software para a indústria de energia solar fotovoltaica, atendendo integradores de geração distribuída (GD).

- **Modelo de negócio:** B2B, SaaS por assinatura anual.
- **Produto principal:** Azume CRM — maior CRM/gerador de propostas para integradores de energia solar do Brasil segundo pesquisa da Greener [PREENCHER: ano da pesquisa].
- **Operação:** enxuta, ~10 colaboradores (incluindo diretores), custos fixos controlados, operando no azul.
- **Estratégia financeira atual:** maximizar lucro e distribuir aos sócios, mantendo caixa saudável. Não focamos em crescimento porque o mercado brasileiro de energia solar está em contração.

## 2. Glossário técnico do domínio

- **GD (Geração Distribuída):** geração de energia elétrica em pequeno porte, próxima ao consumidor final. Pode ser instalada junto à carga da UC ou remotamente, desde que no mesmo distribuidor.
- **UC (Unidade Consumidora):** ponto de conexão onde a energia é consumida (residência, empresa, etc.).
- **On-grid:** sistema fotovoltaico conectado à rede elétrica. Depende de aprovação da distribuidora local.
- **Off-grid:** sistema isolado, sem conexão com a rede. Depende de armazenamento (baterias).
- **Híbrido:** combina on-grid com baterias. Funciona conectado à rede, mas mantém autonomia em quedas.
- **Microgeração:** sistemas de GD até 75 kWp. Foco quase total dos clientes atuais da Azume.
- **Minigeração:** sistemas de GD de 75 kWp até 5 MWp. Minoria dos clientes.
- **Integrador:** empresa que dimensiona, vende e instala sistemas fotovoltaicos. É o cliente típico da Azume.
- **Distribuidora:** concessionária de energia elétrica local (Enel, CPFL, Neoenergia, etc.). Aprova ou rejeita projetos de GD on-grid.

## 3. Produtos

### 3.1. Azume CRM (produto principal)

CRM, gerador de propostas e gerenciador para integradores de energia solar. **Sustenta a empresa hoje.** Totalmente orientado a sistemas on-grid, principalmente microgeração.

**Stack:**
- Backend: Node.js, Express.js, TypeScript, MongoDB (Mongoose), Redis, AWS (S3, SQS), Google Cloud (Storage, Cloud Tasks)
- Frontend CRM (usuário): React 17, TypeScript, Material-UI v4, Sass, PWA
- Frontend Admin: React 17, TypeScript, Material-UI v4

**Repositórios:**
- Backend: `/home/paulo/projects/azume/azume-backend` — `azume-tecnologia/azume-backend`
- Frontend CRM: `/home/paulo/projects/azume/azume-frontend-crm` — `azume-tecnologia/azume-frontend-crm`
- Frontend Admin: `/home/paulo/projects/azume/azume-frontend-adm` — `azume-tecnologia/azume-frontend-adm`

### 3.2. Nexus (em piloto)

Software AI powered. Hoje consiste em:
- **Chat multiprovedor** (GPT, Claude, Gemini, etc.) e **multi-usuários** com IA generativa. Diferente de ChatGPT/Claude Chat/etc., possui contas com múltiplos colaboradores, cada um associável a um ou múltiplos setores da empresa. Atualmente aceita apenas texto e leitura de arquivos (limitação da fase atual do piloto).
- **Leitor smart de faturas de energia** integrado ao Azume CRM, que extrai dados de consumo e cobrança para agilizar a geração de propostas.

**Trajetória de produto:**
1. **Hoje (piloto):** base seleta de usuários para entender custos operacionais reais de inferência de IA.
2. **Próximo passo:** assinatura própria do Nexus, comercializada primeiro para a base do Azume CRM (não é addon do plano do CRM — é uma assinatura paralela do Nexus, ver §3.2.2).
3. **Futuro:** produto standalone, disponível também fora da base do CRM, dimensionado para nichos com ticket médio alto.

#### 3.2.1. Feature estratégica central em desenvolvimento: AI Apps

**O que é:** uma plataforma interna multi-tenant que permite a Azume **configurar (em vez de programar)** novas automações com IA generativa. A unidade que o cliente vê é um **AI App** — automação que combina um Trigger (evento que aciona — manual / WhatsApp / webhook / agendado / evento interno), um Hive (1+ agentes de IA com instruções, conhecimento e ferramentas) e um ou mais Outputs (chat / WhatsApp / arquivo / relatório / webhook / banco / outro App).

**Por que é estratégico:** resolve a restrição fundamental de capacidade (1 engenheiro) movendo a construção de features de IA de **engenharia** para **configuração**. Após o framework ficar pronto, não-engenheiros do time (Lucas e Rodolfo, supervisionados por Victor e Thúlio) constroem novos AI Apps. Cada nicho novo vira uma Suite de AI Apps clonáveis, drasticamente reduzindo o custo de testar mercados.

**Substitui:** a antiga feature estratégica "Multi-Level Context Builder" (spec `multi_level_context_builder.md`) — agora absorvida como subconjunto das fundações de AI Apps (KnowledgeBase + Context + AccessPolicy + RAG).

**Suites planejadas (12-18 meses):**
1. **Suite Azume CRM Smart** — defesa da vaca leiteira. Conjunto de AI Apps que permite operar o Azume CRM por linguagem natural via WhatsApp (geração de propostas on-grid e off-grid/híbrido, relatórios dinâmicos, contratos, gestão de funis).
2. **Suite Nexus Comercial** — produto standalone de ticket mais alto, clonável e adaptável por nicho (prospecção, qualificação, fechamento, pós-venda).
3. **Suite Sucesso do Cliente** — uso interno primeiro (Azume reduz custo operacional de suporte/CS/renovação), depois ofertada como produto.

**Cronograma:** 5-7 meses para o MVP do framework no cenário operacional recomendado (Paulo dividido entre framework e manutenção CRM); 4-6 meses no cenário full-time (CRM congelado). Detalhes em `docs/01-planejamento-estrategico/04-ai-apps.md`.

**Documentos de referência:**
- Blueprint técnico: `/home/paulo/projects/nexus/nexus-core-2/docs/blueprints/ai_apps.md`
- Direção estratégica: `docs/01-planejamento-estrategico/04-ai-apps.md`
- Backlog de features que descem dele: `tmp/features.md`

**Decisões em aberto sobre AI Apps** (consolidadas para não se perderem dentro do `04-ai-apps.md`; resolução esperada antes do mês 5 de desenvolvimento, salvo onde indicado):

| # | Decisão | Quem decide | Prazo |
|---|---------|-------------|-------|
| 1 | `price_per_user_brl` dos tiers PRO_PLUS e PRO_MAX (faixas sugeridas: 30-50 e 60-100) | Victor (lead), com input de Thúlio | Antes do piloto comercial da Suite 1 (mês 5-6) |
| 2 | Provider WhatsApp pós-MVP: Meta Cloud API direto vs. 360dialog | Paulo, com referendo dos sócios | Mês 5 (revisão programada) |
| 3 | Billing de mensagens WhatsApp em BYO: Azume cobra com markup vs. tenant paga direto o provider | Victor + Paulo | Antes do lançamento da Suite 2 externa (pós-MVP) |
| 4 | Estratégia de roteamento entre Apps no mesmo Channel (matcher / intent classifier / fallback) | Paulo | Durante construção do framework, decisão técnica |
| 5 | Modelo de sessão do WebChat (anônimo / autenticado / token emitido pelo tenant) | Paulo + Thúlio (UX) | Durante design das Suites externas |
| 6 | Validação de demanda da Suite 1 (entrevistar 10-15 clientes da base) | Cláudia ou Juliana, acompanhada por Thúlio | Meses 2-4 |
| 7 | Número de contas ativas reais do CRM (campo `[PREENCHER]` em §4) | Victor | O quanto antes — bloqueia recálculo do dimensionamento financeiro |

Detalhes de cada decisão, alternativas avaliadas e trade-offs em `04-ai-apps.md` §6.7, §11.5 e §12.

#### 3.2.2. Modelo de monetização do Nexus

**Assinatura única do Nexus**, paga em BRL via Pagar.me, dá acesso a toda a biblioteca de AI Apps presente e futura. Diferenciação de planos é por **capacidade de inferência de IA reservada por usuário/mês**, não por escopo de funcionalidade.

| Tier | Posicionamento | Capacidade de IA por usuário |
|------|---------------|------------------------------:|
| **PRO** | Tier gratuito de entrada; mantém a base atual sem migração forçada | Capacidade compartilhada do orçamento gratuito da Azume |
| **PRO_PLUS** | Pago; uso regular | 10× capacidade do PRO |
| **PRO_MAX** | Pago; uso intenso ou equipes maiores | 20× capacidade do PRO |

Fórmula de cobrança: `preço_mensal = price_per_user_brl × user_count`. Cliente migra de tier conforme consumo cresce — upsell natural sem renegociação. Spec autoritativa: `/home/paulo/projects/nexus/nexus-core-2/docs/specs/subscription_management.md` (Pagar.me + budget reservado por tier; sistema de budget já em produção desde commit `26a4ffa5`).

**Linha de serviço adjacente (futura, requer expansão de equipe):** consultoria de adaptação de AI Apps/Suites ao negócio específico do cliente. Cobrável à parte (projeto / hora / pacote), não inclusa na assinatura.

**Stack:**
- Backend: Python 3.12+, FastAPI, MongoDB Atlas (com Vector Search), OpenAI Agents SDK, GCP (Cloud Run, Cloud Functions, Cloud Tasks)
- Frontend: Next.js 15, React 19, TypeScript, Material-UI v7

**Repositórios:**
- Backend: `/home/paulo/projects/nexus/nexus-core` — `azume-tecnologia/nexus-core`
- Frontend: `/home/paulo/projects/nexus/nexus-portal` — `azume-tecnologia/nexus-portal`
- Próxima geração do backend (em construção, recebe o framework AI Apps): `/home/paulo/projects/nexus/nexus-core-2` — `azume-tecnologia/nexus-core-2`

### 3.3. Azume Financeiro

Software financeiro simples, addon do Azume CRM. Ticket médio ~R$ 600,00/ano por conta que utiliza o módulo. **Não dá ROI isolado** — existe para agregar valor ao CRM e reduzir churn. Desenvolvimento terceirizado (freelancer externo), o que torna mudanças mais lentas e custosas que nos outros produtos.

- **Clientes ativos no módulo:** [PREENCHER: quantas contas do CRM usam o Financeiro — necessário para avaliar se o módulo cumpre a função de reter clientes]
- **Receita anual do módulo:** [PREENCHER: receita gerada pelo Financeiro isoladamente — necessário para avaliar se vale manter o investimento no módulo]

## 4. Cenário atual (abr/2026)

### Receita e clientes

- **Contas ativas:** [PREENCHER: número de contas ativas no CRM]
- **Faturamento jan-mar/2026:** R$ 931.820,00
- **Faturamento jan-mar/2025:** [PREENCHER: valor para comparativo ano contra ano — essencial para quantificar impacto da contração do mercado]
- **Composição da receita:**
  - 80% renovação de assinaturas anuais
  - 17% novas assinaturas
  - ~3% upgrades de planos e módulos adicionais
- **Natureza dos upgrades:** [PREENCHER: esses upgrades geram receita recorrente (aumento permanente de plano) ou são pagamentos pontuais?]
- **Ticket médio (assinaturas e renovações):** R$ 1.160,00/ano — **baixo**. A empresa depende de alto volume de clientes para se sustentar.
- **Estrutura de planos:** [PREENCHER: quais planos existem e faixa de preço de cada um — importante para avaliar potencial de upsell]
- **Taxa de churn anual:** [PREENCHER: percentual de contas que não renovam — dado crítico para projeções de receita]

### Custos e margem

- **Operando no azul.** Margem de lucro aproximada: [PREENCHER: margem percentual ou faixa de custos fixos mensais — necessário para avaliar capacidade de investimento em novos produtos]
- **Fluxo de caixa:** `docs/00-referencias/img/fluxo-caixa-2026-04-10.png`

### Mercado

- **Mercado em queda:** número de empresas integradoras e quantidade de projetos de GD em queda no Brasil. [PREENCHER: dados quantitativos — ex: queda de X% no número de integradores entre 2024-2026, fonte: ABSOLAR/Greener/ANEEL]. Tendência deve continuar no futuro próximo, o que ameaça diretamente os 80% de receita de renovação.

## 5. Perfil do cliente

- **Cliente:** integradores de energia solar fotovoltaica (empresas que instalam sistemas em residências e empresas).
- **Porte predominante:** microempresas (ME) e autônomos (MEI). Alguns clientes de maior porte, poucos grandes.
- **Colaboradores por conta:**
  - 2 a 5 usuários é a norma
  - Menos de 10% das contas têm mais de 5 usuários
  - Maior cliente tem 150 usuários (caso atípico)
  - Maior parte dos usuários é do setor comercial da empresa integradora
- **Afinidade com tecnologia: BAIXA.** Clientes têm dificuldade em usar qualquer software, mesmo os simples. **Isso é restrição forte de UX em qualquer sugestão de feature ou produto novo** — simplicidade vence poder.
- **Recursos financeiros:** limitados. Ticket médio baixo é reflexo direto do orçamento apertado dos clientes.
- **Operação típica:** GD de sistemas on-grid de microgeração (até 75 kWp). Alguns fazem minigeração (até 1 MWp). Off-grid e híbridos são minoria hoje.

## 6. Ameaças ao mercado

Vários fatores compõem a ameaça estrutural ao setor de energia solar fotovoltaica no Brasil:

1. **Mercado fortemente regulado, com lobby forte de distribuidoras de energia.** Pouca segurança jurídica para integradores.
2. **Saturação da rede elétrica brasileira** (ou pelo menos o que as distribuidoras afirmam). Cada vez mais projetos são rejeitados por alegação de reversão de fluxo de energia.
3. **Distribuidores de equipamentos "atravessando" o integrador**, vendendo direto ao consumidor final com serviço de instalação incluso. Movimento crescente no mercado brasileiro.

Essas ameaças já se traduzem empiricamente em queda no número de integradores ativos e na quantidade de projetos de GD instalados.

## 7. Equipe

### Diretoria (sócios fundadores, mesmo nível hierárquico)

- **Paulo Castagnari** — TI. Desenvolvimento full stack + devops do Azume CRM e do Nexus. **Única pessoa interna de TI.** Desenvolvimento e operação dos softwares são de sua responsabilidade total. **Foco total em produto. Pouco envolvido com o dia a dia operacional da empresa e com a equipe.**
- **Victor Zani** — Financeiro, suporte ao cliente, sucesso do cliente + renovação. Influencer digital (grava conteúdo para YouTube e Instagram junto com Thúlio e Bárbara).
- **Thúlio Nascimento** — Marketing e comercial. Mais envolvido com o dia a dia da empresa. Influencer digital (grava conteúdo junto com Victor e Bárbara).

### Colaboradora chave

- **Cláudia Costa** — Gerente de operações. Braço direito de Thúlio e Victor. **Pessoa mais envolvida com a equipe no dia a dia.** Atua principalmente com comercial, suporte ao cliente e sucesso do cliente + renovação.

### Time operacional

- **Pedro Magalhães** — Analista financeiro (reporta a Victor)
- **Lucas Volotão** — Analista de marketing sênior (reporta a Thúlio)
- **Rodolfo Cortez** — Analista de marketing (reporta a Lucas)
- **Juliana Freitas** — Agente de sucesso do cliente + renovação (reporta a Cláudia)
- **Diógenes Rocha** — Agente de suporte ao cliente (reporta a Cláudia)
- **Letícia** — Vendedora, novas assinaturas (reporta a Cláudia)

### Estrutura organizacional

```
Diretoria:  Paulo + Victor + Thúlio

TI:         Paulo
Financeiro: Victor > Pedro
Marketing:  Thúlio > Lucas > Rodolfo
Comercial:  Thúlio > Cláudia > Letícia
Suporte:    Victor > Cláudia > Diógenes
CS/Renov:   Victor > Cláudia > Juliana
```

## 8. E4.0 Educação (empresa parceira)

Empresa de educação dos mesmos sócios (Paulo + Victor + Thúlio = 50%) em sociedade com **Bárbara Rubim** (50%).

**Bárbara Rubim:** advogada, **presidente da ABSOLAR** (Associação Brasileira de Energia Solar Fotovoltaica). **Provavelmente a maior autoridade do Brasil para o público de integradores de energia solar.** Influencer digital junto com Victor e Thúlio.

**Produto:** Academia do Setor Elétrico — plataforma de cursos online voltada ao público de integradores de energia solar.

**Importância estratégica para a Azume:**
- Possui o **maior canal sério do Brasil** para integradores de energia solar: https://www.youtube.com/@E4EnergiasRenovaveis
- **372 mil inscritos no YouTube / 178 mil no Instagram**
- Forte canal de vendas e branding para a Azume
- Fortalece autoridade e credibilidade da Azume via envolvimento dos sócios no canal
- **Qualquer discussão de distribuição de novo produto deve considerar se o nicho escolhido é adjacente ao público da E4.0** (integradores, setor elétrico) ou se exige construir canal novo do zero.

**Relação financeira E4.0 ↔ Azume:** [PREENCHER: a E4.0 gera receita relevante para os sócios? Há custos compartilhados entre as empresas? O canal da E4.0 é usado como marketing gratuito para a Azume ou há algum arranjo formal?]

## 9. Direção estratégica

### Tese central

O mercado atual de integradores de GD solar não sustenta a empresa no médio/longo prazo. **É necessário pivotar ou expandir para novos nichos** com maior potencial.

### Alvo de ticket médio para novos nichos

**Idealmente acima de R$ 12.000,00/ano** (~10x o ticket atual do CRM). Razões:

- Reduzir dependência de alto volume de clientes
- Cobrir o custo alto de inferência de IA, que é essencial para a proposta de valor dos novos produtos
- Permitir investir em marketing e vendas (recursos limitados hoje)

### Critérios para avaliar um novo nicho

1. **Estratégia de distribuição:** existem influencers ou canais do nicho? Eles já vendem soluções similares? (E4.0 só ajuda se o nicho for adjacente a energia/setor elétrico.)
2. **IA generativa como diferencial:** usar IA para construir valor real, não "IA por IA". Custo alto de inferência obriga ticket alto.
3. **SaaS como oferta principal**, complementado opcionalmente por cursos, treinamentos, consultorias.
4. **B2B.** Mantém o perfil, aproveita a experiência acumulada da empresa.
5. **Nichado.** Evitar soluções "abertas/genéricas" que competem com big techs.
6. **Viável com recursos limitados** para construção, operação, marketing e vendas.

### Nexus como plataforma de expansão — operacionalizado via AI Apps

Independente da direção escolhida, o **Nexus é a plataforma que carrega a expansão**, e a feature **AI Apps** (§3.2.1) é o como — é o que torna essa expansão **operacionalmente viável** com uma equipe de TI de uma pessoa.

O mecanismo: cada nicho novo é atacado por uma **Suite de AI Apps** clonada e adaptada da Suite Nexus Comercial. Construir o ataque a um nicho deixa de ser projeto de engenharia (meses) e vira projeto de configuração (dias a semanas), executado pelos próprios membros do time operacional (Lucas e Rodolfo na construção, Victor e Thúlio na supervisão). O Paulo é gargalo apenas na construção e manutenção do framework, não em cada produto novo.

Capacidades que o framework habilita ao longo do tempo:

- Ferramentas que os agentes do Nexus invocam para interagir com a API do Azume CRM (consultar clientes, gerar propostas, mover funis, gerar contratos).
- Triggers que conectam o Nexus aos canais que os clientes já usam, em especial **WhatsApp** (interface principal para nosso público de baixa afinidade tech) — primitivo de primeira classe do framework.
- Outputs estruturados (arquivos, relatórios interativos, mensagens em múltiplos canais).
- Eventual automação de navegador (Playwright) e tools de código sandboxado em fases posteriores.
- Consultoria de adaptação como linha de serviço adicional conforme expandirmos equipe.

Após essa base, o futuro do Nexus deixa de ser incerto — vira função do nicho que conseguirmos validar primeiro. Detalhes técnicos completos em `/home/paulo/projects/nexus/nexus-core-2/docs/blueprints/ai_apps.md`; direção estratégica em `docs/01-planejamento-estrategico/04-ai-apps.md`.

## 10. Oportunidades em avaliação

Decisão estrutural tomada (mai/2026): todas as oportunidades abaixo serão atacadas via **AI Apps** (§3.2.1), agrupadas em Suites. Isso muda o perfil de cada oportunidade — custo de tentar cada uma cai drasticamente, e várias podem ser exploradas em paralelo após o framework estar pronto.

### 10.1. Suite Azume CRM Smart — defesa da base atual (prioridade alta)

**Contexto:** conjunto de AI Apps que permite operar o Azume CRM por linguagem natural, com **WhatsApp como interface principal**. Endereça as features 1.1 a 1.6 da Suite 1 no `tmp/features.md` (com WhatsApp como primitivo do framework, `features.md` §5.1). AI Apps planejados: gerador de proposta on-grid, gerador de proposta híbrido/off-grid, gerador de relatórios dinâmicos, gerador de contratos, gestão de funis/quadros via comando.

**Tese central:** o cliente integrador roda o Azume CRM inteiro por dentro do WhatsApp. Tira o atrito de "tem que abrir o software". Para nosso público de baixa afinidade tech, é o maior salto de UX possível.

**Inclui a oportunidade antes listada como 10.1 (off-grid / híbrido)** — agora vira um AI App específico dentro da Suite, não mais uma direção de produto isolada que exigiria investimento dedicado.

**Prós:**
- Maior sinergia com clientes atuais (mesmo público, distribuição direta).
- WhatsApp como interface principal — diferencial competitivo real para nosso público.
- Defende a vaca leiteira contra contração do mercado (aumenta valor percebido, reduz churn).
- Dimensionamento financeiro indica payback de 3-4 meses no cenário realista (detalhes em `docs/01-planejamento-estrategico/04-ai-apps.md` §6.7).
- Off-grid/híbrido entra como AI App marginal dentro da Suite, sem precisar dimensionar como produto separado.

**Contras / riscos:**
- Demanda específica das features 1.x ainda não validada com a base — validação prevista para meses 2-4 do desenvolvimento via entrevistas com 10-15 clientes.
- Adoção depende de mudança de hábito (do CRM web para WhatsApp).
- IA aplicada à engenharia de off-grid/híbridos ainda tem performance incerta — começamos com casos mais simples.

### 10.2. Suite Nexus Comercial — produto standalone, expansível por nicho (prioridade média)

**Contexto:** solução ponta-a-ponta de gestão comercial powered by IA (prospecção → qualificação → fechamento → pós-venda). Conjunto de AI Apps clonável e adaptável por nicho. Antes listada como direção isolada (10.2), agora vira uma Suite construída sobre o mesmo framework AI Apps.

**Tese central:** após o framework, atacar nichos novos deixa de ser "construir produto custom" e vira "clonar a Suite Comercial e adaptar para o nicho". Custo de cada experimento cai de meses para semanas — viabiliza atacar 5-10 nichos no tempo que hoje gastamos atacando 1.

**Prós:**
- Sinergia inicial com Azume CRM e Nexus.
- Reduz custo operacional do cliente (proposta de valor mensurável).
- Clonagem por nicho é o mecanismo principal de expansão da Azume daqui em diante.
- Equipes comerciais maiores → contas maiores → naturalmente migram para tier PRO_MAX, atingindo o alvo de ticket >R$ 12k/ano para contas com 10+ usuários.

**Contras / riscos:**
- "Sem diferencial, cai em óbvio/genérico" (§11.4) — mitigação: nichagem em cada clone, nunca lançar a Suite Comercial sem amarração a um nicho específico com canal de distribuição mapeado.
- Tempo longo até produto de valor real em cada nicho — primeiro nicho atacado deve ser adjacente (energia / setor elétrico, via canal E4.0) para minimizar risco de distribuição.

### 10.3. Suite Sucesso do Cliente — uso interno primeiro, produto depois (prioridade alta para uso interno)

**Contexto:** AI Apps para atendimento, relacionamento e renovação. **Para a própria Azume primeiro**, reduzindo trabalho operacional de Diógenes, Juliana e Cláudia. Depois, ofertado como produto a qualquer PME com base de clientes recorrente.

**Por que essa Suite é estratégica para nós internamente:** reduzir 30-40% do trabalho operacional do nosso time de CS/suporte/renovação libera R$ 100-170k/ano de capacidade. Permite crescer base sem crescer equipe. Funciona simultaneamente como (a) economia direta, (b) validação interna do framework antes de expor cliente externo, (c) demonstração prática para usar em vendas da Suite como produto.

**Prós:**
- Cliente piloto somos nós mesmos — risco externo zero, feedback instantâneo.
- Resultado mensurável (horas de trabalho economizadas).
- Validação do framework em condições reais antes de uso externo.

**Contras:**
- Genérica demais se vendida sem nicho — mesmo princípio de §10.2: nunca lançar como produto sem nichagem específica.

### 10.4. Outras direções, agora viáveis

Com AI Apps pronto, qualquer das categorias hoje classificadas como "soluções óbvias com IA" (§11.4) pode ser **revisitada com nichagem específica**, transformando-as em produtos defensáveis. Em particular:

- Software financeiro com IA, nichado em integradores solar (relacionado ao módulo Azume Financeiro). **Atenção:** o Azume Financeiro hoje é desenvolvido por freelancer externo (§3.3). Avançar nesta direção exige decisão prévia sobre internalizar o desenvolvimento ou reorganizar a relação com o freelancer — não é "só configurar uma Suite".
- Solução de atendimento ao cliente nichada em segmentos específicos (extensão natural da Suite Sucesso do Cliente).

Estas só serão atacadas após validar Suites 1 e 3.

## 11. Concorrência a evitar

Direção clara: **não competir diretamente com big techs nem construir soluções "óbvias/genéricas"**. Qualquer ideia que caia em uma destas 4 categorias deve ser sinalizada como risco estratégico:

### 11.1. Chats de IA generativa abertos

ChatGPT, Gemini, Claude, Grok, Perplexity, etc. **Pior concorrência possível** — são os próprios donos dos modelos. Impossível competir em custo ou features gerais.

### 11.2. Ferramentas de engenharia de software

Claude Code, Cursor, GitHub Copilot. Mercado saturado, com praticamente todos os provedores de modelos oferecendo seus próprios SaaS.

### 11.3. NotebookLM e similares (contexto/documentos)

Excelente produto com tier gratuito extremamente generoso. Apenas provedores de modelos conseguem oferecer algo assim devido ao custo de tokens. O Nexus pode se inspirar em capacidades do NotebookLM, mas **competir de frente, sem diferencial, é fracasso garantido.**

Link: https://notebooklm.google

### 11.4. Soluções "óbvias" powered by IA

Soluções genéricas e óbvias, fáceis de construir com IA generativa, onde o diferencial competitivo real é raríssimo. Sobrevive quem tem mais recursos para marketing e quem já domina o mercado. Exemplos:

- CRM genérico com IA (Azume CRM é diferente por ser nichado em integradores de GD solar)
- Gerador de apresentações, PDFs, relatórios com IA (ex: Manus — https://manus.im/)
- Chatbots genéricos com IA
- Solução genérica de IA para atendimento ao cliente
- Solução genérica de IA para geração de conteúdo
- Solução genérica de IA para geração de código (já coberta acima)
- Solução genérica de IA para qualificação de leads
- Solução genérica de IA para planejamento estratégico
- Software financeiro genérico com IA

**Observação importante:** a otimização dessas ideias para **setores ou nichos específicos** pode ser uma boa oportunidade. Em forma genérica, é uma má ideia.

### Princípio-guia

Sem nichagem forte e clara, a solução cai em uma dessas 4 categorias. Toda proposta de produto deve ter o diferencial de nicho definido antes de ser considerada viável.

## 12. Estrutura deste repositório

```
docs/
  00-referencias/              # Materiais de referência (imagens, dados externos, benchmarks)
  01-planejamento-estrategico/ # Documentos de planejamento estratégico
    00-contexto-atual.md       #   Este documento
    01-metodologia.md          #   Metodologias selecionadas para o planejamento
    02-passo-a-passo.md        #   Sequência de execução do planejamento
  02-to-do/                    # Lista de tarefas e ações (TODO.md)
tmp/                           # Arquivos temporários e rascunhos
```

Repositórios de código relacionados:

| Projeto               | Local                                        | GitHub                              |
|-----------------------|----------------------------------------------|-------------------------------------|
| Nexus Backend         | `/home/paulo/projects/nexus/nexus-core`      | `azume-tecnologia/nexus-core`       |
| Nexus Frontend        | `/home/paulo/projects/nexus/nexus-portal`    | `azume-tecnologia/nexus-portal`     |
| Azume CRM Backend     | `/home/paulo/projects/azume/azume-backend`   | `azume-tecnologia/azume-backend`    |
| Azume CRM Frontend    | `/home/paulo/projects/azume/azume-frontend-crm` | `azume-tecnologia/azume-frontend-crm` |
| Azume Admin Frontend  | `/home/paulo/projects/azume/azume-frontend-adm` | `azume-tecnologia/azume-frontend-adm` |
