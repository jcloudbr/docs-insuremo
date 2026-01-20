# Mapeamento — Arquivos SUSEP (Coinsurance) → Policy (Estrutura Definitiva)

> **Objetivo:** mapear, de forma operacional, **onde cada campo dos arquivos (PREMIT/PREMREC/PREMRECEB/RESPREM)** aparece na **Policy** (estrutura definitiva) no InsureMO.
>
> **Referências de código (core):**

> - `CoinsurancePremitService.groovy` (ingestão/validação/persistência do PREMIT)

> - `CoinsurancePremRecService.groovy` (motor de emissão/endosso e montagem da Policy)

> - `CoinsurancePremRecebService.groovy` (recebimento do prêmio)

> - `CoinsuranceResPremService.groovy` (reserva/competência – PPNG)

>


---

## 1) Policy (root) — campos de cabeçalho

Esses campos ficam diretamente no objeto `Policy` (nível raiz) e normalmente vêm do **PREMIT** (e em alguns casos do fluxo de emissão do motor).

| Campo no arquivo | Significado (negócio) | Onde fica na Policy (JSON path) | Fonte principal |
|---|---|---|---|
| `COD_CIA` | Código SUSEP da cia (líder no arquivo) | `Policy.CodCiaLider` | PREMIT |
| `NUM_PROC` | Número de processo SUSEP | `Policy.NumProcLider` | PREMIT |
| `DT_BASE` | Competência/mês base | `Policy.DtBase` | PREMIT |
| `TIPO_MOV` | Tipo de movimento SUSEP | `Policy.TipoMov` | PREMIT |
| `UF_DEP` | UF de dependência/origem | `Policy.UFDep` | PREMIT |
| `UF_RISCO` | UF de risco | `Policy.UFRisco` | PREMIT |
| `DS_GRUPO_RAMO` | Identificação do grupo de ramo | `Policy.DsGrupoRamo` | PREMIT/PREMREC |
| `CD_PRODUTO` | Código do produto | `Policy.CdProduto` | PREMIT/PREMREC |
| `DS_PRODUTO` | Descrição do produto | `Policy.DsProduto` | PREMIT/PREMREC |
| `CD_BU` | BU/canal | `Policy.CdBu` | PREMIT/PREMREC |
| `DS_BU` | Descrição BU/canal | `Policy.DsBu` | PREMIT/PREMREC |
| `CD_OPERACAO` | Operação | `Policy.CdOperacao` | PREMIT/PREMREC |
| `CD_CARTEIRA` | Carteira | `Policy.CdCarteira` | PREMIT/PREMREC |
| `DS_CARTEIRA` | Nome carteira | `Policy.DsCarteira` | PREMIT/PREMREC |
| `CD_SUCURSAL` | Sucursal | `Policy.CdSucursal` | PREMIT/PREMREC |
| `FG_GEB` | Flag GEB | `Policy.FgGeb` | PREMIT/PREMREC |
| `DCR_TIP_OPER` | Descrição tipo operação | `Policy.DcrTipOper` | PREMIT/PREMREC |
| `DS_ORIGEM` | Origem/canal | `Policy.DsOrigem` | PREMIT/PREMREC |

### Datas “principais” do risco (vigência / emissão)

| Campo no arquivo | Significado | Onde fica na Policy | Fonte principal |
|---|---|---|---|
| `DT_INI_VIG` | Início de vigência do risco | `Policy.EffectiveDate` | PREMIT/PREMREC/RESPREM |
| `DT_FIM_VIG` | Fim de vigência do risco | `Policy.ExpiryDate` | PREMIT/PREMREC/RESPREM |
| `DT_EMIS` / `DT_EMIS_PR` | Data de emissão (apólice / parcela) | `Policy.IssueDate` *(quando aplicável)* | PREMIT/PREMREC |

> Observação prática: `MasterPolicyNo`, `PolicyNo`, `PolicyId`, `MasterPolicyId` e afins **não vêm do arquivo** — são gerados pelo motor de emissão (InsureMO) e retornam na Policy.

---

## 2) CoInsuranceInfoList — amarração do cosseguro

A Policy guarda as informações de cosseguro em `CoInsuranceInfoList`.

