# AI Apps — Direção Estratégica

> Direção estratégica de plataforma e produto. Fonte de verdade da decisão de construir o framework AI Apps e do modelo de negócio que roda sobre ele. Originalmente preparado como apresentação executiva para Victor e Thúlio (mai/2026); promovido a documento estratégico permanente. Foco em valor de negócio e nas decisões que moldam o produto, não em detalhes técnicos — esses estão no blueprint em `/home/paulo/projects/nexus/nexus-core-2/docs/blueprints/ai_apps.md`. Última atualização: 2026-05-19.

---

## TL;DR (1 minuto)

**O que é:** AI Apps é uma plataforma interna que permite a Azume **configurar** (em vez de **programar**) novas automações com IA generativa. Cada "AI App" é uma automação inteligente pronta para ser usada por um cliente — gerar uma proposta, responder no WhatsApp, qualificar um lead, gerar um relatório, etc.

**Por que importa:** hoje cada nova feature de IA do Nexus precisa ser codada à mão pelo Paulo, o único engenheiro. Com AI Apps, **não-engenheiros do nosso time** (começando por Lucas e Rodolfo) constroem novas automações. O Paulo constrói a fábrica uma vez; depois disso a equipe inteira fabrica produtos.

**Investimento:** 4 a 6 meses do Paulo focado no framework (cenário base). Sob o cenário operacional recomendado, em que o Paulo divide tempo entre framework e CRM, o cronograma vai a 5-7 meses. Lucas e Rodolfo entram em treinamento desde o mês 1 e começam a construir AI Apps reais conforme cada fase do framework é entregue.

**Modelo de monetização:** assinatura única Nexus que dá acesso a **toda a biblioteca de AI Apps**. Diferenciação de planos é por **volume de consumo de tokens de IA**, não por funcionalidades. Cliente paga proporcional ao uso real; cresce de tier conforme adota mais.

**Retorno esperado em 12 meses (cenário realista, dimensionado em §6.7):**
1. **Suite Azume CRM Smart** — defesa da vaca leiteira. Potencial de +25% de receita nova sobre a base atual no cenário realista (faixa de +8% pessimista a +58% otimista). Payback do investimento em 3-4 meses no cenário realista.
2. **Nexus no WhatsApp** — interface familiar para um público com baixa afinidade tech. Primeira vez que um concorrente direto no Brasil oferece isso para integradores solar.
3. **Capacidade de testar nichos novos com custo de dias** — quando identificarmos o próximo mercado, o tempo de "ir ao mercado" cai de meses para semanas.
4. **Redução de custo operacional interno** — Suite Sucesso do Cliente reduz 30-40% do trabalho manual de Diógenes / Juliana / Cláudia, liberando R$ 100-170k/ano de capacidade.

**Decisão tomada em mai/2026:** comprometer 5-7 meses de engenharia em construir o framework (cenário recomendado), em vez de continuar codando feature por feature e manter o gargalo do dev único. Racional registrado abaixo; este documento serve de referência para checkpoints durante a execução.

---

## Glossário rápido (5 termos que aparecem ao longo do documento)

Para acompanhar este documento sem se prender a jargão:

- **AI App** — uma automação inteligente pronta para uso (ex.: "App Gerador de Propostas"). É o **produto** que o cliente vê.
- **Suite** — agrupamento de AI Apps com objetivo comum (ex.: "Suite Azume CRM Smart" reúne 5 AI Apps).
- **Trigger** — o evento que aciona um AI App (mensagem no WhatsApp, agendamento, chamada de API, etc.).
- **Hive** — o "cérebro" do AI App: 1 ou mais agentes de IA que pensam para resolver o pedido.
- **Output** — para onde o resultado vai (chat, WhatsApp, PDF, outro sistema, banco, outro AI App).

Definições completas no §3.

---

## 1. O problema que estamos resolvendo

Hoje, **toda capacidade nova de IA no Nexus precisa ser construída em código pelo Paulo**. Isso significa:

- Cada feature consome semanas de engenharia.
- O único engenheiro é gargalo permanente — toda iniciativa entra na mesma fila.
- Não conseguimos testar uma ideia barata. Tudo custa caro porque tudo passa pelo mesmo recurso escasso.
- A equipe operacional (marketing, suporte, CS, comercial) não pode "tentar uma coisa" sem pedir desenvolvimento.

Quando olhamos para a lista de features que faria sentido entregar nos próximos 12 meses (gerador de proposta híbrido/off-grid, Nexus consultivo do CRM, geração de propostas via linguagem natural, gestão de funis via WhatsApp, geração de contratos, etc.), fica claro que **fazer todas em código direto não é viável** com um único engenheiro.

A saída é mudar de "construir cada feature" para "construir uma plataforma que produz features".

---

## 2. O que é AI Apps, em linguagem simples

Pense em AI Apps como o **n8n da Azume** — só que pensado para IA generativa, multi-tenant (cada cliente tem seu espaço isolado), e integrado ao Nexus e ao Azume CRM.

Mas o **valor que o cliente vê** não é a plataforma — é cada AI App específico que rodamos em cima dela. O cliente final nunca precisa entender o que é um Trigger ou um Hive. Ele apenas usa um produto pronto: "App Gerador de Propostas", "App de Atendimento via WhatsApp", "App Prospector de Clientes".

A analogia que ajuda:

```
Plataforma            Produtos rodando em cima
─────────────         ────────────────────────
iOS                   WhatsApp, Instagram, Uber, etc.
n8n                   Automações de marketing, vendas, RH
Shopify               Lojas online de cada cliente
AI Apps               Cada AI App que entregamos
```

A plataforma é nossa infraestrutura. Os AI Apps são o produto.

### Como um AI App funciona, em uma frase

Um AI App é uma automação que:

1. **Escuta** um evento (mensagem no WhatsApp, clique em botão, agendamento, chamada de API)
2. **Pensa** usando um ou mais agentes de IA com instruções, conhecimento e ferramentas
3. **Age** entregando o resultado em um ou mais lugares (chat, arquivo, WhatsApp, banco, outro App)

```
   Evento ────▶  IA pensa ────▶  Ação
  (gatilho)     (1+ agentes)    (saída)
```

É isso. A complexidade está em **como configuramos cada uma dessas três partes** — e é exatamente aí que a plataforma ganha seu valor: ela transforma essa configuração em algo que não-engenheiros conseguem fazer.

---

## 3. Os blocos de construção

A plataforma tem alguns blocos. Para a apresentação, cobrimos só os que importam para entender o produto.

### 3.1 App

**O que é:** a unidade que tem valor de negócio. Cada App é uma automação completa que faz uma coisa específica.

**Que tipo de capacidade fornece:** entrega do início ao fim — recebe um gatilho, processa com IA, devolve resultado.

**Exemplos:**
- App "Gerador de Proposta On-Grid" — recebe dados do cliente, gera proposta, salva no CRM.
- App "Atendimento WhatsApp" — recebe mensagem, entende o pedido, responde via WhatsApp.
- App "Qualificador de Lead" — recebe novo contato, pesquisa informações, pontua e classifica.

### 3.2 Trigger (gatilho)

**O que é:** o jeito que o App é acionado. Define quando e como o App "acorda" para executar.

**Tipos de gatilho que a plataforma oferece:**

| Tipo | Quando dispara | Exemplo |
|------|----------------|---------|
| Manual (chat) | Usuário envia mensagem no chat do Nexus | "Gere uma proposta para esse cliente" |
| WhatsApp | Cliente final envia mensagem no WhatsApp | "/proposta João Silva 5kWp" |
| Webhook | Sistema externo chama uma URL nossa | ERP do cliente avisa que houve nova fatura |
| Agendado | Em horários definidos (cron) | Toda segunda 8h, gerar relatório semanal |
| Evento interno | Algo aconteceu no Nexus/CRM | Novo lead foi criado |

**Capacidade que fornece:** **abre o produto para qualquer porta de entrada**. Isso é importante porque podemos atender o cliente onde ele já está (WhatsApp), em vez de forçar ele a abrir um software novo.

### 3.3 Hive — o "cérebro" do App

**O que é:** o conjunto de agentes de IA que o App usa para processar.

**Por que se chama Hive (colmeia):** porque um App pode ter **um único agente** ou **vários agentes trabalhando juntos**, cada um especializado em uma parte do problema, passando trabalho entre si.

**Que tipo de capacidade fornece:** raciocínio de IA em camadas — desde o mais simples ("um agente responde uma pergunta") até o mais sofisticado ("agente A entende o pedido, passa para agente B que pesquisa, passa para agente C que decide e responde").

