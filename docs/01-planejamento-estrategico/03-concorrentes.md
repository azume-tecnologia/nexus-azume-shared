# Mapa de Concorrentes

> Mapeamento dos principais concorrentes (atuais e potenciais) da Azume, organizados por camada de produto. Não pretende ser exaustivo — foca em quem pressiona ou pode pressionar nossas decisões de produto e mercado nos próximos 18-24 meses. Última atualização: 2026-05-19.

## Por que este documento existe

A Azume opera em **três camadas simultaneamente**:

1. **Camada de produto vertical** — Azume CRM para integradores de energia solar.
2. **Camada de framework de agentes** — AI Apps como plataforma de configurar automações de IA.
3. **Camada de canal** — WhatsApp + IA como interface principal de uso.

Cada camada tem dinâmica competitiva diferente. Misturar tudo numa lista única esconde análise. Este documento separa por camada e identifica, em cada uma, **onde queremos competir e onde queremos NÃO competir**, em coerência com o princípio-guia de `00-contexto-atual.md` §11.

## Princípio-guia (recapitulado)

Direção da empresa: **não competir frontalmente com big techs nem construir soluções "óbvias powered by IA"**. Toda escolha de mercado precisa ter um diferencial de nicho claro antes de ser considerada viável (`00-contexto-atual.md` §11.4).

Para cada camada abaixo, classificamos os players em três posturas:

- **Competimos** — disputamos cliente, mercado ou orçamento.
- **Não competimos** — reconhecemos a existência mas decidimos não disputar (custo de oportunidade ou risco estrutural).
- **Pressão indireta** — não disputam o mesmo cliente, mas comoditizam tecnologia que usamos e podem evoluir para concorrência direta.

---

## 1. Camada de produto vertical — CRM solar

Quem disputa diretamente o orçamento de software dos nossos clientes integradores de GD solar no Brasil.

### 1.1. Concorrentes diretos (CRM/proposta solar BR)

**Ranking por pressão competitiva real** segundo a diretoria (mai/2026), calibrado por ciclos de vendas perdidos e percepção da equipe comercial. **Os três players abaixo são os que efetivamente disputam contas com a Azume.**

#### 1.1.1. Luvik — **maior pressão** competitiva hoje

- **URL:** https://luvik.com.br · **CNPJ:** 37.123.063/0001-45 (LUVIK SISTEMAS LTDA)
- **Posicionamento:** **suíte comercial + marketing** para integradores, não apenas CRM. "Conjunto de ferramentas comerciais e de marketing para fazer sua empresa de energia solar decolar."
- **Funcionalidades observadas (mai/2026):** gerador de propostas web e PDF (Lei 14.300), dimensionamento (grupos A/B, autoconsumo remoto, GDII/III), CRM de leads, **site institucional**, **página de links (Luvik Tree)**, **repositório de criativos para redes sociais (Luvik Publi)**, plataforma de tarifas de concessionárias (Fio B).
- **IA / WhatsApp:** **não mencionam** como diferencial.
- **Preço:** não público.
- **Claim relevante:** "maior repositório de criativos para redes sociais de energia solar do Brasil" (Luvik Publi).

**Implicação:** o Luvik não compete só com o Azume CRM — compete com o **stack inteiro do integrador** (CRM + marketing + criativos + site + tarifas). Isso provavelmente explica por que é o mais incômodo. Nossa resposta tradicional (mais features no CRM) endereça só uma parte do escopo deles. **Decisão a tomar:** ampliar oferta da Azume para cobrir as outras dimensões ou especializar verticalmente e abandonar essas frentes para eles.

#### 1.1.2. SolarMarket — **segunda maior pressão**

- **URL:** https://www.solarmarket.com.br
- **Posicionamento:** "**CRM nº 1** para empresas de energia solar crescerem rápido". Foco em integradoras de pequeno/médio porte em fase de estruturação ou expansão.
- **Funcionalidades observadas (mai/2026):** dimensionamento (on-grid, híbrido, zero grid), proposta automatizada, precificação com fórmulas complexas, automações por gatilho/condição, **15+ integrações com distribuidores em tempo real** (WEG, Fortlev, GT Solar etc.), métricas, checklists inteligentes, assinatura eletrônica.
- **IA / WhatsApp:** **WhatsApp automatizado via MyFlows** (integração de terceiro). **Sem IA proprietária** — posicionamento explicitamente "suporte humanizado em até 5 minutos, sem bots ou IA".
- **Preço:** não público; dois planos (BASE e PRO).
- **Claim relevante:** "utilizado por **mais de 1.500 integradoras**".

