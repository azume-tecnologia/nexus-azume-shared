# Ideias de features

> **Contexto (atualizado em 2026-05-19):** todas as features abaixo serão construídas como **AI Apps** sobre o framework descrito em `/home/paulo/projects/nexus/nexus-core-2/docs/blueprints/ai_apps.md`. A direção estratégica está registrada em `docs/01-planejamento-estrategico/04-ai-apps.md` e o contexto estratégico em `docs/01-planejamento-estrategico/00-contexto-atual.md` §3.2.1.
>
> **Sequência de execução pós-framework (mês 5-7):** Suite Sucesso do Cliente (uso interno) → Suite Azume CRM Smart (base CRM) → Suite Nexus Comercial (nicho a definir). Justificativa em `04-ai-apps.md` §5 "Por que essa ordem".
>
> **Modelo de monetização:** assinatura única Nexus, 3 tiers (PRO grátis / PRO_PLUS / PRO_MAX), cobrança `price_per_user_brl × user_count`. Toda a biblioteca de AI Apps disponível em qualquer tier pago. Detalhes em `docs/01-planejamento-estrategico/04-ai-apps.md` §5.4.

---

## 1. Suite Azume CRM Smart (Suite 1)

**Público-alvo:** base atual do Azume CRM (integradores de energia solar).
**Valor de negócio:** defesa da vaca leiteira via "Azume CRM operado por linguagem natural no WhatsApp".
**Prioridade pós-framework:** alta (piloto mês 6-8, rollout mês 9-12).

### 1.1 AI App — Gerador de Propostas Híbridas e Off-Grid

**Descrição:** AI App que gera propostas de sistemas híbridos e off-grid (com baterias). Recebe dados do cliente final via WhatsApp ou chat e gera proposta. Aprimoramentos iterativos conforme demanda. Reuso máximo da lógica do AI App de proposta on-grid.
**AI App primitives:** Trigger WhatsApp/Manual + Hive (agente extrator + agente analista de engenharia) + Output mensagem + Output arquivo (PDF da proposta).
**Esforço:** baixo (configuração + base de conhecimento + tool de criação no CRM).
**Valor:** alto — expande o escopo de atendimento do CRM para um segmento crescente.
**Custo Operacional:** consumo de tokens proporcional ao tier do cliente.
**Riscos:** performance da IA em engenharia de sistemas com baterias é incerta — começamos com casos simples e expandimos.

### 1.2 AI App — Consulta ao Azume CRM via linguagem natural

**Descrição:** Permite consultar clientes, propostas, negócios, métricas de vendas, etc. via comando em linguagem natural no WhatsApp ou chat.
**AI App primitives:** Trigger WhatsApp/Manual + Hive (1 agente) + Tools de consulta ao CRM (clientes, propostas, métricas) + Output mensagem.
**Esforço:** baixo-médio (várias tools de consulta a desenvolver).
**Valor:** alto — destrava acesso aos dados sem precisar abrir software.
**Dependências:** as Tools de consulta criadas aqui são pré-requisito da §1.3 (Relatórios) e fortemente usadas pela §1.4 (Geração de Propostas) e §1.5 (Funis).

### 1.3 AI App — Geração de Relatórios Dinâmicos do CRM

**Descrição:** Geração de relatórios personalizados a partir das métricas do CRM ("quanto vendi esse mês por vendedor?", "qual região com mais conversão?").
**AI App primitives:** Trigger WhatsApp/Manual + Hive (usa Tools de 1.2) + Output mensagem ou arquivo (PDF/Excel).
**Esforço:** médio (reusa tools de 1.2, precisa estruturar saída).
**Valor:** alto para perfil gerencial dos clientes.
**Observação:** Outputs de arquivo/relatório pesados ficam para fase pós-MVP do framework (3b/4).

### 1.4 AI App — Geração de Propostas On-Grid

**Descrição:** Geração de propostas ponta-a-ponta para sistemas on-grid via linguagem natural no WhatsApp.
**AI App primitives:** Trigger WhatsApp/Manual + Hive (extrator de dados + analista + montador) + Tools (CRM, base tarifária, base de equipamentos) + Output mensagem + Output arquivo (PDF).
**Esforço:** médio-alto (caminho crítico do CRM, precisa ser robusto).
**Valor:** muito alto — é o uso mais frequente do CRM hoje.