**Exemplos:**
- Hive simples: 1 agente que recebe a fatura e extrai os dados.
- Hive em cascata: agente A extrai dados da fatura, agente B busca preços no CRM, agente C monta a proposta final.

### 3.4 Agent (agente)

**O que é:** uma "configuração de IA" — um agente é definido por três coisas: instruções (o que ele faz), conhecimento (o que ele sabe), e ferramentas (o que ele pode fazer no mundo).

**Capacidade que fornece:** especialização. Cada agente é especialista em um pedaço do problema.

**Exemplo:** agente "Analista de Propostas Solar" — instruções: "Você é um engenheiro especialista em dimensionar sistemas fotovoltaicos. Sempre considere a tarifa local da distribuidora." | Conhecimento: bases técnicas, tarifas, manuais. | Ferramentas: API do CRM, calculadora de payback.

### 3.5 KnowledgeBase (base de conhecimento)

**O que é:** uma biblioteca de documentos que o agente consulta na hora de responder.

**Capacidade que fornece:** o agente "conhece" coisas específicas do nosso negócio (ou do nicho do cliente) — não fica preso só ao que o modelo de IA já sabe de fábrica.

**Exemplos:**
- Base de manuais técnicos de inversores e baterias.
- Base de contratos modelo da Azume.
- Base de FAQs do suporte.
- Base de regras tarifárias por distribuidora.

### 3.6 Tool (ferramenta)

**O que é:** uma capacidade que o agente pode invocar no meio da execução para fazer algo no mundo. Não é só "responder texto" — é também "consultar um sistema", "criar um registro", "enviar uma mensagem".

**Capacidade que fornece:** **conexão com o mundo real**. Sem ferramentas, agentes só conversam. Com ferramentas, agentes **agem**.

**Exemplos de ferramentas que vamos disponibilizar:**
- Consultar cliente no Azume CRM
- Criar uma proposta no Azume CRM
- Mover um lead no funil de vendas
- Buscar dados públicos de uma empresa (CNPJ, contatos)
- Enviar email
- Disparar assinatura de contrato

### 3.7 Output (saída)

**O que é:** onde o resultado do App é entregue.

**Tipos de saída:**

| Tipo | Para onde vai |
|------|---------------|
| Mensagem de chat | Volta para o chat do Nexus |
| Mensagem WhatsApp | Envia mensagem WhatsApp para um contato |
| Arquivo | Gera PDF / Excel / Word / CSV |
| Relatório | Gera um relatório interativo no Nexus |
| Chamada de webhook | POSTa em um sistema externo |
| Banco de dados | Grava em uma coleção que pode ser consultada depois |
| Outro App | Dispara outro AI App em sequência |

**Capacidade que fornece:** **entrega o valor onde o cliente espera consumir**. Mesmo resultado pode ir para múltiplos lugares ao mesmo tempo (PDF + WhatsApp + chat).

### 3.8 Channel (WhatsApp e WebChat)

**O que é:** o canal externo através do qual um AI App conversa com o mundo. É a abstração que une WhatsApp, WebChat e potencialmente outros providers futuros (Telegram, voz) sob um contrato único — mesmo evento de entrada, mesma execução de App, mesma forma de saída.

**Capacidade que fornece:** **leva o produto até a interface que o usuário já usa**. WhatsApp é estratégico demais para nosso público brasileiro de baixa afinidade tech para ficar escondido como detalhe. WebChat embeddable é alternativa instantânea para quem não quer ou não pode usar WhatsApp.

**Providers de Channel disponíveis no MVP:**
- **WhatsApp** — via Twilio inicialmente (a Zoe já está aprovada e online). Candidatos a substituição pós-MVP: Meta Cloud API direto, 360dialog. Soluções não-oficiais (Z-API, Evolution API, Baileys) **excluídas por política**. Detalhamento em §12.
- **WebChat** — widget JS embeddable em sites de cliente + página standalone com branding por tenant. Funciona contra o mesmo App sem código provider-específico.

**Exemplos:**
- Channel "Zoe" (Azume) — número WhatsApp da Azume; vários AI Apps internos escutam (suporte, comercial, atendimento a integradores).
- Channel BYO (cliente) — integrador registra o próprio número WhatsApp; AI App de pós-venda da empresa dele responde aos clientes finais dele (Suites externas; pós-MVP).
- Channel WebChat embeddable — chat no site do cliente final com tema do integrador.

**Arquitetura completa de Channel** (quem pertence a quem, regras Zoe vs. BYO por Suite, multi-tenancy, decisões em aberto) está detalhada em §11.

### 3.9 Suite (conjunto de AI Apps)

**O que é:** um agrupamento organizacional de AI Apps relacionados, com um objetivo comum.

**Capacidade que fornece:** **navegação e UX do catálogo**. Em vez de uma lista plana de 30 AI Apps, o cliente entra em "Sucesso do Cliente" e encontra os 3 Apps relevantes ali. Suites também são a unidade interna de planejamento — desenhamos uma Suite, depois construímos seus AI Apps.

**Importante — o que Suite NÃO é:** Suite não é SKU. **Não vendemos "a Suite 1"** — vendemos uma assinatura única do Nexus que dá acesso a toda a biblioteca (§5.4). Suite é como organizamos o catálogo internamente e como o cliente navega; não é como cobramos.

### 3.10 Templates

**O que é:** AI Apps e Suites criados pelo nosso time interno e disponibilizados a todos os assinantes. Quem usa um Template está usando um AI App pronto da nossa biblioteca, sem precisar configurar nada.

**Capacidade que fornece:** **escala**. Construímos uma vez (Lucas / Rodolfo configurando), milhares de clientes usam o mesmo AI App.

**Sobre clonagem:** no MVP, todos os Templates rodam centralizados pela Azume — o cliente consome, não clona. Cloning self-service pelo cliente (para customizar) é pós-MVP. Adaptação consultiva via equipe Azume vira linha de serviço cobrável à parte conforme expandirmos equipe (ver §5.4).

### 3.11 Run, Report e observabilidade

Cada execução de um AI App é uma "Run" — registramos tudo que aconteceu (qual agente fez o quê, quais ferramentas usou, quanto custou em IA). Isso garante:

- **Auditoria:** podemos provar para o cliente o que o App fez.
- **Custo:** sabemos exatamente quanto de IA cada cliente está consumindo (consumo aplicado contra o orçamento do tier — base para upgrade natural de PRO_PLUS para PRO_MAX).
- **Diagnóstico:** quando algo dá errado, conseguimos entender por quê e corrigir.

---

## 4. A solução completa, agora com entendimento

Voltando ao desenho do início, agora cada parte faz sentido:

```
   ┌─────────────────────────────────────────────────────────────┐
   │                       AI APP                                │
   │                                                             │
   │   ┌──────────┐      ┌──────────────────┐      ┌──────────┐  │
   │   │ Trigger  │─────▶│       Hive       │─────▶│  Output  │  │
   │   │          │      │                  │      │          │  │
   │   │ WhatsApp │      │  Agent A         │      │ WhatsApp │  │
   │   │ Chat     │      │   ├─ Tools       │      │ PDF      │  │
   │   │ Webhook  │      │   ├─ Knowledge   │      │ Relatório│  │
   │   │ Cron     │      │   └─ Instruções  │      │ Webhook  │  │
   │   │ Evento   │      │                  │      │ DB       │  │
   │   │          │      │  Agent B (opc)   │      │          │  │
   │   │          │      │  Agent C (opc)   │      │          │  │
   │   └──────────┘      └──────────────────┘      └──────────┘  │
   │                                                             │
   └─────────────────────────────────────────────────────────────┘

      Cada AI App vive dentro de uma conta (multi-tenant).
      AI Apps podem ser agrupados em Suites (organização de catálogo).
      AI Apps prontos da biblioteca Azume = Templates consumidos por todos os assinantes.
```

Em uma frase: **AI Apps é uma fábrica de automações inteligentes** que a Azume opera internamente, com produtos que entregamos aos clientes via interfaces que eles já conhecem (WhatsApp principalmente, mas também web e via integrações).

---

## 5. Casos de uso — três Suites concretas

Para tornar tangível, três Suites que pretendemos construir ao longo dos próximos 12-18 meses, em paralelo com a entrega das fases finais do framework e logo após. Nomes "Suite 1 / 2 / 3" são provisórios — marca e nomenclatura comercial ficam para mais tarde.

### Suite 1 — Azume CRM Smart

