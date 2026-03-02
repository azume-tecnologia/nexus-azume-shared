# NEXUS

## LEITOR DE FATURAS DE ENERGIA

### IDEIA

O leitor de faturas de energia será uma feature powered by LLMs que possibilitará aos usuários do Azume CRM + Nexus enviar um arquivo de fatura de energia e receber todos os dados da fatura extraídos, a fim de facilitar e tornar mais eficiente a geração de um orçamento de energia solar.

### AMOSTRAS DE FATURAS DE ENERGIA

Abaixo, algumas amostras de faturas de energia:

**Grupo B:**

1 - samples/faturas-de-energia/fatura-bulbe-grupo-b.pdf
2 - samples/faturas-de-energia/fatura-CERRP-grupo-b.pdf
3 - samples/faturas-de-energia/fatura-CPFL-grupo-b.pdf
4 - samples/faturas-de-energia/fatura-enel-grupo-b.pdf
5 - samples/faturas-de-energia/fatura-enel-grupo-b-2.pdf

**Grupo A:**

1 - samples/faturas-de-energia/fatura-copel-grupo-a.pdf
2 - samples/faturas-de-energia/fatura-enel-grupo-a.pdf
3 - samples/faturas-de-energia/fatura-equatorial-grupo-a.pdf

**Referência:**

1 - samples/faturas-de-energia/guia-calculo-fatura-grupo-a-azume.pdf

### REQUISITOS TÉCNICOS

- Formatos suportados para a v1.0.0: `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.pdf`.

### EXTRAÇÃO DE VALORES

**Modalidade Tarifária:**

Primeiramente, deve-se entender qual a modalidade tarifária da conta de energia (Grupo B ou Grupo A). Isso irá determinar quais valores devem ser extraídos da conta de energia.

Valores:

| Nome                 | Tipo           | Valores    | Campo Backend Azume CRM | Campo Frontend Azume CRM | Obrigatório | Unidade |
| -------------------- | -------------- | ---------- | ----------------------- | ------------------------ | ----------- | ------- |
| Modalidade Tarifária | Enum de string | "A" ou "B" | tariffModality          | tariffModality           | Sim         | -       |

**Grupo B (baixa tensão):**

Para contas do grupo B, devem ser extraídos os valores abaixo:

| Nome               | Tipo                | Valores                                 | Campo Backend Azume CRM | Campo Frontend Azume CRM | Obrigatório | Unidade |
| ------------------ | ------------------- | --------------------------------------- | ----------------------- | ------------------------ | ----------- | ------- |
| Concessionária     | Enum de string      | -                                       | powerDistCompany        | powerDistCompany         | Não         | -       |
| Valor kWh          | number              | > 0 && < 10                             | kwhPrice                | kwhPrice                 | Sim         | R$      |
| Taxa Ilum. Pública | number              | > 0                                     | publicLightBill         | publicLightBill          | Não         | R$      |
| Rede               | Enum de string      | "Trifásica", "Bifásica" ou "Monofásica" | networkClass            | networkClass             | Sim         | -       |
| Consumo Jan        | number              | >= 0                                    | monthlyConsumption[0]   | jan                      | Não         | kWh     |
| Consumo Fev        | number              | >= 0                                    | monthlyConsumption[1]   | feb                      | Não         | kWh     |
| Consumo Mar        | number              | >= 0                                    | monthlyConsumption[2]   | mar                      | Não         | kWh     |
| Consumo Abr        | number              | >= 0                                    | monthlyConsumption[3]   | apr                      | Não         | kWh     |
| Consumo Mai        | number              | >= 0                                    | monthlyConsumption[4]   | may                      | Não         | kWh     |
| Consumo Jun        | number              | >= 0                                    | monthlyConsumption[5]   | jun                      | Não         | kWh     |
| Consumo Jul        | number              | >= 0                                    | monthlyConsumption[6]   | jul                      | Não         | kWh     |
| Consumo Ago        | number              | >= 0                                    | monthlyConsumption[7]   | aug                      | Não         | kWh     |
| Consumo Set        | number              | >= 0                                    | monthlyConsumption[8]   | sep                      | Não         | kWh     |
| Consumo Out        | number              | >= 0                                    | monthlyConsumption[9]   | oct                      | Não         | kWh     |
| Consumo Nov        | number              | >= 0                                    | monthlyConsumption[10]  | nov                      | Não         | kWh     |
| Consumo Dez        | number              | >= 0                                    | monthlyConsumption[11]  | dec                      | Não         | kWh     |
| tusd               | number              | >= 0                                    | tusd                    | tusd                     | Não         | R$      |
| icms               | number (percentual) | > 0 && < 100 %                          | icms                    | icms                     | Não         | %       |

