# Apólice — Visão geral (Coinsurance)

Esta seção descreve o ciclo de **Prêmio** do cosseguro, do ponto de vista do aceito (BMG),
desde a **emissão/endosso da apólice definitiva** até **recebimento** e **reserva mensal (PPNG)**.

## O que entra e o que sai

### Entradas (do líder)
- **PREMIT**: movimento “macro” (emissão/endosso) com valores consolidados
- **PREMREC**: detalhamento por parcelas (cronograma / valores por parcela)
- **PREMRECEB**: recebimento efetivo (baixa bancária)
- **RESPREM**: reserva/PPNG do mês (posição de fechamento)

### Saídas (BMG / aceito)
- Retornos/RO com visão proporcional e enriquecida (ex.: **PREMAC**, **PREMRECAC**, **PREMRECEBC**, **RESPREMC**)

---

## Fluxos do domínio (Apólice)

### Fluxo principal: Emissão da Apólice Definitiva (Master/Certificate)
➡️ **[Emissão da Apólice (Master/Certificate) — PREMIT + PREMREC](fluxos/emissao-apolice-master-certificate.md)**

### Fluxo financeiro: Recebimento do Prêmio (baixa/pagamento)
➡️ **[Recebimento do Prêmio — PREMRECEB](fluxos/recebimento-premio-premreceb.md)**

---

# Eventos do Fluxo (Gatilho → Arquivo → Efeito)

## Evento 1 — Emissão (criação do Master/Certificate)

- **Gatilho:** emissão inicial da apólice/certificado pelo líder.
- **Arquivo:** `PREMIT` + `PREMREC`
- **Regras de Negócio (visão operacional):**
    - `PREMIT` registra o **movimento macro** (o “o que aconteceu”).
    - `PREMREC` traz o **parcelamento** e habilita o processamento que cria a apólice definitiva.
  - O processamento cria/atualiza **Master Policy** e emite **Certificate Policy** (módulo Policy).
- **Transição de Estado:** Não existente → **Emitida (Master + Certificate)**
- **Impacto financeiro:** **(+) Emissão** (gera base para cobrança/parcelas)

---

## Evento 2 — Endosso com cobrança adicional de prêmio

- **Gatilho:** alteração que **aumenta o prêmio** (cobrança complementar).
- **Arquivo:** `PREMIT` + `PREMREC`
- **Tipo de Movimento (TIPO_MOV):** **102**
- **Regras de Negócio:**
  - Atualiza a apólice/certificado com novo movimento e repercute no parcelamento (quando aplicável).
  - Continua usando `PREMIT` como registro pai e `PREMREC` como execução (parcelas).
- **Transição de Estado:** Emitida → **Endossada (prêmio aumentado)**
- **Impacto financeiro:** **(+)** (aumenta base de cobrança)

---

## Evento 3 — Endosso de restituição de prêmio

- **Gatilho:** alteração que **reduz o prêmio** e gera restituição.
- **Arquivo:** `PREMIT` + `PREMREC`
- **Tipo de Movimento (TIPO_MOV):** **103**
- **Regras de Negócio:**
  - Movimento representa redução/restituição; operacionalmente pode gerar ajustes nas parcelas/valores (dependendo da regra do produto).
- **Transição de Estado:** Emitida → **Endossada (prêmio reduzido)**
- **Impacto financeiro:** **(-)** (reduz base de cobrança / pode gerar estorno)

---

## Evento 4 — Cancelamento da Apólice (com ou sem restituição)

- **Gatilho:** cancelamento do contrato (apólice/certificado).
- **Arquivo:** `PREMIT` + `PREMREC`
- **Tipos de Movimento (TIPO_MOV):**
    - **104:** Cancelamento **com** restituição
    - **106:** Cancelamento **sem** restituição
- **Regras de Negócio:**
  - Cancela a apólice/certificado e encerra o ciclo de cobrança (podendo haver restituição, conforme TIPO_MOV).
