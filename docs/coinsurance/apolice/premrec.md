# PREMREC — Descrição negocial (Coinsurance)

> **Definição curta:** **PREMREC** é o **detalhe operacional do movimento**: desdobra o prêmio em **parcelas** e viabiliza o processamento real de emissão/endosso.

## O que ele representa no negócio

O PREMREC descreve, para o mesmo movimento da apólice/certificado:

- **Parcelamento** (nº da parcela, quantidade total, datas relevantes)
- **Valores por parcela** (prêmio, IOF, custos, adicionais, comissão etc.)

## Para que serve no fluxo operacional

- Materializa o **plano de cobrança** do movimento.
- Permite que o sistema monte as estruturas internas de **apólice/certificado/endosso**, incluindo:
  - validações de segurado (party) e produto
  - criação de **Master Policy** e **Certificate Policy**
  - processamento de **endossos** conforme tipo de movimento

## Complementaridade com PREMRECEB

O **PREMRECEB** chega depois informando o **recebimento** (baixa) e é cruzado com o parcelamento definido aqui.