Obs1: Nem todas as contas possuem o consumo dos últimos 12 meses, algumas só exibem 8 meses ou somente 6 meses. Neste caso, os meses que estão faltando devem ser preenchidos com a média de consumo dos meses que estão presentes na fatura de energia. **Importante:** esse preenchimento é pós-processamento executado pelo backend (`nexus-services`), não é responsabilidade do agente de extração (ver seção "Decisões Técnicas > Sem tools de cálculo").
Obs2: O valor de kwhPrice já deve incluir todos os tributos/impostos.
Obs3: No grupo B, os valores de tusd e icms não são essenciais para que o usuário prossiga com a geração do orçamento de energia solar, porém são importantes para que o usuário possa gerar uma proposta mais assertiva. Portanto, deve-se aplicar uma política de "melhor esforço" para extração de tusd e icms, porém sem bloquear o critério de sucesso da leitura da fatura de energia.
Obs4: Mesma coisa que "Obs3" para taxa de iluminação pública (melhor esforço para extração da taxa de iluminação pública, sem bloqueio de sucesso).
Obs5: Mesma coisa que "Obs3" para a concessionária (melhor esforço para extração da concessionária, sem bloqueio de sucesso).

**Critérios de sucesso da leitura Grupo B**:

- Extrair todos os campos obrigatórios
- (OPCIONAL) Definir o consumo médio para os meses que estão faltando na fatura de energia
- Todos os meses possuem valor de consumo (provindo da leitura direta ou do cálculo da média)

Caso os critérios de sucesso não sejam alcançados, deve ser considerado que houve uma falha na leitura.

**Grupo A (alta tensão) - Classificação Horo-Sazonal:**

Uma vez identificado que a fatura é do grupo A (alta tensão), deve-se identificar a Classificação Horo-Sazonal da fatura. O Azume CRM fornece suporte para dimensionamento das classificações "Azul" e "Verde".

Valores:

| Nome                       | Tipo           | Valores           | Campo Backend Azume CRM | Campo Frontend Azume CRM | Obrigatório | Unidade |
| -------------------------- | -------------- | ----------------- | ----------------------- | ------------------------ | ----------- | ------- |
| Classificação Horo-Sazonal | Enum de string | "Azul" ou "Verde" | classification          | classification           | Sim         | -       |

**Grupo A (alta tensão) - Verde:**

| Nome                                                      | Tipo                | Valores        | Campo Backend Azume CRM      | Campo Frontend Azume CRM | Obrigatório | Unidade |
| --------------------------------------------------------- | ------------------- | -------------- | ---------------------------- | ------------------------ | ----------- | ------- |
| Concessionária                                            | Enum de string      | -              | powerDistCompany             | powerDistCompany         | Não         | -       |
| TE Fora Ponta                                             | number              | > 0 && < 10    | kwhPrice                     | kwhPrice                 | Sim         | R$      |
| TE Ponta                                                  | number              | > 0 && < 10    | kwhPricePeak                 | kwhPricePeak             | Sim         | R$      |
| TUSD Fora Ponta                                           | number              | > 0 && < 10    | tusd                         | tusd                     | Sim         | R$      |
| TUSD Ponta                                                | number              | > 0 && < 10    | tusdPeak                     | tusdPeak                 | Sim         | R$      |
| Demanda Contratada                                        | number              | >= 30          | demand                       | demand                   | Sim         | kW      |
| Tarifa da Demanda                                         | number              | > 0            | demandTariff                 | demandTariff             | Sim         | R$      |
| Taxa Ilum. Pública                                        | number              | > 0            | publicLightBill              | publicLightBill          | Não         | R$      |
| Consumo Médio Mensal Fora Ponta                           | number              | > 0            | monthlyConsumption[0-11]     | averageValue             | Sim         | kWh     |
| Consumo Médio Mensal Ponta                                | number              | > 0            | monthlyConsumptionPeak[0-11] | averageValuePeak         | Sim         | kWh     |
| icms                                                      | number (percentual) | > 0 && < 100 % | icms                         | icms                     | Não         | %       |