**Implicação:** marca forte ("CRM nº 1") + cotação em tempo real com distribuidores é o diferencial. **Postura anti-IA explícita** no posicionamento — sinal de que parte do mercado responde melhor a "humano rápido" do que a bot. Nossa tese de IA+WhatsApp precisa **mensurar empiricamente** se o público concorda; isso entra na validação de demanda dos meses 2-4 (ver `04-ai-apps.md` §7.5).

#### 1.1.3. Groner CRM — **terceira maior pressão**

- **URL:** https://lp.gronercrm.com.br
- **Posicionamento:** "O CRM para energia solar **mais completo do mercado**". Atende empresas de qualquer porte.
- **Funcionalidades observadas (mai/2026):** fluxos personalizáveis (vendas/instalação/manutenção), gerador de propostas, integrações (Ecori, Acroni, RD Station, ClickSign, Omie, Zapster, Aldo), automação de mensagens, campos personalizados para contratos, dashboards.
- **IA / WhatsApp: ambos.** **Groner GPT** (IA atuando como SDR 24/7) + **GronerZap** (WhatsApp centralizado com histórico compartilhado).
- **Preço:** não público.
- **Claim relevante:** "mais de **5.000 vendedores** atingindo metas agora".

**Implicação — esta é a observação mais importante desta seção:** **o concorrente que JÁ tem IA + WhatsApp é o que MENOS nos incomoda dos três.** Três leituras possíveis, não excludentes:

1. **Execução de feature vence existência de feature.** Groner GPT pode entregar qualidade fraca; GronerZap pode ser apenas inbox unificado sob outro nome.
2. **Distribuição vence feature.** Luvik e SolarMarket têm marca/canal mais fortes mesmo sem IA generativa.
3. **Público ainda não amadureceu para agente autônomo.** Vendedor solar pode preferir copilot que sugere a agente que age. A validação de demanda precisa medir isso.

Em qualquer das três, a conclusão estratégica é a mesma: **WhatsApp + IA é necessário, mas não é suficiente.** A Suite Azume CRM Smart precisa vencer também em **qualidade percebida e em distribuição** — e é aí que a parceria E4.0/ABSOLAR vira o ativo central, não acessório.

#### 1.1.4. Outros players citados no setor (relevância real a confirmar)

Listados aqui sem detalhamento porque **não fazem parte do top-3 de pressão competitiva** segundo a diretoria. Manter no radar para revisão futura.

| Player | Posicionamento (a verificar) | Postura |
|---|---|---|
| Solfácil | Originalmente financeira solar; oferece ferramentas de proposta/CRM para integradores parceiros | Pressão indireta (verificar) |
| Bright, Genyx, ProGD, Click Solar | Nomes citados no setor; nível real de overlap com nossa base **a confirmar com Cláudia/Thúlio** | A verificar |
| Tess (HelloTess), Aurora Solar | Internacionais, foco em design de sistemas (não CRM) | Pressão indireta |

[PREENCHER: confirmar com Cláudia/Thúlio quais desses aparecem de fato em ciclos de vendas perdidos. Se nenhum deles aparece, retirar do documento na próxima revisão.]

**Implicação estratégica consolidada:** hoje vencemos em volume e em parceria com a E4.0/ABSOLAR. **A defesa de médio prazo não é "IA + WhatsApp" sozinho — é IA + WhatsApp + qualidade de execução + canal de distribuição.** A Suite Azume CRM Smart (`04-ai-apps.md`) endereça os dois primeiros pontos; a parceria com a ABSOLAR/Bárbara endereça o quarto; o terceiro depende de execução disciplinada do framework, não de marketing.

---

## 2. Camada de framework de agentes / AI App builders

Quem oferece "plataforma de configurar automações com IA sem código". Esta é a camada que vamos construir — e é onde a competição é mais sofisticada.

### 2.1. Big techs e provedores de modelos (commodity em ascensão)