### 2.1 Chaves do líder (apólice/endosso/proposta)

| Campo no arquivo | Significado | Onde fica na Policy | Fonte |
|---|---|---|---|
| `NUM_APOL` | Apólice no líder | `Policy.CoInsuranceInfoList[0].LeaderPolicyNo` | PREMIT/PREMREC |
| `NUM_END` | Endosso no líder | `Policy.CoInsuranceInfoList[0].LeaderEndorsementNo` | PREMIT/PREMREC |
| `NUM_PROP` | Proposta no líder | `Policy.CoInsuranceInfoList[0].LeaderProposalNo` | PREMIT/PREMREC |
| `DT_PROP` | Data da proposta | `Policy.CoInsuranceInfoList[0].LeaderProposalDate` | PREMIT/PREMREC |

### 2.2 Participação da aceita (Coseguradora)

| Campo no arquivo | Significado | Onde fica na Policy | Fonte |
|---|---|---|---|
| `PERC_COSS` | Percentual do cosseguro | `Policy.CoInsuranceInfoList[0].PercCoss` | PREMREC |
| `OFER` | Oferta/indicador operacional | `Policy.CoInsuranceInfoList[0].Ofer` | PREMREC |
| `COLET` | Colet/indicador operacional | `Policy.CoInsuranceInfoList[0].Colet` | PREMREC |
| *(fixo no código)* | Código do cossegurador líder (referência) | `Policy.CoInsuranceInfoList[0].CodCoss` | Hardcoded |

### 2.3 CoInsuranceInsurerList — lista de seguradoras (líder/aceita)

No JSON de Policy, os dados SUSEP das seguradoras aparecem dentro de `CoInsuranceInsurerList[*]`.

| Campo no arquivo | Onde fica | Interpretação |
|---|---|---|
| `COD_CIA` / `NUM_PROC` | `...CoInsuranceInsurerList[*].CodCia` / `...NumProc` | No fluxo, normalmente alimenta o registro da seguradora **líder**.
| *(fixo / cadastro)* | `...CoInsuranceInsurerList[*]` do cossegurador | A seguradora aceita costuma ser definida por configuração/cadastro (ex.: `03417` + processo específico).

---

## 3) PolicyCustomerList — segurado, tomador, estipulante

O arquivo ancora principalmente **documentos** (CPF/CNPJ) e alguns indicadores.
Dados como **nome, endereço, sexo, data de nascimento** geralmente vêm do **cadastro/party** (lookup no fluxo).

| Campo no arquivo | Significado | Onde fica na Policy | Fonte |
|---|---|---|---|
| `CPF_SEG` | Documento do segurado (CPF/CNPJ) | `Policy.PolicyCustomerList[*].IdNo` *(para o customer com `IsInsured=Y`)* | PREMIT |
| `QTD_SEG` | Quantidade de segurados (quando aplicável) | `Policy.PolicyCustomerList[*].QtdSeg` | PREMIT |
| `QTD_TOM` | Indicador de tomador (quando aplicável) | `Policy.PolicyCustomerList[*].QtdTom` | PREMIT |
| `CPF_ESTIP` *(se usado)* | Documento do estipulante | `Policy.PolicyCustomerList[*].IdNo` *(para o policy holder)* | PREMIT |

> Nota: no seu OPA do PREMIT, as regras de `CPF_ESTIP` estão comentadas — então pode ser um campo **opcional** no momento.

---

## 4) PolicyLobList → PolicyRiskList → PolicyCoverageList (coberturas e valores)

A estrutura de coberturas (`PolicyCoverageList`) carrega tanto:

- **identidade da cobertura** (ramo, sequência)

- **valores financeiros** (prêmio, IOF, comissão, pró-labore, etc.)

### 4.1 Identidade e ramo

| Campo no arquivo | Significado | Onde fica na Policy | Fonte |
|---|---|---|---|
| `SEQUENCIA` | Identificador do registro no arquivo | `Policy...PolicyCoverageList[*].SequenceNumber` | PREMIT/PREMREC/RESPREM |
| `COD_RAMO` | Ramo SUSEP | `Policy...PolicyCoverageList[*].CodRamo` | PREMIT/PREMREC/RESPREM |
| `IS` | Importância segurada | `Policy...PolicyCoverageList[*].SumInsured` | PREMIT |

