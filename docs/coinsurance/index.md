# Coinsurance — Visão geral do fluxo

## Estrutura macro dos arquivos

- **Apólice**: PREMIT, PREMREC, PREMRECEB, RESPREM
- **Sinistro**: SINAV, SINPEND, SINPAG
- **Retornos (RO BMG)**: PREMAC, PREMRECAC, PREMRECEBC, SINAVAC, SINPENDAC, SINPAGAC

## Estados de processamento (padrão no código)

- `PENDING`: registro ingerido e ainda não processado
- `COMPLETED`: processado com sucesso
- `ERROR`: erro no processamento (em alguns serviços o update está marcado como TODO)
- `DUPLICATED`: duplicado (há trechos comentados/TODO)

## Identificação técnica (FileId)

Nos serviços de ingestão, cada linha do CSV vira um registro com `FileId` derivado:

- `FileId = <etag_do_arquivo_no_S3>_<SEQUENCIA>`

Isso garante unicidade por **arquivo + linha**.

!!! note
    A chave do arquivo (S3 key) também é salva como `FileName` (apenas o nome do arquivo).