| Player | O que oferece | Postura |
|---|---|---|
| **OpenAI Agent Builder / GPTs / Assistants API** | Configurar agentes com Tools, KnowledgeBase, sem código | Não competimos — `§11.1` |
| **Anthropic Skills + MCP** | Skills + Model Context Protocol para Tools | Não competimos — usamos como infra |
| **Microsoft Copilot Studio** | Construtor de copilots corporativos | Não competimos — enterprise, fora do alvo |
| **Google Vertex AI Agent Builder** | Plataforma de agentes na GCP | Não competimos |

**Decisão:** estes são os donos dos modelos. Tentar bater na "plataforma de agentes genérica" é §11.1 do contexto — perda garantida. **Nosso ativo não é o framework — é a biblioteca de AI Apps especializados que roda em cima dele.** O framework é meio de produção, não produto.

### 2.2. Plataformas no-code de agentes (SaaS)

| Player | O que oferece | Postura |
|---|---|---|
| **Lindy.ai** | "AI Employees" configuráveis (vendas, suporte, ops). Foco PME | **Concorrente conceitual mais próximo do que vamos construir** — não competimos diretamente porque eles são genéricos/inglês first, mas é referência forte de UX e modelo comercial |
| **Relevance AI** | Multi-agente no-code, foco ops comerciais | Pressão indireta |
| **Stack AI, Flowise, Voiceflow Agents** | Construtores no-code de agentes/chatbots | Pressão indireta |
| **n8n com AI Agent nodes** | Suporte nativo a LangChain em workflow engine | Pressão indireta — a razão para "não usar n8n" precisa ser revisitada anualmente |
| **Zapier Central / Zapier Agents** | Agentes sobre as 7000 integrações Zapier | Pressão indireta — escala global |

**Decisão:** não competimos frontalmente. Nosso ângulo é **agentes prontos para nichos específicos** (integrador solar com baixa afinidade tech, fluxos do CRM solar, regras tarifárias BR), não agentes genéricos. Se cairmos em "plataforma de agentes para PME generalista", caímos em §11.4.

### 2.3. CRMs grandes com agentes embutidos

| Player | O que oferece | Postura |
|---|---|---|
| **Salesforce Agentforce** | CRM enterprise + framework de agentes configuráveis | Não competimos — enterprise; **define a direção do mercado** |
| **HubSpot Breeze AI** | CRM PME global + agentes IA | Pressão indireta |
| **Pipedrive AI Sales Assistant** | CRM PME simples + IA | Pressão indireta (já no BR) |
| **Kommo (antes amoCRM)** | CRM PME + WhatsApp + chatbot. Popular no Brasil | **Competimos** em algumas contas pequenas do nosso público |

**Decisão:** o **Kommo** é a pressão concreta. Defendemos com **nichagem solar + parceria E4.0/ABSOLAR + integração nativa com o Azume CRM**. Não competimos em features genéricas de CRM, competimos em verticalização.

---

## 3. Camada de canal — WhatsApp + IA no Brasil

Aqui mora a concorrência que pode quebrar a tese da Suite Azume CRM Smart se ignorarmos. **Esta é a categoria que o documento de AI Apps subestimou na sua versão inicial** ("nenhum concorrente direto faz isso" — verdadeiro só dentro de "CRM solar"; falso na categoria mais ampla).

### 3.1. Plataformas BR de WhatsApp + IA generativa

| Player | O que oferece | Postura |
|---|---|---|
| **Take Blip** | Plataforma BR de WhatsApp Business + bots + IA generativa, multi-tenant, integrações CRM. Bem estabelecida em PME e enterprise BR | **Concorrente direto na camada** — se lançar uma vertical solar, pressiona forte |
| **Zenvia** | WhatsApp + IA, foco CX/atendimento. IA generativa lançada em 2024 | Concorrente direto na camada |
| **WATI** | WhatsApp Business + chatbot + IA, popular em PME BR | Concorrente direto na camada |
| **Botconversa** | Chatbots WhatsApp + IA, foco PME BR | Concorrente direto na camada |
| **Yalo** | Comércio conversacional via WhatsApp + IA, América Latina | Pressão indireta — foco em e-commerce |
| **Leadster** | Chatbots para sites + WhatsApp + IA, BR | Adjacente |

**Decisão:** competimos com nichagem. Take Blip e WATI já atendem milhares de PMEs brasileiras — se lançarem vertical solar, é problema sério em 12-24 meses. **Nossa defesa é conhecimento de domínio** (CRM solar, regras tarifárias por distribuidora, contratos do setor, parceria ABSOLAR). É defesa temporal, não permanente — a janela aproveita o gap entre commoditização da tecnologia e expansão deles para verticais nichados.