**Público-alvo:** base atual do Azume CRM (integradores de energia solar).
**Valor de negócio:** defesa da vaca leiteira. Aumenta valor percebido, justifica preço, reduz churn, abre espaço para upsell.
**Como o cliente acessa:** disponível em qualquer plano pago da assinatura Nexus (PRO_PLUS ou PRO_MAX). Detalhes do modelo comercial em §5.4.
**Amarração com o backlog atual:** os AI Apps abaixo materializam diretamente as features 1.1 a 1.6 da Suite 1 no `tmp/features.md` (gerador de propostas híbridas/off-grid, consulta ao CRM, relatórios, propostas on-grid, gestão de funis, contratos). A interface principal é o WhatsApp, tratado como primitivo do framework no `tmp/features.md` §5.1.

| AI App | O que faz | Como o cliente usa | Feature |
|--------|-----------|--------------------|---------|
| Gerador de Proposta On-Grid | Recebe dados do cliente e gera proposta completa | "Gere proposta para João Silva, 5kWp, em Curitiba" via WhatsApp | 1.4 |
| Gerador de Proposta Híbrido/Off-Grid | Idem, mas para sistemas com baterias | Permite o CRM atender sistemas que hoje não consegue | 1.1 + 1.4 |
| Gerador de Relatórios Dinâmicos | Cria relatórios sob demanda da operação do cliente | "Quanto vendi esse mês por vendedor?" via WhatsApp | 1.2 + 1.3 |
| Gerador de Contratos | Monta e envia contrato para assinatura | "Manda contrato para o José" via WhatsApp | 1.6 |
| Gestão de Funis e Quadros | Movimenta leads/negócios/obras no funil | "Move o lead da Maria para 'proposta enviada'" via WhatsApp | 1.5 |

**Interface principal:** WhatsApp (tratado como primitivo da plataforma — `tmp/features.md` §5.1). Web continua disponível, mas a aposta de UX é WhatsApp.

**A tese que une essa Suite:** o cliente integrador roda **o Azume CRM inteiro por dentro do WhatsApp**. Tira o atrito de "tem que abrir o software" e troca por algo que ele já usa o dia todo. Para nosso público de baixa afinidade tech, isso é diferencial gigantesco.

### Suite 2 — Nexus Comercial

**Público-alvo:** qualquer PME B2B que tenha equipe comercial — começando pela própria Azume e pela base atual.
**Valor de negócio:** produto de ticket mais alto, possivelmente o **primeiro produto Nexus standalone** (fora da base do CRM). Tende a atrair contas com mais usuários e maior consumo de IA, levando-as naturalmente ao tier PRO_MAX (ver §5.4).
**Como o cliente acessa:** mesma assinatura única do Nexus. Equipes comerciais maiores tendem a consumir mais tokens, o que move o cliente do tier PRO_PLUS para PRO_MAX. É no PRO_MAX que atingimos o alvo de ticket >R$ 12k/ano definido pelo planejamento estratégico para novos nichos.
**Alavanca de distribuição:** a E4.0 (canal de 372 mil inscritos no YouTube, presidida por Bárbara Rubim, ABSOLAR) é caminho natural para o lançamento desta Suite no nicho de integradores. Para nichos fora de energia, precisaremos identificar canal próprio.

| AI App | O que faz |
|--------|-----------|
| Prospector de Clientes | Busca e organiza leads qualificados a partir de critérios definidos |
| Qualificador de Clientes Prospectados | Conversa inicial com lead para entender fit e nível de interesse |
| Closer de Vendas | Conduz a conversa de venda até fechamento |
| Pós-venda / Relacionamento | Mantém relacionamento ativo após o fechamento |

**Ponto-chave:** essa Suite é **clonável e adaptável por nicho**. O mesmo conjunto de AI Apps pode ser configurado de formas diferentes para nichos com ciclos de vendas distintos (B2B de tecnologia vs. B2B de serviços vs. nicho específico que descobrirmos). Isso conecta diretamente com a tese estratégica de "Nexus como plataforma de expansão para novos nichos" do nosso planejamento.

Importante: o **mesmo framework AI Apps** que constrói a Suite 1 (interna) também constrói a Suite 2 (comercial para nichos novos). Investimento único, múltiplos retornos.

### Suite 3 — Sucesso do Cliente

**Público-alvo:** qualquer PME que tenha base de clientes recorrente — a própria Azume é cliente potencial óbvio aqui.
**Valor de negócio:** reduz custo operacional de suporte/CS/renovação. Aplicável a nós mesmos imediatamente.
**Como o cliente acessa:** mesma assinatura única (ver §5.4). Para uso interno da Azume, é "custo evitado" em vez de receita — o orçamento de IA é consumido pela própria operação Azume.

| AI App | O que faz |
|--------|-----------|
| Atendimento ao Cliente | Resolve tickets de suporte de primeiro nível |
| Relacionamento com o Cliente | Mantém contato proativo, identifica clientes em risco |
| Renovação de Contratos | Conduz processo de renovação anual de forma proativa |

**Por que essa Suite é estratégica para nós internamente:** Diógenes (suporte), Juliana (CS/renovação) e a Cláudia (operações) hoje fazem manualmente. Se a Suite 3 reduzir em 30-40% o trabalho operacional deles, conseguimos crescer base sem crescer equipe.

### 5.4 Modelo comercial — único para toda a biblioteca

**O modelo é uma assinatura única do Nexus.** Não há SKU por Suite nem por AI App — quem assina ganha acesso a toda a biblioteca, presente e futura.

**Três tiers de plano** (definidos na spec `subscription_management.md`):

| Tier | Acesso aos AI Apps | Capacidade de IA por usuário | Posicionamento |
|------|--------------------|-----------------------------:|----------------|
| **PRO** | Todos (com limites) | Capacidade gratuita (compartilhada do orçamento da Azume) | Tier gratuito de entrada; mantém a base atual sem migração forçada |
| **PRO_PLUS** | Todos | 10× a capacidade do PRO | Conta com uso regular de IA |
| **PRO_MAX** | Todos | 20× a capacidade do PRO | Conta com uso intenso ou equipes maiores |

**Como funciona a cobrança:**
- Mensal em BRL, processada pelo Pagar.me.
- Preço calculado por **número de usuários da conta** × **valor por usuário no tier escolhido** (definido em YAML versionado, ajustável conforme estratégia).
- Exemplo ilustrativo (números a definir pelos sócios): conta com 5 usuários no PRO_PLUS pode pagar 5 × R$ 40 = R$ 200/mês.

**Como o cliente "cresce" no produto:**
- Cliente adota o Nexus no PRO_PLUS, experimenta, consome.
- Conforme adoção aumenta (mais AI Apps usados, com mais frequência), consumo de tokens cresce.
- Quando o orçamento do PRO_PLUS começa a apertar, cliente migra para PRO_MAX (upsell natural por valor extraído, sem renegociação de contrato).
- **Não há "venda de feature adicional"** — toda inovação que lançamos (cada AI App novo) é instantaneamente acessível em qualquer tier pago. Isso vira motor de retenção, não complica a régua de cobrança.

**Linha de serviço adjacente (futura, requer expansão de equipe):**
- Consultoria de adaptação: nossa equipe customiza Suites/AI Apps para o negócio específico da conta.
- Cobrança à parte (projeto / hora / pacote), não inclusa na assinatura.
- Ainda não é prioridade do MVP; abre quando tivermos demanda explícita e capacidade interna.

**Por que esse modelo é estratégico para a Azume:**
- **Decisão de compra simples:** uma porta de entrada, sem matriz "qual Suite preciso".
- **Upsell sem atrito:** o cliente migra de tier conforme valor extraído, não por venda ativa de feature.
- **Retenção composta:** cada AI App novo que lançamos beneficia toda a base instantaneamente.
- **Cobrança proporcional ao valor:** quem usa pouco paga pouco; quem usa muito paga proporcional ao uso real (via consumo de tokens).
- **Alinhado com a spec já em construção** (`subscription_management.md`) e com o sistema de budget de IA já em produção.

### Por que essa ordem (Suite 3 → Suite 1 → Suite 2)?

A tese **principal** de negócio é a Suite 1 (defesa do CRM via WhatsApp + linguagem natural). Mesmo assim, a sequência de execução pós-framework prioriza Suite 3 primeiro, por três razões:

