# NEXUS

## Construtor de contexto multi-nível

Esta feature fornece contexto para os agentes do Nexus com controle multi-nível [A]-[F] e separação operacional entre:

- **Policy**: regras/instruções comportamentais curtas, sempre injetadas no prompt.
- **Knowledge**: documentos e fatos, sempre recuperados via RAG (não entram no prompt por padrão).

Na V1, o RAG adotado será o padrão: **vector embeddings + hybrid search** em **MongoDB Atlas** (Atlas Vector Search + Atlas Search), com observabilidade/auditoria mínima viável.

### 1. Funcionamento

- Um novo menu de configurações será adicionado à plataforma.
- Neste menu, uma das opções será o construtor de contexto (nome pendente; escolher nome objetivo para usuários).
- No construtor, o usuário poderá fazer upload de documentos para compor o contexto dos agentes.
- Na etapa de aprovação, o usuário deverá classificar o conteúdo em uma das classes abaixo:
  - **Policy (sempre no prompt)**.
  - **Knowledge (busca automática via RAG)**.
- **Pinned Summary (opcional)**:
  - Para conteúdos grandes (principalmente Knowledge), o sistema pode gerar um **resumo curto** (“pinned summary”) e permitir que o usuário **opte por incluí-lo no prompt** como âncora.
  - O pinned summary não substitui o RAG; ele só adiciona um contexto fixo mínimo quando ativado.
- Formatos suportados para a v1.0.0: `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.pdf`, `.txt`, `.md`, `.csv`, `.docx`, `.docm`, `.xlsx`, `.xlsm`.
- Formatos suportados para versões futuras: `.mp3`, `.m4a`, `.wav`, `.webm`, `.ogg`, `.mp4`, `.mov`.

Impacto mínimo no front-end (apenas onde necessário):
- Substituir “crítico / não crítico” por **“Policy (sempre no prompt)”** e **“Knowledge (busca via RAG)”** na etapa de aprovação.
- Para Knowledge, expor opção **“Incluir resumo fixo (Pinned Summary) no prompt”** quando houver pinned summary disponível.

### 2. Multi-nível

O “nível” define em quais condições um conteúdo se aplica ao usuário. O acesso a diferentes níveis depende do tipo de usuário.

**Níveis:**

- **[A] — Global (todos os usuários do Nexus):** aplicado a toda a plataforma.
- **[B] — "SystemSource" específico:** aplicado condicionalmente com base no "SystemSource" do usuário (multi-tenant).
- **[C] — "Account" específica:** aplicado condicionalmente com base na conta do usuário.
- **[D] — "Scopes" ("UserScope[]") de uma "Account" específica:** aplicado condicionalmente com base nos "scopes" do usuário (basta estar no array).
  Exemplo: na conta "ID_ACCOUNT_1", existem os usuários "ID_USER_1" e "ID_USER_2", ambos com `scopes = ["sales", "marketing"]`. "ID_USER_1" tem `main_scope = "sales"` e "ID_USER_2" tem `main_scope = "marketing"`. Se for criada uma regra para incluir conteúdo para `scope = "marketing"`, o conteúdo será aplicado para ambos.
- **[E] — "Main scope" ("UserScope") de uma "Account" específica:** aplicado condicionalmente com base no `main_scope` do usuário.
  Exemplo: usando o cenário acima, se for criada uma regra para incluir conteúdo para `main_scope = "marketing"`, o conteúdo será aplicado somente para "ID_USER_2".
- **[F] — Usuário específico:** aplicado condicionalmente para um único usuário.

**Como o multi-nível vira RAG (filtros por metadados / ABAC-like):**
- Para **Knowledge**, os níveis [A]-[F] são materializados como **metadados persistidos por documento/chunk** e aplicados como **filtros obrigatórios** na recuperação.
- A “cascata” de [A]→[F] não é feita “por prompt”; ela vira um conjunto de **condições de elegibilidade** (ex.: `systemSource`, `accountId`, `scopes`, `mainScope`, `userId`) para o retriever.

**Permissões por tipo de usuário:**

Admins do sistema:
- Definição: usuários com "AccountType/UserType" = "internal", "SystemSource" = "nexus" e "UserRole" = "owner".
- Níveis disponíveis: todos ([A] a [F]).
- Níveis exclusivos: [A] e [B].
- Observação: será necessário implementar um sistema de busca por contas e usuários para que admins possam editar contas específicas (incluindo seus escopos) e usuários individuais.