Obs sobre consumo mensal (Grupo A): O agente deve extrair **um único valor médio** para cada tipo de consumo (fora ponta e ponta). O backend (`nexus-services`) será responsável por replicar esse valor médio nos 12 meses dos arrays `monthlyConsumption[0-11]` e `monthlyConsumptionPeak[0-11]`. Diferentemente do Grupo B, onde o agente extrai os 12 meses individuais, no Grupo A a extração é de um valor médio por tipo.

Obs1: Para a classificação "Verde", a demanda contratada não é diferenciada entre ponta e fora ponta, há apenas um valor para demanda contratada. Na classificação "Azul", os campos "Demanda Contratada" e "Tarifa da Demanda" da tabela Verde correspondem aos valores de fora ponta, e os campos adicionais "Demanda Contratada Ponta" e "Tarifa da Demanda Ponta" da tabela Azul representam os valores de ponta.
Obs2: No grupo A Verde, o valor de icms não é essencial para que o usuário prossiga com a geração do orçamento de energia solar, porém é importante para que o usuário possa gerar uma proposta mais assertiva. Portanto, deve-se aplicar uma política de "melhor esforço" para extração de icms, porém sem bloquear o critério de sucesso da leitura da fatura de energia.
Obs3: Mesma coisa que "Obs2" para taxa de iluminação pública (melhor esforço para extração da taxa de iluminação pública, sem bloqueio de sucesso).
Obs4: Mesma coisa que "Obs3" para a concessionária (melhor esforço para extração da concessionária, sem bloqueio de sucesso).

**Grupo A (alta tensão) - Azul:**

Além de todos os valores do Grupo A (alta tensão) - Verde, devem TAMBÉM ser extraídos os valores abaixo para classificação "Azul":

| Nome                     | Tipo   | Valores | Campo Backend Azume CRM | Campo Frontend Azume CRM | Obrigatório | Unidade |
| ------------------------ | ------ | ------- | ----------------------- | ------------------------ | ----------- | ------- |
| Demanda Contratada Ponta | number | > 0     | demandPeak              | demandPeak               | Sim         | kW      |
| Tarifa da Demanda Ponta  | number | > 0     | demandTariffPeak        | demandTariffPeak         | Sim         | R$      |

Obs: Para a classificação "Azul", a demanda contratada é diferenciada entre ponta e fora ponta.

**Critérios de sucesso da leitura Grupo A**:

- Extrair todos os campos obrigatórios (incluindo classificação horo-sazonal e campos específicos da classificação Verde ou Azul)
- Extrair o valor de consumo médio mensal fora ponta e ponta
- O consumo médio mensal será replicado para os 12 meses pelo backend (`nexus-services`) — não há extração de meses individuais no Grupo A

Caso os critérios de sucesso não sejam alcançados, deve ser considerado que houve uma falha na leitura.

**Definições:**

1. Campo Backend Azume CRM: nome do campo na coleção de dados "proposal" que armazena o valor.
2. Campo Frontend Azume CRM: ID do input do formulário no frontend do Azume que armazena o valor.
3. Obrigatório: Se o campo for obrigatório e a LLM não conseguir extrair o valor da conta de energia, significa que a leitura da fatura de energia falhou.

### SUPPORT TOOLS PARA AGENTES