1. **Validação interna sem risco externo.** A Suite 3 roda dentro da Azume com nossos próprios funcionários. Se algo não funciona como esperado, ajustamos sem expor cliente.
2. **Ganho operacional imediato.** Reduzir 30-40% do trabalho de suporte/CS já paga parte do investimento antes mesmo de gerarmos receita nova.
3. **Stress-test do framework com casos reais.** A Suite 3 exercita Triggers (WhatsApp + evento interno), Hives multi-agente, Tools de API. Quando partirmos para a Suite 1 com clientes, o framework já está rodado.

A Suite 1 entra logo após — meses 6-8 em piloto, mês 9+ em rollout geral. A Suite 2 entra quando tivermos clareza sobre qual nicho atacar primeiro.

---

## 6. Importância estratégica para a Azume

### 6.1 Resolve a restrição fundamental

A maior restrição da empresa hoje, dado o cenário, é: **um único engenheiro de software**. Toda iniciativa que queremos rodar passa pelo Paulo. Esse gargalo é absoluto.

AI Apps **muda isso estruturalmente**: depois do framework pronto, Lucas e Rodolfo (com supervisão de Victor e Thúlio) constroem AI Apps. O Paulo continua sendo o único engenheiro, mas a empresa inteira ganha capacidade de produzir produtos.

É a única forma realista que vemos de aumentar throughput sem aumentar equipe de TI no curto prazo — e aumento de equipe de TI conflita com a estratégia atual de maximizar lucro e distribuir aos sócios.

### 6.2 Reduz drasticamente o custo de errar

Hoje, se decidimos atacar um nicho novo, gastamos meses construindo um produto custom para ele. Se o nicho não responder, perdemos meses.

Com AI Apps, atacamos o mesmo nicho com uma Suite clonada e adaptada em dias/semanas. Se o nicho não responder, perdemos dias/semanas. Podemos atacar 5-10 nichos no tempo que hoje gastamos atacando 1.

Em um momento em que **não sabemos qual o próximo nicho** (e o documento de planejamento estratégico reconhece isso explicitamente), ter a capacidade de testar muitos é mais valioso do que apostar tudo em um.

### 6.3 Defende a base atual com features de alto valor

A receita de renovação anual do CRM é 80% da nossa receita. Mercado em contração ameaça essa renovação. A Suite Azume CRM Smart é a resposta direta para essa ameaça:

- Aumenta valor percebido do produto atual.
- Justifica preço atual e abre espaço para upsell.
- Cria barreira de saída (o cliente acostuma com produtividade nova).
- Reduz churn.

### 6.4 Diferencial competitivo no nicho

WhatsApp como interface principal + automação de tarefas via linguagem natural **não é feito hoje por nenhum concorrente direto no nicho de integradores solar BR**. Para um público com baixa afinidade tech, isso é diferencial muito mais relevante do que features visuais sofisticadas.

**Qualificação importante:** existem players brasileiros fortes na camada WhatsApp+IA generalista (Take Blip, Zenvia, WATI) e plataformas de agentes no-code (Lindy, Relevance AI) — eles não atendem o nicho solar hoje, mas podem entrar em 12-24 meses. Nossa defesa não é tecnológica (a tecnologia comoditiza); é **conhecimento de domínio + canal ABSOLAR/E4.0 + integração nativa com Azume CRM**.

A janela de diferenciação é temporal. Quanto mais cedo entregarmos a Suite 1 e firmarmos posição, melhor. Mapa competitivo completo em `03-concorrentes.md`.

### 6.5 Habilita o Nexus como produto standalone no futuro

A trajetória do Nexus está definida no planejamento estratégico: hoje é piloto, depois ofertado primeiro à base do Azume CRM (assinatura própria do Nexus, comercializada para clientes do CRM antes de qualquer outro público), depois produto standalone para nichos novos. AI Apps é a tecnologia que faz a etapa "standalone" virar realidade — porque sem AI Apps, o Nexus precisaria ser custom-codado para cada nicho que atacar.

### 6.6 Alinhamento com os critérios do planejamento estratégico

O documento `00-contexto-atual.md` define 6 critérios que qualquer novo nicho/produto deve atender e 4 categorias de concorrência que devemos evitar. AI Apps respeita todos:

| Critério do planejamento estratégico | Como AI Apps atende |
|---|---|
| (1) Estratégia de distribuição | Para a Suite 1, base atual do CRM (canal direto). Para a Suite 2, E4.0 (canal de 372k inscritos) para nichos adjacentes a energia |
| (2) IA generativa como diferencial real | IA não é decoração; é o motor de cada AI App. Sem IA, nenhum dos AI Apps existe |
| (3) SaaS como oferta principal | Todos os modelos de cobrança são assinatura SaaS |
| (4) B2B | Mantido. Todas as Suites são B2B |
| (5) Nichado | A plataforma é genérica; cada **AI App** é nichado por construção (CRM solar, comercial específico, sucesso do cliente). Vendemos AI Apps específicos, não a plataforma |
| (6) Viável com recursos limitados | É exatamente o motivo da decisão: maximizar o que 1 dev produz |

Sobre as 4 categorias de concorrência a evitar: o framework em si poderia cair em "solução óbvia com IA" se fosse vendido como plataforma de construção. Mitigação: **não vendemos a plataforma** — vendemos os AI Apps prontos, nichados por construção. A plataforma é nosso meio de produção, invisível ao cliente final.

### 6.7 Dimensionamento financeiro (back-of-envelope)

Cálculos baseados no modelo da §5.4 (assinatura única, tiers PRO/PRO_PLUS/PRO_MAX, cobrança por usuário). Os números absolutos dependem de variáveis ainda em aberto (`[PREENCHER: número de contas ativas]` no `00-contexto-atual.md`), então usamos faixas e premissas explícitas.

**Premissas:**
- Faturamento jan-mar/2026: R$ 931.820 ≈ R$ 310k/mês.
- 80% renovação → ~R$ 248k/mês recorrente do CRM.
- Ticket médio anual do CRM: R$ 1.160 (~R$ 97/mês por conta).
- **Estimativa derivada de contas ativas:** ~R$ 248k de renovação ÷ R$ 97 ≈ **2.500-3.000 contas ativas**. Confirmar com Victor.
- Usuários médios por conta: 3 (faixa típica 2-5 segundo o contexto).
- **Preços ilustrativos** (sócios definem o número exato; usamos faixa razoável):
  - PRO_PLUS: ~R$ 40/usuário/mês → conta típica de 3 usuários paga R$ 120/mês
  - PRO_MAX: ~R$ 80/usuário/mês → conta típica de 3 usuários paga R$ 240/mês

#### Cenários de receita (Suite 1 — Azume CRM Smart na base atual)

| Variável | Pessimista | Realista | Otimista |
|---|---|---|---|
| Adoção da base (% de contas que assinam plano pago) | 5% | 15% | 30% |
| Mix entre tiers (PRO_PLUS / PRO_MAX) | 80% / 20% | 65% / 35% | 50% / 50% |
| Ticket médio mensal por conta paga | ~R$ 144 | ~R$ 162 | ~R$ 180 |
| Contas pagantes (base de 2.700 contas estimada) | ~135 | ~405 | ~810 |
| Receita nova mensal | ~R$ 19k | ~R$ 66k | ~R$ 146k |
| Receita nova anual | ~R$ 234k | ~R$ 787k | ~R$ 1,75M |
| **% sobre receita atual de renovação (R$ 3M/ano)** | **+8%** | **+26%** | **+58%** |
| **Payback do investimento de dev (≈ R$ 200k em tempo do Paulo)** | ~10-12 meses | ~3-4 meses | ~1-2 meses |

#### Cenário Suite 3 (uso interno Azume — custo evitado)

- Equipe atual de 3 pessoas em CS/suporte/renovação ≈ R$ 25-35k/mês de custo.
- Suite 3 reduzindo 30-40% do trabalho operacional = ~R$ 8-14k/mês de capacidade liberada.
- **Anualizado: R$ 100-170k/ano de espaço para crescer base sem contratar.**

#### Cenário Suite 2 (Nexus Comercial standalone — médio prazo)

- 50 contas no PRO_PLUS × R$ 200/mês médio = R$ 120k/ano (cenário mínimo de entrada em um nicho).
- 200 contas, sendo 40% no PRO_MAX × R$ 400/mês médio = R$ 720k/ano (cenário médio prazo).
- Tickets nos PRO_MAX (R$ 400-600/mês = R$ 4.800-7.200/ano por conta) **atingem o alvo estratégico de ticket >R$ 12k/ano para contas maiores** com 10+ usuários, conforme planejamento.

#### Custos durante a construção (meses 1-7, pré-lançamento)

