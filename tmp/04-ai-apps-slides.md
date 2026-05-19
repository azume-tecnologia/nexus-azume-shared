# Planejamento de slides — AI Apps (apresentação executiva)

> Fonte: `docs/01-planejamento-estrategico/04-ai-apps.md`
> Destino: NotebookLM (gerar slides)
> Público: Victor e Thúlio
> Objetivo: explicar **o que é AI Apps**, **por que é estratégico**, **quais Suites** conseguimos construir, e o **tamanho do investimento** (5-8 meses, Paulo a 70%).
> Cronograma detalhado, checkpoints mensais e fases 3a/3b/3c **fora de escopo** desta apresentação (vivem no doc estratégico).

---

## Notas da revisão (gaps endereçados nesta versão)

A versão anterior cobria bem "o que é" e "quais Suites" mas deixava de fora seis elementos importantes para uma decisão executiva. Esta revisão os incorpora sem sair de 15 slides:

| Gap detectado | Onde foi endereçado |
|---|---|
| Riscos e mitigações ausentes | Novo slide 14 dedicado |
| Concorrência (Take Blip, GPTs, big tech) ausente | Incorporada ao slide 12 |
| "Por que agora" (janela temporal) ausente | Incorporada ao slide 12 |
| Caminho A vs. B (continuar custom vs. framework) ausente | Fechamento do slide 15 |
| Distinção Zoe vs. BYO ausente | Incorporada ao slide 7 (interfaces) + reforçada nas Suites |
| Alinhamento com os 6 critérios estratégicos ausente | Incorporada ao slide 12 |

Para abrir espaço, "Anatomia" (antigo slide 5) e "Blocos de construção" (antigo slide 6) foram **consolidados em um único slide 5** — o diagrama e o glossário de 5 termos ficam bem juntos.

**Inconsistências verificadas e resolvidas:**

- Strategic doc fala em "5-7 meses recomendado, podendo esticar para 8-9 no pior caso"; slides usam **"5 a 8 meses"** conforme orientação do Paulo — captura o cenário recomendado + buffer realista sem mostrar pior-caso explícito.
- "70% no AI Apps / 30% no CRM em manutenção conservadora" consistente entre slides e doc estratégico.
- Tiers PRO/PRO_PLUS/PRO_MAX com capacidade 1×/10×/20× consistentes.
- Ordem de execução Suite 3 → Suite 1 → Suite 2 consistente com §5 do doc estratégico.
- Cenários financeiros (pessimista 5% / realista 15% / otimista 30%) consistentes com §6.7.

---

## Estrutura — 15 slides

| # | Bloco narrativo | Slide |
|---|-----------------|-------|
| 1 | Abertura | Capa |
| 2 | Abertura | TL;DR |
| 3 | O problema | Gargalo do dev único |
| 4 | O que é | AI Apps em linguagem simples |
| 5 | O que é | Anatomia + blocos de construção |
| 6 | O que é | Plataforma vs. produto (analogia) |
| 7 | O que é | Como o cliente acessa (WhatsApp + Web + Zoe/BYO) |
| 8 | Suites | Suite 1 — Azume CRM Smart |
| 9 | Suites | Suite 2 — Nexus Comercial |
| 10 | Suites | Suite 3 — Sucesso do Cliente |
| 11 | Suites | Modelo comercial (assinatura única) |
| 12 | Por que importa | Importância estratégica + concorrência + por que agora |
| 13 | Por que importa | Retorno esperado (cenários) |
| 14 | Por que importa | Riscos e mitigações |
| 15 | Decisão | Esforço, equipe e decisão (caminho A vs. B) |

---

## Slide 1 — Capa

**Título:** AI Apps — Direção estratégica da Azume

**Subtítulo:** Como vamos produzir produtos pelos próximos anos

**Rodapé:** Apresentação executiva · Mai/2026

**Visual sugerido:** logo Azume + ilustração simples de "fábrica" / blocos compondo apps.

---

## Slide 2 — TL;DR (uma página resumindo tudo)

**Título:** O que está em jogo

**Conteúdo (4 blocos):**

- **O que é:** uma plataforma interna que permite **configurar** (em vez de programar) novas automações com IA generativa. Cada AI App é uma automação pronta para o cliente.
- **Por que importa:** hoje cada feature de IA precisa ser codada pelo Paulo (único dev). Com AI Apps, **não-engenheiros** (Lucas, Rodolfo) constroem novas automações. Paulo constrói a fábrica uma vez.
- **Investimento:** **5 a 8 meses** do Paulo no framework, dedicando **~70%** do tempo (CRM continua em manutenção conservadora).
- **Retorno esperado em 12 meses:** defesa do CRM via WhatsApp, redução de custo operacional interno (R$ 100-170k/ano), capacidade de testar nichos novos em dias.

