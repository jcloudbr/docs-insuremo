# PREMIT — Descrição negocial (Coinsurance)

> **Definição curta:** **PREMIT** é o **cabeçalho/espelho do movimento** de emissão/endosso do prêmio (consolidado), no contexto de cosseguro.

## O que ele representa no negócio

O PREMIT representa o movimento de prêmio emitido relacionado a uma apólice/certificado, trazendo:

- **Chaves principais** (ramo, apólice, endosso, proposta, cia)
- **Datas de referência** (base/competência, proposta, emissão, início e fim de vigência)
- **Valores consolidados do movimento** (prêmio emitido, prêmio cedido, comissões, IOF, custos, adicional fracionamento, pró-labore etc.)

## Para que serve no fluxo operacional

- Dá rastreabilidade do evento: **“o que aconteceu”** com a apólice naquele ciclo.
- Serve como **registro pai** para correlacionar o detalhamento (PREMREC) e suportar conciliação.

## Como ele se complementa com PREMREC

- **PREMIT (pai)**: 1 registro por movimento com o consolidado.
- **PREMREC (filhos)**: N registros por movimento, tipicamente por parcela, e é o que viabiliza o processamento real.

!!! note
    No fluxo interno, o PREMIT costuma entrar como **PENDING** e não “emite apólice” sozinho — quem executa a emissão/alteração é o **PREMREC**.