- **Tokens de IA para desenvolvimento e testes**: ~R$ 500-1.500/mês durante testes de fluxos. Volume baixo (não há clientes pagantes ainda).
- **Twilio sandbox + 1-2 números reais para piloto**: ~R$ 100-300/mês de uso interno + custo inicial de aprovação do número WhatsApp Business (taxa única ~R$ 0-500 dependendo do plano contratado).
- **Infra adicional para dev/staging** (GCP / Mongo): ~R$ 200-800/mês.
- **Total estimado de custo direto durante construção: R$ 800-2.600/mês × 7 meses = R$ 6-18k**. Pequeno em relação ao investimento de tempo do Paulo.

#### Custos operacionais diretos esperados (pós-lançamento)

- **Inferência de IA**: o orçamento por usuário/mês já está modelado nos env vars `RESERVED_PER_USER_*_USD`. A capacidade reservada por tier (10× ou 20× a do PRO) é dimensionada para casar com o ticket cobrado, com margem positiva.
- **Twilio WhatsApp Business API**: ~R$ 0,05 a R$ 0,30 por mensagem. Para 400 contas × 200 mensagens/mês = ~R$ 4-24k/mês — coberto pela receita.
- **Infraestrutura adicional** (GCP / Mongo / vetores): incremento estimado em R$ 500-2.000/mês conforme volume.

#### Decisão pendente: definição dos preços por usuário

Os números nas tabelas acima são **ilustrativos**. Os sócios precisam decidir antes do lançamento:

- `price_per_user_brl` do PRO_PLUS — sugestão para discussão: R$ 30-50/usuário/mês.
- `price_per_user_brl` do PRO_MAX — sugestão para discussão: R$ 60-100/usuário/mês.
- Faixas de `min_users` / `max_users` se quisermos descontos por volume.

Decisão pode ficar para o checkpoint do mês 3-5 (sem bloquear o desenvolvimento), mas precisa estar fechada antes do piloto comercial da Suite 1.

#### Linha de receita adicional (futura): consultoria de adaptação

- Após expansão da equipe, oferta de adaptação personalizada por projeto/hora.
- Ticket típico estimado: R$ 5-30k por projeto, dependendo de escopo.
- Não contabilizada nos cenários acima — é upside.

#### O ponto importante

**Mesmo no cenário pessimista da Suite 1**, o investimento se paga em ~12 meses. **No cenário realista** (15% de adoção, mix moderado dos tiers), o payback é de 3-4 meses e a receita anual nova é equivalente a 26% do que renovamos hoje. As Suites 2 e 3 e a consultoria são upside adicional não-contabilizado.

**Observação importante:** todos esses números dependem da base de contas ativas reais. Pedimos a Victor confirmação do número (campo `[PREENCHER]` em `00-contexto-atual.md`) antes da decisão final.

---

## 7. Esforço e cronograma

### 7.1 Estimativa: 4 a 7 meses para o framework rodando ponta a ponta

Duas linhas de cenário, dependendo da decisão sobre manutenção do CRM (§7.3):

| Cenário | Foco do Paulo | Cronograma |
|---------|---------------|------------|
| **Framework full-time** | 100% AI Apps; CRM em manutenção mínima | 4 meses otimista / 5 realista / 6 pessimista |
| **Framework + CRM dividido** (recomendado) | 70-80% AI Apps / 20-30% CRM | 5 meses otimista / 6 realista / 7 pessimista |

**Por que essa estimativa é defensável (baseline empírica):**
O Nexus inteiro — backend, frontend, infra, CI/CD, todas as features atuais — foi construído pelo Paulo em **7 meses**, partindo do zero e sem as ferramentas que ele tem hoje. AI Apps começa numa base muito mais favorável:

- CI/CD já montado para backend e frontend.
- Paulo já domina a arquitetura do Nexus.
- Uso maduro de agentes de codificação em paralelo (Claude) — produtividade muito superior ao início do Nexus.
- Cerca de 60% da infraestrutura necessária já existe e é reaproveitada.

**O que está dentro do MVP do framework (versão mínima viável):**
- Fundações: bases de conhecimento, contexto, RAG, política de acesso.
- Agentes, Hives, ferramentas (incluindo APIs do Azume CRM).
- Triggers: manual, webhook, agendado, evento interno, **WhatsApp**.
- Outputs: mensagem de chat, **WhatsApp**, e o suficiente para os primeiros AI Apps.
- App, Run, observabilidade administrativa.

**O que fica para fases seguintes (não bloqueia lançamento):**
- Outputs mais pesados (PDF / Excel / Word complexos, Reports interativos).
- Suites e templates clonáveis pelo cliente (cliente clonar é pós-MVP; nós da Azume clonamos templates desde o início).
- Interface drag-and-drop (durante MVP os AI Apps são montados via formulários).
- Marketplace público de templates.

### 7.2 Quem faz o quê

| Pessoa | Papel | Quando entra |
|--------|-------|--------------|
| Paulo | Construção do framework | Dia 1 ao final do MVP (mês 5-7 no cenário recomendado) |
| Lucas (mkt) | Builder de AI Apps | Treinamento desde o mês 1; construção em ambiente de desenvolvimento a partir do mês 3; AI Apps em produção a partir do mês 5-6 |
| Rodolfo (mkt) | Builder de AI Apps | Idem Lucas |
| Victor | Supervisão e validação de AI Apps de CS/Sucesso do Cliente | A partir do mês 4 (revisão de designs); revisão de AI Apps em produção a partir do mês 5-6 |
| Thúlio | Supervisão e validação de AI Apps de Comercial/Marketing | Idem Victor |

**Por que Lucas e Rodolfo:** ambos já usam n8n (familiaridade com pensamento de baixo-código). Rodolfo está estudando programação, o que ajuda na curva. A alocação proposta é 20-30% do tempo deles para AI Apps (sem afundar marketing) — a definir com Thúlio.

**Como a construção em camadas funciona** (importante para sócios entenderem que Lucas e Rodolfo não ficam ociosos enquanto o framework é construído):

| Mês | Estado do framework | O que Lucas e Rodolfo fazem |
|---|---|---|
| 1-2 | Fundações em construção | Treinamento (n8n avançado, conceitos de prompt engineering, anatomia de um AI App, fluxos do Azume CRM); desenho em papel dos primeiros AI Apps; estudo de bases de conhecimento candidatas |
| 3-4 | Agentes, ferramentas, primeiros Triggers prontos | Protótipos dos AI Apps em ambiente de desenvolvimento (não rodam em produção, mas validam configuração e fluxos); construção das bases de conhecimento; redação de instruções de agente |
| 5-6 | MVP completo | AI Apps reais entrando em produção; primeiros pilotos com usuários internos (Suite 3) e clientes selecionados (Suite 1) |
| 7+ | Pós-MVP | Operação plena; Paulo libera capacidade para retomar features do CRM e/ou iniciar outputs mais pesados |

### 7.3 Manutenção do Azume CRM durante o período

**Esta é a pergunta operacional mais importante para os sócios alinharem.** Enquanto o Paulo está focado no framework, o CRM precisa continuar rodando. Três cenários possíveis:

| Cenário | Implicação | Cronograma do framework |
|---------|------------|-------------------------|
| **A — Manutenção mínima** | Paulo só atende bug crítico. CRM congela em features. Mais barato; risco de stress operacional acumulado. | 4-6 meses |
| **B — Dia dividido** (recomendado) | Paulo divide 70-80% framework / 20-30% CRM. CRM continua respirando. | 5-7 meses |
| **C — Reforço terceirizado** | Freelancer/contratado para manter o CRM em modo conservador. Custo extra; mais previsibilidade. | 4-6 meses (similar a A) |

Recomendação: **cenário B**, com sócios alinhados de que durante o período o CRM entra em modo "estável, sem features novas — apenas bugs e ajustes pequenos".

### 7.4 Pontos de controle

Para evitar atraso silencioso ou escopo crescendo sem controle:

- **Checkpoint mês 3:** revisão formal do progresso. Se as fases fundacionais (KnowledgeBase + Context + RAG + AccessPolicy + primeiros Agents) não estiverem concluídas, ajustamos escopo ou cronograma.
- **Checkpoint mês 5:** Triggers WhatsApp e primeiros Hives multi-agente funcionando em ambiente de desenvolvimento; Lucas e Rodolfo com protótipos prontos.
- **Checkpoint mês 6-7:** MVP em produção; primeiro AI App da Suite 3 rodando internamente.
- **Validação de demanda em paralelo:** entre mês 2 e mês 4, entrevistar 10-15 clientes do CRM sobre o cenário "imagine pedir 'gere proposta para João via WhatsApp' e a IA fazer". Não para validar tecnicamente — para confirmar disposição a pagar.

