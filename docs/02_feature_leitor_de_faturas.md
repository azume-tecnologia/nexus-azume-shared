# NEXUS

## LEITOR DE FATURAS DE ENERGIA

### IDEIA

O leitor de faturas de faturas de energia será uma feature powered by LLMs que possibilitará aos usuaŕios do Azume CRM + Nexus enviar um arquivo de fatura de energia e receber todos os dados da fatura extraídos, a fim de de facilitar e tornar mais eficiente a geração de um orçamento de energia solar.

### AMOSTRAS DE FATURAS DE ENERGIA

Abaixo algumas amostras de faturas de energia

**Grupo B:**

1 - samples/faturas-de-energia/fatura-bulbe-grupo-b.pdf
2 - samples/faturas-de-energia/fatura-CERRP-grupo-b.pdf
3 - samples/faturas-de-energia/fatura-CPFL-grupo-b.pdf
4 - samples/faturas-de-energia/fatura-enel-grupo-b.pdf

**Grupo A:**

1 - samples/faturas-de-energia/fatura-copel-grupo-a.pdf
2 - samples/faturas-de-energia/fatura-enel-grupo-a.pdf
3 - samples/faturas-de-energia/fatura-equatorial-grupo-a.pdf

### REQUISITOS TÈCNICOS

- Formatos suportados para a v1.0.0: `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.pdf`.

### EXTRAÇÂO DE VALORES

**Modalidade Tafifária:**

Primeiramente, deve se entender qual a modalidade tafifária da conta de energia (Grupo B ou Grupo A). Isso irá determinar quais valores devem ser extraídos da conta de energia.

Valores:

| Nome                 | Tipo           | Valores    | Campo Backend Azume CRM | Campo Frontend Azume CRM | Opcional | Unidade |
| -------------------- | -------------- | ---------- | ----------------------- | ------------------------ | -------- | ------- |
| Modalidade Tarifária | Enum de string | "A" ou "B" | tariffModality          | tariffModality           | Não      | -       |

**Grupo B (baixa tensão):**

Para contas do grupo B, devem ser extraídos os valores abaixo:

| Nome               | Tipo                | Valores                                 | Campo Backend Azume CRM | Campo Frontend Azume CRM | Obrigatório | Unidade |
| ------------------ | ------------------- | --------------------------------------- | ----------------------- | ------------------------ | ----------- | ------- |
| Concessionária     | Enum de string      | -                                       | powerDistCompany        | powerDistCompany         | Sim         | -       |
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

Obs1: nem todas as contas possuem o consumo dos últimos 12 meses, algumas só exivem 8 meses ou somente 6 meses... Neste caso, os meses que estão faltando devem ser preenchidos com a média de consumo dos meses que estão presentes na fatura de energia
Obs2: o valor de kwhPrice já deve incluir todos tributos/impostos
Obs3: no grupo B, os valores de tusd e icms não são essenciais para que o usuário prossiga com a geração do orçamento de energia solar, porém, são importantes para que o usuário possa gerar uma proposta mais assertiva. Portanto, deve-se aplicar uma política "maior esforço" para extração de tusd e icms, porém sem bloquear o critério de sucesso da leitura da fatura de energia.Obs4: Mesma coisa que "Obs3" para taxa de iluminação pública.

**Critérios de sucesso da leitura Grupo B**:

- Extrair todos os campos obrigátórios
- (OPCIONAL) Definir o consumo médio para os meses que estão faltando na fatura de energia
- Todos os meses possuem valor de consumo (provindo da leitura direta ou do cálcula da média)

Caso os critérios de sucesso não sejam alcançado, deve ser considerado que houve uma falha na leitura.

**Grupo A (alta tensão) - Classificação Horo-Sazonal:**

Uma vez que for identificado que a fatura é do grupo A (alta tensão), deve-se identificar a Classificação Horo-Sazonal da fatura. O Azume CRM fornece suporte para dimensionamento das classificações "Azul" e "Verde"

Valores:

| Nome                       | Tipo           | Valores           | Campo Backend Azume CRM | Campo Frontend Azume CRM | Opcional | Unidade |
| -------------------------- | -------------- | ----------------- | ----------------------- | ------------------------ | -------- | ------- |
| Classificação Horo-Sazonal | Enum de string | "Azul" ou "Verde" | classification          | classification           | Não      | -       |