### 4.2 Totalizadores/valores (lado líder)

| Campo no arquivo | Significado | Onde fica na Policy | Fonte |
|---|---|---|---|
| `PR_EMIT` | Prêmio emitido do movimento | `...PolicyCoverageList[*].OverrideBeforeVatPremium` | PREMIT/PREMREC |
| `IOF` | Imposto | `...PolicyCoverageList[*].OverrideVat` | PREMIT/PREMREC |
| `COMIS` | Comissão (líder) | `...PolicyCoverageList[*].OverrideCommission` | PREMIT |
| `AD_FRAC` | Adicional fracionamento | `...PolicyCoverageList[*].AdFracLider` | PREMIT/PREMREC |
| `CUST_APOL` | Custo de apólice | `...PolicyCoverageList[*].CustApol` | PREMIT/PREMREC |
| `PRO_LAB` | Pró-labore (líder) | `...PolicyCoverageList[*].LaborAmountLider` | PREMIT |

### 4.3 Valores do cosseguro (lado aceito / cedido)

| Campo no arquivo | Significado | Onde fica na Policy | Fonte |
|---|---|---|---|
| `PR_COS_CED` | Prêmio cedido no cosseguro | `...PolicyCoverageList[*].PrCosCed` | PREMIT |
| `COMIS_COSS` | Comissão cosseguro | `...PolicyCoverageList[*].ComisCoss` | PREMIT |

> Observação: no JSON da Policy aparecem campos como `PrCossAc`, `ComCosAc`, `Cofee`, `PrNGanho*` etc. Esses tendem a ser preenchidos por cálculos/rotinas de cosseguro (ex.: PREMAC/PREMRECAC/RESPREMC) e/ou regras do motor.

---

## 5) PolicyPaymentInfoList → InstallmentList (parcelas / cobrança)

Essa parte é predominantemente **PREMREC** (parcelamento, dados bancários e cronograma de parcelas).

### 5.1 Cabeçalho de pagamento

| Campo no arquivo (PREMREC) | Significado | Onde fica na Policy | Fonte |
|---|---|---|---|
| `QTDE_PREST` | Qtde total de parcelas | `Policy.PolicyPaymentInfoList[0].InstallmentPeriodCount` | PREMREC |
| `NUM_BAN` | Banco | `Policy.PolicyPaymentInfoList[0].NumBan` | PREMREC |
| `CNPJ_BAN` | CNPJ do banco/beneficiário (layout) | `Policy.PolicyPaymentInfoList[0].CNPJBan` | PREMREC |
| `NUM_BOL` | Número do boleto | `Policy.PolicyPaymentInfoList[0].NumBol` | PREMREC |
| `NUM_DOC` | Número do documento | `Policy.PolicyPaymentInfoList[0].NumDoc` | PREMREC |
| `NUM_AGE` | Agência | `Policy.PolicyPaymentInfoList[0].NumAge` | PREMREC |
| `NUM_NOS` | Nosso número | `Policy.PolicyPaymentInfoList[0].NumNos` | PREMREC |

### 5.2 Parcela (InstallmentList)

| Campo no arquivo (PREMREC) | Significado | Onde fica na Policy | Fonte |
|---|---|---|---|
| `PRESTACAO` | Número da parcela | `...InstallmentList[*].InstallmentPeriodSeq` | PREMREC |
| `DT_VENCTO` *(ou data de parcela no layout)* | Vencimento | `...InstallmentList[*].InstallmentDate` | PREMREC |
| `DT_EMIS_PR` | Data emissão da parcela | `...InstallmentList[*].DtEmisPre` | PREMREC |

> Valores como `ValDoc`, `ValDesc`, `ValMul`, `ValCob` aparecem no JSON e tendem a ser populados pelo fluxo/cálculo de cobrança.

---

## 6) PREMRECEB — recebimento do prêmio (importante: destino não é “Policy”)

No desenho de negócio, o **PREMRECEB** representa o **realizado/recebido**.

