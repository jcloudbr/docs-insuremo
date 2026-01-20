# PREMRECEB — Descrição negocial (Coinsurance)

> **Definição curta:** **PREMRECEB** registra o **recebimento/liquidação** do prêmio (por parcela/documento), viabilizando conciliação financeira e baixa.

## O que ele representa no negócio

No negócio, o PREMRECEB materializa o fato financeiro:

- **“o valor foi recebido”** (data de recebimento, valor do documento, desconto/multa/cobrança)
- **por qual documento/parcela** (nº de boleto, nosso número, documento, parcela, vencimento)
- **com quais dados bancários** (banco/agência/conta, identificadores)

## Para que serve no fluxo operacional

- Concilia **emitido (PREMREC)** vs **recebido**.
- Alimenta (ou deveria alimentar) a baixa/liquidação no downstream financeiro (ex.: BCP), conforme implementação.

!!! warning
    No código atual, existem **TODOs e valores hardcoded** em partes do processamento (ex.: busca de master policy). Isso indica fluxo ainda em evolução.
