# Lista de Features

## Finalizar follow-ups simples do gerador de propostas híbrido e BESS

**Objetivo:**
Implementar features pendentes de follow-ups do gerador de propostas híbrido e BESS.

**Esforço (1-5):** 2

**Dependências:**
- Nenhuma

**Repositórios envolvidos:**
- azume-backend
- azume-frontend-crm

**Atividades:**
- Remover bloqueio de superdimensionamento de 200% para PCS, banco de baterias e potência fotovoltaica, mantenha somente a UI mostrando que o sistema está superdimensionado (gauges em vermelho).
- Integrar upload (importação) de vendas, projetos, obras, administrativo, pós-vendas e pré-vendas com o gerador de propostas híbrido e BESS.
- Integrar download (exportação) de propostas, vendas, projetos, obras, administrativo, pós-vendas e pré-vendas com o gerador de propostas híbrido e BESS.
- Integrar relatórios (métricas) de página inicial, pré-vendas, pós-vendas, vendas, projetos, obras, administrativo, com o gerador de propostas híbrido e BESS.
- Integrar geração de contratos (novas tags) com o gerador de propostas híbrido e BESS.
- Tornar OPCIONAIS todos os campos de cadastro de baterias, inversores híbridos e containers BESS que: (a) não são utilizados em nenhum cálculo - ou seja, não são consumidos em nenhum cálculo OU (b) possuem fallback para valores configurados no "settings" da conta do usuário (ver no gerador de propostas híbrido e BESS).

## Implementar validação detalhada do gerador de propostas híbrido e BESS

**Objetivo:**
Deixar EXTREMAMENTE CLARO (considerando todas as soluções inclusas em cada cenário) para o engenheiro:
- Como o sistema chegou nos valores dimensionados do banco de baterias, PCS e potência fotovoltaica. (isso nem o on-grid faz)
- Como o sistema chegou nos KPIs financeiros e fluxo de caixa (tabela de geração ano a ano e mês a mês, custo, economia, etc). Se basear na tabela de validação do sistema on-grid, adaptando para a realidade do sistema híbrido e BESS.

**Esforço (1-5):** 2

**Dependências:**
- Nenhuma

**Repositórios envolvidos:**
- azume-backend
- azume-frontend-crm

**Atividades:**
- Implementar validação detalhada do DIMENSIONAMENTO do sistema híbrido e BESS (tamanho dos alvos do banco de baterias, PCS e potência fotovoltaica).
- Implementar validação detalhada do KPIs financeiros e fluxo de caixa do sistema híbrido e BESS (ver tabela de validação do sistema on-grid).

## Implementar copilot Nexus IA no Azume CRM (requer atualização de /home/paulo/projects/nexus/nexus-core/packages/nexus/nexus-contexts/src/nexus_contexts/data/azume_knowledge para incluir o novo gerador de propostas híbrido e BESS na base de conhecimento do Nexus IA + liberar o Nexus IA para toda a base do Azume CRM)

**Objetivo:**
Implementar copilot Nexus IA no Azume CRM. A ideia é fornecer um assitente EFETIVO de IA 24/7 para o usuário do Azume CRM. Serve como um novo pilar de suporte e onboarding para o usuário.

**Esforço (1-5):** 3

**Dependências:**
- Atualização de /home/paulo/projects/nexus/nexus-core/packages/nexus/nexus-contexts/src/nexus_contexts/data/azume_knowledge para incluir o novo gerador de propostas híbrido e BESS na base de conhecimento do Nexus IA
- Liberar o Nexus IA para toda a base do Azume CRM

**Repositórios envolvidos:**
- nexus-core (onde é definido o agente de IA, a base de conhecimento e consumido os APIs)
- azume-backend (middleware entre o Azume CRM e o Nexus IA)
- azume-frontend-crm (frontend do Azume CRM)

**Atividades:**
- Atualizar a base de conhecimento do Nexus IA para incluir o novo gerador de propostas híbrido e BESS
- Liberar o Nexus IA para toda a base do Azume CRM
- Implementar o copilot Nexus IA no Azume CRM

**Requisitos:**
- Deve ser capaz de fornecer um suporte técnico efetivo e eficiente para o usuário do Azume CRM, baseado em uma base de conhecimento rica e atualizada.
- Deve ser capaz de auxiliar no onboarding de novos usuários do Azume CRM.
- Deve ser capaz de automaticamente identificar o contexto da tela em que o usuário está, para fornecer um suporte mais efetivo e eficiente.
- Verificar viabilidade técnica de enviar estado da tela do Azume CRM para o Nexus IA automaticamente quando o usário enviar uma mensagem para o copilot, para fornecer um suporte mais efetivo e eficiente.
- Verificar viabilidade técnica de capacidade de automação de navegadores para realizar tarefas repetitivas, como preenchimento de formulários, navegação entre telas, etc.

## Incluir capacidade de leitura de fatura do grupo B com modadilade tarifária branca no Nexus IA

**Objetivo:**
Incluir capacidade de leitura de fatura do grupo B com modadilade tarifária branca no Nexus IA, para o novo gerador de propostas híbrido e BESS.

**Esforço (1-5):** 1

**Dependências:**
- Encontrar exemplos de faturas do grupo B com modadilade tarifária branca de diferentes concessionárias

**Repositórios envolvidos:**
- nexus-core
- azume-backend
- azume-frontend-crm

**Atividades:**
- Incluir capacidade de leitura de fatura do grupo B com modadilade tarifária branca no Nexus IA, para o novo gerador de propostas híbrido e BESS.

## Gerar mais 9 modelos de propostas para o novo gerador de propostas híbrido e BESS + sistema para o usuário escolher o modelo de proposta que deseja gerar (deve ser extramamente visual para o usuário)

**Objetivo:**
Criar valor para o usuário do Azume CRM, com mais 9 modelos de propostas para o novo gerador de propostas híbrido e BESS + sistema para o usuário escolher o modelo de proposta que deseja gerar (deve ser extremamente visual para o usuário).

**Esforço (1-5):** 3

**Dependências:**
- Nenhuma

**Repositórios envolvidos:**
- azume-backend
- azume-frontend-crm

**Atividades:**
- Gerar modelo de proposta BESS/Híbrido #2
- Desenvolver sistema para o usuário escolher o modelo de proposta que deseja gerar (deve ser extremamente visual para o usuário) - incluso na etapa 7 "Finalizar" do wizard de geração de proposta de BESS/Híbrido.
- Gerar modelo de proposta BESS/Híbrido #3
- Gerar modelo de proposta BESS/Híbrido #4
- Gerar modelo de proposta BESS/Híbrido #5
- Gerar modelo de proposta BESS/Híbrido #6
- Gerar modelo de proposta BESS/Híbrido #7
- Gerar modelo de proposta BESS/Híbrido #8
- Gerar modelo de proposta BESS/Híbrido #9
- Gerar modelo de proposta BESS/Híbrido #10

**Requisitos:**
- Queremos entregar um modelo por vez, de forma que a gente coloque cada novo modelo em produção sem depender de todos os modelos estarem prontos.
- Deve ser extremamente visual para o usuário, com uma interface amigável e intuitiva. Usuário deve ser capaz de visualizar o modelo completo de uma forma fácil.
- Deve ter "awareness" das soluções que o usuário incluiu em cada cenário, tal como no modelos que já existem.