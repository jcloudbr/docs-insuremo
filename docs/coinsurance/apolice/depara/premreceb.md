# PREMRECEB — De/Para (arquivo físico → persistência)

> **Service:** `CoinsurancePremRecebService.groovy`  > **Entidade:** `CoinsuranceRepository.CnsPremReceb`

## Origem (arquivo físico)

- Separador CSV: `|` (pipe)
- Leitura: arquivo pendente no S3 → parse em mapa de campos
- **Validação Velora:** `bmg_prereceb`

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
| PRESTACAO | InstallmentNumber |  |
| DT_REC_PRE | InstallmentReceiptDate |  |
| DT_VEN_PRE | InstallmentDueDate |  |
| NUM_BAN | BankNumber |  |
| CNPJ_BAN | BankDocumentId |  |
| NUM_BOL | TicketNumber |  |
| NUM_DOC | DocumentNumber |  |
| NUM_AGE | AgencyNumber |  |
| NUM_NOS | OurNumber |  |
| VAL_DOC | DocumentValue |  |
| VAL_DESC | DiscountValue |  |
| VAL_MUL | FineValue |  |
| VAL_COB | ChargedValue |  |
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
