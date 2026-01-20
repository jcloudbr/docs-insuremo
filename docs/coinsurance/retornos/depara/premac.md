# PREMAC — De/Para (arquivo físico → persistência)

> **Service:** `CoinsurancePremacService.groovy`  > **Entidade:** `CoinsuranceRepository.CnsPremAc`

## Origem (arquivo físico)

- Separador CSV: `|` (pipe)
- Leitura: arquivo pendente no S3 → parse em mapa de campos
- **Validação Velora:** `bmg_premac`

## De/Para dos campos (CSV → Entidade)

| Campo no arquivo (CSV) | Campo persistido (Entidade) | Observações |
| --- | --- | --- |
| RecordNo | SEQUENCIA |  |
| CompanyCode | COD_CIA |  |
| Protocol | NUM_PROC |  |
| BaseDate | DT_BASE |  |
| MovementType | TIPO_MOV |  |
| IssuingState | UF_DEP |  |
| RiskStates | UF_RISCO |  |
| BranchCode | COD_RAMO |  |
| PolicyNumber | NUM_APOL |  |
| EndorsementNumber | NUM_END |  |
| InsuredQuantity | QTD_SEG |  |
| BorrowerDocumentId | CPF_TOM |  |
| BorrowersQuantity | QTD_TOM |  |
| IssueDate | DT_EMIS |  |
| EffectiveDate | DT_INI_VIG |  |
| EndDate | DT_FIM_VIG |  |
| ProposalNumber | NUM_PROP |  |
| ProposalDate | DT_PROP |  |
| InsuredDocumentId | CPF_SEG |  |
| IssuedPremium | PR_EMIT |  |
| CededCoinsurancePremium | PR_COS_CED |  |
| FractionalSurchargeValue | AD_FRAC |  |
| PolicyCostValue | CUST_APOL |  |
| TaxAmount | IOF |  |
| BrokerageCommissionAmount | COMIS |  |
| CoinsuranceCommissionAmount | COMIS_COSS |  |
| ProLaboreValue | PRO_LAB |  |
| StipulatorDocumentId | CPF_ESTIP |  |
| IsValue | IS |  |
| BranchGroupDescription | DS_GRUPO_RAMO |  |
| ProductCode | CD_PRODUTO |  |
| ProductDescription | DS_PRODUTO |  |
| BusinessUnitCode | CD_BU |  |
| BusinessUnitDescription | DS_BU |  |
| OperationCode | CD_OPERACAO |  |
| WalletCode | CD_CARTEIRA |  |
| WalletDescription | DS_CARTEIRA |  |
| SucursalCode | CD_SUCURSAL |  |
| FgGeb | FG_GEB |  |
| OperationDescription | DCR_TIP_OPER |  |
| OriginDescription | DS_ORIGEM |  |

## Campos técnicos e derivados

| Campo lógico | Campo persistido | Como é preenchido |
| --- | --- | --- |
| FileId | FileId | Derivado: `${etag}_${RecordNo}` (etag do S3 sem aspas). |
| FileName | FileName | Derivado: nome do arquivo (última parte da key no S3). |
| Status | Status | Default: `PENDING` na ingestão. Em erro, pode virar `ERROR` (há TODOs de update em alguns serviços). |
| Sequence | Sequence | Derivado: PK interna gerada (`PrimaryKeyUtils.getSequenceId()`) no insert. |