### 7.5 Métricas de sucesso

Para que "deu certo" não seja uma sensação subjetiva, definimos métricas concretas por checkpoint.

| Checkpoint | Métrica de sucesso | Critério de "ajustar curso" |
|---|---|---|
| **Mês 3** | Fundações funcionais (KB + RAG + AccessPolicy + Context); Lucas/Rodolfo passaram pelo treinamento básico | Fundações incompletas ou Lucas/Rodolfo sem progresso no treinamento |
| **Mês 5-6** | MVP rodando ponta a ponta com 1 AI App piloto interno (Suite 3); validação de demanda concluída com sinal positivo de ≥6 dos 10-15 clientes entrevistados | Apenas componentes isolados, sem App ponta-a-ponta; validação inconclusiva ou negativa |
| **Mês 9** | 3+ AI Apps em produção (todos da Suite 3 + primeiros da Suite 1); piloto Suite 1 com 3-5 clientes pagantes ou em trial | Menos de 2 AI Apps em produção; nenhum cliente externo testando |
| **Mês 12** | Suite 1 disponível para toda base; **≥5% de adoção** (cenário pessimista do §6.7) e curva ascendente; receita nova mensurável | Adoção abaixo de 3% sem sinal de crescimento; sinal forte de necessidade de pivot na proposta de valor |

---

## 8. Riscos e mitigações

Sendo honesto com a diretoria, há quatro riscos relevantes que precisamos olhar de frente.

### 8.1 Risco de dependência do framework (bus factor)

**O risco:** uma vez que muitos AI Apps dependam do framework, um bug no framework afeta todos os AI Apps simultaneamente. O Paulo passa a ser ponto único de falha de uma plataforma inteira, não só de features individuais.

**Mitigação:** disciplina de testes automatizados especialmente para Hive runner, política de acesso e integração com orçamento de IA. Documentação operacional clara desde o início. No médio prazo (12-18 meses), reavaliar contratação de um segundo engenheiro.

### 8.2 Risco de escopo crescendo silenciosamente

**O risco:** o desenho completo do AI Apps é grande (15+ entidades, várias fases). Sem disciplina, 5-7 meses (cenário recomendado) podem virar 9-12.

**Mitigação:** MVP definido com escopo radicalmente menor que o desenho completo. Funcionalidades como cloning pelo cliente, drag-and-drop UI, geração de PDF/relatórios complexos ficam para depois do MVP. Checkpoints mensais com escopo congelado.

### 8.3 Risco de construir framework para AI Apps que ninguém quer

**O risco:** investir 4-7 meses no framework e descobrir que os AI Apps que pretendíamos construir não têm demanda real.

**Mitigação:** validação de demanda em paralelo durante meses 2-4 (mencionada acima). Se a hipótese principal (Nexus + WhatsApp + Azume CRM via linguagem natural) for derrubada pelas entrevistas, pivotamos o framework antes do mês 5.

**Mitigação adicional:** mesmo que a Suite 1 não engaje a base externamente, a Suite 3 (uso interno) ainda gera retorno via redução de custo operacional. O downside da decisão não é zero retorno — é "só" o retorno interno.

### 8.4 Risco específico de WhatsApp como dependência

**O risco:** WhatsApp é canal regulado pela Meta. Mudanças de política, ban de número, falha do provedor (Twilio inicialmente) podem derrubar AI Apps inteiros.

**Mitigação:** arquitetura prevê troca de provedor (Twilio → Meta direto / 360dialog) sem reescrita. Status do canal é exposto na interface. AI Apps críticos sempre devem ter caminho alternativo de uso (web/chat) — WhatsApp é interface preferida, não obrigatória. **WebChat entra no MVP justamente como plano B arquitetural** (ver §3.8 e §11).

### 8.5 Risco de segurança em agentes multi-tenant

**O risco:** AI Apps recebem inputs de usuários finais (WhatsApp, WebChat) e executam Tools que tocam dados sensíveis — CRM, propostas, contratos, financeiro. Prompt injection é vetor de ataque crítico nessa classe de produto. Quatro vetores principais:

- **Direta** — input do usuário inclui instrução maliciosa.
- **Indireta** — dado retornado de uma Tool (ex.: campo "observações" do CRM preenchido por terceiro) contém instrução que o agente obedece.
- **Via RAG** — documento da KnowledgeBase contém payload adversarial.
- **Cross-tenant** — payload crafted em uma conta vaza para outra através de templates ou bases compartilhadas.

Em produto multi-tenant que toca dados financeiros e contratos, isso é problema sério, não acadêmico.

**Mitigação:** segurança contra prompt injection é tratada como cidadã de primeira classe da arquitetura no blueprint técnico em `/home/paulo/projects/nexus/nexus-core-2/docs/blueprints/ai_apps.md`, não como hardening tardio. Princípios principais:

- **Least privilege por Agent** — cada Agent declara Tools permitidas; runner valida no servidor, não no prompt.
- **Separação read vs. write** — Tools com side effect têm autorização mais estrita que Tools de leitura.
- **Tenant context imutável** — `tenant_id` injetado pelo runtime, nunca extraído do contexto do agente.
- **Validação de argumentos antes de side effect** — schema, ranges, whitelists no servidor.
- **Human-in-the-loop para operações irreversíveis** — envio de contrato, mudança de preço, e-mail para cliente final exigem confirmação humana por padrão.
- **Suite de testes adversariais em CI** — nenhuma Tool nova com side effect entra em produção sem caso de teste adversarial.

**Política de produto:** o item anterior (testes adversariais obrigatórios) é **bloqueador de release**, não recomendação. Detalhes operacionais (lista de Tools que exigem confirmação humana, procedimento de resposta a incidente, capability scopes) vivem no blueprint.

---

## 9. O que está sendo pedido

Esta apresentação não é só informativa — há decisões concretas para Victor e Thúlio.

### 9.1 Decisão principal

**Aprovar o investimento de 5-7 meses do Paulo no framework AI Apps** (cenário recomendado: tempo dividido entre framework e manutenção CRM; cenário full-time seria 4-6 meses mas com CRM congelado), sabendo que:

- Durante o período, o CRM entra em modo "manutenção conservadora" — sem features novas, apenas bugs e ajustes pequenos.
- Lucas e Rodolfo terão 20-30% do tempo alocado para AI Apps (treinamento desde mês 1, protótipos a partir do mês 3, produção a partir do mês 5-6).
- Existe um risco real de que a estimativa estique para 8-9 meses no pior caso (scope creep + dia dividido).

### 9.2 Decisões operacionais que dependem da aprovação

- **Thúlio:** confirmar alocação parcial de Lucas e Rodolfo. Quem treina, como, quando começa.
- **Victor:** validar a inclusão dos AI Apps de CS/Sucesso do Cliente como prioridade da Suite 3 (uso interno imediato).
- **Sócios em conjunto:** alinhar comunicação para a equipe sobre o que muda no curto prazo (CRM em modo conservador) e o que vem no médio prazo (capacidade de produzir Apps rapidamente).

### 9.3 Próximos passos sugeridos se aprovado

**Semana 0 (logo após aprovação):**
- Kickoff de 1 hora entre os três sócios para alinhar a comunicação à equipe (CRM em modo conservador, expectativas de Lucas/Rodolfo, prioridade do framework).
- Thúlio bloqueia 20-30% da agenda de Lucas e Rodolfo a partir da Semana 2.
- Victor agenda checkpoint mensal recorrente para acompanhar progresso.
- Paulo cria o repositório do framework e divide o backlog do MVP em fases entregáveis.

**Semana 1-2:**
- Paulo inicia Fase 0 (fundações: KnowledgeBase, Context, RAG, AccessPolicy).
- Plano de treinamento Lucas/Rodolfo definido (conteúdo, tempo semanal, quem ministra).

**Mês 1-2:**
- **Entrevistas de validação de demanda.** Quem conduz: alguém da operação próximo dos clientes (sugestão: Cláudia ou Juliana, com Thúlio acompanhando 1-2 entrevistas iniciais para calibrar). Quem é entrevistado: 10-15 clientes da base atual, mix de portes e perfis de uso. Perguntas-chave: (a) "Hoje, em que parte do seu fluxo você gostaria de ter alguém te ajudando?", (b) "Imagine que você pudesse pedir 'gere proposta para João via WhatsApp' e a IA fizesse — quanto pagaria a mais por mês por isso?", (c) "Se eu disponibilizasse 5 automações desse tipo no WhatsApp, mudaria seu dia a dia?". Critério de sucesso: ≥6 dos 10-15 sinalizam disposição a pagar valor compatível com PRO_PLUS.