**Visual sugerido:** 4 quadrantes com ícones.

---

## Slide 3 — O problema que estamos resolvendo

**Título:** Hoje, todo IA novo passa por um único engenheiro

**Conteúdo:**

- 1 desenvolvedor (Paulo) → toda iniciativa entra na mesma fila
- Cada feature de IA consome **semanas** de engenharia
- Não dá para testar uma ideia barata — tudo custa caro
- Marketing, suporte, CS, comercial **não podem "tentar uma coisa"** sem pedir desenvolvimento
- Lista de features razoáveis para os próximos 12 meses **não cabe** no ritmo atual

**Conclusão visual destacada:**
> Precisamos parar de "construir cada feature" e começar a "construir uma plataforma que produz features".

---

## Slide 4 — O que é AI Apps, em linguagem simples

**Título:** AI Apps é o "n8n da Azume", pensado para IA generativa

**Conteúdo:**

- Plataforma multi-tenant, integrada ao Nexus e ao Azume CRM
- Mas o que o **cliente vê não é a plataforma** — é cada AI App específico rodando em cima dela
- Exemplos de AI Apps:
  - "App Gerador de Propostas"
  - "App de Atendimento via WhatsApp"
  - "App Prospector de Clientes"
- O cliente final nunca precisa entender Trigger ou Hive — ele usa o produto pronto

**Visual sugerido:** caixa "Plataforma AI Apps" embaixo, vários AI Apps em cima (com ícones simples).

---

## Slide 5 — Anatomia e blocos de construção

**Título:** Três partes, cinco termos — é só isso

**Diagrama (topo do slide):**

```
   Evento  ───────▶   IA pensa   ───────▶   Ação
  (Trigger)         (Hive de agentes)      (Output)
```

- **Escuta** um evento (WhatsApp, agendamento, API, etc.)
- **Pensa** usando agentes de IA com instruções, conhecimento e ferramentas
- **Age** entregando o resultado onde o cliente espera

**Tabela (base do slide):**

| Termo | O que é |
|-------|---------|
| **App** | A automação inteligente pronta. O produto que o cliente vê. |
| **Trigger** | O evento que aciona o App (WhatsApp, agendamento, API…) |
| **Hive** | O "cérebro" — 1 ou mais agentes de IA que pensam para resolver o pedido |
| **Output** | Para onde o resultado vai (chat, WhatsApp, PDF, banco, outro App) |
| **Suite** | Agrupamento de AI Apps com objetivo comum (ex.: "Sucesso do Cliente") |

**Nota lateral:** Suite **não é** SKU — vendemos assinatura única que dá acesso a toda a biblioteca. Suite é como organizamos o catálogo, não como cobramos.

---

## Slide 6 — Plataforma vs. produto

**Título:** A plataforma é nosso meio de produção. O produto são os AI Apps.

**Conteúdo (tabela de analogia):**

| Plataforma | Produtos rodando em cima |
|------------|--------------------------|
| iOS | WhatsApp, Instagram, Uber |
| n8n | Automações de marketing, vendas, RH |
| Shopify | Lojas online de cada cliente |
| **AI Apps** | **Cada AI App que entregamos** |

**Mensagem-chave (destaque):**
> Construímos a plataforma uma vez. Depois disso, o time inteiro fabrica produtos em paralelo.

---

## Slide 7 — Como o cliente acessa

**Título:** WhatsApp é interface principal. Web é alternativa. Dois modelos de canal.

**Interfaces:**

- **WhatsApp** como porta de entrada principal — nosso público de baixa afinidade tech já usa o dia todo
- **WebChat** embeddable + página standalone com branding por tenant — alternativa para quem não quer ou não pode usar WhatsApp
- Também: chamadas de API, agendamento, eventos internos do Nexus

**Dois modelos de canal (importante para o escopo do MVP):**

| Modelo | Quando se aplica | Quem é dono do número |
|--------|------------------|------------------------|
| **Zoe** (canal Azume) | Cliente da Azume conversando com a Azume — Suite 1 (CRM Smart), versões internas das Suites 2 e 3 | Azume |
| **BYO** (Bring Your Own) | Cliente da Azume conversando com seu cliente final — Suites 2 e 3 externas | Cada cliente registra o próprio número |

