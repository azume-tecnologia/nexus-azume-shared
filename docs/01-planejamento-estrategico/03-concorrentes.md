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

| Player | Posicionamento | Postura |
|---|---|---|
| **Solfácil** | Originalmente financeira solar; oferece ferramentas de proposta/CRM para integradores parceiros [PREENCHER: nível de competição direta com Azume CRM em contas reais] | Competimos |
| **Bright** | Software de gestão e proposta solar BR | Competimos |
| **Genyx** | Gestão para integradoras solar | Competimos |
| **ProGD** | Gerador de projetos e propostas para GD | Competimos |
| **Click Solar** | Ferramentas de proposta e gestão | Competimos |
| **Tess (HelloTess)** | Internacional, foco em design de sistemas solar | Pressão indireta (não BR ainda) |
| **Aurora Solar** | EUA, dominante em design solar | Pressão indireta |

[PREENCHER: market share aproximado e overlap real com nossa base — Thúlio/Cláudia provavelmente têm anedotas de quem aparece em ciclos de vendas perdidos]

**Implicação estratégica:** hoje vencemos em volume e em parceria com a E4.0/ABSOLAR. A defesa de médio prazo é a **Suite Azume CRM Smart** (WhatsApp + IA via AI Apps, ver `04-ai-apps.md`), não features visuais.

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
- Quando entrarmos em nicho novo via Suite Nexus Comercial — adicionar coluna de concorrentes do nicho.

Próxima revisão programada: **2026-11-19**.
