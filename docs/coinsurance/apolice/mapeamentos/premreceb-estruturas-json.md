# PREMRECEB — Estruturas usadas no processamento e como recuperar os JSONs

> **Objetivo:** documentar **quais estruturas** o fluxo de **PREMRECEB** usa (entrada, Policy/Endorsement e BCP Payment) e **quais JSONs** você precisa recuperar para montar o **mapeamento** dos campos do arquivo na **estrutura definitiva**.

---

## Visão geral (em linguagem de negócio)

- **PREMRECEB** representa o **recebimento do prêmio** (baixa/recebido) no contexto de cosseguro.
- No fluxo atual, ele **não “grava recebimento” dentro do JSON da Policy** diretamente.
- Ele **localiza a apólice/certificado** correspondente e **registra um Payment no BCP**, usando os dados do arquivo (valores e dados bancários) + `policyNo` / `endoNo`.

**Consequência prática para mapeamento:**
- O mapeamento de PREMRECEB se divide em:
  1) **Arquivo → `CnsPremReceb`** (persistência espelho)
  2) **`CnsPremReceb` → Policy/Endorsement (lookup)** (para obter `policyNo` / `endoNo`)
  3) **`CnsPremReceb` → `Payment` (BCP)** (onde o “recebido” efetivamente vira dado operacional)

---

## 1) Estrutura de entrada/persistência (espelho do arquivo)

### 1.1 Entidade gravada: `CnsPremReceb`

**Papel:** é o **espelho do CSV** dentro do sistema. Tudo começa por aqui.

**Onde aparece:**
- Classe/entidade: `CnsPremReceb` (camada de repository)
- Inserção/atualização: métodos do service de ingestão e persistência.

**O que você recupera em JSON aqui:**
- Um registro de `CnsPremReceb` (ou o DTO equivalente do service) contendo:
  - **chaves**: `cod_cia`, `dt_base`, `tipo_mov`, `cod_ramo`, `num_apol`, `num_end`, `num_prop`, `dt_prop`
  - **vigência**: `dt_ini_vig`, `dt_fim_vig`
  - **valores**: `val_doc`, `val_desc`, `val_mul`, `val_cob`
  - **bancário/boleto**: `num_ban`, `num_nos`, `num_bol`, `num_doc`, `num_age` (e eventuais variações)
  - **controle**: `status`, `fileId`, `originalFileName`, `recordNo/sequence`

**Por que esse JSON é obrigatório para o mapeamento?**
- Porque ele preserva os nomes de origem (layout) e a forma como o sistema “entendeu” e armazenou o arquivo.

---

## 2) Estruturas usadas para localizar a apólice definitiva (Policy/Endorsement)

### 2.1 `Policy` (Master e Certificate)

**Papel:** o fluxo precisa descobrir o **`policyNo` do Certificate** (e, às vezes, o **endosso**) para registrar o pagamento.

**Como é obtido no fluxo (alto nível):**
1) **Query (descobrir `policyId`)**
2) **Load Core (carregar JSON completo da apólice)**

#### A) Master Policy (quando o fluxo tenta localizar)
- Usado quando o código precisa do contexto de **Master**.
- Nem sempre é obrigatório para o pagamento — o crítico costuma ser o **Certificate**.

**JSON a recuperar (se necessário):**
- `Policy Core` do Master (`PolicyType = MASTER`).

#### B) Certificate Policy (o mais importante)
- É o **Certificate** que contém o **`policyNo`** usado no Payment.

**JSON a recuperar (obrigatório):**
- **Policy Core** do Certificate (JSON completo).

**Campos normalmente usados/observáveis nesse JSON:**
- `PolicyNo`, `MasterPolicyNo`, `MasterPolicyId`
- `EffectiveDate`, `ExpiryDate`, `IssueDate`
- `TipoMov`, `DtBase`, `UFDep`, `UFRisco`
- `CoInsuranceInfoList` (LeaderPolicyNo, LeaderEndorsementNo, LeaderProposalNo, ShareRate etc.)
- (eventualmente) `PolicyPaymentInfoList` / `InstallmentList` (nem sempre é “o” destino do recebido)

---

### 2.2 `Endorsement` (quando aplicável)

**Papel:** se o recebimento estiver associado a endosso, o fluxo obtém `endoNo`.

**Como é obtido no fluxo (alto nível):**
1) **Query (descobrir `endorsementId`)**
2) **Load Core (carregar JSON completo do endosso)**