### 1.5 AI App — Gestão de Funis e Quadros via comando

**Descrição:** Movimentação/ganho/perdas/retorno ao funil de leads, negócios, obras, projetos, operações administrativas, pós-venda via comando em linguagem natural.
**AI App primitives:** Trigger WhatsApp/Manual + Hive (1 agente) + Tools de mutação no CRM + Output mensagem (confirmação).
**Esforço:** médio (várias tools de mutação a desenvolver com cuidado — operações destrutivas/irreversíveis).
**Valor:** alto.

### 1.6 AI App — Assinatura de Contratos via comando

**Descrição:** Geração e disparo de contrato para assinatura via linguagem natural no WhatsApp.
**AI App primitives:** Trigger WhatsApp/Manual + Hive (1 agente) + Tools (CRM + serviço de assinatura) + Output mensagem.
**Esforço:** médio (depende de integração com serviço de assinatura — provavelmente já existe no CRM).
**Valor:** alto — fecha o ciclo da venda dentro do WhatsApp.

---

## 2. Suite Sucesso do Cliente (Suite 3 — prioridade alta para uso interno)

**Público-alvo:** uso interno da Azume primeiro (redução de custo operacional dos times de CS/suporte/renovação). Depois, ofertado como produto a qualquer PME com base recorrente.
**Valor de negócio:** ~R$ 100-170k/ano de capacidade liberada para a Azume; validação do framework antes de uso externo.
**Prioridade pós-framework:** alta (primeiro AI App em produção, mês 5-6).

### 2.1 AI App — Atendimento ao Cliente

**Descrição:** Resolve tickets de suporte de primeiro nível via WhatsApp e chat.
**AI App primitives:** Trigger WhatsApp + Hive (1 agente) + KnowledgeBase (FAQs Azume, manuais do CRM) + Tools de consulta ao CRM (status do cliente, último pagamento, etc.) + Output mensagem.
**Esforço:** médio (precisa de base de conhecimento robusta de FAQs).
**Valor:** alto internamente — desonera Diógenes.

### 2.2 AI App — Relacionamento com o Cliente

**Descrição:** Contato proativo, identificação de clientes em risco, follow-up de NPS.
**AI App primitives:** Trigger agendado + Hive (1-2 agentes) + Tools de consulta CRM + Output WhatsApp.
**Esforço:** médio.
**Valor:** alto internamente — desonera Juliana / Cláudia.

### 2.3 AI App — Renovação de Contratos

**Descrição:** Conduz processo de renovação anual de forma proativa — identifica contratos próximos do vencimento, abre conversa via WhatsApp, leva até o pagamento.
**AI App primitives:** Trigger agendado + Hive (1-2 agentes) + Tools de consulta e mutação no CRM + Output WhatsApp + Output webhook (cobrança).
**Esforço:** médio-alto (caminho crítico — receita de renovação é 80% do faturamento).
**Valor:** muito alto — automatiza o processo que sustenta a empresa.

---

## 3. Suite Nexus Comercial (Suite 2 — prioridade média, depende de definição de nicho)

**Público-alvo:** PMEs B2B em geral, lançada primeiro em nicho adjacente a energia/setor elétrico (canal E4.0). Cada nicho subsequente é uma clonagem da Suite com adaptação.
**Valor de negócio:** primeiro produto Nexus standalone; alvo de ticket >R$ 12k/ano para contas grandes.
**Prioridade pós-framework:** média (planejamento mês 9-12, depende de nicho-alvo emergir).

### 3.1 AI App — Prospector de Clientes

**Descrição:** Busca e organiza leads qualificados a partir de critérios definidos (porte, segmento, sinais externos).
**AI App primitives:** Trigger agendado/manual + Hive multi-agente (pesquisa + classificação) + Tools (busca web, CNPJ, redes sociais) + Output banco de dados (lista qualificada).

### 3.2 AI App — Qualificador de Clientes Prospectados