**Implicação prática:**
> MVP roda inteiro na **Zoe** (já aprovada na Twilio). Onboarding BYO entra pós-MVP — a partir do momento que abrirmos as Suites externas.

---

## Slide 8 — Suite 1: Azume CRM Smart

**Título:** Defesa da vaca leiteira

**Para quem:** base atual do Azume CRM (integradores solares).

**Canal:** Zoe (cliente conversa com a Azume).

**O que entrega (via WhatsApp):**

| AI App | Exemplo de comando |
|--------|-------------------|
| Gerador de Proposta On-Grid | "Gere proposta para João Silva, 5kWp, em Curitiba" |
| Gerador de Proposta Híbrido/Off-Grid | (sistemas com baterias — hoje o CRM não atende) |
| Gerador de Relatórios | "Quanto vendi esse mês por vendedor?" |
| Gerador de Contratos | "Manda contrato para o José" |
| Gestão de Funis | "Move o lead da Maria para 'proposta enviada'" |

**A tese:** o cliente integrador roda **o Azume CRM inteiro por dentro do WhatsApp**. Tira o atrito de "tem que abrir o software".

**Valor de negócio:** aumenta valor percebido, justifica preço, abre upsell, reduz churn.

---

## Slide 9 — Suite 2: Nexus Comercial

**Título:** Primeiro produto Nexus standalone

**Para quem:** qualquer PME B2B com equipe comercial — começando pela própria Azume e pela base do CRM.

**Canal:** Zoe (versão interna Azume) → BYO (versão externa, pós-MVP).

**AI Apps:**

- **Prospector de Clientes** — busca e organiza leads qualificados
- **Qualificador de Leads** — conversa inicial para entender fit
- **Closer de Vendas** — conduz a venda até o fechamento
- **Pós-venda / Relacionamento** — mantém relacionamento ativo

**Ponto-chave (destaque):**
> Essa Suite é **clonável e adaptável por nicho**. O mesmo framework constrói a Suite 1 (interna) e a Suite 2 (para nichos novos). Investimento único, múltiplos retornos.

**Alavanca de distribuição:** E4.0 + ABSOLAR (Bárbara Rubim) — 372k inscritos no YouTube.

---

## Slide 10 — Suite 3: Sucesso do Cliente

**Título:** Uso interno imediato + receita externa depois

**Para quem (interno):** a própria Azume — Diógenes (suporte), Juliana (CS/renovação), Cláudia (operações).

**Para quem (externo, depois):** qualquer PME com base recorrente.

**Canal:** Zoe (uso interno) → BYO (uso externo).

**AI Apps:**

- **Atendimento ao Cliente** — resolve tickets de primeiro nível
- **Relacionamento com o Cliente** — contato proativo, identifica risco
- **Renovação de Contratos** — conduz renovação anual de forma proativa

**Por que essa Suite estreia ANTES da Suite 1 (apesar da Suite 1 ser a tese principal):**

1. **Validação sem risco externo** — roda dentro da Azume primeiro; se algo não funcionar, ajustamos sem expor cliente
2. **Ganho operacional imediato** — reduz 30-40% do trabalho operacional = **R$ 100-170k/ano de capacidade liberada**
3. **Stress-test do framework com casos reais** — quando a Suite 1 chegar nos clientes, o framework já está rodado

**Sequência de execução pós-framework:** Suite 3 (interna) → Suite 1 (rollout para base CRM) → Suite 2 (quando o nicho-alvo emergir).

---

## Slide 11 — Modelo comercial

**Título:** Uma assinatura. Toda a biblioteca. Três tiers.

**Princípio:** não vendemos "Suite 1" ou "AI App X". Vendemos **uma assinatura única do Nexus** que dá acesso a toda a biblioteca, presente e futura.

| Tier | Acesso aos AI Apps | Capacidade de IA por usuário |
|------|-------------------|------------------------------|
| **PRO** | Todos (com limites) | Capacidade gratuita (orçamento Azume) |
| **PRO_PLUS** | Todos | 10× a capacidade do PRO |
| **PRO_MAX** | Todos | 20× a capacidade do PRO |

**Cobrança:** mensal, em BRL, **por número de usuários** × valor por usuário no tier.

**Como o cliente cresce:** começa no PRO_PLUS → consome → quando o orçamento aperta, migra para PRO_MAX. **Upsell por valor extraído, sem renegociação.**

**Diferencial:** cada AI App novo que lançamos beneficia toda a base instantaneamente — motor de retenção, não complica a régua.

