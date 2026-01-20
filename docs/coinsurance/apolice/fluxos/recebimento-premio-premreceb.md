# Recebimento de Prêmio — PREMRECEB (Coinsurance)

> **Escopo:** registrar o **prêmio recebido** (por parcela) e **materializar o recebimento no BCP** como **Payment (boleto pago)**, amarrado à apólice **Certificate** (e ao **Endorsement**, quando aplicável).  
> **Referências de código:** `CoinsurancePremRecebService`  
> **Entidades:** `CnsPremReceb` (ingestão) + integração BCP (`Payment`, `PaymentDetail`)

---

## Visão geral

- **PREMRECEB** representa, no negócio, o **evento de recebimento** do prêmio (normalmente por boleto) — isto é: **pagou**.
- No fluxo interno, ele funciona como “ponte” entre:
  - **o mundo SUSEP/arquivo** (parcela recebida + dados bancários + valores)
  - e **o mundo operacional** (registrar no **BCP** como `Payment` com status **Paid**).
- Diferente do PREMREC (parcelamento/cronograma), o PREMRECEB é o **realizado**: comprovação/registro de recebimento.

---

## O que é o PREMRECEB no negócio (SUSEP / operacional)

### O que ele representa
Um registro de **prêmio efetivamente recebido**, por parcela, contendo:
- a **chave de negócio** (ramo, apólice, endosso, proposta, movimento)
- dados de **parcela** (número, vencimento, data de recebimento)
- dados de **cobrança** (boleto, nosso número, documento, agência, banco)
- valores do título (valor do documento, desconto, multa, valor cobrado)

### Para que ele serve
Negocialmente, serve para:
- **dar baixa financeira** (parcela paga)
- permitir **conciliação** entre:
  - **emitido/parcelado** (PREMREC)
  - **recebido** (PREMRECEB)
- acionar a camada operacional (no seu caso, **BCP**) para registrar o pagamento como **Payment Paid**.

---

## 4.1 PREMRECEB — Recebimento do prêmio (por parcela)

### Código e entidade
- **Service:** `CoinsurancePremRecebService`
- **Entidade:** `CnsPremReceb`

!!! note "Validação de entrada (OPA/Velora)"
    O service chama `veloraService.validate("bmg_prereceb", mapData)`.
    As **policies OPA/Velora** ficam **fora deste repositório**.

---

## A) Ingestão do PREMRECEB

### 1) Leitura do arquivo (S3)
O service lista arquivos pendentes via:
- `s3FileService.listPendingProcessFiles(s3FileService.PREMRECEB)`

Para cada arquivo:
- lê o conteúdo (`getResponseBundleFromKey`)
- transforma em bytes UTF-8
- faz parse CSV com delimitador `|`:
  - `csvReader.readFile("|", bytes)`

### 2) Mapeamento CSV → Entidade (`CnsPremReceb`)
O mapeamento é feito por `mapper` no código (tabela completa em **De/Para**).

Além disso, o código:
- carrega `declaredFields` da entidade para cast tipado (`castValue(...)`)
- injeta defaults (Status, FileId, FileName)

### 3) Validação de entrada
Para cada linha do arquivo (`mapData`):
- `veloraService.validate("bmg_prereceb", mapData)`
- **Se válido:** `coinsuranceRepository.insertPremReceb(entity)`
- **Se inválido:** `processAsError(...)` (gera `CnsError`)

### 4) Pós-processamento do arquivo (move/delete)
Depois de processar o arquivo:
- move para:
  - `processed/` se **não houve erro**
  - `error/` se **houve erro**
- e apaga o original (`deleteFile(key)`)

!!! warning "Risco operacional (marcação de erro do arquivo)"
    No código atual, a variável `hasError` depende do resultado de validação ao longo do loop.
    Se houver mistura de linhas válidas e inválidas, a decisão de mover o arquivo inteiro para `processed/` ou `error/` pode ficar **inconsistente**.

---

## B) Processamento dos registros pendentes (materializar pagamento)

Após a ingestão, o service processa registros `PENDING` a partir do índice:
- `coinsuranceRepository.searchByModuleAndStatusFromIndex(CnsPremRecebEntity, PENDING)`
- para cada registro: `processEntity(entity)`

