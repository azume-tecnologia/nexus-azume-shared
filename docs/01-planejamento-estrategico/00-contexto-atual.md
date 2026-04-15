# Contexto Atual — Azume

> Documento de contexto consolidado sobre a Azume, seus produtos, sua equipe, seu mercado e sua direção estratégica. Fonte única de verdade para planejamento. Última atualização: 2026-04-15.

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
2. **Próximo passo:** addon com assinatura mensal para os usuários do Azume CRM.
3. **Futuro:** quando "andar com as próprias pernas", vira produto standalone de ticket médio alto, disponível também fora da base do CRM.

**Feature estratégica em desenvolvimento: Multi-Level Context Builder**

Sistema de contexto multi-nível (global > source > account > scope > user) com policies e knowledge — RAG com busca híbrida BM25 + Vector + RRF.

**Por que é estratégico:** é o habilitador para "moldar" o Nexus para diferentes nichos de mercado — instruções de LLMs, policies e bases de conhecimento específicas por segmento. Pode vir acompanhado de ferramentas customizáveis por usuário. É a base para o Nexus se especializar em qualquer nicho novo que a Azume venha a atacar.

Spec: `/home/paulo/projects/nexus/nexus-core/docs/specs/pending/multi_level_context_builder.md`

**Stack:**
- Backend: Python 3.12+, FastAPI, MongoDB Atlas (com Vector Search), OpenAI Agents SDK, GCP (Cloud Run, Cloud Functions, Cloud Tasks)
- Frontend: Next.js 15, React 19, TypeScript, Material-UI v7

**Repositórios:**
- Backend: `/home/paulo/projects/nexus/nexus-core` — `azume-tecnologia/nexus-core`
- Frontend: `/home/paulo/projects/nexus/nexus-portal` — `azume-tecnologia/nexus-portal`

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

### Nexus como plataforma de expansão

Independente da direção escolhida, o **Nexus é a plataforma que carrega a expansão**. Com o Context Builder + ferramentas customizáveis por usuário, é possível moldar o Nexus para qualquer nicho. Pode também crescer em profundidade integrando cada vez mais com o Azume CRM, incluindo:

- Novas ferramentas para que os agentes do Nexus interajam diretamente com a API do Azume CRM
- Automação de navegador via Playwright para capacidades mais amplas de interação
- Consultoria de implantação de tecnologias em pequenos e médios negócios do nicho atendido

Após essa base, o futuro do Nexus é incerto e depende do caminho de nicho escolhido.

## 10. Oportunidades em avaliação

Nenhuma validada ainda. Precisam de validação real antes de qualquer investimento significativo.

### 10.1. Expansão para sistemas off-grid e híbridos (com armazenamento)

**Contexto:** expandir o Azume CRM (e Nexus) para dimensionar e gerar propostas de sistemas off-grid e híbridos com baterias. Para sucesso, o uso de IA para auxiliar na engenharia é essencial, já que o público atual é pouco capacitado tecnicamente nessa área.

**Prós identificados:**
- Maior sinergia possível com os clientes atuais da Azume
- Azume CRM pode ser adaptado com relativa facilidade
- Solução já nasce integrada ao CRM e ao software financeiro
- Nexus pode ser integrado para auxiliar na engenharia via IA
- Mercados mais maduros (EUA, Europa, China) têm tendência crescente a off-grid/híbridos — sinal de direção
- Saturação da rede elétrica brasileira tende a aumentar a demanda por off-grid/híbridos
- Pode ser expandido para soluções mais amplas de energia (mercado livre, etc.)
- Sem concorrente direto identificado — chance de pioneirismo
- Pode abrir portas para consultorias e treinamentos de engenharia

**Contras identificados:**
- Engenharia de off-grid/híbridos é muito mais complexa que on-grid
- Público atual, na média, não tem conhecimento técnico para isso
- Mercado brasileiro de off-grid/híbridos é pequeno hoje em relação ao on-grid
- Incerteza sobre preço de baterias no Brasil e ROI para o cliente final
- Não há volume atual suficiente para manter a empresa com ticket baixo — o módulo de baterias precisaria ter ticket médio muito mais alto
- Público atual tem recursos muito limitados para investir em sistemas com baterias
- Incerteza sobre performance da IA generativa em projetos de engenharia (fora engenharia de software, onde já está comprovada)

### 10.2. Solução comercial ponta-a-ponta powered by IA

**Contexto:** criar uma solução ponta-a-ponta (qualificação de leads → geração de propostas → negociação → venda → onboarding do produto → pós-venda → renovação) com IA generativa. A IA reduziria a necessidade de equipe comercial (SDRs e closers) e suporte ao cliente. Nasceria como piloto com os maiores clientes da Azume, mas precisa expandir para mercados mais amplos para se sustentar. Produto de ticket alto, até porque o custo com inferência de IA é alto.

**Prós identificados:**
- Sinergia inicial com Azume CRM e Nexus
- Software que reduz custo operacional do cliente (equipe comercial e suporte) — proposta de valor clara e mensurável
- Se rodar bem, resolve dores muito latentes de gestão comercial, algo notoriamente difícil

**Contras identificados:**
- **Sem diferencial, cai em "óbvio/genérico"** — milhares de soluções com IA sendo desenvolvidas para a área comercial. Precisaria encontrar um diferencial de nicho ao longo do caminho
- Gestão comercial ponta-a-ponta é linda no papel mas muito difícil de entregar. Tempo longo até produto de valor real
- Custo operacional alto (alto consumo de tokens de IA generativa + custos de API do WhatsApp)

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