**Descrição:** Conversa inicial com lead para entender fit e nível de interesse.
**AI App primitives:** Trigger WhatsApp/manual + Hive (1-2 agentes) + Tools de consulta interna + Output mensagem.

### 3.3 AI App — Closer de Vendas

**Descrição:** Conduz a conversa de venda até fechamento.
**AI App primitives:** Trigger WhatsApp + Hive (1-2 agentes, com handoff humano configurável) + Tools (propostas, contratos) + Output mensagem + Output webhook (pagamento).

### 3.4 AI App — Pós-venda / Relacionamento

**Descrição:** Mantém relacionamento ativo após o fechamento; ativa cross-sell e upsell.
**AI App primitives:** Trigger agendado + Hive (1 agente) + Tools de consulta + Output WhatsApp.

---

## 4. Capacidades base do Nexus (necessárias para todas as Suites)

São features de plataforma, não AI Apps. Construídas pelo Paulo como parte do framework AI Apps (`docs/blueprints/ai_apps.md`).

### 4.1 Framework AI Apps

**Descrição:** A plataforma interna que torna possível construir todos os AI Apps acima por configuração ao invés de código. **Esta é a feature estratégica central** (ver `docs/01-planejamento-estrategico/00-contexto-atual.md` §3.2.1).
**Componentes:** AccessPolicy multi-nível, KnowledgeBase + RAG, Agents, Hives, Tools, Triggers (incluindo WhatsApp Channel), Outputs, Suites, Runs/Reports, observabilidade administrativa.
**Esforço:** 4-6 meses (full-time) / 5-7 meses (cenário recomendado).
**Valor:** desbloqueio sistêmico — sem ele, nenhum dos AI Apps acima é viável com 1 dev.
**Histórico:** absorve e supera duas features que apareciam em versões anteriores deste documento (antes de 2026-05-19) — "Construtor de agentes, contextos e ferramentas multi-nível" e "Multi-Level Context Builder". Ambas viraram subcomponentes (KnowledgeBase + Context + AccessPolicy + RAG) das fundações do framework.

### 4.2 Observabilidade de RAG no painel administrativo

**Descrição:** Permite analisar a performance do RAG do Nexus em tempo real, com métricas de performance, consultas, respostas.
**Inclusa no framework:** parcialmente (admin observability surface — `ai_apps.md` §13). Detalhes específicos de RAG podem ficar para fase pós-MVP.

### 4.3 Gerenciamento de usuários do Nexus

**Descrição:** Listar e gerenciar os usuários do Nexus (tal como no Azume ADM).
**Esforço:** baixo (reuso de padrões já existentes no Azume ADM).
**Valor:** operacional — necessário para qualquer cliente externo.

---

## 5. Features que NÃO são AI Apps (a fazer fora do framework)

### 5.1 Nexus no WhatsApp (= primitivo do framework + UX)

**Descrição:** Integração do Nexus com o WhatsApp, permitindo conversar com a IA do Nexus via WhatsApp. **Não é uma feature isolada** — é o conjunto (a) entidade `WhatsAppChannel` no framework (`ai_apps.md` §3.3.2/§3.3.3 — Phase 3c), (b) Trigger subtype `whatsapp` e Output subtype `whatsapp_message`, (c) integração Twilio (provedor inicial, abstraído).
**Status:** parte do MVP do framework (Phase 3c do blueprint).
**Por que estratégico:** interface familiar para nosso público de baixa afinidade tech; tese north-star de "operar o Azume CRM inteiro pelo WhatsApp".

---

## Notas finais

- **Esforço / Valor / Custo Operacional** foram preenchidos qualitativamente. Quantificação precisa por feature pode ser feita conforme cada uma entrar no planejamento de execução.
- **Sequência de implementação** segue o roadmap em `docs/01-planejamento-estrategico/04-ai-apps.md` §9.3 (Suite 3 antes da Suite 1 antes da Suite 2; framework é pré-requisito de tudo).
- **Decisões pendentes que afetam essa lista:** preço por usuário dos tiers PRO_PLUS e PRO_MAX (ver `04-ai-apps.md` §6.7); número de contas ativas do CRM (campo `[PREENCHER]` em `00-contexto-atual.md` §4); validação de demanda das features 1.x via entrevistas meses 2-4.
