# NEXUS

## Construtor de contexto multi-nível

Esta feature fornece contexto para os agentes do Nexus com controle multi-nível [A]-[F] e separação operacional entre:

- **Policy**: regras/instruções comportamentais curtas, sempre injetadas no prompt.
- **Knowledge**: documentos e fatos, sempre recuperados via RAG (não entram no prompt por padrão).

Na V1, o RAG adotado será o padrão: **vector embeddings + hybrid search** em **MongoDB Atlas** (Atlas Vector Search + Atlas Search), com observabilidade/auditoria mínima viável. Em produção, esse workload de busca será executado em **Search Nodes dedicados** no Atlas (isolamento de workload).

### 1. Funcionamento

- Um novo menu de configurações será adicionado à plataforma.
- Neste menu, uma das opções será o construtor de contexto (nome pendente; escolher nome objetivo para usuários).
- No construtor, o usuário poderá fazer upload de documentos para compor o contexto dos agentes.
- Na etapa de aprovação, o usuário deverá classificar o conteúdo em uma das classes abaixo (V1):
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
- **[B] — SystemSource específico (campo: `system_source`):** aplicado condicionalmente com base no `system_source` do usuário (multi-tenant).
- **[C] — Account específica (campo: `account_id`):** aplicado condicionalmente com base na conta do usuário.
- **[D] — "Scopes" (`user_scopes`, "UserScope[]") de uma "Account" específica:** aplicado condicionalmente com base nos scopes do usuário (basta estar no array).
  Exemplo: na conta "ID_ACCOUNT_1", existem os usuários "ID_USER_1" e "ID_USER_2", ambos com `user_scopes = ["sales", "marketing"]`. "ID_USER_1" tem `main_scope = "sales"` e "ID_USER_2" tem `main_scope = "marketing"`. Se for criada uma regra para incluir conteúdo para `user_scope = "marketing"`, o conteúdo será aplicado para ambos.
- **[E] — "Main scope" ("UserScope") de uma "Account" específica:** aplicado condicionalmente com base no `main_scope` do usuário.
  Exemplo: usando o cenário acima, se for criada uma regra para incluir conteúdo para `main_scope = "marketing"`, o conteúdo será aplicado somente para "ID_USER_2".
- **[F] — Usuário específico (campo: `user_id`):** aplicado condicionalmente para um único usuário.

**Como o multi-nível vira RAG (filtros por metadados / ABAC-like):**
- Para **Knowledge**, os níveis [A]-[F] são materializados como **metadados persistidos por documento/chunk** e aplicados como **filtros obrigatórios** na recuperação.
- A “cascata” de [A]→[F] não é feita “por prompt”; ela vira um conjunto de **condições de elegibilidade** (ex.: `system_source`, `account_id`, `user_scopes`, `main_scope`, `user_id`) para o retriever.
- Guardrail (V1): o retriever deve operar em modo *fail-closed* — queries de Knowledge devem sempre aplicar filtros obrigatórios de tenant (ex.: `system_source`, `account_id`) quando aplicável.
- **ACL (V1):** a V1 **não implementa ACL por documento/chunk além do modelo multi-nível [A]-[F]** (ABAC-like por metadados). Na prática, todo conteúdo de nível [C] é potencialmente visível/recuperável por qualquer usuário da `account_id`, e níveis [D]/[E] por qualquer usuário elegível conforme seus scopes. Implicação prática: conteúdo sensível que não pode ser visto por todos os usuários elegíveis de um nível deve ser colocado em um nível mais específico (ex.: [F]).

**Permissões por tipo de usuário:**

Admins do sistema:
- Definição: usuários com `account_type`/`user_type = "internal"`, `system_source = "nexus"` e `user_role = "owner"`.
- Níveis disponíveis: todos ([A] a [F]).
- Níveis exclusivos: [A] e [B].
- Observação: será necessário implementar um sistema de busca por contas e usuários para que admins possam editar contas específicas (incluindo seus escopos) e usuários individuais.

Donos de contas:
- Definição: usuários com `user_role = "owner"`.
- Níveis disponíveis: [C], [D], [E] e [F].
- Observação: será necessário implementar um sistema de busca por usuários vinculados à conta para que donos possam editar usuários específicos.

