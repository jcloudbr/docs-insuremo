# Documentação Funcional: Geração do Arquivo PREMAC

**Processo:** Exportação Mensal de Prêmios de Cosseguro Aceito (SUSEP)

**Contexto:** Operação BMG Seguridade (Cosseguradora Aceita) & Generali (Seguradora Líder)

**Frequência:** Mensal (Fechamento)

---

## 1. Objetivo do Processo

Este processo tem como objetivo gerar e disponibilizar o arquivo regulatório **PREMAC**. Este arquivo reporta à SUSEP toda a movimentação financeira de prêmios onde a **BMG Seguridade** atua como "Cosseguradora Aceita" (participante), recebendo riscos repassados pela **Generali** (Seguradora Líder).

O sistema deve varrer a base de dados de apólices, identificar as movimentações do mês de referência e calcular a participação da BMG nos valores.

## 2. Critérios de Seleção (Quem entra no arquivo?)

Para que uma apólice ou endosso conste neste arquivo, o sistema aplica os seguintes filtros de negócio:

1. **Empresa:** A apólice deve pertencer à operação da **BMG**.
2. **Seguradora Líder:** A apólice deve ter a **Generali (Código SUSEP 05908)** identificada como a Seguradora Líder.
3. **Competência:** A data de registro da movimentação (Data Base) deve pertencer ao mês e ano selecionados para a execução (ex: Janeiro de 2026).

---

## 3. Regras de Negócio e Cálculos

O sistema processa cada apólice individualmente e aplica as seguintes regras para compor os valores do relatório:

### 3.1. Consolidação de Valores

Uma apólice pode ter diversas coberturas (ex: Morte, Invalidez, Assistência Funeral). O arquivo PREMAC exige o valor total do movimento.

* **Regra:** O sistema soma os valores de todas as coberturas da apólice para chegar ao montante total.

### 3.2. Aplicação da Participação (Cosseguro)

Os valores originais na apólice representam o contrato total (100%) fechado pela Líder.

* **Regra:** O sistema aplica o percentual de **40% (Participação BMG)** sobre os valores da Líder para determinar o montante a ser reportado no arquivo.
* *Prêmio Aceito = Prêmio Líquido da Líder x 40%*
* *Comissão Aceita = Comissão da Líder x 40%*
* *Adicional Aceito = Adicional da Líder x 40%*



### 3.3. Tratamento da Taxa de Custo (COFEE)

O Custo de Apólice (conhecido como COFEE) tem tratamento específico:

* **Valor Monetário:** O sistema captura o valor exato de custo já calculado e gravado na apólice.
* **Percentual:** O sistema reporta a taxa utilizada (ex: 6,8%), priorizando a taxa específica gravada no momento do endosso, se houver.

### 3.4. Regra de Emissão vs. Endosso

Para identificar corretamente o documento na SUSEP:

* **Emissões (Movimento 101):** O número do endosso é preenchido com zeros, pois é uma apólice nova.
* **Demais Movimentos:** O número do endosso original da Líder é mantido.

---

## 4. Estrutura do Arquivo Gerado (Visão Geral)

Com base no código final validado e no arquivo gerado (`PREMAC_202601_20260120_190410.csv`), detalhei a estrutura técnica do arquivo **PREMAC**.

Esta especificação define exatamente como cada coluna é formatada e qual o seu significado.

---

### 4.1 Layout do Arquivo: PREMAC (Prêmios de Cosseguro Aceito)

**Nome do Arquivo:** `PREMAC_{YYYYMM}_{TIMESTAMP}.csv`

**Formato:** Texto (CSV)

**Delimitador:** Pipe (`|`)

**Encoding:** UTF-8

**Separador Decimal:** Ponto (`.`)

**Formato de Data:** `DDMMAAAA`

---

### 4.2 Detalhe das Colunas

