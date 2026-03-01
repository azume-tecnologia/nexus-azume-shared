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

### EXTRAÇÂO

**Modalidade Tafifária:**

Primeiramente, deve se entender qual a modalidade tafifária da conta de energia (Grupo B ou Grupo A). Isso irá determinar quais valores devem ser extraídos da conta de energia.

Valores:

| Nome | Tipo | Valores | Campo Azume CRM | Opcional |
|------|------|---------|-----------------|----------|
| Modalidade Tarifária | Enum de string | "A" ou "B" | tariffModality | Não |

**Grupo B:**

Para contas do grupo B, devem ser extraídos os valores abaixo:

| Nome | Tipo | Valores | Campo Azume CRM | Opcional |
|------|------|---------|-----------------|----------|
| Concessionária | Enum de string | - | tariffModality | Não |
| Valor kWh | number | > 0 && < 10 | kwhPrice | Não |
| Taxa Ilum. Pública | number | > 0 | publicLightBill | Sim |
| Rede | Enum de string | "Trifásica", "Bifásica" ou "Monofásica" | publicLightBill | Não |
| Consumo Jan | number | >= 0 | monthlyConsumption[0] | Sim |
| Consumo Fev | number | >= 0 | monthlyConsumption[1] | Sim |
| Consumo Mar | number | >= 0 | monthlyConsumption[2] | Sim |
| Consumo Abr | number | >= 0 | monthlyConsumption[3] | Sim |
| Consumo Mai | number | >= 0 | monthlyConsumption[4] | Sim |
| Consumo Jun | number | >= 0 | monthlyConsumption[5] | Sim |
| Consumo Jul | number | >= 0 | monthlyConsumption[6] | Sim |
| Consumo Ago | number | >= 0 | monthlyConsumption[7] | Sim |
| Consumo Set | number | >= 0 | monthlyConsumption[8] | Sim |
| Consumo Out | number | >= 0 | monthlyConsumption[9] | Sim |
| Consumo Nov | number | >= 0 | monthlyConsumption[10] | Sim |
| Consumo Dez | number | >= 0 | monthlyConsumption[11] | Sim |

Obs: nem todas as contas possuem o consumo dod últimos 12 meses, algumas só exivem 8 meses ou somente 6 meses... Neste caso, os meses que estão faltando devem ser preenchidos com a média de consumo dos meses que estão presentes na fatura de energia

### CONCESSIONÁRIAS

A lista de valores de concessionárias do Azume será acessada via endpoint GET.