Donos de contas:
- Definição: usuários com "UserRole" = "owner".
- Níveis disponíveis: [C], [D], [E] e [F].
- Observação: será necessário implementar um sistema de busca por usuários vinculados à conta para que donos possam editar usuários específicos.

Colaboradores:
- Definição: qualquer usuário que não seja admin do sistema nem dono de uma conta ("UserRole" = "employee").
- Níveis disponíveis: apenas [F].
- Observação: colaboradores só podem personalizar o contexto de sua própria conta.

### 3. Formas de inclusão de contexto no prompt (V1)

Na V1, a inclusão é determinística:

- **Policy:** sempre injetada no prompt (curta; com hard-caps).
- **Knowledge:** sempre recuperada via RAG (não entra no prompt por padrão).
- **Pinned Summary:** opcionalmente injetado no prompt quando ativado (curto; com cap próprio).

Conteúdo “longo” deve ser Knowledge (RAG). Conteúdo em prompt deve ser Policy/pinned summary (curto).

Fora da V1:
- **"Knowledge-hoarder agents"** serão tratados como ideia futura (não implementar na V1).

### 4. Conversão de arquivos em texto (markdown) e CRUD do contexto

- Ao fazer upload de um arquivo, ele deve ser processado e convertido em **markdown canônico** (texto-base do sistema).
- O markdown gerado será salvo na base. O usuário poderá visualizá-lo no frontend para revisar o conteúdo.
- Logo após o upload, haverá uma etapa de aprovação:
  - o usuário revisa/edita o markdown gerado (inclusive com auxílio de um agente (LLM) via chat, se já existir esse fluxo);
  - o usuário classifica o conteúdo como **Policy** ou **Knowledge**;
  - se **Knowledge**, o usuário pode (opcionalmente) solicitar/ativar um **Pinned Summary** para inclusão no prompt.
- O usuário poderá aprovar ou rejeitar o conteúdo. Em ambos os casos, o arquivo original será descartado após a decisão.
- Se o conteúdo for rejeitado, nada será persistido na base.
- Após aprovado, o conteúdo ficará disponível para visualização, edição e exclusão no menu de construção de contexto.

Indexação/persistência (V1):
- **Policy:** persistir texto final + metadados do nível [A]-[F]. Aplicar validação de tokens ao ativar (seção 5).
- **Knowledge (RAG):**
  - fazer chunking do markdown e persistir chunks + embeddings + metadados;
  - habilitar busca híbrida (lexical + vetorial) no MongoDB Atlas:
    - **Atlas Search** (texto / BM25);
    - **Atlas Vector Search** (similaridade vetorial);
    - estratégia “hybrid” na V1: recuperar candidatos por ambos e combinar em um ranking final auditável.
- **Pinned Summary:** persistir como artefato separado, versionado (texto do resumo + referência ao(s) documento(s) origem + modelo/versão de geração).

Permissões de visualização/edição:
- Contextos dos níveis [A] e [B] só poderão ser visualizados por admins.
- Admins e donos de contas devem poder visualizar, editar e excluir o markdown de todos os níveis aos quais têm acesso, incluindo o de cada usuário individualmente.
- Colaboradores poderão visualizar o conteúdo adicionado por admins e donos de conta, porém não poderão editá-lo.
- Colaboradores poderão visualizar, editar e excluir o conteúdo adicionado por eles próprios.

### 5. Gerenciamento de tokens (somente in-prompt: Policy + Pinned Summary)

O sistema deverá calcular tokens apenas para conteúdo que entra no prompt: **Policy** e **Pinned Summary**. **Knowledge via RAG não entra nesse cálculo**.

Caps por categoria (substitui o modelo 15k/15k/15k):
- **System policy** \([A] + [B]\) in-prompt: **3k–6k tokens**.
- **Account policy** \([C] + [D] + [E]\) in-prompt: **3k–6k tokens**.
- **User policy** \([F]\) in-prompt: **1k–3k tokens**.

Pinned Summary:
- Conta como in-prompt e deve ter cap próprio (curto e estável).
- Deve entrar no cálculo e na validação da categoria aplicável ao nível onde foi definido.

Visibilidade:
- Admins e donos de contas devem poder visualizar os tokens in-prompt por usuário. Colaboradores devem ver os próprios.
- No frontend, não usar o termo "overhead". Rótulos:
  - "memória ocupada por contexto de sistema (Policy)"
  - "memória ocupada por contexto da conta (Policy)"
  - "memória ocupada por contexto individual (Policy)"
  - e, quando aplicável, “resumos fixos (Pinned Summary)”.

