# Emissão da Apólice (Master/Certificate) — PREMIT + PREMREC (Coinsurance)

> **Escopo:** fluxo de **criação/atualização da Apólice Definitiva no módulo Policy** (**Master Policy + Certificate Policy**), a partir de **PREMIT (pai)** e **PREMREC (parcelas/motor)**.  
> **Referências de código:** `CoinsurancePremitService.groovy`, `CoinsurancePremRecService.groovy`  
> **Entidades:** `CnsPremit`, `CnsPremRec`  
> **Links rápidos:** [PREMIT](../premit.md) · [PREMREC](../premrec.md) · [De/Para PREMIT](../depara/premit.md) · [De/Para PREMREC](../depara/premrec.md)

---

## Visão geral

- **PREMIT** entra como **registro “pai”** (fotografia da emissão/movimento) e fica **PENDING**.
- **PREMREC** entra como **filho/parcelas** e é o **motor operacional** que:
  - Cria/atualiza **Master Policy** e **Certificate Policy** (apólice definitiva no módulo Policy)
  - Processa **endossos**
  - Dispara integração **BCP** no **pós-emissão**
  - Atualiza status para **COMPLETED** quando sucesso

---

## 3.1 PREMIT — Registro mestre do movimento (base da emissão)

### Código e entidade
- **Service:** `CoinsurancePremitService.groovy`
- **Entidade:** `CnsPremit`
- **Campos principais (exemplos):**
  - `BaseDate`, `MovementType`, `BranchCode`
  - `PolicyNumber`, `EndorsementNumber`, `ProposalNumber`
  - datas de vigência
  - prêmios, comissões etc.

### Fluxo real no código

1) **Ingestão do arquivo**
- Lê `PREMIT.CSV` em `uploaded/`

2) **Validação**
- Validação de entrada (OPA/Velora) — **fora deste repositório**

3) **Persistência**
- `insertPremit()` com `Status = PENDING`

!!! note "Comportamento importante"
    O **PREMIT não emite apólice sozinho**. Ele funciona como **registro mestre (pai)** para correlacionar e orientar o processamento do **PREMREC**, que é quem efetivamente **cria/atualiza a apólice definitiva** (Master/Certificate).

### Papel negocial (SUSEP / operacional)
- **PREMIT** representa a “fotografia” do **movimento macro** (emissão/endosso) com totalizadores e vigência.
- No fluxo interno, é a **referência pai** para o processamento de emissão/endosso no módulo Policy via **PREMREC**.

---

## 3.2 PREMREC — Motor de emissão (Master/Certificate) e endosso

### Código e entidade
- **Service:** `CoinsurancePremRecService.groovy`
- **Entidade:** `CnsPremRec` (parcelas)
- **Campos principais (exemplos):**
  - `InstallmentNumber`, `InstallmentQuantity`
  - datas e valores por parcela

---

## A) Ingestão do PREMREC

1) **Leitura**
- Lê `PREMREC.CSV`

2) **Validação**
- Validação de entrada (OPA/Velora) — **fora deste repositório**

3) **Persistência**
- Persiste `CnsPremRec` com `Status = PENDING`

---

## B) Processamento (criação/alteração da apólice definitiva)

### B.1 Seleção do “pai” (PREMIT) e dos “filhos” (PREMREC)

O service busca **PREMIT pendentes** e, para cada pai, busca os **PREMREC pendentes** correspondentes usando a chave:

- **Chave de correlação (no próprio código):**
  - `MovementType + PolicyNumber + EndorsementNumber + ProposalNumber + Status = PENDING`

---

## Validações e decisões (como o código decide)

### 1) Segurado (Party) precisa existir
- Antes de emitir, o fluxo **busca party no cadastro por documento** (CPF/CNPJ).
- Se não existir: **falha** (gera erro no fluxo).

### 2) Produto precisa existir na rate table
- Lookup em `BMG_PolicyProduct` por:
  - `LeaderPolicyNo (PolicyNumber)` + `CdProduto (ProductCode)`