- O agente de extração não utilizará tools auxiliares (calculadora, Python, MCP, etc.)
- O agente será focado exclusivamente na extração visual dos valores da fatura de energia
- Operações de cálculo (ex: média de meses faltantes no Grupo B) serão executadas por código de regra de negócio no backend (`nexus-services`), após a extração

### CONCESSIONÁRIAS

As concessionárias de energia são as seguintes. O campo Concessionária deve ser um enum de string com os valores de "Company" abaixo. São os valores que o Azume CRM aceita para o campo "powerDistCompany".

Os valores de "aliases" são para ajudar o agente a localizar o valor da concessionária na fatura de energia.

| Company                | Aliases                                                                              |
| ---------------------- | ------------------------------------------------------------------------------------ |
| AMAZONAS ENERGIA       | AmE; CEAM; Amazonas Distribuidora de Energia                                         |
| CASTRO-DIS             | Castro Distribuidora de Energia                                                      |
| CEA EQUATORIAL         | CEA; Companhia de Eletricidade do Amapa; Equatorial Amapa; EQUATORIAL ENERGIA AMAPÁ  |
| CEDRAP                 | Cooperativa de Eletrificacao e Desenvolvimento Rural do Alto Paraiba                 |
| CEDRI                  | Cooperativa de Energizacao e Desenvolvimento Rural do Vale do Itariri                |
| CEEE EQUATORIAL        | CEEE; CEEE-D; CEEE Distribuicao; Equatorial RS; EQUATORIAL ENERGIA RIO GRANDE DO SUL |
| CEGERO                 | Cooperativa de Eletricidade de Sao Ludgero                                           |
| CEJAMA                 | Cooperativa de Eletricidade Jacinto Machado                                          |
| CELESC-DIS             | CELESC; CELESC-D; CELESC Distribuicao; Centrais Eletricas de Santa Catarina          |
| CELETRO                | Cooperativa de Eletrificacao Centro Jacui                                            |
| CEMIG-D                | CEMIG; CEMIG Distribuicao; Companhia Energetica de Minas Gerais                      |
| CEMIRIM                | Cooperativa de Eletrificacao e Desenvolvimento da Regiao de Mogi Mirim               |
| CEPRAG                 | Cooperativa de Eletricidade Praia Grande                                             |
| CERACA                 | Cooperativa Distribuidora de Energia Vale do Araca                                   |
| CERAL ANITAPOLIS       | Cooperativa de Distribuicao de Energia Eletrica de Anitapolis                        |
| CERAL ARARUAMA         | Cooperativa de Eletrificacao Rural de Araruama                                       |
| CERAL DIS              | Cooperativa de Distribuicao de Energia Eletrica de Arapoti                           |
| CERBRANORTE            | Cooperativa de Eletrificacao de Braco do Norte                                       |
| CERCI                  | Cooperativa de Eletrificacao Rural Cachoeiras de Macacu - Itaborai                   |
| CERCOS                 | Cooperativa de Eletrificacao e Desenvolvimento Rural Centro Sul de Sergipe           |
| CEREJ                  | Cooperativa Senador Esteves Junior                                                   |
| CERES                  | Cooperativa de Eletrificacao Rural de Resende                                        |
| CERFOX                 | Cooperativa de Distribuicao de Energia Fontoura Xavier                               |
| CERGAL                 | Cooperativa de Eletrificacao Anita Garibaldi                                         |
| CERGAPA                | Cooperativa de Eletricidade Grao Para                                                |
| CERGRAL                | Cooperativa de Eletricidade Gravatal                                                 |
| CERILUZ                | Cooperativa Regional de Energia e Desenvolvimento Ijui                               |
| CERIM                  | Cooperativa de Eletrificacao Rural de Itu-Mairinque                                  |
| CERIPA                 | Cooperativa de Eletrificacao Rural de Itai-Paranapanema-Avare                        |
| CERIS                  | Cooperativa de Eletrificacao da Regiao de Itapecerica da Serra                       |
| CERMC                  | Cooperativa de Eletrificacao e Desenvolvimento da Regiao de Mogi das Cruzes          |
| CERMISSOES             | CERMISSÕES; Cooperativa de Distribuicao e Geracao de Energia das Missoes             |
| CERMOFUL               | Cooperativa Fumacense de Eletricidade; CERMOFUL Energia                              |
| CERNHE                 | Cooperativa de Eletrificacao e Desenvolvimento Rural da Regiao de Novo Horizonte     |
| CERPALO                | Cooperativa de Eletrificacao Rural de Paulo Lopes                                    |
| CERPRO                 | Cooperativa de Eletrificacao Rural da Regiao de Promissao                            |
| CERRP                  | Cooperativa de Eletrificacao e Desenvolvimento da Regiao de Sao Jose do Rio Preto    |
| CERSAD                 | Cooperativa de Distribuicao de Energia Eletrica Salto Donner; Cersad Distribuidora   |
| CERSUL                 | Cooperativa de Distribuicao de Energia Sul Catarinense                               |
| CERTAJA                | CERTAJA Energia; Cooperativa Regional de Energia Taquari Jacui                       |
| CERTEL                 | CERTEL Energia; Cooperativa de Distribuicao de Energia Teutonia                      |
| CERTHIL                | Cooperativa de Energia e Desenvolvimento Rural Entre Rios                            |
| CERTREL                | Cooperativa de Energia Treviso                                                       |
| CERVAM                 | Cooperativa de Energizacao e de Desenvolvimento do Vale do Mogi                      |
| CETRIL                 | Cooperativa de Eletrificacao de Ibiuna e Regiao; CERI                                |
| CHESP                  | Companhia Hidroeletrica Sao Patricio                                                 |
| COCEL                  | Companhia Campolarguense de Energia                                                  |
| CODESAM                | Cooperativa de Distribuicao de Energia Eletrica Santa Maria                          |
| COOPERA                | Cooperativa Pioneira de Eletrificacao                                                |
| COOPERALIANCA          | COOPERALIANÇA; Cooperativa Alianca                                                   |
| COOPERCOCAL            | Cooperativa Energetica Cocal                                                         |
| COOPERLUZ              | Cooperativa Distribuidora de Energia Fronteira Noroeste                              |
| COOPERMILA             | Cooperativa de Eletrificacao Lauro Muller                                            |
| COOPERNORTE            | Cooperativa Regional de Distribuicao de Energia do Litoral Norte                     |
| COOPERSUL              | Cooperativa Regional de Eletrificacao Rural Fronteira Sul                            |
| COOPERZEM              | Cooperativa de Distribuicao de Energia Eletrica Armazem                              |
| COORSEL                | Cooperativa Regional Sul de Eletrificacao Rural                                      |
| COPEL-DIS              | COPEL; COPEL-D; COPEL Distribuicao; Companhia Paranaense de Energia                  |
| COPREL                 | COPREL Energia; Coprel Cooperativa de Energia                                        |
| CPFL PAULISTA          | CPFL; Companhia Paulista de Forca e Luz                                              |
| CPFL PIRATININGA       | Companhia Piratininga de Forca e Luz; PIRATININGA                                    |
| CPFL SANTA CRUZ        | Companhia Luz e Forca Santa Cruz; CLFSC                                              |
| CRELUZ-D               | CRELUZ; CRELUZ Distribuicao; CRELUZ - COOPERATIVA DE DISTRIBUIÇÃO DE ENERGIA         |
| CRERAL                 | CRERAL Cooperativa Regional de Eletrificação Rural do Alto Uruguai                   |
| DCELT                  | DCELT Energia; Distribuidora Catarinense de Energia Elétrica S/A                     |
| DEMEI                  | Departamento Municipal de Energia de Ijui                                            |
| DMED                   | DME Distribuicao; DME; DME-D                                                         |
| EDP ES                 | ESCELSA; EDP Escelsa; EDP Espirito Santo                                             |
| EDP SP                 | BANDEIRANTE; EDP Bandeirante; Bandeirante Energia; EDP Sao Paulo                     |
| EFLJC                  | Empresa Forca e Luz Joao Cesa                                                        |
| EFLUL                  | Empresa Forca e Luz Urussanga                                                        |
| ELETROCAR              | Centrais Eletricas de Carazinho                                                      |
| ELFSM                  | Empresa Luz e Forca Santa Maria                                                      |
| ENEL CE                | COELCE; Companhia Energetica do Ceara; Enel Distribuicao Ceara                       |
| ENEL GO                | Enel Distribuicao Goias                                                              |
| ENEL RJ                | AMPLA; CERJ; Ampla Energia e Servicos; Enel Distribuicao Rio                         |
| ENEL SP                | ELETROPAULO; AES ELETROPAULO; Eletropaulo Metropolitana; Enel Distribuicao Sao Paulo |
| ENERGISA AC            | EAC; ELETROACRE; Empresa de Eletricidade do Acre                                     |
| ENERGISA BORBOREMA     | EBO; CELB; Companhia Energetica da Borborema                                         |
| ENERGISA MG            | EMG; CATAGUASES; CFLCL; Companhia Forca e Luz Cataguazes-Leopoldina                  |
| ENERGISA MS            | EMS; ENERSUL; Empresa Energetica de Mato Grosso do Sul                               |
| ENERGISA MT            | EMT; CEMAT; Centrais Eletricas Matogrossenses                                        |
| ENERGISA NOVA FRIBURGO | ENF; CENF; Companhia de Eletricidade de Nova Friburgo                                |
| ENERGISA PB            | EPB; SAELPA; Sociedade Anonima de Eletrificacao da Paraiba                           |
| ENERGISA RO            | ERO; CERON; Centrais Eletricas de Rondonia                                           |
| ENERGISA SE            | ESE; ENERGIPE; Empresa Energetica de Sergipe                                         |
| ENERGISA SUL SUDESTE   | ESS; ENERGISA SUL-SUDESTE                                                            |
| ENERGISA TO            | ETO; CELTINS; Companhia de Energia Eletrica do Estado do Tocantins                   |
| EQUATORIAL AL          | EAL; CEAL; Companhia Energetica de Alagoas                                           |
| EQUATORIAL GO          | EGO; CELG-D; CELG Distribuicao; Companhia Energetica de Goias                        |
| EQUATORIAL MA          | EMA; CEMAR; Companhia Energetica do Maranhao                                         |
| EQUATORIAL PA          | EPA; CELPA; Centrais Eletricas do Para                                               |
| EQUATORIAL PI          | EPI; CEPISA; Companhia Energetica do Piaui                                           |
| FORCEL                 | Forca e Luz Coronel Vivida                                                           |
| HIDROPAN               | Hidroeletrica Panambi                                                                |
| LIGHT                  | LIGHT S.A.; Light Servicos de Eletricidade                                           |
| MUXENERGIA             | MUX Energia; MUX                                                                     |
| NEOENERGIA BRASILIA    | CEB-DIS; CEB-D; CEB; CEB Distribuicao; Companhia Energetica de Brasilia              |
| NEOENERGIA COELBA      | COELBA; Companhia de Eletricidade do Estado da Bahia                                 |
| NEOENERGIA COSERN      | COSERN; Companhia Energetica do Rio Grande do Norte                                  |
| NEOENERGIA ELEKTRO     | ELEKTRO; Elektro Redes; Elektro Eletricidade e Servicos                              |
| NEOENERGIA PERNAMBUCO  | CELPE; Companhia Energetica de Pernambuco; NEOENERGIA PE                             |
| NOVA PALMA             | Nova Palma Energia                                                                   |
| RGE                    | RGE SUL; Rio Grande Energia; CPFL RGE; AES SUL                                       |
| RORAIMA ENERGIA        | CERR; Boa Vista Energia; Companhia Energetica de Roraima                             |
| SULGIPE                | Companhia Sul Sergipana de Eletricidade                                              |

