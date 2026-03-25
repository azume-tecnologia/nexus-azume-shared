# AZUME CRM

### User-facing features (full-stack)

[X] Novos filtros de exportação de vendas: vendedor específico / equipe específica | [X] Documented
[X] Abrir endpoint de API para POST de um novo lead / negócio | [X] Documented
[X] Feature de "batch add" proprietários | [X] Documented
[X] Adicionar campo de estado civil para customers + tag para geração de contrato | [X] Documented
[X] Adicionar notificação de novo lead para o colaborador que o lead foi atribuído

AFTER NEXUS DEPLOY TO PRODUCTION:

[ ] Add proposal export feature to Azume CRM
[ ] Augument sales export feature to Azume CRM with "note type"
[ ] Augument lead export feature to Azume CRM with "lead owner"

### Documentation Updates

### Correções

[X] Fix bug ativação do Nexus IA
[X] Fix fetch other users chats not filtering by account
[X] Fix leitura smart 10x o valor extraído indo para o formulário
[X] Fix bug leitura smart warning de ponta e fora ponta trocado mostrando campo de taxa iluminação pública incorretamente
[ ] Cap de 1 página (PDF) no leitura smart. Verificar se imagem precisa de algum tipo de cap de segurança também.
[ ] Tirar doc de telas do contexto estático do Nexus IA (mover para RAG)
[ ] Atualizar documentação para colocar a leitura smart e o Nexus IA como módulo adicional do Azume CRM
[ ] Sistema para liberar o Nexus IA para o usuário

[X] Update documentation for new energy invoice extraction feature

### Before Deploy to production

[X] Setup Nexus production IaC and environment variables
[ ] Setup env vars on Heroku and DigitalOcean
[X] Update Azume knowledge files with new features
[X] Re-seed Azume Knowledge

### Backend commit stage pipeline (CI/CD)

[ ] Create CLAUDE.md file in root of repository
[ ] Create CODING_STANDARDS.md file in root of repository