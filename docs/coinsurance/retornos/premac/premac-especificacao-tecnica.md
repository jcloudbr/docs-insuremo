# Especificação Técnica: Geração Arquivo PREMAC

**Módulo:** Coinsurance (Cosseguro Aceito)

**Entidade Origem:** `Policy` (Plataforma InsureMO)

**Formato de Saída:** CSV (`|` delimitado)

**Frequência:** Mensal (Fechamento Contábil)

---

## 1. Objetivo

Gerar o arquivo regulatório contendo os dados de **Prêmios de Cosseguro Aceito** para a operação onde a **BMG (03417)** atua como Cosseguradora Aceita e a **Generali (05908)** atua como Seguradora Líder.

O arquivo consolida as movimentações (Emissões, Endossos, Cancelamentos) ocorridas na competência, baseando-se nos dados processados e persistidos na entidade definitiva de Apólice.

## 2. Estratégia de Extração e Performance

### 2.1 Fonte de Dados

A extração não utiliza tabelas temporárias. Os dados são consumidos diretamente das APIs definitivas:

* **Busca (Search):** `ProposalSdkClient` (Índice `Policy`).
* **Detalhamento (Load):** `PolicySdkClient` (Carregamento do `Policy Core` completo).

### 2.2 Paginação (Keyset Pagination)

Devido ao alto volume de dados, a extração utiliza paginação por cursor para evitar limites de *offset* do Elasticsearch (máximo 10.000 registros):

* **Cursor:** Campo `entity_id` (Numérico sequencial único).
* **Lógica:** Busca lotes de 1.000 registros onde `entity_id > ultimo_cursor_processado`.

### 2.3 Critérios de Seleção (Filtros)

Para compor o arquivo, são selecionados registros que atendam a **todas** as condições:

1. **Tenant:** `FullOrgCode` = `bmg`.
2. **Líder:** `CodCoss` = `05908` (Generali).
3. **Competência:** `BaseDate` = Data inicial do mês de referência (formato `YYYY-MM-01`).
* *Exemplo:* Para competência `202601`, filtra-se por `2026-01-01`.



---

## 3. Mapeamento de Campos (De-Para)

Abaixo, a relação entre as colunas do arquivo CSV e os campos no JSON da entidade `Policy`.

### 3.1 Cabeçalho e Identificação

| Coluna CSV | Origem (JSON Policy) | Regra de Negócio / Transformação |
| --- | --- | --- |
| **SEQUENCIA** | *Gerado* | Contador sequencial incremental iniciando em 1. Formatado em 10 dígitos com zeros à esquerda. |
| **COD_CIA** | *Fixo* | Valor fixo: `03417` (BMG). |
| **NUM_PROC** | `NumProcLider` | Número do Processo SUSEP. Se vazio na raiz, busca no objeto da Líder em `CoInsuranceInsurerList`. |
| **DT_BASE** | *Parâmetro* | Ano e Mês da competência (ex: `202601`). |
| **TIPO_MOV** | `TipoMov` | Tipo de movimento (ex: 101, 102). Formatado com 3 dígitos. |
| **UF_DEP** | `UFDep` | UF do Departamento/Emissão. |
| **UF_RISCO** | `UFRisco` | UF do Risco. |
| **COD_RAMO** | `CodRamoProduto` | Código do Ramo SUSEP. |
| **NUM_APOL** | `CoInsuranceInfoList[0].LeaderPolicyNo` | Número da Apólice na Seguradora Líder. |
| **NUM_END** | `CoInsuranceInfoList[0].LeaderEndorsementNo` | Número do Endosso na Líder.<br><br>**Regra:** Se `TIPO_MOV == 101`, preencher com 20 zeros (`000...`). Caso contrário, usar o valor original. |
| **COD_COSS** | *Fixo* | Valor fixo: `05908` (Generali). |
| **DT_EMIS** | `IssueDate` | Data de Emissão. Formato `DDMMYYYY`. |
| **DT_INI_VIG** | `EffectiveDate` | Início de Vigência. Formato `DDMMYYYY`. |
| **DT_FIM_VIG** | `ExpiryDate` | Fim de Vigência. Formato `DDMMYYYY`. |