- **Transição de Estado:** Emitida/Endossada → **Cancelada**
- **Impacto financeiro:**
  - **104:** **(-)** (reduz/estorna prêmio)
  - **106:** **(0/-)** (sem restituição; efeito depende de parcelas já recebidas e competência)

---

## Evento 5 — Endosso sem movimentação de prêmio

- **Gatilho:** alteração de condições sem mexer em valores de prêmio.
- **Arquivo:** `PREMIT` + `PREMREC`
- **Tipo de Movimento (TIPO_MOV):** **108**
- **Regras de Negócio:**
  - Atualiza dados/condições (ex.: parâmetros, informações cadastrais/contratuais) sem alterar prêmio.
- **Transição de Estado:** Emitida → **Endossada (sem efeito financeiro)**
- **Impacto financeiro:** **(0)** (não altera cobrança)

---

## Evento 6 — Recebimento do prêmio (baixa de parcela)

- **Gatilho:** pagamento efetivo (ex.: boleto liquidado / comprovante de recebimento).
- **Arquivo:** `PREMRECEB`
- **Campos chave (visão de negócio):**
  - Identificadores do movimento (ramo/apólice/endosso/proposta)
  - **Parcela** (`PRESTACAO`, vencimento e data de recebimento)
  - **Valores do título** (`VAL_DOC`, `VAL_COB`, desconto, multa etc.)
- **Regras de Negócio:**
  - PREMRECEB representa o **realizado** (recebido) e é base de conciliação com o emitido/parcelado.
  - Operacionalmente, o sistema pode registrar o pagamento como **Payment Paid** (quando integrado).
- **Transição de Estado:** Parcela em aberto → **Parcela recebida**
- **Impacto financeiro:** **(+) Caixa / Liquidação** (confirma recebimento)

---

## Evento 7 — Reserva mensal (RESPREM / PPNG)

- **Gatilho:** fechamento mensal / apuração de competência do prêmio (risco decorrido vs não decorrido).
- **Arquivo:** `RESPREM`
- **Regras de Negócio:**
  - Representa a **posição mensal** de prêmio não ganho (PPNG) e ajustes de competência.
  - Usado para controle/regulatórios e conciliação de resultado por competência.
- **Transição de Estado:** Competência (mês) → **Reserva apurada/atualizada**
- **Impacto financeiro:** **(+/-)** (ajuste de competência/reserva — não é “caixa”)

---

## Tabela-resumo (Apólice)

| Evento (visão de apólice) | Arquivos | TIPO_MOV (SUSEP) | Impacto principal | Transição principal |
|---|---|---:|---|---|
| **1. Emissão** | `PREMIT` + `PREMREC` | 101 | **(+)** cria base de cobrança | Não existente → Emitida (Master/Certificate) |
| **2. Endosso adicional** | `PREMIT` + `PREMREC` | 102 | **(+)** aumenta prêmio | Emitida → Endossada (+) |
| **3. Restituição** | `PREMIT` + `PREMREC` | 103 | **(-)** reduz prêmio | Emitida → Endossada (-) |
| **4. Cancelamento** | `PREMIT` + `PREMREC` | 104 / 106 | **(-)/(0)** depende restituição | Emitida/Endossada → Cancelada |
| **5. Endosso sem prêmio** | `PREMIT` + `PREMREC` | 108 | **(0)** sem efeito financeiro | Emitida → Endossada (0) |
| **6. Recebimento (baixa)** | `PREMRECEB` | — | **(+) caixa/liquidação** | Parcela em aberto → Recebida |
| **7. Reserva mensal (PPNG)** | `RESPREM` | — | **(+/-) competência/reserva** | Competência → Reserva apurada |

!!! note "Observação importante"
    Os eventos **1–5** (em “Emissão/Endosso”) dependem do processamento que cria/atualiza a **apólice definitiva (Master/Certificate)**.  
    O evento **6** (Recebimento) representa o **realizado** e pode integrar com o motor operacional de pagamentos.  
    O evento **7** (Reserva) é **competência/regulatório** e não representa “caixa”.