Colaboradores:
- Definição: qualquer usuário que não seja admin do sistema nem dono de uma conta (`user_role = "employee"`).
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
- Diretriz de conversão (V1): preferir **extração estruturada** quando disponível (ex.: texto nativo de PDF/DOCX) e aplicar pós-processamento; evitar “completar lacunas” e reduzir risco de alucinação na conversão.
- **Fluxo de upload/aprovação (V1 — draft temporário com TTL):**
  - o resultado do processamento (markdown + metadados do nível [A]-[F]) é persistido como **draft** com expiração (TTL);
  - V1 default: `draft_ttl_hours = 72`; o draft expira por TTL e é limpo automaticamente;
  - decisão V1: **o TTL é renovado a cada edição salva** do draft;
  - o usuário revisa/edita o draft no frontend e seleciona **Policy** ou **Knowledge** (e, se Knowledge, se quer pinned summary);
  - ao **aprovar**, o draft é promovido para ativo e os artefatos definitivos são gerados (Policy ativa ou Knowledge indexada);
  - ao **rejeitar**, o draft é **excluído** e nenhum artefato definitivo permanece;
  - o arquivo original é descartado após a decisão (aprovar/rejeitar).
- Após aprovado, o conteúdo ficará disponível para visualização, edição e exclusão no menu de construção de contexto.

Indexação/persistência (V1):
- **Policy:** persistir texto final + metadados do nível [A]-[F]. Aplicar validação de tokens ao ativar (seção 5).
- **Knowledge (RAG):**
  - fazer chunking do markdown e persistir chunks + embeddings + metadados;
  - habilitar busca híbrida (lexical + vetorial) no MongoDB Atlas:
    - **Atlas Search** (texto / BM25);
    - **Atlas Vector Search** (similaridade vetorial);
    - estratégia “hybrid” na V1: recuperar candidatos por ambos e combinar em um ranking final auditável.
- **Pinned Summary:**
  - V1: pinned summary é gerado **por documento** (por padrão) e persistido como artefato separado, versionado.
  - O artefato deve manter `source_document_id` e `source_document_version`.
  - Pinned summary composto (multi-doc) **não é suportado na V1**; se introduzido no futuro, deve ser um artefato separado com `source_document_ids` e versionamento próprio.

Edição de Knowledge após aprovado (V1 — reindex assíncrono):
- Atualizações de Knowledge após aprovado disparam **reindexação assíncrona** com estados operacionais: `active`, `reindexing`, `failed`.
- Durante `reindexing`, o retriever pode continuar usando a versão ativa anterior até a nova versão estar pronta; ao concluir, troca-se a versão ativa de forma atômica.
- Durante `failed`, o retriever continua usando a última versão ativa; o UI deve sinalizar falha para o usuário autorizado.

Versionamento mínimo (V1):
- Knowledge é versionado por `document_version`.
- Chunks e embeddings são vinculados à versão do documento (a unidade de consistência de retrieval é “documento + document_version”).
- Reindex escreve uma nova versão e, ao final, troca-se o “ponteiro” de versão ativa para a nova versão.

#### 4.1 Algoritmo de hybrid search (V1 defaults; sem rerank)

Procedimento mínimo (por requisição, dentro do escopo permitido pelos filtros [A]-[F]):

1) Recuperar `topk_text` chunks via **Atlas Search (BM25)**.
2) Recuperar `topk_vector` chunks via **Atlas Vector Search**.
3) Deduplicar por `chunk_id`.
4) Normalizar `score_text` e `score_vector` para \([0, 1]\) (min-max no conjunto de candidatos da requisição).
   - Edge case (V1): se `min == max` em um dos métodos (texto ou vetor), definir `score_*_norm = 1.0` para todos os candidatos daquele método (evita divisão por zero e mantém ranking determinístico).
5) Combinar scores com média ponderada:
   - `score_final = 0.5 * score_text_norm + 0.5 * score_vector_norm`
6) Ordenar por `score_final` e selecionar `topk_final` chunks para montagem do contexto.
7) **Sem rerank na V1** (rerank fica para V1.1).