### USER STORY

Como usuário, quero poder gerar um orçamento de energia solar. Quero que o sistema leia a fatura de energia e popule os campos do formulário para eu ganhar tempo e facilidade na geração do orçamento.

1. O usuário deve poder fazer upload de uma fatura de energia.
2. O sistema deve ler a fatura de energia e extrair os valores dos campos obrigatórios.
3. O sistema deve se esforçar para extrair os valores dos campos opcionais, mas não deve bloquear o sucesso da leitura da fatura de energia.
4. Se o sistema não conseguir extrair os valores dos campos obrigatórios, deve ser considerado que houve uma falha na leitura da fatura de energia.
5. Se o sistema conseguir extrair os valores dos campos obrigatórios, deve ser considerado que houve sucesso na leitura da fatura de energia.
6. Em caso de sucesso, o sistema deve mostrar os valores extraídos de forma clara e organizada para o usuário visualizar e editar caso necessário (validação), antes de popular os campos extraídos no formulário de geração de orçamento de energia solar.
7. Se algum campo não obrigatório não for extraído, deve ser exibido um feedback mostrando quais campos o agente não encontrou na fatura de energia.
8. Uma vez que o usuário tenha validado os valores extraídos, o sistema deve popular o formulário de geração de orçamento de energia solar com os valores extraídos.
9. O usuário poderá prosseguir com a geração do orçamento de energia solar OU adição de novas unidades consumidoras.

