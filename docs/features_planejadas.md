# NEXUS

## Construtor de contexto multi-nível

Esta feature fornecerá todo o conhecimento que os agentes no Nexus terão. O contexto será dividido em múltiplos níveis, desde o mais genérico — aplicado a todos os chats de todos os usuários da plataforma — até o mais granular — aplicado somente a um papel ("role") de uma conta de usuário específica.

### 1. Funcionamento

- Um novo menu de configurações será adicionado à plataforma.
- Neste menu, uma das opções será o construtor de contexto (nome pendente; verificar melhor nome para apresentar aos usuários).
- No construtor, o usuário poderá fazer upload de múltiplos tipos de documentos para compor o contexto dos agentes.
- Formatos suportados para a v1.0.0: .jpg, .jpeg, .png, .gif, .webp, .pdf, .txt, .md, .csv, .docx, .docm, .xlsx, .xlsm.
- Formatos suportados para versões futuras: .mp3, .m4a, .wav, .webm, .ogg, .mp4, .mov.

### 2. Multi-nível

O "nível" do contexto define em quais condições o conhecimento será incluso para um agente. O acesso a diferentes níveis de construção de contexto dependerá do tipo de usuário.

**Níveis:**

- **[A] — Global (todos os usuários do Nexus):** contexto mais amplo; aplicado a toda a plataforma.

- **[B] — "SystemSource" específico:** contexto incluso condicionalmente com base no "SystemSource" do usuário. Permite que admins adaptem contextos distintos para diferentes "tenants".

- **[C] — "Account" específica:** contexto incluso condicionalmente com base na conta do usuário. Permite que admins e donos de conta definam contextos para todos os usuários de uma mesma conta.

- **[D] — "Scopes" ("UserScope[]") de uma "Account" específica:** contexto incluso condicionalmente com base nos "scopes" do usuário. O usuário não precisa ter o "scope" como "main_scope" para receber o contexto; basta que o "scope" esteja presente no seu array de "scopes".
Exemplo: na conta "ID_ACCOUNT_1", existem os usuários "ID_USER_1" e "ID_USER_2", ambos com "scopes" = ["sales", "marketing"]. "ID_USER_1" tem "main_scope" = "sales" e "ID_USER_2" tem "main_scope" = "marketing". Se for criada uma regra para incluir contexto para o "scope" = "marketing", o contexto será incluso para ambos os usuários.
Permite que admins e donos da conta definam contextos personalizados por setor da empresa.

- **[E] — "Main scope" ("UserScope") de uma "Account" específica:** contexto incluso condicionalmente com base no "main_scope" do usuário. Diferente do nível [D], aqui o usuário precisa ter o "scope" como "main_scope" para receber o contexto.
Exemplo: usando o mesmo cenário do nível [D], se for criada uma regra para incluir contexto para "main_scope" = "marketing", o contexto será incluso somente para "ID_USER_2".
Permite que admins e donos da conta definam contextos personalizados por setor, restritos a usuários cujo setor principal corresponda ao especificado.

- **[F] — Usuário específico:** contexto incluso condicionalmente para um único usuário. Permite que admins, donos da conta e o próprio colaborador definam contextos personalizados para uma conta de usuário individual.

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

### 3. Formas de inclusão de contexto no prompt

A melhor estratégia de inclusão de contexto ainda é uma incógnita para esta feature. A partir de um brainstorming inicial, surgiram as seguintes abordagens:

- **Inclusão direta no prompt de sistema:** forma mais básica e mais assertiva, porém com menor eficiência de janela de contexto. Reservada para conhecimento e instruções que nunca podem deixar de fazer parte do contexto do agente.
- **Inclusão via pipeline de RAG:** forma mais balanceada, porém requer vector embeddings e a construção de toda a infraestrutura de RAG.
- **Inclusão via leitura de documentos:** uma espécie de "doc-read tool-based RAG". Requer fornecer aos agentes ferramentas de leitura de documentos e pesquisa por palavra-chave, além de acesso a documentos em markdown.
- **Inclusão via "knowledge-hoarder agents":** ideia ainda em fase de validação. Um "knowledge-hoarder" seria um agente especializado no conhecimento de um documento ou conjunto de documentos (via prompt de sistema). O agente principal (orchestrator) teria acesso a esses "knowledge-hoarders" e instruções para consultá-los quando necessário. O ponto negativo é o consumo de tokens na inicialização do swarm (contexto injetado no prompt de sistema de múltiplos agentes). Avaliar se faz sentido um modelo híbrido combinando "knowledge-hoarders" com as demais abordagens listadas acima.

A forma como o conteúdo de cada arquivo será adicionado ao contexto da conversa — via prompt de sistema ou via RAG — será definida pelo usuário. Na etapa de aprovação, o usuário poderá classificar o contexto como "crítico" (sempre presente no prompt) ou "não crítico" (recuperável via RAG). A possibilidade de marcar o contexto como crítico dependerá do tamanho do arquivo e do espaço disponível de tokens de overhead. Se não houver espaço, a única opção será a inclusão via RAG.

### 4. Conversão de arquivos em texto (markdown) e CRUD do contexto