Defaults iniciais (configuráveis):
- `topk_text = 40`
- `topk_vector = 40`
- `topk_final = 16`
- `hybrid_weight_text = 0.5`
- `hybrid_weight_vector = 0.5`

Nota de infraestrutura (produção, V1): a carga de busca (Atlas Search + Atlas Vector Search) deve ser isolada em **Search Nodes dedicados** no MongoDB Atlas.

Nota de estabilidade (V1): a normalização min-max é feita por requisição; isso é aceitável na V1 e pode ser refinado em V1.1 com normalização mais robusta e/ou rerank.

Permissões de visualização/edição:
- Contextos dos níveis [A] e [B] só poderão ser visualizados por admins.
- Admins e donos de contas devem poder visualizar, editar e excluir o markdown de todos os níveis aos quais têm acesso, incluindo o de cada usuário individualmente.
- Colaboradores poderão visualizar o conteúdo adicionado por admins e donos de conta, porém não poderão editá-lo.
- Colaboradores poderão visualizar, editar e excluir o conteúdo adicionado por eles próprios.

### 5. Gerenciamento de tokens (somente in-prompt: Policy + Pinned Summary)

O sistema deverá calcular tokens apenas para conteúdo que entra no prompt: **Policy** e **Pinned Summary**. **Knowledge via RAG não entra nesse cálculo**.

Nota (multi-provedor, V1): tokens são estimativas baseadas no tokenizer/modelo do provedor selecionado; a V1 usa uma aproximação consistente para validação e exibição.

Caps por categoria (substitui o modelo 15k/15k/15k; V1 defaults):
- `system_policy_cap_tokens = 4000` (para \([A] + [B]\) in-prompt)
- `account_policy_cap_tokens = 4000` (para \([C] + [D] + [E]\) in-prompt)
- `user_policy_cap_tokens = 2000` (para \([F]\) in-prompt)

Pinned Summary:
- Conta como in-prompt e deve ter cap próprio (curto e estável).
- Deve entrar no cálculo e na validação da categoria aplicável ao nível onde foi definido.
- V1 default: `pinned_summary_cap_tokens = 800`.
- Regra de validação de budget (V1):
  1) truncar o pinned summary para `pinned_summary_cap_tokens`;
  2) somar tokens do pinned summary com a Policy aplicável da categoria;
  3) validar contra o cap da categoria (`system_policy_cap_tokens` / `account_policy_cap_tokens` / `user_policy_cap_tokens`).
- Regras quando múltiplos pinned summaries forem aplicáveis (V1; simples e implementável):
  - a montagem do prompt inclui **no máximo 1 pinned summary por resposta**;
  - justificativa: manter o prompt estável e evitar explosão de tokens;
  - critério de escolha: mais específico vence (prioridade \([F] > [C]/[D]/[E] > [A]/[B]\));
  - em empate (mesmo nível/especificidade), escolher o mais recentemente atualizado;
  - o texto incluído é truncado para `pinned_summary_cap_tokens` (quando necessário).

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

- Definir `rag_context_cap_tokens` (teto recomendável) e como dividir o budget com outros RAGs do sistema (ex.: memória recente).
- Definir parâmetros finais de chunking (tamanho/overlap) e critérios de corte (score mínimo, deduplicação por documento, etc.).
- Definir modelo de embeddings (`embedding_model_version`) e política de reindexação em caso de troca de modelo.
- Definir política de retenção e acesso aos logs de auditoria (quem pode consultar e por quanto tempo).
- Decidir se haverá limite máximo de renovação do TTL de drafts (para evitar drafts “eternos” mesmo com renovação por edição).
- Definir limites de tamanho de arquivo por upload e limites de volume total por nível/conta (V1), por custo e UX.

### 8. Decisões tomadas

- V1 adotará RAG com **vector embeddings + busca híbrida** (MongoDB Atlas Vector Search + Atlas Search).
- V1 em produção usará **Search Nodes dedicados no MongoDB Atlas** para Atlas Search + Atlas Vector Search (isolamento de workload).
- Conteúdo será separado em duas classes:
  - **Policy**: sempre injetada no prompt (curta; capada).
  - **Knowledge**: sempre recuperada via RAG (não entra no prompt por padrão).