**Mês 3:**
- Checkpoint formal — fundações entregues, validação de demanda concluída, decisão "seguir / ajustar / pivotar".

**Mês 5-6:**
- Primeiros AI Apps internos rodando em produção (Suite 3 — Sucesso do Cliente — para nós mesmos primeiro).
- Decisão sobre `price_per_user_brl` finalizada (Victor lidera, com input de Thúlio sobre percepção comercial).

**Mês 6-8:**
- Primeiro AI App da Suite 1 (Azume CRM Smart) em piloto com 3-5 clientes selecionados.

**Mês 9-12:**
- Suite 1 liberada para toda base. Início de planejamento da Suite 2 conforme nicho-alvo emergir.

---

## 10. Perguntas frequentes antecipadas

Perguntas que provavelmente surgirão na reunião, com respostas curtas.

### "Por que não usar n8n, Zapier, Make ou similar?"

Três razões:

1. **Multi-tenancy.** n8n e similares são engines de workflow genéricos, single-tenant por design (uma instância por cliente, ou multi-tenant via gambiarra). Não conseguimos isolar dados de cliente A e cliente B com segurança nativa.
2. **Integração profunda com o nosso stack.** Para acoplar n8n ao Azume CRM e ao Nexus de forma que faça sentido (APIs internas, base de conhecimento compartilhada, sistema de orçamento de IA, observabilidade), o esforço é comparável a construir o nosso. Ganharíamos poucos meses.
3. **Onde mora o valor competitivo.** Nosso ativo não é a plataforma de configuração em si (n8n já tem isso comoditizado) — é a **biblioteca de AI Apps especializados para nosso público** (integrador solar com baixa afinidade tech, fluxos específicos do CRM solar, regras tarifárias brasileiras, contratos do setor). Essa biblioteca precisa de runtime, observabilidade e cobrança que casem com nossas regras de negócio. Plataforma terceira nos amarraria às decisões de produto de outra empresa.

### "E se OpenAI / Anthropic lançarem algo parecido (GPTs, Anthropic Skills, etc.)?"

Eles já lançaram. Mas:
- Eles não têm o **cliente nichado** de integrador solar com baixa afinidade tech.
- Eles não têm **WhatsApp como interface principal**.
- Eles não integram com **nosso CRM**, nem com regras tarifárias brasileiras, nem com nossas bases de contratos.

Nosso valor competitivo está na **especialização para nosso público**, não na tecnologia genérica. GPTs competem com nós no mercado em que **decidimos não competir** (categoria §11.4 do plano estratégico — "soluções óbvias powered by IA").

### "E os concorrentes brasileiros de WhatsApp+IA (Take Blip, Zenvia, WATI)?"

Eles existem e são fortes. Take Blip atende milhares de PMEs brasileiras, é bem capitalizado e tem distribuição massiva. **Hoje não atacam o nicho solar**, mas tecnicamente podem em 12-24 meses.

A defesa segue a mesma lógica que aplicamos para big techs: **conhecimento de domínio que eles não têm**. Especificamente:
- Fluxos do Azume CRM solar (geração de proposta on-grid/híbrido/off-grid, regras tarifárias por distribuidora, contratos do setor).
- Parceria E4.0 + autoridade ABSOLAR (Bárbara Rubim) — barreira de marca difícil de replicar.
- Integração nativa com o Azume CRM — ativo único que dá vantagem operacional concreta.

A janela é temporal, não permanente. Análise competitiva detalhada por camada em `03-concorrentes.md` §3.1.

### "Quanto isso custa para nós em IA por mês?"

A spec de assinatura (`subscription_management.md`) já trata isso pelo desenho: cada tier reserva um orçamento em USD de inferência **por usuário/mês**, definido em env vars (`RESERVED_PER_USER_PRO_PLUS_*_USD`, `RESERVED_PER_USER_PRO_MAX_*_USD`). O preço cobrado em BRL é dimensionado pelos sócios para deixar margem positiva sobre essa reserva.

No regime de operação plena (cenário realista — ~400 contas pagantes):
- Inferência de IA: limitada pela capacidade reservada que o cliente paga — não cresce além disso sem upgrade de tier.
- Twilio WhatsApp Business API: ~R$ 4-24k/mês conforme volume.
- Infraestrutura adicional: ~R$ 500-2.000/mês.

**O custo de IA é estruturalmente coberto pelo preço por tier**, não é risco operacional fora de controle. Estado pré-MVP, durante construção: custo direto baixo (testes), majoritariamente o Claude para coding do Paulo.

### "Por que agora, e não daqui 1 ano?"

Três razões:
1. **Mercado em contração.** Quanto mais cedo defendermos a base, menos churn acumulado.
2. **Janela de IA generativa madura.** Modelos confiáveis o suficiente para produção existem agora. Há 18 meses, não existiam.
3. **Pico de produtividade.** O Paulo está em alta de produtividade com Claude paralelo. Esperar significa perder essa janela enquanto outros se posicionam.

### "Qual o pior cenário se aprovarmos?"

Cronograma estica para 8-9 meses, Suite 1 não engaja externamente como esperado. Mesmo assim ganhamos:
- Capacidade interna para produzir features de IA rápido daqui em diante.
- Suite 3 gerando redução de custo operacional interno (~R$ 100-170k/ano de capacidade liberada).
- WhatsApp como canal de produto estabelecido.

**Pior cenário não é zero. É retorno menor que o esperado.**

### "E se decidirmos NÃO fazer?"

Mantemos o status quo: cada feature nova de IA leva semanas/meses do Paulo. A lista completa de features e AI Apps no `tmp/features.md` seria atendida, no mesmo ritmo de hoje, em **3-5 anos** se mantida prioridade. A base atual do CRM continua sem o salto de UX que diferenciaria a Azume. Concorrentes (incluindo big tech via GPTs e Skills) ganham espaço enquanto esperamos.

### "Quem revisa o que Lucas e Rodolfo constroem?"

Pipeline de revisão proposto:
1. **Lucas/Rodolfo constroem em Draft** (não vai para produção).
2. **Victor ou Thúlio revisam** a configuração e os testes do AI App (depende do domínio — comercial, CS, etc.).
3. **Paulo aprova publicação** dos primeiros AI Apps. Conforme o time pega ritmo, essa etapa relaxa.

A plataforma tem audit trail completo de quem fez o quê em cada AI App, com versionamento e capacidade de reverter (Draft → Published → Archived) — então o risco de "alguém publicou e quebrou tudo" é controlado pela arquitetura.

### "Templates clonáveis pelo cliente — quando?"

Duas formas de "adaptação" do AI App ao negócio do cliente:

1. **Self-service de cloning pelo cliente** (cliente clona e adapta sozinho via UI) — pós-MVP. Feature de Phase 5 do blueprint técnico, adicionada quando houver demanda explícita.
2. **Adaptação consultiva pela equipe Azume** — disponível conforme expandirmos equipe. Modelo cobrável à parte (projeto/hora), não incluso na assinatura. Pode começar pequeno: o próprio time de operações/CS adapta para 1-2 clientes-piloto antes de virar oferta formal.

No MVP, a Azume mantém os templates centralizados; clientes consomem os AI Apps prontos via assinatura única. Conforme o produto amadurece, abrem-se as duas opções de adaptação.

---

## 11. Arquitetura de canal — Zoe, BYO WhatsApp e WebChat

Esta seção registra decisões de arquitetura que afetam o produto e a estratégia comercial. O detalhamento técnico vive no blueprint em `/home/paulo/projects/nexus/nexus-core-2/docs/blueprints/ai_apps.md`; aqui registramos as decisões e o racional.

### 11.1. Regra fundamental — a quem pertence o canal

**Um Channel pertence ao "outro lado" da conversa**, não a quem usa o AI App.

| Lado oposto da conversa | Channel a usar |
|---|---|
| Cliente da Azume (integrador) falando com a Azume | **Zoe** (número Azume) |
| Azume falando consigo mesma (operação interna — Suite Sucesso do Cliente uso interno, Suite Comercial uso interno) | **Zoe** |
| Cliente final do nosso cliente falando com a empresa dele | **BYO** — número registrado pelo próprio tenant |

Aplicação concreta nas Suites planejadas:

