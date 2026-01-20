# PREMREC — Descrição negocial (Coinsurance)

> **Definição curta:** **PREMREC** é o **detalhe operacional do movimento de prêmio**, normalmente no nível de **parcelas**, que materializa **como o prêmio foi (ou será) cobrado** e viabiliza o **processamento real** de emissão/endosso da **apólice definitiva** (Master/Certificate).

---

## O que ele representa no negócio

O **PREMREC** representa o **desdobramento do movimento** (registrado no PREMIT) em informações que suportam a operação de cobrança e consistência financeira.

Em linguagem de negócio, ele descreve:
- **Qual contrato/movimento** está sendo processado (mesmas chaves do PREMIT)
- **Como o prêmio se distribui por parcela** (número e quantidade de parcelas)
- **Quais datas e valores** compõem cada parcela (competência, emissão, vigência e valores por parcela)

> **Leitura de negócio:** se o PREMIT diz “o que aconteceu e quanto totaliza”, o PREMREC diz “como isso vira cobrança, parcela a parcela”.

---

## O que ele contém (visão conceitual)

### 1) Identidade do movimento (chaves)
O PREMREC repete as chaves que conectam o detalhe ao movimento macro:

- `COD_CIA` (companhia)
- `DT_BASE` (competência/base)
- `TIPO_MOV` (tipo de movimento)
- `COD_RAMO` (ramo SUSEP)
- `NUM_APOL` (apólice)
- `NUM_END` (endosso)
- `NUM_PROP` (proposta)
- `DT_PROP` (data proposta)

> **Por que repetir?** Para o arquivo ser autocontido e para garantir correlação com PREMIT e demais fluxos.

---

### 2) Parcelamento (o “plano de cobrança”)
Este é o coração do PREMREC no negócio:

- `PRESTACAO` → número da parcela (ex.: 1, 2, 3…)
- `QTDE_PREST` → quantidade total de parcelas (ex.: 12)

> **Leitura de negócio:** “este movimento será cobrado em N parcelas e esta linha é a parcela X”.

---

### 3) Datas operacionais por parcela
O PREMREC traz datas usadas para cobrança, vigência e conciliação:

- `DT_EMIS_PR` (data de emissão do prêmio/parcela)
- `DT_INI_VIG` / `DT_FIM_VIG` (vigência do risco)

Dependendo do desenho do produto, também pode ser interpretado como:
- referência para cálculo de proporcionalidade/competência
- validação “não cobrar fora da vigência”

---

### 4) Valores por parcela (micro totalizadores)
O PREMREC traz valores no nível de parcela, que explicam e sustentam o consolidado do PREMIT:

Exemplos comuns no layout/validações:
- `PR_EMIT` (prêmio emitido da parcela)
- `AD_FRAC` (adicional fracionamento)
- `CUS_APOL` / `CUST_APOL` (custo apólice)
- `IOF` (imposto)
- `OFER` (quando aplicável)
- `PERC_COSS` (percentual de cosseguro, quando informado)

> **Leitura de negócio:** esses valores, somados, devem bater com o PREMIT (ou pelo menos permitir explicar diferenças).

---

### 5) Contexto de produto/canal (enriquecimento)
Assim como no PREMIT, o PREMREC normalmente carrega também:

- `DS_GRUPO_RAMO`
- `CD_PRODUTO` / `DS_PRODUTO`
- `CD_BU` / `DS_BU`
- `CD_OPERACAO` / `DCR_TIP_OPER`
- `CD_CARTEIRA` / `DS_CARTEIRA`
- `CD_SUCURSAL`
- `FG_GEB`
- `DS_ORIGEM`

> **Uso negocial:** classificação, roteamento, validação de consistência e auditoria.

---

## Para que serve no fluxo operacional

### 1) Materializar o parcelamento para cobrança e conciliação
O PREMREC habilita:
- conferência do “planejado” (parcelas) versus “realizado” (PREMRECEB)
- análise de inadimplência (se existir)
- reprocessamento e controle por parcela (quando aplicável)

---

### 2) Viabilizar a emissão/endosso da apólice definitiva (Master/Certificate)
No fluxo interno de Coinsurance, o PREMREC é o **motor operacional**:

- ele é processado correlacionado ao PREMIT (pai)
- executa as validações operacionais (party/produto/apólice existente)
- cria/atualiza a apólice definitiva no módulo **Policy**:
  - **Master Policy** (pré-condição)
  - **Certificate Policy** (o certificado emitido)
- processa endossos (quando `TIPO_MOV` indicar)

> **Em linguagem de negócio:** o PREMREC é “o arquivo que faz a apólice acontecer” no sistema.

---

### 3) Base para downstream financeiro (fees / integrações)
Como o PREMREC é o que “constrói” a apólice e define o parcelamento, ele também suporta:
- integração de pós-emissão (ex.: taxas/fees)
- consistência de dados para etapas posteriores (recebimento, reserva)

---

## Relação com PREMIT (pai) e PREMRECEB (recebido)

### PREMIT x PREMREC
- **PREMIT:** movimento macro (evento + totalizadores)
- **PREMREC:** detalhamento (parcelas + valores por parcela)

### PREMREC x PREMRECEB
- **PREMREC:** diz “como foi parcelado/cobrado”
- **PREMRECEB:** diz “o que foi efetivamente recebido”

> **Conciliação natural:** parcela X do PREMREC deve ter correspondência no PREMRECEB quando liquidada.

---

## Leitura de TIPO_MOV no contexto do PREMREC (negócio)

O `TIPO_MOV` influencia o sentido do movimento e como tratar o evento:

| TIPO_MOV | Significado negocial | Efeito típico |
|---:|---|---|
| 101 | Emissão | Cria Master/Certificate e estrutura base |
| 102 | Cobrança adicional | Aumenta prêmio (ajuste para cima) |
| 103 | Restituição | Reduz prêmio (ajuste para baixo) |
| 104 / 106 | Cancelamento apólice | Encerra contrato (com/sem restituição) |
| 105 / 107 | Cancelamento endosso | Reverte um endosso (com/sem restituição) |
| 108 | Endosso sem prêmio | Ajuste cadastral/contratual sem efeito financeiro |

!!! note "Nota"
    A leitura acima é negocial (layout). O comportamento exato depende do motor de endosso/integração no sistema.

---

## Como identificar o PREMREC no dia a dia (tradução objetiva)

- PREMREC = **“parcela + valores da parcela”** para um movimento de apólice
- Responde: **“como o prêmio foi parcelado?”**
- Serve para:
  - **emitir/endossar** apólice definitiva (Master/Certificate)
  - **operar cobrança** e **conciliar com recebimento (PREMRECEB)**

---

## Checklist de entendimento rápido

- PREMREC repete as **chaves** do PREMIT para correlação
- PREMREC traz o **parcelamento** (`PRESTACAO`, `QTDE_PREST`)
- PREMREC traz **valores por parcela** (micro totalizadores)
- PREMREC é o **motor operacional** da emissão/endosso no fluxo interno