**Grupo A (alta tensão) - Verde:**

| Nome                                                      | Tipo                | Valores        | Campo Backend Azume CRM      | Campo Frontend Azume CRM | Obrigatório | Unidade |
| --------------------------------------------------------- | ------------------- | -------------- | ---------------------------- | ------------------------ | ----------- | ------- |
| Concessionária                                            | Enum de string      | -              | powerDistCompany             | powerDistCompany         | Sim         | -       |
| TE Fora Ponta                                             | number              | > 0 && < 10    | kwhPrice                     | kwhPrice                 | Sim         | R$      |
| TE Ponta                                                  | number              | > 0 && < 10    | kwhPricePeak                 | kwhPricePeak             | Sim         | R$      |
| TUSD Fora Ponta                                           | number              | > 0 && < 10    | tusd                         | tusd                     | Sim         | R$      |
| TUSD Ponta                                                | number              | > 0 && < 10    | tusdPeak                     | tusdPeak                 | Sim         | R$      |
| Demanda Contratada (Fora Ponta para classificação "Azul") | number              | >= 30          | demand                       | demand                   | Sim         | kW      |
| Tarifa da Demanda (Fora pPonta para classificação "Azul") | number              | > 0            | demandTariff                 | demandTariff             | Sim         | R$      |
| Taxa Ilum. Pública                                        | number              | > 0            | publicLightBill              | publicLightBill          | Não         | R$      |
| Consumo Médio Mensal Fora Ponta                           | number              | > 0            | monthlyConsumption[1-12]     | averageValue             | Sim         | kWh     |
| Consumo Médio Mensal Ponta                                | number              | > 0            | monthlyConsumptionPeak[1-12] | averageValuePeak         | Sim         | kWh     |
| icms                                                      | number (percentual) | > 0 && < 100 % | icms                         | icms                     | Não         | %       |

Obs1: Para a classificação "Verde", a demanda contratada não é diferenciada entre ponta e fora ponta, há apenas um valor para demanda contratada.
Obs2: no grupo B, o valor de icms não é essencial para que o usuário prossiga com a geração do orçamento de energia solar, porém, é importante para que o usuário possa gerar uma proposta mais assertiva. Portanto, deve-se aplicar uma política "maior esforço" para extração de icms, porém sem bloquear o critério de sucesso da leitura da fatura de energia.
Obs3: Mesma coisa que "Obs2" para taxa de iluminação pública.

**Grupo A (alta tensão) - Azul:**

Além de todos os valores do Grupo A (alta tensão) - Verde, devem TAMBÉM ser extraídos os valores abaixo para classificação "Azul":

| Nome                     | Tipo   | Valores | Campo Backend Azume CRM | Campo Frontend Azume CRM | Obrigatório | Unidade |
| ------------------------ | ------ | ------- | ----------------------- | ------------------------ | ----------- | ------- |
| Demanda Contratada Ponta | number | > 0     | demandPeak              | demandPeak               | Sim         | kW      |
| Tarifa da Demanda Ponta  | number | > 0     | demandTariffPeak        | demandTariffPeak         | Sim         | R$      |

Obs: Para a classificação "Azul", a demanda contratada é diferenciada entre ponta e fora ponta.

**Critérios de sucesso da leitura Grupo A**:

- Extrair todos os campos obrigátórios

**Definições:**

1. Campo Backend Azume CRM: nome do campo na coleção de dados "proposal" que armazena o valor.
2. Campo Frontend Azume CRM: ID do input do formulário no frontend do Azume que armazena o valor.
3. Obrigatório: Se o campo for obrigatorio e a LLM não conseguir extrair o valor da conta de energia, significa que a leitura da fatura de energia falhou.

### Suport tools para agentes

- Para que o agent (LLM) consiga fornecer valores de maneira mais assertiva, seria interessante fornecer ao agente ferramentas de operações artiméticas (nível de calculadora científica simples)
- Dessa forma, quando necessário, o agente será capaz de executar operações aritméticas para extrair valores da conta. Um exemplo simples é o cálculo
- Deve ser pesquisado a melhor forma de fornecer ao agente essa capacidade (pacote python + @function_tool? MCP server?)
- Deve ser avaliado se é melhor fornecer diretamente ao agente a capacidade de executar python

### CONCESSIONÁRIAS

A lista de valores de concessionárias do Azume será acessada via endpoint GET.
