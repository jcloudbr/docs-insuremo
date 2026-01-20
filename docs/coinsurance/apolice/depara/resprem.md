# RESPREM — De/Para (arquivo físico → persistência)

> **Service:** `CoinsuranceResPremService.groovy`  > **Entidade:** `CoinsuranceRepository.CnsResPrem`

## Origem (arquivo físico)

- Separador CSV: `|` (pipe)
- Leitura: arquivo pendente no S3 → parse em mapa de campos
- **Validação Velora:** `bmg_resprem`

## De/Para dos campos (CSV → Entidade)

| Campo no arquivo (CSV) | Campo persistido (Entidade) | Observações |
| --- | --- | --- |
| SEQUENCIA | RecordNo | Nº do registro no arquivo (SEQUENCIA). |
| COD_CIA | CompanyCode |  |
| DT_BASE | BaseDate | Data base (competência). |
| TIPO_MOV | MovementType | Tipo de movimento (ex.: emissão/endosso). |
| COD_RAMO | BranchCode |  |
| NUM_APOL | PolicyNumber |  |
| NUM_END | EndorsementNumber |  |
| NUM_PROP | ProposalNumber |  |
| DT_PROP | ProposalDate |  |
| DT_EMIS | IssueDate |  |
| DT_INI_VIG | EffectiveDate |  |
| DT_FIM_VIG | EndDate |  |
| PR_EMIT | IssuedPremium |  |
| PR_COS_CED | CededCoinsurancePremium |  |
| PR_N_GANHO | UnwonPremiumValue |  |
| COLET | Collective |  |
| DS_GRUPO_RAMO | BranchGroupDescription |  |
| CD_PRODUTO | ProductCode |  |
| DS_PRODUTO | ProductDescription |  |
| CD_BU | BusinessUnitCode |  |
| DS_BU | BusinessUnitDescription |  |
| CD_OPERACAO | OperationCode |  |
| CD_CARTEIRA | WalletCode |  |
| DS_CARTEIRA | WalletDescription |  |
| CD_SUCURSAL | SucursalCode |  |
| FG_GEB | FgGeb |  |
| DCR_TIP_OPER | OperationDescription |  |
| DS_ORIGEM | OriginDescription |  |

## Campos técnicos e derivados

| Campo lógico | Campo persistido | Como é preenchido |
| --- | --- | --- |
| FileId | FileId | Derivado: `${etag}_${RecordNo}` (etag do S3 sem aspas). |
| FileName | FileName | Derivado: nome do arquivo (última parte da key no S3). |
| Status | Status | Default: `PENDING` na ingestão. Em erro, pode virar `ERROR` (há TODOs de update em alguns serviços). |
| Sequence | Sequence | Derivado: PK interna gerada (`PrimaryKeyUtils.getSequenceId()`) no insert. |