Obs: 1 Fatura de energia = 1 unidade consumidora.

### UI

Para satisfazer o user story, deve-se criar uma interface de usuário que permita o upload de uma fatura de energia e a extração dos valores dos campos obrigatórios.

1. Um botão com a logo do Nexus + texto "Leitura Smart" será adicionado no form da etapa 1 do wizard de geração de proposta de energia solar, antes da seção "Modalidade Tarifária"
2. Ao clicar no botão, uma modal de upload de fatura de energia será exibida (botão de upload + zona de drop de arquivo)
3. A validação será feita por meio de um pop-up, que mostrará os campos extraídos e permitirá ao usuário validar os valores. O pop-up também mostrará os campos que não foram extraídos (não encontrados na fatura de energia).
4. No backend, os valores extraídos ser validados de acordo com as regras de negócio definidas ("Valores" das tabelas mostram regras de validação).
5. No backend do Azume CRM, a URL da conta de energia deverá ser armazenada na coleção de dados "archive" e na coleção de dados "proposal".
6. Para persistir a URL da conta de energia na coleção "archive", o sistema deverá localizar um folder denominado "Faturas de Energia". Se não for localizado, deverá ser criado um novo folder com o nome "Faturas de Energia".
7. Fluxo de múltiplas unidades consumidoras: Após a validação e preenchimento dos campos da primeira fatura, o usuário pode clicar em "Adicionar unidade consumidora" para abrir novamente a modal de upload. Os dados da unidade consumidora anterior são preservados no formulário. Cada nova fatura corresponde a uma nova unidade consumidora (1 fatura = 1 unidade consumidora).

