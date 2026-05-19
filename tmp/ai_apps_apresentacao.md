# AI Apps — Apresentação para a diretoria

> Documento de apresentação para Victor e Thúlio. Foco em valor de negócio e entendimento do produto, não em detalhes técnicos. Última atualização: 2026-05-19.

---

## TL;DR (1 minuto)

**O que é:** AI Apps é uma plataforma interna que permite a Azume **configurar** (em vez de **programar**) novas automações com IA generativa. Cada "AI App" é uma automação inteligente pronta para ser usada por um cliente — gerar uma proposta, responder no WhatsApp, qualificar um lead, gerar um relatório, etc.

**Por que importa:** hoje cada nova feature de IA do Nexus precisa ser codada à mão pelo Paulo, o único engenheiro. Com AI Apps, **não-engenheiros do nosso time** (começando por Lucas e Rodolfo) constroem novas automações. O Paulo constrói a fábrica uma vez; depois disso a equipe inteira fabrica produtos.

**Investimento:** 4 a 6 meses do Paulo focado no framework (cenário base). Sob o cenário operacional recomendado, em que o Paulo divide tempo entre framework e CRM, o cronograma vai a 5-7 meses. Lucas e Rodolfo entram em treinamento desde o mês 1 e começam a construir AI Apps reais conforme cada fase do framework é entregue.

**Retorno esperado em 12 meses (estimativas conservadoras):**
1. **Suite Azume CRM Smart** — defesa da vaca leiteira. Potencial de gerar 15-40% de receita nova adicional sobre a base atual via addon mensal, e reduzir churn.
2. **Nexus no WhatsApp** — interface familiar para um público com baixa afinidade tech. Primeira vez que um concorrente direto no Brasil oferece isso para integradores solar.
3. **Capacidade de testar nichos novos com custo de dias** — quando identificarmos o próximo mercado, o tempo de "ir ao mercado" cai de meses para semanas.
4. **Redução de custo operacional interno** — Suite Sucesso do Cliente reduz 30-40% do trabalho manual de Diógenes / Juliana / Cláudia, criando espaço para crescer base sem crescer equipe.

**A decisão na mesa:** comprometer 4-7 meses de engenharia em construir o framework ou continuar codando feature por feature, mantendo o gargalo do dev único.

---

## Glossário rápido (5 termos que aparecem ao longo do documento)

Para acompanhar a apresentação sem se prender a jargão:

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

### 3.8 WhatsApp Channel

**O que é:** um número de WhatsApp registrado e configurado para conversar via Nexus. É um bloco de construção próprio porque WhatsApp é estratégico demais para ficar escondido como detalhe.

**Capacidade que fornece:** **a interface mais familiar para o público brasileiro**. Nosso cliente típico (integrador solar) tem alta familiaridade com WhatsApp e baixa familiaridade com praticamente qualquer outro software. Trazer o Nexus para dentro do WhatsApp é potencialmente o desbloqueio de UX mais importante que conseguimos fazer.

**Exemplos:**
- Número do "Atendimento Azume" — vários AI Apps escutam (suporte, comercial, financeiro).
- Número próprio do cliente — o cliente registra o WhatsApp dele, AI App de pós-venda da empresa dele responde.

### 3.9 Suite (conjunto de AI Apps)

**O que é:** um agrupamento de AI Apps com um objetivo comum.

**Capacidade que fornece:** **empacotamento comercial e UX**. Em vez de vender 5 AI Apps isolados, vendemos a "Suite Azume CRM Smart" — produto único com identidade própria.

**Por que importa:** é a unidade de comercialização. Suites são o que clonamos para nichos novos.

### 3.10 Templates (mencionar brevemente)

**O que é:** AI Apps e Suites criados pelo nosso time interno que ficam disponíveis para os clientes clonarem para a conta deles.

**Capacidade que fornece:** **escala**. Construímos uma vez, milhares de clientes usam.

### 3.11 Run, Report e observabilidade (mencionar brevemente)

Cada execução de um AI App é uma "Run" — registramos tudo que aconteceu (qual agente fez o quê, quais ferramentas usou, quanto custou em IA). Isso garante:

