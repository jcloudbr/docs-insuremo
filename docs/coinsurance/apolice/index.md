# Apólice — Visão geral (Coinsurance)

Esta seção descreve o ciclo de **Prêmio** do cosseguro, do ponto de vista do aceito,
desde a emissão/endosso até recebimento e reserva mensal.

## O que entra e o que sai

### Entradas (do líder)
- **PREMIT**: movimento “macro” (emissão/endosso) com valores consolidados
- **PREMREC**: detalhamento por parcelas (cronograma / valores por parcela)
- **PREMRECEB**: recebimento efetivo (baixa bancária)
- **RESPREM**: reserva/PPNG do mês (posição de fechamento)

### Saídas (BMG / aceito)
- Retornos/RO com visão proporcional e enriquecida (ex.: **PREMRECEBC**, **RESPREMC**)

---

## Fluxos do domínio (Apólice)

### Emissão e Endosso (Apólice Definitiva no módulo Policy)

Este é o fluxo central do bloco **Apólice**, responsável por **criar/atualizar a apólice definitiva**
(**Master Policy + Certificate Policy**) a partir dos arquivos SUSEP:

- **PREMIT (pai)**: registro mestre do movimento
- **PREMREC (motor)**: parcelas e execução operacional (emissão/endosso)

➡️ **Leia o fluxo completo aqui:**  
**[Emissão da Apólice (Master/Certificate) — PREMIT + PREMREC](fluxos/emissao-apolice-master-certificate.md)**

---

## Papel de cada arquivo no negócio

- **PREMIT (pai)**  
  Registro mestre do movimento. Responde:
  - “O que aconteceu com a apólice/certificado?”
  - “Quais os totalizadores e a vigência do movimento?”

- **PREMREC (motor)**  
  Detalha parcelas e viabiliza a execução operacional. Responde:
  - “Como o prêmio foi parcelado?”
  - “Quais vencimentos/valores por parcela?”

- **PREMRECEB (caixa)**  
  Confirma recebimento efetivo. Responde:
  - “Quais parcelas foram pagas, quando e por qual valor?”

- **RESPREM (competência/PPNG)**  
  Controla reserva no fechamento mensal. Responde:
  - “Quanto do prêmio ainda não foi ganho (risco não decorrido) no mês?”

---

## Como usar esta documentação

1) Leia o **fluxo de emissão** (Master/Certificate) para entender a lógica principal de criação/atualização.

2) Em seguida, vá arquivo a arquivo:
  - PREMIT / PREMREC / PREMRECEB / RESPREM

3) Depois, consulte o **De/Para** para ver:
  - campo do arquivo físico → campo persistido
  - normalizações e tipos
  
4) Use as páginas para:
  - troubleshooting de divergências (chaves, status, duplicidade)
  - entendimento do ciclo ponta-a-ponta