- Ao fazer upload de um arquivo, ele deve ser processado: seu conteúdo será lido por um agente (LLM) e convertido em texto (markdown).
- O markdown gerado será salvo na base de dados. O usuário poderá visualizá-lo no frontend para revisar o conteúdo.
- Logo após o upload, haverá uma etapa de aprovação. O usuário poderá revisar e editar o markdown gerado, inclusive com auxílio de um agente (LLM) via chat para melhorar, adicionar ou remover conteúdo.
- O usuário poderá aprovar ou rejeitar o conteúdo. Em ambos os casos, o arquivo original será descartado após a decisão.
- Se o conteúdo for rejeitado, nada será persistido na base.
- Após aprovado, o conteúdo ficará disponível para visualização, edição e exclusão no menu de construção de contexto.
- Se o conteúdo for injetado via RAG, o sistema deverá gerar vector embeddings e persistir os vetores na base de dados juntamente com o conteúdo de texto (habilitando pesquisas híbridas de vetores + texto no MongoDB).
- Contextos dos níveis [A] e [B] só poderão ser visualizados por admins.
- Admins e donos de contas devem poder visualizar, editar e excluir o markdown de todos os níveis aos quais têm acesso, incluindo o de cada usuário individualmente.
- Colaboradores poderão visualizar o conteúdo de contexto adicionado por admins e donos de conta, porém não poderão editá-lo.
- Colaboradores poderão visualizar, editar e excluir o contexto adicionado por eles próprios.

### 5. Gerenciamento de tokens de prompt de sistema

- O sistema deverá calcular o overhead (em tokens) do contexto que será injetado diretamente no prompt.
- Admins e donos de contas devem poder visualizar o overhead de cada usuário individualmente. Colaboradores devem poder visualizar o overhead de suas próprias contas.
- O overhead deverá ser calculado considerando todos os conteúdos de todos os níveis aplicáveis ao usuário, em cascata do mais geral ao mais específico, até compor o overhead total da conta.
- O overhead proveniente dos níveis [A] e [B] será exibido como "overhead de sistema".
- O overhead proveniente dos níveis [C], [D] e [E] será exibido como "overhead da conta".
- O overhead proveniente do nível [F] será exibido como "overhead do usuário".
- No frontend, não será usado o termo "overhead". Os rótulos serão: "memória ocupada por contexto de sistema", "memória ocupada por contexto da conta" e "memória ocupada por contexto individual".
- Um limite máximo de 15k tokens será reservado para o "overhead de sistema" (soma dos níveis [A] + [B]).
- Um limite máximo de 15k tokens será reservado para o "overhead da conta" (soma dos níveis [C], [D] e [E]).
- Um limite máximo de 15k tokens será reservado para o "overhead do usuário" (nível [F]).
- O limite de cada categoria funciona como um teto para a soma de todos os conteúdos dos níveis que a compõem, considerando o usuário com maior overhead dentro daquela categoria. Em outras palavras, ao adicionar contexto em um nível mais amplo (ex.: [A]), o espaço restante para os demais níveis da mesma categoria diminui proporcionalmente.
Exemplo para overhead de sistema: se o nível [A] consome 10k tokens, restam apenas 5k tokens para o nível [B] de cada "SystemSource". Se um "SystemSource" já consome 5k tokens no nível [B], não é possível adicionar mais contexto em [A] nem em [B] para os usuários daquele "SystemSource" sem antes reduzir o conteúdo existente.
- A mesma lógica se aplica ao "overhead da conta": os escopos com maior volume de overhead delimitam quanto ainda pode ser adicionado nos demais níveis. Nenhum grupo de usuários pode ficar com overhead da conta acima de 15k tokens.

OBS: contextos injetados via pipeline de RAG, leitura de documentos ou "knowledge-hoarder agents" não serão contabilizados nesses overheads — somente contextos injetados diretamente nos prompts.

### 6. Gerenciamento de tokens reservados para RAG

Como ainda há incerteza sobre qual método de RAG será adotado (vector embeddings, leitura de arquivos ou "knowledge-hoarders"), esta seção define "RAG" como qualquer um dos três métodos de recuperação de contexto.

- Diferentemente do hard-cap fixo para tokens de prompt de sistema, a quantidade de tokens reservados para RAG será calculada dinamicamente com base na janela de contexto do modelo: um percentual da janela total disponível, descontando-se o overhead do prompt de sistema.
- É importante considerar que o RAG de contexto do construtor não será o único RAG do sistema. Também estão planejados mecanismos de RAG para recuperação das conversas mais recentes do usuário com os agentes (memória recente).

### 7. Pontos pendentes

- Definir a melhor abordagem de RAG: vector embeddings, leitura de arquivos ou "knowledge-hoarders".
- Validar se o plano como um todo faz sentido ou se já existe alguma solução mais consolidada para essa "engenharia de contexto" que seja padrão de mercado.
- Definir o percentual da janela de contexto reservado para RAG.
- Avaliar se faz sentido manter a "inclusão direta no prompt de sistema" ou se tudo deveria ser feito via RAG. Motivo: a inclusão direta introduz bastante complexidade à feature — todo o "Gerenciamento de tokens de prompt de sistema" (um sistema consideravelmente complexo) precisaria ser implementado para permitir que usuários adicionem contexto diretamente no prompt.
- Avaliar a ideia de um único "knowledge-hoarder" (agente extra) para armazenar todo o contexto crítico — ou seja, todo o contexto que seria incluso diretamente no prompt de sistema (overhead de sistema + overhead da conta + overhead do usuário). Isso possibilitaria aumentar significativamente os hard-caps de contexto armazenado, visto que modelos como o Gemini suportam até 1M de tokens de contexto. A contrapartida é um aumento considerável de custo.

# AZUME
