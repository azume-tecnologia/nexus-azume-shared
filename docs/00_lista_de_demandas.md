# AZUME CRM

### User-facing features (full-stack)

[X] Novos filtros de exportação de vendas: vendedor específico / equipe específica | [X] Documented
[X] Abrir endpoint de API para POST de um novo lead / negócio | [X] Documented
[X] Feature de "batch add" proprietários | [X] Documented
[X] Adicionar campo de estado civil para customers + tag para geração de contrato | [X] Documented
[X] Adicionar notificação de novo lead para o colaborador que o lead foi atribuído | [X] Documented
[X] Melhorar feature de exportação de vendas para Azume CRM com "tipo de nota" | [ ] Documented
[X] Melhorar feature de exportação de leads para Azume CRM com "proprietário do lead" | [ ] Documented
[X] Adicionar feature de exportação de propostas para Azume CRM | [ ] Documented


### Documentation Updates

### Correções

[X] Fix bug ativação do Nexus IA
[X] Fix fetch other users chats not filtering by account
[X] Fix leitura smart 10x o valor extraído indo para o formulário
[X] Fix bug leitura smart warning de ponta e fora ponta trocado mostrando campo de taxa iluminação pública incorretamente
[X] Cap de 1 página (PDF) no leitura smart. Verificar se imagem precisa de algum tipo de cap de segurança também.
[X] Tirar doc de telas do contexto estático do Nexus IA (mover para RAG)
[X] Sistema para liberar o Nexus IA para o usuário
[X] Atualizar documentação para o novo sistema de liberação de acesso ao Nexus IA
[X] Gerar OpenAI e Gemini API keys para produção do Nexus IA e Invoice Extraction
[X] Update Secret Manager with new API keys (prod and dev)
[X] Atualizar documentação para colocar a leitura smart e o Nexus IA como módulo adicional do Azume CRM
[X] Re-seed Azume Knowledge (dev and prod)
[X] Re-run integration tests
[X] Redeploy dev
[X] Re-run e2e tests
[X] Redeploy prod
[X] Account toggling asynchronously with polling

### Before Deploy to production

[X] Setup Nexus production IaC and environment variables
[X] Setup env vars on Heroku and DigitalOcean
[X] Update Azume knowledge files with new features
[X] Re-seed Azume Knowledge
[X] Update documentation for new energy invoice extraction feature

### Backend commit stage pipeline (CI/CD)

[ ] Create CLAUDE.md file in root of repository
[ ] Create CODING_STANDARDS.md file in root of repository