### 3.2. Infraestrutura de WhatsApp (BSPs)

Não são concorrentes — são fornecedores. Mas a escolha afeta margem e arquitetura.

| Player | Posicionamento | Postura |
|---|---|---|
| **Twilio** | BSP global, robusto, caro. **Em uso hoje (Zoe)** | Usamos no MVP; revisitar pré-lançamento |
| **Meta Cloud API direto** | Acesso direto à Meta, sem intermediário. Mais trabalho técnico, custo mínimo | Candidato a migração pré-lançamento |
| **360dialog** | BSP alemão, "near-cost" (assinatura por número + tarifa Meta sem markup). Popular em SaaS BR | Candidato a migração pré-lançamento |
| **Gupshup, Infobip, Sinch** | BSPs globais alternativos | Backup |
| **Z-API, Evolution API, Baileys, Waha** | **Soluções NÃO-oficiais** (WhatsApp Web protocol / emulação Android) | **Excluídos por política** — risco de ban incompatível com posicionamento B2B sério da Azume |

Calendário de migração, justificativa de cada candidato e exclusão explícita das soluções não-oficiais em `04-ai-apps.md` §12.

---

## 4. Adjacentes — parecem competir, mas não competem

Players frequentemente citados em discussões internas que **não disputam o mesmo cliente**:

- **ChatGPT, Claude, Gemini, Perplexity, Grok** — chats genéricos. §11.1. Não competem com produto vertical.
- **Manus, Anara, Genspark** — geradores genéricos de apresentações/relatórios. §11.4. Não competem em verticais.
- **NotebookLM** — pesquisa sobre documentos. §11.3. Inspiração, não concorrência.
- **Claude Code, Cursor, GitHub Copilot** — engenharia de software. §11.2. Fora de escopo.

Listados aqui para fechar a fronteira do debate. Se aparecerem como "concorrente" em conversa interna, o reflexo certo é: "concorrem com o ChatGPT da vida, não com a Azume".

---

## 5. Síntese — onde competimos, onde não competimos

### Vantagens defensáveis hoje

1. **Nichagem solar + autoridade ABSOLAR.** Bárbara Rubim + canal E4.0 (372k inscritos) — barreira difícil de copiar.
2. **Conhecimento operacional do integrador solar.** Fluxos do CRM solar, regras tarifárias, contratos do setor, dores específicas — não disponível para player generalista sem investimento de tempo significativo.
3. **Integração nativa com o Azume CRM.** Ativo único — concorrentes na camada WhatsApp+IA não têm.

### Vulnerabilidades

1. **Janela de diferenciação no canal WhatsApp+IA é temporal.** Take Blip ou similar pode lançar vertical solar em 12-24 meses. A urgência da Suite Azume CRM Smart é maior do que parece.
2. **Comoditização da camada de framework.** GPTs, Anthropic Skills, Lindy reduzem o "moat técnico" do AI Apps a cada release. A defesa é a biblioteca de AI Apps especializados, não o framework em si.
3. **Concorrência indireta no CRM PME generalista** (Kommo, Pipedrive AI) — sobre contas pequenas, baixo ticket.

### Categorias onde NUNCA queremos competir

1. **Modelos genéricos e chats abertos** (§11.1).
2. **Engenharia de software** (§11.2).
3. **NotebookLM-likes sem nicho** (§11.3).
4. **"Óbvio com IA" sem nichagem forte** (§11.4) — em particular: CRM genérico, chatbot genérico, gerador genérico, qualificação genérica de leads.

---

## Atualização

Revisar este documento:
- A cada 6 meses como rotina.
- Imediatamente se qualquer player da camada 3.1 (WhatsApp+IA BR) lançar vertical solar.
- Imediatamente se OpenAI/Anthropic/Google lançar plataforma de agentes verticalizada por setor.
- Imediatamente se qualquer um dos três top concorrentes BR solar (Luvik, SolarMarket, Groner) lançar feature relevante (especialmente Luvik adicionando IA/WhatsApp ou SolarMarket abandonando o posicionamento anti-IA).
- Quando entrarmos em nicho novo via Suite Nexus Comercial — adicionar coluna de concorrentes do nicho.

Próxima revisão programada: **2026-11-19**.
