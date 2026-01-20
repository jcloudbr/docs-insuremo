# PREMRECEBC — De/Para (arquivo físico → persistência)

> **Service:** `CoinsurancePremRecebcService.groovy`  > **Entidade:** `CoinsuranceRepository.CnsPremRecebC`

## Origem (arquivo físico)

- Separador CSV: `|` (pipe)
- Leitura: arquivo pendente no S3 → parse em mapa de campos
- **Validação Velora:** `bmg_premrecebc`

## De/Para dos campos (CSV → Entidade)

| Campo no arquivo (CSV) | Campo persistido (Entidade) | Observações |
| --- | --- | --- |
| RecordNo | RecordNo | Nº do registro no arquivo (SEQUENCIA). |
| CompanyCode | CompanyCode |  |
| BaseDate | BaseDate | Data base (competência). |
| MovementType | MovementType | Tipo de movimento (ex.: emissão/endosso). |
| BranchCode | BranchCode |  |
| PolicyNumber | PolicyNumber |  |
| EndorsementNumber | EndorsementNumber |  |
| ProposalNumber | ProposalNumber |  |
| ProposalDate | ProposalDate |  |
| InstallmentNumber | InstallmentNumber |  |
| InstallmentReceiptDate | InstallmentReceiptDate |  |
| InstallmentDueDate | InstallmentDueDate |  |
| BankNumber | BankNumber |  |
| BankDocumentId | BankDocumentId |  |
| TicketNumber | TicketNumber |  |
| DocumentNumber | DocumentNumber |  |
| AgencyNumber | AgencyNumber |  |
| OurNumber | OurNumber |  |
| DocumentValue | DocumentValue |  |
| DiscountValue | DiscountValue |  |
| FineValue | FineValue |  |
| ChargedValue | ChargedValue |  |
| BranchGroupDescription | BranchGroupDescription |  |
| ProductCode | ProductCode |  |
| ProductDescription | ProductDescription |  |
| BusinessUnitCode | BusinessUnitCode |  |
| BusinessUnitDescription | BusinessUnitDescription |  |
| OperationCode | OperationCode |  |
| WalletCode | WalletCode |  |
| WalletDescription | WalletDescription |  |
| SucursalCode | SucursalCode |  |
| FgGeb | FgGeb |  |
| OperationDescription | OperationDescription |  |
| OriginDescription | OriginDescription |  |

## Campos técnicos e derivados

| Campo lógico | Campo persistido | Como é preenchido |
| --- | --- | --- |
| FileId | FileId | Derivado: `${etag}_${RecordNo}` (etag do S3 sem aspas). |
| FileName | FileName | Derivado: nome do arquivo (última parte da key no S3). |
| Status | Status | Default: `PENDING` na ingestão. Em erro, pode virar `ERROR` (há TODOs de update em alguns serviços). |
| Sequence | Sequence | Derivado: PK interna gerada (`PrimaryKeyUtils.getSequenceId()`) no insert. |