### REPOSITÓRIOS

Os repositórios abaixo serão utilizados para o desenvolvimento da feature:

- [backend-azume-crm](/home/paulo/projects/azume/azume-backend)
- [frontend-azume-crm](/home/paulo/projects/azume/azume-frontend-crm)
- [backend-nexus](/home/paulo/projects/nexus/nexus-core)

### NETWORK WORKFLOW

1. Usuário no frontend do Azume CRM faz o upload da fatura de energia.
2. O backend do Azume CRM recebe a requisição, valida os dados e faz uma requisição para o backend do Nexus, onde os valores serão extraídos da fatura de energia.
3. Nexus faz o upload da fatura para o Cloud Storage (armazenamento temporário para processamento pelo agente de extração).
4. Se o arquivo for PDF, Nexus renderiza as páginas como imagens (PyMuPdf). Se o arquivo for uma imagem (`.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`), é enviado diretamente ao agente sem etapa de renderização. Em seguida, cria um agente de tarefa com Structured Output (OpenAI Agents SDK) → executa `agent.run()` com as imagens como input → valida os valores extraídos contra as regras de negócio → aplica pós-processamento (preencher meses faltantes com média, Grupo B) → retorna o resultado estruturado.
5. Nexus valida os valores extraídos e retorna os valores extraídos para o backend do Azume CRM.
6. O backend do Azume CRM valida os valores extraídos e retorna para o frontend do Azume CRM. Após validados os dados, o backend do Azume CRM faz o upload do arquivo da fatura de energia para o S3 do Azume CRM (armazenamento permanente vinculado ao arquivo do cliente) e persiste a URL do arquivo no banco de dados do Azume CRM (coleções "archive" e "proposal"). Para "proposal" deverá ser definido um novo campo para armazenar a URL do arquivo da fatura de energia.