O objetivo dessa etapa é:

1) localizar a **apólice definitiva** (Master/Certificate) no módulo Policy

2) localizar **endosso** (quando aplicável)

3) registrar o recebimento no **BCP** como **Payment Paid**

---

## Validações e decisões (como o código decide)

### 1) Master Policy precisa existir
O código tenta localizar a master:
- `fetchMasterPolicy(entity.PolicyNumber)`

**Se não existir:**
- erro: “Não existe uma Master Policy para a apólice …”
- `processAsError(entity, ERROR, ...)`

!!! warning "Ponto crítico do código (TODO/hardcode)"
    No código atual, `fetchMasterPolicy(...)` está com **TODO** e usa `LeaderPolicyNo` hardcoded, em vez de usar plenamente o `entity.PolicyNumber`.

### 2) Certificate Policy emitida precisa existir
O código valida se existe certificate emitida (no contexto da master):
- `existPolicyIssued(masterPolicy, entity)`

**Se não existir:**
- erro: “Não existe uma apólice criada para o ramo…, apólice…, proposta…”
- `processAsError(entity, ERROR, ...)`

!!! warning "Ponto crítico do código (TODO/hardcode)"
    `existPolicyIssued(...)` também contém **TODO** e, no estado atual, utiliza valores hardcoded para busca (ex.: ramo e leader policy).

### 3) Carrega a primeira Certificate (para obter PolicyNo)
- `fetchFirstCertificate(masterPolicy, entity)`

Se não achar:
- erro de “Não encontrada apólice para o ramo…, apólice…, proposta…”

### 4) Endorsement (para obter EndoNo)
- `fetchFirstEndorsement(certificatePolicy, entity)`

!!! warning "Ponto crítico do código (TODO/hardcode)"
    `fetchFirstEndorsement(...)` está com **TODO** e faz buscas com IDs/PolicyNo hardcoded.
    Isso impacta diretamente a capacidade de amarrar o recebimento ao endosso correto.

---

## Integração com BCP (registro do pagamento)

Se passou nas validações, o código monta um `Payment` e envia para o BCP:
- `paymentApi().newSinglePaymentRequestBuilder().payment(payments).doRequest()`

### Como o PREMRECEB vira `Payment` no BCP
- `Payment.paymentMethod = "70"` → boleto
- `Payment.paymentStatus = "3"` → Paid
- `Payment.paymentType = "1"` → Direct Billing
- `Payment.currencyCode = "BRL"`
- `Payment.policyNo = certificatePolicy.policyNo`
- `Payment.endoNo = endo.endoNo` (se encontrado)

---

## Atualização de status (sucesso vs erro)

### Sucesso
- O código retorna `processedSuccessfully = true`.
- **Mas** o registro `CnsPremReceb` não é marcado como `COMPLETED` (não há update efetivo persistido nessa etapa).

### Erro
- `processAsError(...)`:
  - seta `entity.Status = ERROR`
  - **não persiste** a mudança (update está comentado): `// TODO coinsuranceRepository.updatePremReceb(entity)`
  - grava `CnsError` (com `Status = COMPLETED`)

!!! warning "Risco operacional (reprocessamento infinito)"
    Como o `updatePremReceb(...)` está comentado, registros podem permanecer **PENDING** no índice e serem **reprocessados** continuamente, mesmo após erro/sucesso.

---

## De/Para — CSV (arquivo físico) → Entidade (`CnsPremReceb`)

> **Fonte:** `Map<String, String> mapper` no `CoinsurancePremRecebService`

