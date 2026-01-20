# RESPREM — Descrição negocial (Coinsurance)

> **Definição curta:** **RESPREM** é o arquivo de **reserva/competência do prêmio** no fechamento mensal — a “posição do mês” que indica quanto do prêmio está **ganho** vs **não ganho (PPNG)**, suportando apuração contábil/regulatória no contexto de cosseguro.

---

## O que ele representa no negócio

O **RESPREM** representa a **fotografia mensal** do prêmio em termos de **competência** (resultado) e **reserva técnica**.

Em linguagem de negócio, ele responde:
- “Quanto do prêmio já foi **ganho** até o mês-base?”
- “Quanto ainda está **não ganho** (risco a decorrer) e precisa ficar como **reserva**?”
- “Qual a posição consolidada por apólice/certificado e suas chaves?”

> **Leitura de negócio:** diferente de PREMRECEB (caixa) e PREMREC (parcelamento), o RESPREM é **competência**: ele existe para refletir o **risco decorrido** e a **PPNG** no mês.

---

## Por que ele existe (visão regulatória e operacional)

### Competência ≠ Caixa
No seguro, o resultado é reconhecido por **competência** (risco ao longo do tempo), e não apenas pelo recebimento.

- Você pode ter **prêmio recebido** (caixa) sem estar totalmente **ganho** (risco ainda não decorrido).
- Você pode ter **prêmio emitido** sem ter sido **recebido** ainda.

O RESPREM é a peça que amarra:
- **vigência do risco** (tempo decorrido)
- **prêmio relacionado**
- **posição de reserva** no mês-base

---

## O que ele contém (visão conceitual)

### 1) Identidade da posição (mês-base)
O RESPREM sempre está associado a um **mês-base** (competência):

- `DT_BASE` (ou equivalente) → mês de referência do fechamento

> **Uso negocial:** é o “carimbo” do fechamento: posição do prêmio no mês X.

---

### 2) Chaves do contrato (apólice/certificado)
Para identificar sobre qual contrato a posição está sendo calculada, tipicamente inclui:

- ramo (`COD_RAMO`)
- apólice (`NUM_APOL`)
- endosso (`NUM_END`) quando aplicável
- proposta/certificado (`NUM_PROP`)
- segurado / produto (dependendo do layout do seu caso)

> **Uso negocial:** permite “posicionar” a reserva por contrato e cruzar com PREMIT/PREMREC.

---

### 3) Vigência e parâmetros de competência
RESPREM costuma trazer:
- início e fim de vigência (base para cálculo de risco decorrido)
- datas/indicadores necessários para determinar o “quanto do risco já passou”

> **Leitura de negócio:** sem vigência, não existe “não ganho”; o tempo é o motor.

---

### 4) Valores de reserva e competência (PPNG)
O arquivo expressa valores como:
- **Prêmio ganho no mês / acumulado**
- **Prêmio não ganho (PPNG)**
- **ajustes** (quando há mudanças de vigência, cancelamentos, endossos etc.)

> **Leitura de negócio:** o RESPREM é a visão do “quanto ainda está a decorrer” e por isso deve ficar reservado.

---

## Para que serve no fluxo operacional

### 1) Fechamento mensal e base de relatórios
O RESPREM é o insumo para:
- fechamento por competência (mês-base)
- relatórios regulatórios/contábeis
- conciliação de saldo e consistência com outras posições (emitido, recebido, cancelado)

---

### 2) Consistência entre “movimento” e “posição”
Enquanto PREMIT/PREMREC descrevem **movimentos** (eventos):
- emissão
- endosso
- cancelamento

o RESPREM descreve **posição**:
- “como ficou a reserva no mês X considerando todos os eventos e a vigência”

> **Regra mental:** movimentos mudam o estado; RESPREM mostra o resultado consolidado do mês.

---

### 3) Integração com geração de RO mensal
No seu fluxo Coinsurance, o RESPREM normalmente:
- alimenta o cálculo/geração de retorno mensal (RO BMG)
- e serve como base de consistência para o aceito reportar a posição proporcional.

---

## Relações importantes com outros arquivos (visão de negócio)

### PREMIT / PREMREC (emitido e parcelado)
- PREMIT/PREMREC dizem “o que foi emitido e como foi parcelado”
- RESPREM diz “quanto desse prêmio está ganho vs não ganho no mês-base”

> Um cancelamento/endosso (movimento) altera a base do RESPREM daquele mês e dos seguintes.

---

### PREMRECEB (recebido / caixa)
- PREMRECEB diz “o que entrou de caixa”
- RESPREM diz “o que pode ser reconhecido como resultado (competência)”

> Pode haver caixa sem competência (ainda não ganhou) e competência sem caixa (emitido a receber).

---

## Como interpretar RESPREM no dia a dia (tradução objetiva)

- RESPREM = **“posição do prêmio no mês”** (competência/reserva)
- Responde:
  - “qual é a **PPNG** da apólice/certificado no mês-base?”
  - “qual parte do prêmio já pode ser reconhecida como **ganha**?”
- Não responde “pagou?” (isso é PREMRECEB)

---

## Checklist de entendimento rápido

- Sempre ligado a um **mês-base** (fechamento)
- Reflete **competência** e **PPNG** (reserva), não caixa
- Amarra contrato + vigência + valores de posição
- Serve para **fechamento**, **regulatório** e **conciliação**