Obs sobre duplo armazenamento: O arquivo é armazenado em dois locais distintos com propósitos diferentes. O Cloud Storage do Nexus (passo 3) é armazenamento temporário usado durante o processamento de extração. O S3 do Azume CRM (passo 6) é armazenamento permanente vinculado ao arquivo do cliente. O arquivo no Cloud Storage do Nexus pode ser limpo após o processamento.
7. O frontend do Azume CRM exibe os valores extraídos para o usuário visualizar e editar caso necessário (validação), antes de popular os campos extraídos no formulário de geração de orçamento de energia solar.
8. O usuário poderá prosseguir com a geração do orçamento de energia solar OU adição de novas unidades consumidoras.

Workflow completo: [FRONTEND_AZUME_CRM] -> [BACKEND_AZUME_CRM] -> [BACKEND_NEXUS] -> [CLOUD_STORAGE] -> [LLM] -> [BACKEND_NEXUS] -> [BACKEND_AZUME_CRM] -> [FRONTEND_AZUME_CRM]

### DECISÕES TÉCNICAS

#### Abordagem: LLM Vision + Structured Output

- Usar modelos de LLM com visão para ler a fatura (PDF renderizado como imagem)
- Usar Structured Output (`output_type` do OpenAI Agents SDK) com Pydantic models para forçar o LLM a retornar os campos exatos da spec
- Agente de **tarefa** (não chat) — execução síncrona via `agent.run()`, sem streaming
- Single-step: o LLM classifica (Grupo A/B, Verde/Azul) e extrai valores na mesma chamada
- O PDF será renderizado em imagens (PyMuPdf, já existe no Nexus) e enviado como input visual ao agent

#### Onde no Nexus

- Toda a inteligência mora no **nexus packages layer** (`packages/nexus/`), não no domain layer
- Pacotes envolvidos:
  - `nexus-models`: Pydantic models de extração (schemas de output)
  - `nexus-services`: Serviço de extração + validação + pós-processamento
  - `nexus-ai`: Agent factory e configuração do agente especializado
  - `nexus-contexts`: System prompt do agente de extração

#### Sem tools de cálculo para o agente

- O agente NÃO terá tools de calculadora
- O agente fica 100% focado na extração de valores
- O preenchimento de meses faltantes (Grupo B) será feito por código de regra de negócio no `nexus-services`, após a extração:
  - Validar quais meses foram retornados
  - Calcular a média dos meses presentes
  - Preencher os meses faltantes com a média

#### Requisito de custo: Mid Tier

- Usar exclusivamente modelos de **Mid Tier** (modelos de High Tier adicionam custo excessivo)
- Modelos Mid Tier com suporte a visão disponíveis no Nexus:
  - GPT-5 Mini (`gpt-5-mini`)
  - Gemini 3 Flash (`gemini-3-flash-preview`)
  - Claude Sonnet 4.5 (`claude-sonnet-4-5`)
  - Grok 4.1 Fast (`grok-4-1-fast`)
- O modelo deve ser configurável (Nexus já suporta multi-provider)
- Testar com as amostras existentes para validar acurácia por modelo

#### Generalização sem samples de todas as concessionárias

- LLM Vision generaliza por design — não precisa de um template por concessionária
- System prompt robusto com instruções detalhadas de onde localizar cada valor
- A tabela de concessionárias com aliases já serve como referência para o agente
- Validação de negócio (ranges das tabelas) funciona como rede de segurança
- Se valores obrigatórios não passarem na validação, a extração falha graciosamente
- Refinamento iterativo do prompt com base em erros reais de produção