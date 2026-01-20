# PREMIT — Descrição negocial (Coinsurance)

> **Definição curta:** **PREMIT** é o **registro mestre do movimento de prêmio** (emissão/endosso/cancelamento) no cosseguro — um **“espelho consolidado”** do evento no nível **apólice/certificado**, com as **chaves** e **totalizadores** que amarram o movimento.

---

## O que ele representa no negócio

O **PREMIT** representa o **evento macro** que aconteceu com a apólice/certificado naquele ciclo.  
Ele descreve **qual movimento ocorreu**, **em que competência** esse movimento é reportado, **qual apólice/proposta/endosso foi afetado** e **quais totalizadores financeiros** compõem esse movimento.

### 1) Identidade do movimento (chaves)
O PREMIT traz as chaves que identificam univocamente o movimento:

- **Companhia / origem**
  - `COD_CIA` (companhia de referência do arquivo)
  - `DS_ORIGEM` (origem do dado/canal)

- **Chaves do contrato**
  - `COD_RAMO` (ramo SUSEP)
  - `NUM_APOL` (apólice)
  - `NUM_END` (endosso)
  - `NUM_PROP` (proposta)

- **Natureza do evento**
  - `TIPO_MOV` (emissão/endosso/cancelamento etc.)

> **Leitura de negócio:** com esses campos você consegue responder **“o que aconteceu?”** e **“qual contrato foi impactado?”**.

---

### 2) Contexto temporal (datas)
O PREMIT é fortemente “temporal”: ele diz **quando** (competência e vigência) o movimento vale.

- **Competência / base**
  - `DT_BASE` → “mês base”/competência que organiza fechamento, conciliação e relatórios.

- **Lifecycle comercial**
  - `DT_PROP` (data da proposta)
  - `DT_EMIS` (data de emissão do movimento)

- **Vigência do risco**
  - `DT_INI_VIG` (início vigência)
  - `DT_FIM_VIG` (fim vigência)

> **Leitura de negócio:** PREMIT separa bem **competência** (`DT_BASE`) de **vigência de risco** (`DT_INI_VIG/DT_FIM_VIG`).  
> Isso é importante porque: *o caixa pode acontecer em data diferente da competência, e a competência pode acontecer em data diferente da vigência do risco.*

---

### 3) Totalizadores financeiros do movimento (consolidado)
O PREMIT carrega os **valores consolidados** do evento. Ele é a “foto” do movimento na visão macro.

Valores típicos (exemplos do seu OPA/layout):
- `PR_EMIT` → prêmio emitido do movimento
- `PR_COS_CED` → prêmio cedido no cosseguro (parcela cedida)
- `AD_FRAC` → adicional fracionamento
- `CUST_APOL` → custo apólice
- `IOF` → imposto
- `COMIS` / `COMIS_COSS` → comissões
- `PRO_LAB` → pró-labore

> **Leitura de negócio:** são totalizadores usados para:
> - comparar com o detalhamento (`PREMREC`)
> - conciliar com o recebido (`PREMRECEB`)
> - explicar divergência de saldo / diferença de fechamento

---

### 4) Identidade do segurado e contexto comercial
Além do movimento e valores, o PREMIT ancora **quem é o contrato** e “qual produto”.

- **Segurado**
  - `CPF_SEG` (CPF/CNPJ do segurado)
  - `QTD_SEG` (quantidade de segurados quando aplicável)
  - `QTD_TOM` (indicador de tomador quando aplicável)

- **Produto e canal (enriquecimento de negócio)**
  - `DS_GRUPO_RAMO` (grupo de ramo)
  - `CD_PRODUTO` / `DS_PRODUTO`
  - `CD_BU` / `DS_BU`
  - `CD_OPERACAO` / `DCR_TIP_OPER`
  - `CD_CARTEIRA` / `DS_CARTEIRA`
  - `CD_SUCURSAL`
  - `FG_GEB` (flag operacional)

> **Leitura de negócio:** esses campos permitem classificação, segmentação e validações de consistência (produto, BU, operação, carteira).

---

## Para que serve no fluxo operacional

### 1) Rastreabilidade (“o que aconteceu”)
O PREMIT responde rapidamente:
- **Qual evento** (TIPO_MOV) ocorreu
- **Em qual contrato** (ramo/apólice/endosso/proposta)
- **Em qual competência** (DT_BASE)
- **Com quais totalizadores** (prêmio, IOF, comissão, custos…)

Isso é usado para:
- auditoria e trilha regulatória
- conciliação por competência
- investigação de divergências (emitido x parcelado x recebido)

---

### 2) Registro “pai” do detalhamento (PREMREC)
No seu fluxo de Coinsurance, o PREMIT funciona como “pai” do PREMREC:

- PREMIT consolida o movimento
- PREMREC detalha a cobrança (parcelas) e viabiliza a execução operacional

**Na prática:**
- o sistema ingere PREMIT e guarda `PENDING`
- depois usa PREMIT como base para correlacionar e processar PREMREC

---

### 3) Validações de consistência e governança do lote
Mesmo sem colocar OPA no repo, o conceito de negócio é:

- PREMIT garante que o movimento possui:
  - chaves mínimas
  - datas obrigatórias
  - códigos de ramo coerentes
  - segurado identificado
  - totalizadores presentes

**Por que isso importa?**
- evita criar “apólice definitiva” com dados incompletos
- garante que o lote de integração tem consistência regulatória mínima

---

## Como o PREMIT se complementa com PREMREC

### Relação pai/filho (visão de domínio)
- **PREMIT (pai):** 1 registro por movimento **macro** (evento + totalizadores)
- **PREMREC (filhos):** N registros por movimento (parcelas / plano de cobrança)

### O que cada um resolve
- PREMIT responde: **“qual foi o movimento e qual o total?”**
- PREMREC responde: **“como esse total foi parcelado/cobrado?”**

### O que dá errado se um faltar
- **Sem PREMIT:** você perde o “contexto do evento” e os totalizadores para auditoria/conciliação.
- **Sem PREMREC:** você tem o evento, mas não consegue operar o ciclo de cobrança/parcelas com consistência.

---

## Relações importantes com outros arquivos (visão de negócio)

### PREMRECEB (recebido)
- **PREMREC** define o “planejado/parcelado”
- **PREMRECEB** confirma o “realizado/recebido”
- **PREMIT** ancora o “evento macro” que dá contexto aos dois

### RESPREM (competência/PPNG)
- RESPREM usa o universo de apólices e vigências para apurar competência/reserva.
- O PREMIT fornece parte do contexto (movimento + vigência + competência) para rastreabilidade.

---

## Status no fluxo interno (visão operacional)

- **Entrada:** PREMIT normalmente é gravado como `PENDING`.
- **Processamento real (emissão/endosso da apólice definitiva):** acontece no motor do **PREMREC**.
- **Saída esperada:** quando o movimento é processado com sucesso, PREMIT tende a ir para `COMPLETED` junto dos PREMREC relacionados (quando a atualização de status está ativa).

!!! note "Nota operacional"
    O PREMIT **não “emite” apólice sozinho**. Ele é o registro mestre do movimento, usado para correlação e conciliação.
    A criação/alteração da apólice definitiva (Master/Certificate) ocorre via processamento do **PREMREC**.

---

## Checklist de entendimento (tradução bem objetiva)

- PREMIT = **evento macro + totalizadores + vigência + competência**
- Ele responde: “**o que aconteceu** e **qual o total** do movimento?”
- Ele ancora: **auditoria**, **conciliação** e **correlação** com PREMREC/PREMRECEB