- **Auditoria:** podemos provar para o cliente o que o App fez.
- **Custo:** sabemos exatamente quanto de IA cada cliente está consumindo (cobrança justa).
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
      AI Apps podem ser agrupados em Suites.
      Suites podem ser Templates clonáveis por clientes.
```

Em uma frase: **AI Apps é uma fábrica de automações inteligentes** que a Azume opera internamente, com produtos que entregamos aos clientes via interfaces que eles já conhecem (WhatsApp principalmente, mas também web e via integrações).

---

## 5. Casos de uso — três Suites concretas

Para tornar tangível, três Suites que pretendemos construir nos primeiros 6-12 meses pós-framework:

### Suite 1 — Azume CRM Smart

**Público-alvo:** base atual do Azume CRM (integradores de energia solar).
**Valor de negócio:** defesa da vaca leiteira. Aumenta valor percebido, justifica preço, reduz churn, abre espaço para upsell.
**Modelo de cobrança:** addon mensal do CRM. **Faixa estimada: R$ 99 a R$ 249/mês por conta**, dependendo dos AI Apps inclusos (compatível com o ticket atual de ~R$ 97/mês do CRM — vira essencialmente "dobrar o ticket" do cliente que adota).
**Amarração com o backlog atual:** os AI Apps abaixo materializam diretamente as features 1.2 a 1.6 do `tmp/features.md`, mais a feature 1.1 (proposta híbrido/off-grid) e 3.1 (Nexus no WhatsApp) como interface principal.

| AI App | O que faz | Como o cliente usa | Feature |
|--------|-----------|--------------------|---------|
| Gerador de Proposta On-Grid | Recebe dados do cliente e gera proposta completa | "Gere proposta para João Silva, 5kWp, em Curitiba" via WhatsApp | 1.4 |
| Gerador de Proposta Híbrido/Off-Grid | Idem, mas para sistemas com baterias | Permite o CRM atender sistemas que hoje não consegue | 1.1 + 1.4 |
| Gerador de Relatórios Dinâmicos | Cria relatórios sob demanda da operação do cliente | "Quanto vendi esse mês por vendedor?" via WhatsApp | 1.2 + 1.3 |
| Gerador de Contratos | Monta e envia contrato para assinatura | "Manda contrato para o José" via WhatsApp | 1.6 |
| Gestão de Funis e Quadros | Movimenta leads/negócios/obras no funil | "Move o lead da Maria para 'proposta enviada'" via WhatsApp | 1.5 |

**Interface principal:** WhatsApp (feature 3.1 do `features.md`). Web continua disponível, mas a aposta de UX é WhatsApp.

**A tese que une essa Suite:** o cliente integrador roda **o Azume CRM inteiro por dentro do WhatsApp**. Tira o atrito de "tem que abrir o software" e troca por algo que ele já usa o dia todo. Para nosso público de baixa afinidade tech, isso é diferencial gigantesco.

### Suite 2 — Nexus Comercial

**Público-alvo:** qualquer PME B2B que tenha equipe comercial — começando pela própria Azume e pela base atual.
**Valor de negócio:** produto de ticket mais alto, possivelmente o **primeiro produto Nexus standalone** (fora da base do CRM).
**Modelo de cobrança:** SaaS por assinatura. **Faixa estimada: R$ 500 a R$ 2.500/mês por conta**, conforme volume e funcionalidades. Atinge o alvo de ticket >R$ 12k/ano que o planejamento estratégico definiu para novos nichos.
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
**Modelo de cobrança:** SaaS por assinatura. **Faixa estimada: R$ 200 a R$ 800/mês por conta**, conforme volume de tickets/contatos. Para uso interno da Azume, é "custo evitado" em vez de receita.

| AI App | O que faz |
|--------|-----------|
| Atendimento ao Cliente | Resolve tickets de suporte de primeiro nível |
| Relacionamento com o Cliente | Mantém contato proativo, identifica clientes em risco |
| Renovação de Contratos | Conduz processo de renovação anual de forma proativa |

**Por que essa Suite é estratégica para nós internamente:** Diógenes (suporte), Juliana (CS/renovação) e a Cláudia (operações) hoje fazem manualmente. Se a Suite 3 reduzir em 30-40% o trabalho operacional deles, conseguimos crescer base sem crescer equipe.

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

### 6.4 Diferencial competitivo real

WhatsApp como interface principal do Nexus + automação de tarefas via linguagem natural é algo que **nenhum concorrente nosso direto faz hoje**. Para um público com baixa afinidade tech, isso é diferencial muito mais relevante do que features visuais sofisticadas.

### 6.5 Habilita o Nexus como produto standalone no futuro

A trajetória do Nexus está definida no planejamento estratégico: hoje é piloto, depois addon do CRM, depois produto standalone. AI Apps é a tecnologia que faz a etapa "standalone" virar realidade — porque sem AI Apps, o Nexus precisaria ser custom-codado para cada nicho que atacar.

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

Cálculos conservadores baseados nos dados disponíveis. Faixas usadas em vez de números exatos por causa dos campos `[PREENCHER]` ainda abertos em `00-contexto-atual.md`.

**Premissas:**
- Faturamento jan-mar/2026: R$ 931.820 ≈ R$ 310k/mês.
- 80% renovação → ~R$ 248k/mês recorrente do CRM.
- Ticket atual ~R$ 97/mês por conta.

**Cenário conservador da Suite 1 (Azume CRM Smart):**

| Variável | Cenário pessimista | Cenário realista | Cenário otimista |
|---|---|---|---|
| Taxa de adoção da base | 10% | 25% | 40% |
| Ticket addon mensal | R$ 99 | R$ 149 | R$ 249 |
| Receita nova/mês | ~R$ 25k | ~R$ 92k | ~R$ 250k |
| % sobre receita atual | +10% | +30% | +80% |
| Payback do investimento | ~12-15 meses | ~5-7 meses | ~3 meses |

**Cenário Suite 3 (uso interno Azume — custo evitado):**
- Hoje: equipe de 3 pessoas em CS/suporte/renovação ≈ R$ 25-35k/mês de custo.
- Suite 3 reduzindo 30-40% do trabalho operacional = ~R$ 8-14k/mês de capacidade liberada.
- Anualizado: R$ 100-170k/ano de espaço para crescer base sem contratar.

**Cenário Suite 2 (Nexus Comercial standalone):**
- 50 clientes × R$ 1.000/mês = R$ 600k/ano (cenário realista mínimo se a Suite encaixar em um nicho).
- 200 clientes × R$ 1.500/mês = R$ 3.6M/ano (cenário otimista médio prazo).
- Atinge alvo estratégico de ticket >R$ 12k/ano por cliente.

**Custos operacionais diretos esperados (pós-lançamento):**
- Inferência de IA: estimativa de R$ 1-5/conta/mês por AI App ativo (Suite 1 com 5 Apps × R$ 3 médio × 100 contas = ~R$ 1.500/mês).
- Twilio WhatsApp Business API: aproximadamente R$ 0,05 a R$ 0,30 por mensagem dependendo do tipo (sessão x template). Para 100 contas × 200 mensagens/mês = ~R$ 1.000-6.000/mês.
- Infraestrutura adicional (GCP / Mongo / vetores): incremento estimado em R$ 500-2.000/mês conforme volume.
- **Total operacional esperado: R$ 3-10k/mês em estado de regime** — coberto com folga pelos modelos de cobrança propostos.

**O ponto importante:** mesmo no cenário pessimista da Suite 1 isolada, o investimento de 4-7 meses de dev se paga em 12-15 meses. O cenário realista paga em 5-7 meses. E as Suites 2 e 3 são upside adicional ainda não contabilizado nesse cálculo.

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
| **Mês 12** | Suite 1 disponível para toda base; ≥10% de adoção na base (taxa pessimista do dimensionamento); receita nova mensurável | Adoção abaixo de 5%; necessidade de pivot na proposta de valor |

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

**Mitigação:** arquitetura prevê troca de provedor (Twilio → Meta direto / 360dialog) sem reescrita. Status do canal é exposto na interface. AI Apps críticos sempre devem ter caminho alternativo de uso (web/chat) — WhatsApp é interface preferida, não obrigatória.

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

1. **Semana 1-2:** Paulo inicia Fase 0 (fundações). Em paralelo, plano de treinamento Lucas/Rodolfo definido com Thúlio.
2. **Mês 1-2:** entrevistas de validação com 10-15 clientes do CRM sobre o uso WhatsApp + linguagem natural. Cláudia e Juliana conduzem.
3. **Mês 3:** checkpoint formal — escopo, cronograma, validação de demanda.
4. **Mês 5-6:** primeiros AI Apps internos rodando em produção (Suite 3 — Sucesso do Cliente — para nós mesmos primeiro).
5. **Mês 6-8:** primeiro AI App da Suite 1 (Azume CRM Smart) em piloto com 3-5 clientes selecionados.
6. **Mês 9-12:** Suite 1 liberada para toda base. Início de planejamento da Suite 2 conforme nicho-alvo emergir.

---

## 10. Perguntas frequentes antecipadas

Perguntas que provavelmente surgirão na reunião, com respostas curtas.

### "Por que não usar n8n, Zapier, Make ou similar?"

n8n e similares são engines de workflow genéricos, **single-tenant por design** (uma instância por cliente, ou multi-tenant via gambiarra). Não conseguimos isolar dados de cliente A e cliente B com segurança nativa. Além disso, integrar n8n ao Azume CRM e ao Nexus de forma profunda exigiria tanto trabalho quanto construir o nosso. Avaliamos: o ganho de tempo seria de poucos meses; a perda de controle de IP seria permanente.

### "E se OpenAI / Anthropic lançarem algo parecido (GPTs, Anthropic Skills, etc.)?"

Eles já lançaram. Mas:
- Eles não têm o **cliente nichado** de integrador solar com baixa afinidade tech.
- Eles não têm **WhatsApp como interface principal**.
- Eles não integram com **nosso CRM**, nem com regras tarifárias brasileiras, nem com nossas bases de contratos.

Nosso valor competitivo está na **especialização para nosso público**, não na tecnologia genérica. GPTs competem com nós no mercado em que **decidimos não competir** (categoria §11.4 do plano estratégico — "soluções óbvias powered by IA").

### "Quanto isso custa para nós em IA por mês?"

No regime de operação plena (estimativa, baseada em testes do piloto Nexus):
- Inferência de IA: ~R$ 1-5 por conta por AI App ativo por mês.
- Twilio WhatsApp: ~R$ 0,05-0,30 por mensagem (mais agressivo se uso de templates).
- Infra adicional: ~R$ 500-2.000/mês de incremento.

**Total no cenário realista (100 contas, 5 AI Apps cada): R$ 3-10k/mês**, coberto pelos modelos de cobrança propostos. Estado pré-MVP, durante construção: custo direto baixo (testes), majoritariamente Claude para coding do Paulo.

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

Mantemos o status quo: cada feature nova de IA leva semanas/meses do Paulo. A lista de features do `tmp/features.md` (1.1 a 3.2) seria atendida, no mesmo ritmo de hoje, em **3-5 anos** se mantida prioridade. A base atual do CRM continua sem o salto de UX que diferenciaria a Azume. Concorrentes (incluindo big tech via GPTs e Skills) ganham espaço enquanto esperamos.

### "Quem revisa o que Lucas e Rodolfo constroem?"

Pipeline de revisão proposto:
1. **Lucas/Rodolfo constroem em Draft** (não vai para produção).
2. **Victor ou Thúlio revisam** a configuração e os testes do AI App (depende do domínio — comercial, CS, etc.).
3. **Paulo aprova publicação** dos primeiros AI Apps. Conforme o time pega ritmo, essa etapa relaxa.

A plataforma tem audit trail completo de quem fez o quê em cada AI App, com versionamento e capacidade de reverter (Draft → Published → Archived) — então o risco de "alguém publicou e quebrou tudo" é controlado pela arquitetura.

### "Templates clonáveis pelo cliente — quando?"

Pós-MVP (após o mês 7). No MVP, a Azume mantém os templates centralizados; clientes consomem os AI Apps prontos via assinatura. O cloning pelo cliente é feature de Phase 5 do blueprint técnico — adicionada quando houver demanda explícita por personalização.

---

## 11. Em resumo

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

A recomendação é o **caminho B**, com a disciplina operacional descrita acima.

A pergunta que fica para a reunião é: **estamos alinhados para comprometer os próximos 5-7 meses nessa direção?**