### O que você deve documentar como destino

- **Regra prática (estado atual do código):** o `CoinsurancePremRecebService.groovy` faz ingestão/validação/persistência e depois tenta encaminhar/registrar a baixa (pagamento) no módulo apropriado.
- **Ou seja:** PREMRECEB é **fluxo de recebimento** e **não** um “atualizador direto” da Policy.

Mesmo assim, os campos do PREMRECEB costumam espelhar chaves do movimento e podem ser usados para:
- localizar `PolicyPaymentInfo/Installment` (por chaves e competência)
- registrar quitação/baixa
- conciliação com o parcelado (PREMREC)

**Campos típicos a mapear (por chave/consulta):**

| Campo no arquivo (PREMRECEB) | Uso no fluxo | Observação |
|---|---|---|
| `COD_CIA`, `NUM_PROC` | Amarra SUSEP/companhia | Mesma semântica do PREMIT |
| `DT_BASE`, `TIPO_MOV` | Competência e tipo | Usado para conciliar lote |
| `COD_RAMO`, `NUM_APOL`, `NUM_END`, `NUM_PROP` | Chave do contrato/movimento | Usado para localizar Policy/CoInsuranceInfo |
| `DT_INI_VIG`, `DT_FIM_VIG` | Vigência | Pode reforçar consistência |
| `VAL_DOC` / valores de recebimento | Baixa/quitado | Normalmente vai para módulo de pagamentos/baixa |

---

## 7) RESPREM — competência / PPNG (reserva de prêmio não ganho)

O **RESPREM** se relaciona com vigência + competência e alimenta cálculo/registro de prêmio ganho/não ganho.

### Estado do código

- No fluxo atual, o `CoinsuranceResPremService.groovy` possui partes **pendentes/TODO** (em implementações típicas, ele persiste o arquivo e depois gera registros contábeis/gerenciais).
- Então, o que dá para afirmar com segurança é:
  - RESPREM reforça/consome `DT_BASE`, `DT_INI_VIG`, `DT_FIM_VIG`, `COD_RAMO`, e chaves do contrato.
  - O destino pode ser `PolicyCoverage.PrNGanho*` / campos de “earned/unearned premium”, mas isso depende da implementação completa.

**Campos que aparecem no JSON e costumam ser alvo do RESPREM:**

| Conceito | Onde aparece na Policy | Observação |
|---|---|---|
| Prêmio não ganho (líder) | `...PolicyCoverageList[*].PrNGanhoLider` | Pode ser calculado a partir de RESPREM |
| Prêmio não ganho (cosseguro) | `...PolicyCoverageList[*].PrNGanhoCoss` | Mesma lógica, lado aceito |

---

## 8) Notas de consistência para seu repositório

### 8.1 “Fonte do dado” (padrão recomendado)

Para cada campo, mantenha sempre:
- **origem**: PREMIT / PREMREC / PREMRECEB / RESPREM
- **destino**: `Policy` (path) **ou** “módulo externo” (pagamentos/baixa/contábil)
- **comentário**: “hardcoded”, “lookup cadastro/party”, “gerado pelo motor”, “TODO no código”

### 8.2 Campos gerados pelo motor (não mapeiam do arquivo)

Alguns exemplos comuns:
- `PolicyId`, `PolicyNo`, `MasterPolicyId`, `MasterPolicyNo`
- `OrgCode`, `PolicyStatus`, `PolicyType`
- `ProductId`, `TechProductId`, `BusinessObjectId`

---

## Apêndice — exemplos de paths úteis no JSON

- `Policy.EffectiveDate`, `Policy.ExpiryDate`, `Policy.IssueDate`
- `Policy.CoInsuranceInfoList[0].LeaderPolicyNo`, `LeaderEndorsementNo`, `LeaderProposalNo`
- `Policy.PolicyCustomerList[*].IdNo`, `IsInsured`, `IsPolicyHolder`
- `Policy.PolicyLobList[0].PolicyRiskList[0].PolicyCoverageList[*].OverrideBeforeVatPremium`
- `Policy.PolicyPaymentInfoList[0].InstallmentList[*].InstallmentDate`, `InstallmentPeriodSeq`
