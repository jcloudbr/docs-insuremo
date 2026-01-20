# Apólice — Mapeamentos internos (Policy / CoinsuranceInfo / Coverage)

> **Origem:** `CoinsurancePremRecService.groovy` (métodos de montagem de apólice/endosso)

## A) PolicyInfo (campos adicionais no Policy/Endosso)

| Chave no PolicyInfo | Origem (entidade) | Destino |
| --- | --- | --- |
| AdjustedPremium | entity.FractionalSurchargeValue | Policy/Endosso (campos adicionais no PolicyInfo). |
| DuePremium | entity.IssuedPremium | Policy/Endosso (campos adicionais no PolicyInfo). |
| GrossPremium | entity.PolicyCostValue | Policy/Endosso (campos adicionais no PolicyInfo). |
| Commission | entity.CoinsuranceCommissionAmount | Policy/Endosso (campos adicionais no PolicyInfo). |
| CommissionLocal | entity.BrokerageCommissionAmount | Policy/Endosso (campos adicionais no PolicyInfo). |
| OriginalFileName | entity.FileName | Policy/Endosso (campos adicionais no PolicyInfo). |
| EffectiveDate | entity.EffectiveDate | Policy/Endosso (campos adicionais no PolicyInfo). |
| ExpiryDate | entity.EndDate | Policy/Endosso (campos adicionais no PolicyInfo). |
| IssueDate | entity.IssueDate | Policy/Endosso (campos adicionais no PolicyInfo). |
| TipoMov | entity.MovementType | Policy/Endosso (campos adicionais no PolicyInfo). |
| DtBase | entity.BaseDate | Policy/Endosso (campos adicionais no PolicyInfo). |
| CdProduto | entity.ProductCode | Policy/Endosso (campos adicionais no PolicyInfo). |
| DsProduto | entity.ProductDescription | Policy/Endosso (campos adicionais no PolicyInfo). |
| CdBu | entity.BusinessUnitCode | Policy/Endosso (campos adicionais no PolicyInfo). |
| DsBu | entity.BusinessUnitDescription | Policy/Endosso (campos adicionais no PolicyInfo). |
| CdOperacao | entity.OperationCode | Policy/Endosso (campos adicionais no PolicyInfo). |
| CdCarteira | entity.WalletCode | Policy/Endosso (campos adicionais no PolicyInfo). |
| DsCarteira | entity.WalletDescription | Policy/Endosso (campos adicionais no PolicyInfo). |
| CdSucursal | entity.SucursalCode | Policy/Endosso (campos adicionais no PolicyInfo). |
| FgGeb | entity.FgGeb | Policy/Endosso (campos adicionais no PolicyInfo). |
| DcrTipOper | entity.OperationDescription | Policy/Endosso (campos adicionais no PolicyInfo). |
| DsOrigem | entity.OriginDescription | Policy/Endosso (campos adicionais no PolicyInfo). |
| DsGrupoRamo | entity.BranchGroupDescription | Policy/Endosso (campos adicionais no PolicyInfo). |
| UFDep | entity.IssuingState | Policy/Endosso (campos adicionais no PolicyInfo). |
| UFRisco | entity.RiskStates | Policy/Endosso (campos adicionais no PolicyInfo). |
| CodCiaLider | entity.CompanyCode | Policy/Endosso (campos adicionais no PolicyInfo). |
| NumProcLider | entity.Protocol | Policy/Endosso (campos adicionais no PolicyInfo). |

## B) CoinsuranceInfo (lista)

| Chave no CoinsuranceInfo | Origem (entidade) | Destino |
| --- | --- | --- |

## C) Coverage (coberturas)

| Campo no Coverage | Origem (entidade) | Destino |
| --- | --- | --- |
| SumInsured | entity.IsValue | Coverage (itens/coberturas no endorsement). |
| SequenceNumber | entity.Sequence | Coverage (itens/coberturas no endorsement). |
| SequenceBMG | entity.RecordNo | Coverage (itens/coberturas no endorsement). |
| OverrideBeforeVatPremium | entity.IssuedPremium | Coverage (itens/coberturas no endorsement). |
| OverrideVat | entity.TaxAmount | Coverage (itens/coberturas no endorsement). |
| OverrideCommission | entity.BrokerageCommissionAmount | Coverage (itens/coberturas no endorsement). |
| AdFracLider | entity.FractionalSurchargeValue | Coverage (itens/coberturas no endorsement). |
| CustApol | entity.PolicyCostValue | Coverage (itens/coberturas no endorsement). |
| LaborAmountLider | entity.ProLaboreValue | Coverage (itens/coberturas no endorsement). |
| PrCosCed | entity.CededCoinsurancePremium | Coverage (itens/coberturas no endorsement). |
| ComisCoss | entity.CoinsuranceCommissionAmount | Coverage (itens/coberturas no endorsement). |

!!! note
    Esses mapeamentos são *internos* (não vêm diretamente do CSV). Eles são gerados no processamento do **PREMREC** ao montar **apólice/endosso**.