**Linha de receita adjacente (futura):** consultoria de adaptação por projeto/hora quando expandirmos equipe. Não está nos cenários financeiros — é upside.

---

## Slide 12 — Por que AI Apps é estratégico (agora)

**Título:** Cinco motivos pelos quais essa é a decisão certa **agora**

**Os cinco motivos:**

1. **Resolve a restrição fundamental** — depois do framework, o time inteiro produz AI Apps em paralelo. Paulo deixa de ser gargalo.
2. **Reduz drasticamente o custo de errar** — testar nicho novo passa de meses para dias/semanas. Em um momento em que não sabemos qual o próximo nicho, ter capacidade de testar muitos vale mais do que apostar em um.
3. **Defende a base atual** — renovação do CRM é 80% da receita. Suite 1 aumenta valor percebido, cria barreira de saída, reduz churn.
4. **Diferencial competitivo no nicho** — WhatsApp + IA + Azume CRM nativo **não é feito por nenhum concorrente direto** no nicho de integradores solar BR hoje.
5. **Habilita o Nexus standalone** — sem AI Apps, o Nexus precisaria ser custom-codado para cada nicho novo.

**Alinhamento com os critérios do planejamento estratégico (todos os 6 atendidos):**

> Distribuição (base CRM + E4.0) · IA como diferencial real · SaaS · B2B · Nichado (cada AI App é nichado por construção) · Viável com recursos limitados.

**Mas e a concorrência?**

| Camada | Quem joga | Por que ainda ganhamos |
|--------|-----------|------------------------|
| WhatsApp + IA generalista BR | Take Blip, Zenvia, WATI | **Não atendem o nicho solar** — podem entrar em 12-24 meses |
| Plataformas no-code globais | Lindy, Relevance AI | **Sem conhecimento de domínio nem distribuição BR** |
| Big tech (GPTs, Skills) | OpenAI, Anthropic | Competem **na categoria que decidimos não competir** ("soluções óbvias com IA") |

> Nossa defesa **não é tecnológica** (a tecnologia comoditiza). É **conhecimento de domínio + canal E4.0/ABSOLAR + integração nativa com o Azume CRM**. A janela é **temporal**.

**Por que agora, não daqui 1 ano:**

- Mercado em contração → quanto mais cedo defendermos a base, menos churn acumulado
- Modelos de IA confiáveis para produção **existem agora** (há 18 meses, não existiam)
- Paulo está em **pico de produtividade** com agentes de codificação — esperar é perder essa janela

---

## Slide 13 — Retorno esperado em 12 meses

**Título:** Mesmo no pior cenário, o investimento se paga

**Cenários de Suite 1 (Azume CRM Smart, sobre a base atual):**

| Cenário | Adoção | Receita nova anual | % sobre renovação atual | Payback |
|---------|--------|--------------------|-----------------------:|---------|
| Pessimista | 5% | ~R$ 234k | +8% | 10-12 meses |
| **Realista** | **15%** | **~R$ 787k** | **+26%** | **3-4 meses** |
| Otimista | 30% | ~R$ 1,75M | +58% | 1-2 meses |

**Cenário Suite 3 — uso interno (custo evitado):**

- Equipe atual de 3 pessoas em CS/suporte/renovação ≈ R$ 25-35k/mês
- Suite 3 reduzindo 30-40% do trabalho = **R$ 100-170k/ano de capacidade liberada**

**Cenário Suite 2 — Nexus Comercial standalone:**

- 50 contas no PRO_PLUS = R$ 120k/ano (entrada em um nicho)
- 200 contas, 40% no PRO_MAX = R$ 720k/ano (médio prazo)

**Mensagem-chave (destaque):**
> Mesmo se a Suite 1 não engajar como esperado, a Suite 3 (uso interno) **já paga parte do investimento via custo evitado**. Pior cenário não é zero. É retorno menor que o esperado.

---

## Slide 14 — Riscos e mitigações

**Título:** Sendo honesto: quatro riscos relevantes