**JSON a recuperar (quando aplicável):**
- **Endorsement Core** (JSON completo), principalmente para:
  - `endoNo`
  - datas/referências do endosso

---

## 3) Estruturas usadas para registrar o recebimento (BCP)

### 3.1 `Payment` (BCP) + `PaymentDetail`

**Papel:** é aqui que o **recebido** é efetivado operacionalmente.

#### A) `Payment` (campos típicos)
Preenchimentos esperados a partir do `CnsPremReceb`:

- `amount` \u2190 `val_doc` (DocumentValue)
- `bankCode` \u2190 `num_ban`
- `paymentNo` \u2190 `num_bol` ou `num_doc` (depende do que o código escolhe)
- `insurerAccountCode` \u2190 `num_age`
- `bankAccountNo` \u2190 `num_nos` *(atenção: “Nosso Número” pode estar sendo usado como “AccountNo”)*
- `policyNo` \u2190 `certificatePolicy.policyNo`
- `endoNo` \u2190 `endorsement.endoNo` (quando existir)

Campos fixos comuns no código:
- `currencyCode = "BRL"`
- `paymentMethod = "70"` (Boleto)
- `paymentType = "1"` (Direct Billing)
- `paymentStatus = "3"` (Paid)
- `payerPayeeType = "1"`

#### B) `PaymentDetail`
- `amount` \u2190 `val_doc`
- `currencyCode = "BRL"`
- `expenseType = "1"`
- `isExpense = "N"`

**JSON a recuperar (obrigatório):**
- **Request JSON** do Payment (lista `payments`) — normalmente o service já serializa/loga.

### 3.2 Resposta do BCP

**Papel:** confirma o registro do Payment.

**JSON a recuperar (recomendado):**
- Response do endpoint do BCP (sucesso/erro) — para evidência e troubleshooting.

---

## 4) Quais JSONs recuperar para fazermos o mapeamento (checklist)

### Obrigatórios
1) **`CnsPremReceb`** (registro persistido do arquivo)
2) **Policy Certificate (core)** (para capturar `policyNo` e a estrutura definitiva)
3) **Payment Request JSON** (BCP) (onde o “recebido” efetivamente vira dado operacional)

### Condicionais
4) **Policy Master (core)** (se você também quer mapear Master e regras/atributos adicionais)
5) **Endorsement (core)** (se o fluxo usa/precisa de `endoNo` no Payment)
6) **Payment Response JSON** (BCP) (auditoria e diagnóstico)

---

## 5) Como coletar esses JSONs na prática

### A) A partir de logs do próprio serviço
- Procure por logs onde o service serializa objetos (ex.: `toJSON(payments)` ou equivalente).
- Ideal para capturar:
  - **Payment request**
  - IDs utilizados (`policyId`, `endorsementId`)

### B) A partir do Policy/Endorsement Core (load)
- Com o `policyId` do Certificate, faça o **load** e salve o JSON.
- Com o `endorsementId`, faça o **load** e salve o JSON.

> Dica operacional: se você já tem exemplos de chamadas como `/platform/proposal/core/proposal/v1/load?policyId=...`, salve o payload completo para usarmos como “base de mapeamento”.

---

## 6) Observações importantes (para orientar o mapeamento)

1) **Destino do “recebido”**
- No fluxo atual, o destino primário do recebido é o **BCP Payment**, não necessariamente `PolicyPaymentInfoList.InstallmentList`.

2) **Campos bancários**
- Há risco de “semântica trocada” (ex.: `Nosso Número` indo como `bankAccountNo`).
- No mapeamento, registre explicitamente:
  - campo do arquivo
  - campo do `CnsPremReceb`
  - campo do `Payment`
  - observação de semântica

3) **Endorsement lookup**
- Se o código está com **hardcode/TODO** (IDs fixos), isso pode afetar a coleta de JSON real.
- Para mapeamento, foque primeiro no **happy path** (quando lookup funciona corretamente) e depois documente as lacunas.

---

## 7) Próximo passo sugerido (para o seu mapeamento)

Quando você tiver os JSONs (1) `CnsPremReceb`, (2) Policy Certificate core e (3) Payment request, a gente monta a tabela:

| Campo no arquivo (layout) | Campo em `CnsPremReceb` | Campo no JSON `Policy` (se existir) | Campo no JSON `Payment` (BCP) | Regra/observação |
|---|---|---|---|---|