### 6. Gerenciamento de tokens reservados para Knowledge via RAG (budget dinâmico + teto recomendável)

- A quantidade de tokens utilizada por Knowledge via RAG será calculada dinamicamente por requisição, com base no espaço remanescente após:
  - prompt base do sistema (incluindo ferramentas), e
  - **Policy + pinned summary**.
- Para manter a V1 estável e previsível, definir um **teto recomendável** (limite global) de tokens de contexto recuperado por resposta (independente de modelos com janelas enormes).
- Considerar que o RAG do construtor coexistirá com outros RAGs do sistema (ex.: memória recente); o budget total de recuperação deve ser dividido por regra simples.

### 7. Pontos pendentes

- Definir valores default (dentro dos ranges) para os caps de tokens das três categorias de Policy.
- Definir cap do pinned summary e regras quando múltiplos pinned summaries se aplicarem ao mesmo usuário.
- Definir a estratégia exata de hybrid search (combinação/normalização de scores lexical + vetorial; re-ranking ou não na V1).
- Definir parâmetros iniciais de chunking (tamanho/overlap) e seleção (top-k, score mínimo, deduplicação).
- Definir modelo de embeddings (provedor/versão) e política de versionamento/reindexação.
- Definir estratégia de consistência quando Knowledge for editado (invalidar/regenerar chunks/embeddings) e como versionar.
- Validar modelagem de metadados/filtros [A]-[F] para isolamento multi-tenant e performance em Atlas.

### 8. Decisões tomadas

- V1 adotará RAG com **vector embeddings + busca híbrida** (MongoDB Atlas Vector Search + Atlas Search).
- Conteúdo será separado em duas classes:
  - **Policy**: sempre injetada no prompt (curta; capada).
  - **Knowledge**: sempre recuperada via RAG (não entra no prompt por padrão).
- **Pinned Summary** será suportado como resumo curto gerado automaticamente e opcionalmente injetado no prompt como âncora.
- **"Knowledge-hoarder agents"** ficam fora da V1 (ideia futura).
- Hard-caps de tokens serão aplicados apenas a conteúdo in-prompt (Policy/pinned summary), substituindo o modelo 15k/15k/15k por caps menores e estáveis:
  - System policy \([A]+[B]\): 3k–6k
  - Account policy \([C]+[D]+[E]\): 3k–6k
  - User policy \([F]\): 1k–3k
- O conceito multi-nível [A]-[F] será mantido e, no RAG, vira filtros por metadados (ABAC-like).

### 9. Pendências remanescentes

- Valores default exatos dentro dos ranges de tokens de Policy e se serão configuráveis.
- Teto recomendável exato para tokens de Knowledge via RAG por resposta e divisão do budget com outros RAGs.
- Estratégia final de hybrid search e necessidade de re-ranking.
- Especificação final de chunking e limites (top-k, score mínimo, deduplicação).
- Estratégia de versionamento/reindexação para mudanças de embeddings e edições de documentos.
- Retenção e acesso aos logs de auditoria (quem pode consultar e por quanto tempo).

### 10. Observabilidade e Auditoria do RAG (mínimo viável)

Para cada resposta gerada com RAG, registrar (log estruturado com correlação por request):

- Identidade e escopo: `requestId`, `conversationId`, `userId`, `accountId`, `systemSource`, `userScopes`, `mainScope`.
- Versões do contexto in-prompt efetivamente aplicado:
  - IDs/versões das Policies aplicadas (por categoria) e tokens estimados por categoria;
  - `pinnedSummaryId`/versão (se usado) e tokens estimados.
- Consulta e filtros do retriever:
  - representação do query usado para retrieval (ou hash se sensível),
  - filtros de metadados aplicados (níveis [A]-[F] materializados).
- Resultados do retrieval (top-N usados):
  - para cada chunk: `documentId`, `chunkId`, `documentVersion` (ou `chunkVersion`), `scoreVector`, `scoreText`, `scoreFinal`, posição no ranking,
  - `embeddingModelVersion` e identificação da configuração/versão do índice.
- Métricas de execução:
  - latência total e por etapa (vector, lexical, merge/rank, montagem de contexto),
  - contagens: candidatos recuperados, chunks selecionados, tokens estimados do contexto recuperado.
- Saída do modelo:
  - `modelId`/versão, tokens de prompt/completion e um `responseId` para auditoria.

# AZUME