| Risco | Mitigação |
|-------|-----------|
| **Bus factor do framework** — bug no framework afeta todos os AI Apps; Paulo vira ponto único de falha de uma plataforma inteira | Disciplina de testes automatizados (Hive runner, política de acesso, orçamento IA). Documentação operacional desde o início. Reavaliar contratação de 2º engenheiro em 12-18 meses. |
| **Escopo crescendo silenciosamente** — desenho completo é grande (15+ entidades); 5-8 meses podem virar 9-12 | MVP com escopo radicalmente menor que o desenho completo. Cloning self-service, drag-and-drop UI, geração de PDF complexos ficam para depois. Checkpoints mensais com escopo congelado. |
| **Construir framework para AI Apps que ninguém quer** — investir e descobrir que não tem demanda | Entrevistas de validação com 10-15 clientes em paralelo (meses 2-4). Se a hipótese cair, pivotamos antes do mês 5. **Mesmo que Suite 1 não engaje, Suite 3 (interna) ainda gera retorno via custo evitado** — downside da decisão não é zero. |
| **WhatsApp como dependência regulada** — mudanças de política Meta, ban de número, falha do provedor | Arquitetura prevê troca de provedor sem reescrita. `quality_rating` exposto + alertas operacionais (não silencioso). **WebChat entra no MVP justamente como plano B arquitetural**. AI Apps críticos sempre têm caminho alternativo. |

**Risco extra (segurança):** prompt injection em produto multi-tenant que toca dados financeiros e contratos é problema sério.

**Mitigação:** segurança tratada como **cidadã de primeira classe** da arquitetura — least privilege por agente, separação read/write, tenant context imutável, validação server-side, human-in-the-loop para operações irreversíveis, **suite de testes adversariais bloqueando release**.

---

## Slide 15 — Esforço, equipe e decisão

**Título:** O que está sendo pedido

**Investimento:**

- **5 a 8 meses** do Paulo no framework
- Dedicação de **~70%** ao AI Apps; **30%** mantendo o CRM em modo conservador (sem features novas, apenas bugs e ajustes pequenos)
- **Lucas e Rodolfo** entram em treinamento desde o mês 1; constroem AI Apps em produção a partir do mês 5-6

**Quem faz o quê:**

- **Paulo** — constrói o framework (a fábrica)
- **Lucas, Rodolfo** — constroem AI Apps em cima do framework (os produtos)
- **Victor** — supervisão dos AI Apps de CS/Sucesso do Cliente
- **Thúlio** — supervisão dos AI Apps de Comercial/Marketing

**O que está sendo pedido aos sócios:**

1. **Aprovar o investimento** de 5-8 meses do Paulo no framework
2. **Aceitar o CRM em modo conservador** durante o período (sem features novas)
3. **Alocar 20-30% do tempo de Lucas e Rodolfo** para construção dos primeiros AI Apps

**A escolha que está na mesa:**

| Caminho A — continuar custom-codando | Caminho B — investir no framework |
|--------------------------------------|-----------------------------------|
| Cada feature consome semanas do Paulo | Investimento único de 5-8 meses |
| Lucas, Rodolfo, Victor, Thúlio dependem de fila de TI | Time inteiro produz AI Apps em paralelo |
| Testar nicho novo custa meses | Testar nicho novo custa dias/semanas |
| WhatsApp seria feature isolada e cara | WhatsApp vira primitiva da plataforma |
| Lista de features razoáveis leva 3-5 anos | Lista de features viável em 12-18 meses |

**E se decidirmos NÃO fazer?**

> Status quo. Cada feature nova de IA leva semanas/meses. Concorrentes (incluindo big tech) ganham espaço enquanto esperamos. Base do CRM continua sem o salto de UX que diferenciaria a Azume.

**Encerramento (destaque):**
> AI Apps não é uma feature. É uma decisão de arquitetura sobre **como a Azume vai produzir produtos pelos próximos anos**.

---

## Observações para quem montar os slides no NotebookLM

- **Tom:** profissional, direto, sem emojis. Linguagem clara para Victor e Thúlio (Paulo já está alinhado).
- **Densidade:** cada slide tem 1 ideia central + 3-5 bullets de suporte. Não encher. Os slides 12, 14 e 15 são naturalmente mais densos — manter tabelas enxutas.
- **Visuais sugeridos onde notei:** diagrama de Trigger → Hive → Output (slide 5), analogia plataforma/produto (slide 6), tabelas comparativas (slides 7, 12, 14, 15), tabela de cenários (slide 13). Os demais são listas e texto.
- **Pular cronograma detalhado:** checkpoints mensais, fases 3a/3b/3c, métricas de sucesso por checkpoint — **tudo fora do escopo** desta apresentação. Quem quiser mergulhar, consulta o doc estratégico.
- **Reforçar 3x ao longo da apresentação:** (a) o cliente vê AI Apps prontos, não a plataforma; (b) assinatura única, não SKU por Suite; (c) Paulo constrói a fábrica uma vez, time inteiro produz depois.
- **Fechar com o caminho A vs. B** (slide 15) — é o quadro mental que faz a decisão ficar óbvia.