### 3) Master Policy (apólice master) — pré-condição para o Certificate
- `fetchMasterPolicy(leaderPolicyNo)` busca `Policy` no módulo “Policy” por:
  - `LeaderPolicyNo`
  - `PolicyType = MASTER`
  - `PolicyStatus = ISSUED`
  - `FullOrgCode = tenant`
- **Se movimento for de emissão e master não existir:** cria master (issuance)

### 4) Certificate Policy (certificado) — apólice definitiva emitida para o segurado
- Checa se certificado já existe: `existPolicyIssued(...)`
  - Se existe: trata como **duplicado**
  - Se não existe: **cria certificate** e integra BCP

### 5) Endosso
- Para `MovementType > 102`, usa `processEndorsementIssuance(...)`
- Mapeamento de movimentos (101–108) para `endoType` via `CnsPremRec.getEndoType()`:

| MovementType (TIPO_MOV) | Descrição (SUSEP / Layout) | O que significa na prática | endoType (se aplicável) |
|---:|---|---|---:|
| 101 | Emissão de Apólice | Criação da apólice/certificado (emissão inicial) | — |
| 102 | Endosso de cobrança adicional de prêmio | Endosso que aumenta prêmio (cobrança complementar) | — |
| 103 | Endosso de restituição de prêmio | Endosso que reduz prêmio e gera restituição | 37 |
| 104 | Cancelamento de Apólice com restituição de prêmio | Cancela a apólice e devolve prêmio | 51 |
| 105 | Cancelamento de Endosso com restituição de prêmio | Cancela um endosso e devolve prêmio | 17 |
| 106 | Cancelamento de Apólice sem restituição de prêmio | Cancela a apólice sem devolução de prêmio | 50 |
| 107 | Cancelamento de Endosso sem restituição de prêmio | Cancela um endosso sem devolução de prêmio | 3 |
| 108 | Endosso sem movimentação de prêmio | Endosso que altera condições sem mexer no prêmio | 20 |
| 109 | Transferência de carteira | Movimentação para transferência de carteira (quando aplicável no layout) | — |

!!! note "Nota importante (layout x implementação)"
    A tabela acima reflete a **leitura negocial** (layout SUSEP).  
    O `endoType` reflete o que o **código implementa** em `getEndoType()` e pode não cobrir todos os cenários como o layout descreve.

---

## Integração com BCP (pós-emissão)

Após emissão, executa `integrateBcpAfterPolicyIssuance(policy)`:

1) Gera taxa/fee: `GenerateFeeService`  
2) Carrega `PolicyFeeInfo`  
3) Monta payload  
4) Envia via `DirectPostFeeReceiver`

---

## Atualização de status (sucesso vs erro)

### Sucesso
- Se processou OK:
  - Atualiza **PREMREC** e **PREMIT** relacionados para `COMPLETED`

### Erro
- `processParentAndChildrenAsError(...)` monta erro e grava `CnsError`
- **Ponto crítico:** as linhas que atualizariam **PREMREC/PREMIT** para `ERROR/DUPLICATED` estão **comentadas (TODO)**

!!! warning "Risco operacional"
    Se o update para `ERROR/DUPLICATED` está comentado, registros podem ficar **eternamente `PENDING`** e serem **reprocessados** continuamente.

---

## Tabelas de decisão (resumo do motor PREMREC)

### A) Emissão / Master / Certificate

| Condição | Ação principal | Resultado esperado |
|---|---|---|
| Party não existe | Bloqueia emissão | Erro registrado (`CnsError`) |
| Produto não existe em rate table | Bloqueia emissão | Erro registrado (`CnsError`) |
| Emissão + Master não existe | Criar Master | `Policy (MASTER)` emitida |
| Certificate já existe (ISSUED) | Tratar como duplicado | (deveria marcar DUPLICATED, mas pode ficar PENDING) |
| Certificate não existe | Criar Certificate + integrar BCP | Apólice emitida + fee/pós-issue |

### B) Endosso

| Condição | Ação principal |
|---|---|
| `MovementType > 102` | `processEndorsementIssuance(...)` |
| `MovementType 101–108` | `getEndoType()` define `endoType` |