| Suite | Versão | Channel |
|---|---|---|
| Azume CRM Smart | Integrador opera o CRM dele via comando | **Zoe** — o integrador é cliente da Azume falando com a Azume |
| Nexus Comercial | Uso interno (Azume vendendo Nexus) | **Zoe** |
| Nexus Comercial | Externa (cliente da Azume operando comercial dele) | **BYO** |
| Sucesso do Cliente | Uso interno (Azume reduzindo custo de CS/suporte) | **Zoe** |
| Sucesso do Cliente | Externa (cliente da Azume atendendo clientes finais) | **BYO** |

Implicação prática: **a maior parte do MVP roda com a Zoe que já está aprovada na Twilio**. BYO entra como requisito do pós-MVP para as versões externas das Suites 2 e 3.

### 11.2. Channel como primitivo por-tenant (desde o dia 1)

Mesmo que o MVP rode com um único Channel em produção (Zoe), o modelo de dados precisa suportar N channels por tenant **desde o início**. Refatorar isso depois custa caro porque toca AccessPolicy, roteamento de webhooks, billing e UI.

Princípios não-negociáveis registrados no blueprint:
- Channel tem `owner_tenant_id`. Zoe pertence ao tenant interno da Azume.
- Channel é compartilhável entre tenants sob `AccessPolicy` explícita (permite que múltiplos AI Apps internos escutem a Zoe).
- Channel é desacoplado de Trigger — múltiplos Triggers (de Apps diferentes) podem apontar para o mesmo Channel; o runtime faz o roteamento.

### 11.3. WebChat como gêmeo arquitetural do WhatsApp

Decisão: tratar `webchat` como provider de Channel com **o mesmo contrato** que `whatsapp_twilio`. Mesmo evento de entrada, mesmo App Run, mesma saída.

No MVP entrega:
- Widget JS embeddable para sites de clientes.
- Página standalone em subdomínio/path branded por tenant.
- Customização de tema (logo, cores, nome) por tenant.

**Por que está no MVP, não no pós:**
- Onboarding instantâneo para clientes que não querem ou não podem usar WhatsApp.
- Plano B se um WhatsApp do cliente cair (ban Meta).
- Custo marginal pequeno — Outputs `chat_message` já existem para o canal Nexus interno; reutilização direta.
- Resiliência arquitetural — força o framework a ser provider-agnóstico desde o início.

### 11.4. BYO WhatsApp — onboarding pós-MVP, data model day-1

O fluxo de provisionamento de WhatsApp por tenant **não é MVP**. Mas o modelo de dados, a roteamento de webhooks e a `AccessPolicy` precisam acomodar BYO desde a primeira linha — caso contrário, paga-se em refatoração no momento do lançamento das Suites externas.

Caminho preferido para BYO documentado no blueprint: **Meta Embedded Signup** (Cloud API direto) como default; Twilio Sender Flow como fallback. Justificativa em §12 abaixo.

### 11.5. Decisões em aberto (registradas para resolução futura)

1. **Billing de mensagens em BYO** — Azume cobra do tenant com markup ou tenant paga direto o provider? Trade-off comercial pendente. Discutir com Victor antes do lançamento da Suite 2 externa.
2. **Roteamento entre Apps no mesmo Channel** — matcher por regex, intent classifier, fallback. Quais estratégias entram no MVP, quais no pós.
3. **Modelo de sessão do WebChat** — anônimo, autenticado, token emitido pelo tenant. Decisão de UX a tomar no design das Suites externas.

---

## 12. Estratégia de provider WhatsApp

### 12.1. Decisão consolidada

| Fase | Provider | Justificativa |
|---|---|---|
| **MVP (meses 1-7)** | Twilio | Zoe já aprovada e online (sender ativo, 80 MPS). Refatorar agora é distração. Arquitetura prevê troca sem reescrita |
| **Pré-lançamento Suite 1 (mês 5-6)** | Revisar entre **Meta Cloud API direto** e **360dialog** | Twilio cobra markup gordo sobre tarifa Meta. Em escala, economia projetada de R$ 4-12k/mês com 400 tenants × 200 msg/mês |
| **BYO pós-MVP (Suites 2/3 externas)** | **Meta Embedded Signup** (Cloud API direto) como default; Twilio Sender Flow como fallback para tenants já na Twilio | Embedded Signup tem UX bem mais limpa para onboarding do cliente |

### 12.2. Candidatos avaliados

| Provider | Posicionamento | Custo relativo | Quando faz sentido |
|---|---|---|---|
| **Meta Cloud API direto** | Acesso direto, sem intermediário. Mais engenharia (webhooks, templates, media), zero markup | Mínimo | Operação em escala; BYO |
| **360dialog** | BSP alemão "near-cost". Assinatura por número + tarifa Meta sem markup. Popular em SaaS BR sério | Quase Meta-direto | Alternativa de menor esforço de engenharia que Meta direto |
| **Gupshup** | BSP global competitivo | Médio | Backup |
| **Twilio** | BSP confortável, caro | Alta | MVP (já aprovado) |

### 12.3. Exclusão explícita — soluções não-oficiais

**Z-API, Evolution API, Baileys, Waha, Chat-API, UnoAPI e similares estão FORA da mesa.** Operam via WhatsApp Web protocol ou emulação Android, violam TOS da Meta, e a Meta bane números sem aviso.

Razões da exclusão:
1. **Risco de ban sem aviso.** Volume sobe, padrão fica visível, número cai. Imprevisível e irreversível.
2. **Quando o cliente do integrador toma ban, o culpado é a Azume.** Reclamação chega no nosso suporte, não na Z-API. Esse tipo de incidente quebra confiança que comunicado não conserta.
3. **Conflita com a posição de marca.** ABSOLAR + E4.0 + posicionamento de "software sério para o setor" não comporta solução cinza para economizar R$ 5/cliente.
4. **Risco legal.** Cláusulas de compliance e certificações de segurança em contratos B2B sérios não passam.
5. **Pior efeito justamente nos clientes que mais queremos manter** — contas com 10+ usuários (PRO_MAX) têm o maior dano se o WhatsApp comercial cair de repente.

**Esta decisão é política de produto, não preferência técnica.** Revisitar exigiria mudança de posicionamento de marca. Quando aparecer como "alternativa para baratear" em conversa futura, a resposta é a mesma: não.

### 12.4. Critérios para reavaliar a decisão de provider

Revisão programada do provider de produção: **mês 5 de desenvolvimento**, durante a fase de testes da Suite 3 interna. Critérios objetivos para decidir migrar do Twilio:

- Economia mensal projetada × custo de migração (engenharia + risco).
- Maturidade do provider em PT-BR (suporte, documentação, casos de sucesso).
- Compatibilidade com o modelo BYO (importante para o pós-MVP).
- Disponibilidade de Meta Embedded Signup (decisivo para BYO).

Trigger para revisão fora desse calendário: incidente operacional relevante com Twilio (qualidade, custo, indisponibilidade) ou mudança comercial significativa em algum dos candidatos.

### 12.5. Referência cruzada

Detalhes técnicos de implementação do `Channel` provider e da troca entre providers vivem no blueprint em `/home/paulo/projects/nexus/nexus-core-2/docs/blueprints/ai_apps.md`. Este documento registra **a decisão**; o blueprint registra **a implementação**.

---

## 13. Em resumo

**AI Apps não é uma feature. É uma decisão de arquitetura sobre como a Azume vai produzir produtos pelos próximos anos.**

A escolha que está na mesa:

| Caminho A — continuar custom-codando | Caminho B — investir no framework |
|--------------------------------------|-----------------------------------|
| Cada feature consome semanas do Paulo | Investimento único de 5-7 meses (cenário recomendado) |
| Lucas, Rodolfo, Victor, Thúlio dependem de fila de TI | Time inteiro produz Apps em paralelo após o MVP |
| Testar nicho novo custa meses | Testar nicho novo custa dias/semanas |
| Throughput limitado a ~3-5 features grandes por ano | Throughput multiplicado por ordem de grandeza |
| WhatsApp seria feature isolada e cara | WhatsApp vira primitiva da plataforma |
| Lista do `features.md` levaria 3-5 anos | Lista do `features.md` viável em 12-18 meses |

A recomendação foi o **caminho B**, com a disciplina operacional descrita acima.

**Decisão tomada em mai/2026.** Este documento registra o racional e serve de referência para os checkpoints previstos em §7.4 ao longo dos próximos 5-7 meses de execução. Atualizações de escopo, prazo ou trade-off material devem ser refletidas aqui — não em conversas paralelas.