- **Pinned Summary** será suportado como resumo curto gerado automaticamente e opcionalmente injetado no prompt como âncora.
- **"Knowledge-hoarder agents"** ficam fora da V1 (ideia futura).
- Hard-caps de tokens serão aplicados apenas a conteúdo in-prompt (Policy/pinned summary), substituindo o modelo 15k/15k/15k por caps menores e estáveis:
  - `system_policy_cap_tokens = 4000` (\([A]+[B]\))
  - `account_policy_cap_tokens = 4000` (\([C]+[D]+[E]\))
  - `user_policy_cap_tokens = 2000` (\([F]\))
- O conceito multi-nível [A]-[F] será mantido e, no RAG, vira filtros por metadados (ABAC-like).
- Upload/aprovação usará draft temporário com TTL e promoção para `active` somente após aprovação.
- Knowledge aprovado será reindexado de forma assíncrona em edições, com fallback para versão anterior durante `reindexing`.

### 9. Observabilidade e Auditoria do RAG (mínimo viável)

Para cada resposta gerada com RAG, registrar (log estruturado com correlação por request):

- Identidade e escopo: `request_id`, `conversation_id`, `user_id`, `account_id`, `system_source`, `user_scopes`, `main_scope`.
- Controle de versão/configuração do retriever:
  - `retriever_version` (versão do componente/estratégia de retrieval ativa),
  - `hybrid_config_id` (identificação da configuração efetiva de `topk_*` e pesos aplicados).
- Versões do contexto in-prompt efetivamente aplicado:
  - IDs/versões das Policies aplicadas (por categoria) e tokens estimados por categoria;
  - `pinned_summary_id`/versão (se usado) e tokens estimados.
- `prompt_assembly_hash`: hash do prompt final (ou, no mínimo, do bloco de contexto montado) para reprodutibilidade/auditoria.
- Consulta e filtros do retriever:
  - representação do query usado para retrieval (ou hash se sensível),
  - filtros de metadados aplicados (níveis [A]-[F] materializados).
- Resultados do retrieval (top-N usados):
  - para cada chunk: `document_id`, `chunk_id`, `document_version` (ou `chunk_version`), `score_vector`, `score_text`, `score_final`, posição no ranking,
  - por chunk, registrar “por que entrou”: `matched_levels` (ex.: `["A","C"]`) e/ou `eligibility_source` (ex.: `"account"`),
  - `embedding_model_version` e identificação da configuração/versão do índice.
- Métricas de execução:
  - latência total e por etapa (vector, lexical, merge/rank, montagem de contexto),
  - contagens: candidatos recuperados, chunks selecionados, tokens estimados do contexto recuperado.
- Saída do modelo:
  - `model_id`/versão, tokens de prompt/completion e um `response_id` para auditoria.

### 10. Resumo das melhorias

- Padronização de nomenclatura de backend para `snake_case` e inclusão de mapeamentos mínimos (ex.: SystemSource → `system_source`, UserRole → `user_role`).
- Fluxo consistente de persistência no upload/aprovação usando **draft com TTL** (default 72h), limpeza automática e renovação do TTL a cada edição salva.
- Clarificação explícita de permissões/ACL: V1 **não tem ACL por documento/chunk** além do ABAC-like por metadados [A]-[F].
- Decisão de infraestrutura: produção V1 executará Atlas Search + Atlas Vector Search em **Search Nodes dedicados** (isolamento de workload).
- Pinned summary clarificado: **por documento na V1**, cap default e regra única de seleção; budget fechado com validação contra cap da categoria.
- Edição de Knowledge pós-aprovação definida como **reindex assíncrono**, com versionamento mínimo por `document_version` e troca atômica da versão ativa.
- Hybrid search V1 mantido simples, com nota de estabilidade sobre normalização min-max por requisição e evolução prevista para V1.1.
- Observabilidade/auditoria ampliada com `retriever_version`, `hybrid_config_id`, `prompt_assembly_hash` e campos por chunk (`matched_levels`/`eligibility_source`).
