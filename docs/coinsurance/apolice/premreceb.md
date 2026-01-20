# PREMRECEB — Descrição negocial (Coinsurance)

> **Definição curta:** **PREMRECEB** é o arquivo de **recebimento efetivo do prêmio** (baixa financeira), normalmente no nível de **parcela/título**, registrando **quando** e **quanto** foi pago e os **identificadores bancários** do recebimento.

---

## O que ele representa no negócio

O **PREMRECEB** representa o **fato gerador de caixa** no ciclo de prêmio do cosseguro:  
**a parcela do prêmio foi liquidada** (ex.: boleto pago).

Ele descreve, em linguagem de negócio:
- **Qual contrato/movimento** está sendo baixado (ramo/apólice/endosso/proposta e tipo de movimento)
- **Qual parcela** foi paga (número da parcela e datas)
- **Como foi pago** (identificadores bancários do título/boleto)
- **Quais valores** foram liquidados (valor do documento, desconto, multa, valor cobrado)

> **Leitura de negócio:** se o PREMREC é o “planejado/cobrado (parcelas)”, o PREMRECEB é o “realizado/recebido”.

---

## O que ele contém (visão conceitual)

### 1) Identidade do movimento (chaves)
O PREMRECEB traz as chaves que ligam o recebimento ao contrato e ao evento:

- `COD_CIA` (companhia)
- `DT_BASE` (competência/base do registro)
- `TIPO_MOV` (tipo do movimento relacionado)
- `COD_RAMO` (ramo SUSEP)
- `NUM_APOL` (apólice)
- `NUM_END` (endosso)
- `NUM_PROP` (proposta)
- `DT_PROP` (data da proposta)

> **Uso negocial:** correlação com PREMIT/PREMREC e rastreabilidade do recebimento por movimento.

---

### 2) Identificação da parcela e datas de baixa
O PREMRECEB identifica a parcela liquidada e suas datas:

- `PRESTACAO` → número da parcela (ex.: 1/12, 2/12…)
- `DT_VEN_PRE` → data de vencimento
- `DT_REC_PRE` → data de recebimento (liquidação)

> **Leitura de negócio:** “parcela X venceu em Y e foi paga/liquidada em Z”.

---

### 3) Identificadores bancários do título (boleto)
Elementos que permitem conciliação com cobrança e auditoria:

- `NUM_BOL` → número do boleto (quando existente)
- `NUM_DOC` → número do documento/título
- `NUM_NOS` → nosso número
- `NUM_BAN` → código/número do banco
- `NUM_AGE` → agência
- `CNPJ_BAN` → documento do banco (CNPJ)

> **Uso negocial:** conciliar com a baixa bancária e garantir unicidade do recebimento (evitar duplicidade).

---

### 4) Valores do recebimento (caixa)
O PREMRECEB registra valores relacionados à liquidação do título:

- `VAL_DOC` → valor do documento
- `VAL_DESC` → valor de desconto
- `VAL_MUL` → valor de multa
- `VAL_COB` → valor cobrado/recebido (valor efetivamente liquidado)

> **Leitura de negócio:** o que interessa para o “caixa” normalmente é `VAL_COB` (ou a combinação de doc+ajustes conforme regra).

---

### 5) Contexto de produto/canal (enriquecimento)
Também aparecem campos de categorização do negócio:

- `DS_GRUPO_RAMO`
- `CD_PRODUTO` / `DS_PRODUTO`
- `CD_BU` / `DS_BU`
- `CD_OPERACAO` / `DCR_TIP_OPER`
- `CD_CARTEIRA` / `DS_CARTEIRA`
- `CD_SUCURSAL`
- `FG_GEB`
- `DS_ORIGEM`

> **Uso negocial:** classificação, roteamento, relatórios e conciliação por produto/canal.

---

## Para que serve no fluxo operacional

### 1) Confirmar a liquidação do prêmio (evento de caixa)
No ciclo de vida de prêmio, PREMRECEB é o que permite afirmar:
- “**o dinheiro entrou**” (ou foi liquidado no agente arrecadador)

Isso habilita:
- conciliação emitido x recebido
- controle de inadimplência (quando comparado ao PREMREC)
- base para repasses, comissionamentos e controles internos (conforme desenho do produto)

---

### 2) Conciliação com PREMREC (planejado/cobrado)
O caso clássico de uso:

- PREMREC diz: “parcela X existe, vence em Y, vale Z”
- PREMRECEB diz: “parcela X foi recebida em W, valor liquidado foi V”

> **Conceito-chave:** PREMRECEB é o “espelho do extrato/baixa”, e PREMREC é o “espelho do plano de cobrança”.

---

### 3) Materialização do recebimento em sistemas operacionais (pagamento)
Operacionalmente, o PREMRECEB pode ser usado para:
- registrar o recebimento como um **pagamento efetivo** (ex.: `Payment` com status **Paid**)
- vinculado à **apólice/certificado** e, quando aplicável, ao **endosso**

> **Leitura de negócio:** PREMRECEB vira o “registro de pagamento” no sistema operacional.

---

## Relações importantes com outros arquivos (visão de negócio)

### PREMIT (movimento macro)
- PREMIT ancora “o evento” (emissão/endosso/cancelamento) e seus totalizadores.
- PREMRECEB ancora “o caixa” decorrente daquele universo de cobrança.

### PREMREC (parcelas)
- PREMREC define o parcelamento e valores planejados.
- PREMRECEB confirma o pagamento/baixa por parcela.

### RESPREM (competência / PPNG)
- RESPREM trata a reserva/competência.
- PREMRECEB é caixa (pode acontecer em data diferente da competência do prêmio).

---

## Como identificar o PREMRECEB no dia a dia (tradução objetiva)

- PREMRECEB = **“parcela paga” + “dados bancários do título” + “valor liquidado”**
- Responde:
  - “**qual parcela** foi paga?”
  - “**quando** foi paga?”
  - “**quanto** foi liquidado?”
  - “**qual título/boleto** foi baixado?”

---

## Checklist de entendimento rápido

- Tem as **chaves** do contrato/movimento (ramo/apólice/endosso/proposta)
- Tem a **parcela** (`PRESTACAO`) e datas (`DT_VEN_PRE`, `DT_REC_PRE`)
- Tem **dados bancários** do título (nosso número, doc, boleto, banco, agência)
- Tem os **valores do recebimento** (`VAL_DOC`, `VAL_DESC`, `VAL_MUL`, `VAL_COB`)
- É o evento que fecha a pergunta: **“o prêmio foi recebido?”**