| Campo no arquivo (CSV) | Campo salvo na entidade |
|---|---|
| `SEQUENCIA` | `RecordNo` |
| `COD_CIA` | `CompanyCode` |
| `DT_BASE` | `BaseDate` |
| `TIPO_MOV` | `MovementType` |
| `COD_RAMO` | `BranchCode` |
| `NUM_APOL` | `PolicyNumber` |
| `NUM_END` | `EndorsementNumber` |
| `NUM_PROP` | `ProposalNumber` |
| `DT_PROP` | `ProposalDate` |
| `PRESTACAO` | `InstallmentNumber` |
| `DT_REC_PRE` | `InstallmentReceiptDate` |
| `DT_VEN_PRE` | `InstallmentDueDate` |
| `NUM_BAN` | `BankNumber` |
| `CNPJ_BAN` | `BankDocumentId` |
| `NUM_BOL` | `TicketNumber` |
| `NUM_DOC` | `DocumentNumber` |
| `NUM_AGE` | `AgencyNumber` |
| `NUM_NOS` | `OurNumber` |
| `VAL_DOC` | `DocumentValue` |
| `VAL_DESC` | `DiscountValue` |
| `VAL_MUL` | `FineValue` |
| `VAL_COB` | `ChargedValue` |
| `DS_GRUPO_RAMO` | `BranchGroupDescription` |
| `CD_PRODUTO` | `ProductCode` |
| `DS_PRODUTO` | `ProductDescription` |
| `CD_BU` | `BusinessUnitCode` |
| `DS_BU` | `BusinessUnitDescription` |
| `CD_OPERACAO` | `OperationCode` |
| `CD_CARTEIRA` | `WalletCode` |
| `DS_CARTEIRA` | `WalletDescription` |
| `CD_SUCURSAL` | `SucursalCode` |
| `FG_GEB` | `FgGeb` |
| `DCR_TIP_OPER` | `OperationDescription` |
| `DS_ORIGEM` | `OriginDescription` |

---

## De/Para — Defaults e rastreabilidade (como o registro é “carimbado”)

| Campo interno | Como é preenchido no código |
|---|---|
| `Status` | `PENDING` |
| `FileName` | nome do arquivo (`key.split("/").last()`) |
| `FileId` | `"{etagSemAspas}_{RecordNo}"` |

---

## De/Para — Entidade → BCP Payment (como o “recebido” é operacionalizado)

| Origem | Campo | Destino BCP |
|---|---|---|
| `CnsPremReceb` | `DocumentValue` | `Payment.amount` e `PaymentDetail.amount` |
| `CnsPremReceb` | `BankNumber` | `Payment.bankCode` |
| `CnsPremReceb` | `OurNumber` | `Payment.bankAccountNo` |
| `CnsPremReceb` | `AgencyNumber` | `Payment.insurerAccountCode` |
| `CnsPremReceb` | `TicketNumber` | `Payment.paymentNo` |
| `Certificate Policy` | `policyNo` | `Payment.policyNo` |
| `Endorsement` | `endoNo` | `Payment.endoNo` |
| Constante | `"70"` | `Payment.paymentMethod` (boleto) |
| Constante | `"3"` | `Payment.paymentStatus` (Paid) |
| Constante | `"1"` | `Payment.paymentType` (Direct Billing) |
| Constante | `"BRL"` | `Payment.currencyCode` / `PaymentDetail.currencyCode` |

---

## Tabelas de decisão (resumo do motor PREMRECEB)

### A) Pré-condições para registrar pagamento

| Condição | Ação | Resultado |
|---|---|---|
| Master Policy não encontrada | `processAsError(ERROR)` | Não registra no BCP |
| Certificate emitida não existe | `processAsError(ERROR)` | Não registra no BCP |
| Certificate não carregou | `processAsError(ERROR)` | Não registra no BCP |
| Tudo OK | Monta `Payment` e chama BCP | Registra `Payment` como **Paid** |

---

## Pontos pendentes/incompletos no código (impacto direto no negócio)

1) **Lookup de apólice com valores hardcoded (TODO)**
- Master/Certificate/Endorsement ainda não usam plenamente a chave do arquivo (`BranchCode`, `PolicyNumber`, `EndorsementNumber`, `ProposalNumber`).

2) **Status do registro não é atualizado**
- `updatePremReceb(entity)` está comentado, então o registro pode permanecer `PENDING` e reprocessar indefinidamente.

3) **Marcação de sucesso do arquivo pode ser inconsistente**
- A decisão de mover para `processed/` ou `error/` pode não refletir corretamente um arquivo com linhas mistas (válidas e inválidas).