| Ordem | Nome da Coluna | Tipo | Formato / Tamanho | Descrição e Regra de Preenchimento |
| --- | --- | --- | --- | --- |
| **01** | **SEQUENCIA** | Numérico | 10 dígitos (com zeros) | Contador sequencial único por linha no arquivo (Ex: `0000000001`). |
| **02** | **COD_CIA** | Texto | 5 caracteres | Código SUSEP da Seguradora (BMG). Valor Fixo: `03417`. |
| **03** | **NUM_PROC** | Texto | Variável | Número do Processo SUSEP do produto (Ex: `15414.901147/2014-38`). Vem da apólice líder. |
| **04** | **DT_BASE** | Data | AAAAMM | Ano e Mês da competência contábil (Ex: `202601`). |
| **05** | **TIPO_MOV** | Numérico | 3 dígitos | Código do tipo de movimento (Ex: `101` para Emissão, `102` para Endosso). |
| **06** | **UF_DEP** | Texto | 2 caracteres | Sigla da UF do Departamento/Emissão (Ex: `SP`). |
| **07** | **UF_RISCO** | Texto | 2 caracteres | Sigla da UF de localização do Risco (Ex: `AM`). |
| **08** | **COD_RAMO** | Texto | 4 caracteres | Código do Ramo SUSEP (Ex: `0993`). |
| **09** | **NUM_APOL** | Texto | Variável | Número da Apólice na Seguradora Líder. |
| **10** | **NUM_END** | Texto | Variável | Número do Endosso na Líder.<br><br>⚠️ **Regra:** Se `TIPO_MOV` for `101`, preenche com 20 zeros. Caso contrário, usa o número real.|
| **11** | **COD_COSS** | Texto | 5 caracteres | Código SUSEP da Seguradora Líder (Generali). Valor Fixo: `05908`. |
| **12** | **DT_EMIS** | Data | DDMMAAAA | Data de Emissão da Apólice/Endosso (Ex: `20012026`). |
| **13** | **DT_INI_VIG** | Data | DDMMAAAA | Data de Início de Vigência (Ex: `02012026`). |
| **14** | **DT_FIM_VIG** | Data | DDMMAAAA | Data de Fim de Vigência (Ex: `02122026`). |
| **15** | **PR_COSS_AC** | Valor | 0.00 | **Prêmio de Cosseguro Aceito.**<br><br>Valor da participação da BMG. Soma do campo `PrCossAc` de todas as coberturas. |
| **16** | **COM_COSS_AC** | Valor | 0.00 | **Comissão de Cosseguro Aceito.**<br><br>Valor da comissão sobre a parte aceita. Soma do campo `ComCosAc`. |
| **17** | **PR_EMIT** | Valor | 0.00 | **Prêmio Emitido Total (Líder).**<br><br>Valor total líquido da apólice na líder (100%), sem impostos. |
| **18** | **AD_FRAC** | Valor | 0.00 | **Adicional de Fracionamento Aceito.**<br><br>Soma do campo `AdFracCoss`. |
| **19** | **COFEE** | Valor | 0.00 | **Custo de Apólice (Aceito).**<br><br>Valor monetário do custo de emissão (taxa Cofee) proporcional à BMG. |
| **20** | **PRO_LAB** | Valor | 0.00 | **Pró-Labore.**<br><br>Valor de administração. Soma do campo `LaborAmountCoss`. |
| **21** | **CD_PRODUTO** | Texto | Variável | Código interno do produto no sistema (Ex: `93911`). |
| **22** | **DS_PRODUTO** | Texto | Variável | Descrição/Nome comercial do produto. |
| **23** | **NUM_PROP** | Texto | Variável | Número da Proposta na Seguradora Líder. |
| **24** | **DT_PROP** | Data | DDMMAAAA | Data da Proposta na Líder. |
| **25** | **CPF_SEG** | Texto | Variável | CPF ou CNPJ do Segurado principal. |
| **26** | **PRC_COMISS_COFEE** | Valor | 0.00#### | Percentual utilizado para cálculo do COFEE (Ex: `0.068` para 6.8%). |

---

## Exemplo de Linha (Raw Data)

```csv
SEQUENCIA|COD_CIA|NUM_PROC|DT_BASE|TIPO_MOV|UF_DEP|UF_RISCO|COD_RAMO|NUM_APOL|NUM_END|COD_COSS|DT_EMIS|DT_INI_VIG|DT_FIM_VIG|PR_COSS_AC|COM_COSS_AC|PR_EMIT|AD_FRAC|COFEE|PRO_LAB|CD_PRODUTO|DS_PRODUTO|NUM_PROP|DT_PROP|CPF_SEG|PRC_COMISS_COFEE
0000000001|03417|15414.901147/2014-38|202601|101|SP|AM|0993|789322900007005|00000000000000000000|05908|20012026|02012026|02122026|0.34|0.00|0.85|0.00|0.02|0.00|93911|BMG 2.0 Vida Cartão Consignado INSS|20260120|01012026|12345678912|0.068

```

## Regras de Negócio Aplicadas na Geração

1. **Soma de Coberturas:** Os valores monetários (colunas 15 a 20) são resultantes da soma de todas as coberturas da apólice. O arquivo não quebra por cobertura, apenas por Apólice/Movimento.
2. **Valores Líquidos:** O campo `PR_EMIT` refere-se ao **Prêmio Líquido** (`BeforeVatPremium`), ou seja, desconta-se o IOF/Impostos.
3. **Origem dos Dados:**
* Campos com sufixo `_AC` ou `Coss` vêm dos campos calculados de Cosseguro Aceito.
* Campos como `PR_EMIT` vêm dos valores originais (100%) da Líder.


4. **Chave Única:** A chave única lógica de negócio de cada linha é composta por `COD_RAMO + NUM_APOL + NUM_END`.

## 5. Resultados Esperados

Ao final da execução do processo mensal:

1. **Sucesso:** Um arquivo nomeado como `PREMAC_AAAAMM_DataHora.csv` será depositado na área de exportação para envio à SUSEP/Contabilidade.
2. **Exceções:** Caso alguma apólice tenha dados incompletos (ex: falta de cadastro do Líder), ela não será incluída no arquivo final e um relatório de erros separado será gerado para análise da equipe operacional.