### 3.2 Valores Financeiros

Os valores devem ser consolidados por movimento. O sistema varre toda a estrutura de `PolicyLobList -> PolicyRiskList -> PolicyCoverageList` e **soma** os valores encontrados nas coberturas.

| Coluna CSV | Origem (JSON Policy Coverage) | Descrição |
| --- | --- | --- |
| **PR_COSS_AC** | Soma de `PrCossAc` | Prêmio de Cosseguro Aceito (Valor já calculado na persistência). |
| **COM_COSS_AC** | Soma de `ComCosAc` | Comissão de Cosseguro Aceito. |
| **PR_EMIT** | `BeforeVatPremium` (Raiz) | Prêmio Líquido Total (Líder) sem impostos. |
| **AD_FRAC** | Soma de `AdFracCoss` | Adicional de Fracionamento (Parcela Aceita). |
| **COFEE** | Soma de `Cofee` | Valor monetário da taxa de Cofee (Custo de apólice). |
| **PRO_LAB** | Soma de `LaborAmountCoss` | Valor de Pró-Labore (Parcela Aceita). |

*Nota: Formatação numérica utilizada: `0.00` (Ponto como separador decimal).*

### 3.3 Identificação Complementar (Campos Novos)

| Coluna CSV | Origem (JSON Policy) | Regra de Negócio |
| --- | --- | --- |
| **CD_PRODUTO** | `CdProduto` | Código interno do produto. |
| **DS_PRODUTO** | `DsProduto` | Descrição do produto. |
| **NUM_PROP** | `CoInsuranceInfoList[0].LeaderProposalNo` | Número da proposta na Líder. |
| **DT_PROP** | `CoInsuranceInfoList[0].LeaderProposalDate` | Data da proposta na Líder. Formato `DDMMYYYY`. |
| **CPF_SEG** | `PolicyCustomerList` | Busca o `IdNo` (CPF/CNPJ) do cliente onde `IsInsured` for `Y` ou `true`. |
| **PRC_COMISS_COFEE** | `CoInsuranceInfoList[0].PrcComissCofee` | Percentual de comissão de Cofee.<br>

<br>**Prioridade:** Tenta `PrcComissCofee102`, se nulo usa `PrcComissCofee`. Formata com até 4 casas decimais. |

---

## 4. Regras de Negócio Implementadas

1. **Agregação por Movimento:** O arquivo gera **uma linha por `PolicyId**`. Não há quebra por cobertura. Todos os valores financeiros das coberturas são somados para compor o total do movimento.
2. **Cálculos Pré-Existentes:** O script **não recalcula** percentuais (como 40% de participação ou 6.8% de Cofee). Ele confia que os valores persistidos no JSON da Apólice (`PrCossAc`, `Cofee`, etc.) já foram calculados corretamente pelo motor de cálculo no momento da emissão/endosso.
3. **Tratamento de Endosso de Emissão (101):** Para movimentos do tipo 101, o campo `NUM_END` é forçado para uma string de zeros, conforme padrão de arquivos regulatórios que não esperam número de endosso na emissão original.
4. **Sanitização:** Todos os campos de texto têm quebras de linha (`\n`, `\r`) e o caractere delimitador (`|`) removidos para não quebrar o layout do CSV.

## 5. Especificação do Arquivo Físico

* **Nome do Arquivo:** `PREMAC_{YYYYMM}_{timestamp}.csv`
* Exemplo: `PREMAC_202601_20260120_143000.csv`


* **Encoding:** UTF-8
* **Delimitador:** Pipe (`|`)
* **Destino:** Bucket S3, pasta `export`.
* **Log de Erros:** Caso ocorra falha no processamento de linhas específicas, um arquivo `LOG_PREMAC_{YYYYMM}_{timestamp}.csv` é gerado na pasta `error` contendo o ID da apólice e a mensagem de erro.