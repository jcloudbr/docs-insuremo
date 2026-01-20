# Template de Documentação

Este repositório consolida a documentação funcional e técnica do fluxo **Coinsurance / SUSEP (BMG)**, com foco em:

- **Descrição negocial** do que cada arquivo representa
- **Fluxo operacional** (ingestão → persistência → processamento → retorno)
- **Estruturas de dados** e **chaves de correlação**
- **De/Para** (campo do arquivo físico → como foi salvo/persistido)

!!! note "O que NÃO entra aqui"
    As **policies OPA/Velora** e os validadores ficam em um repositório próprio.
    Aqui mantemos **a descrição do comportamento de negócio**, as chaves e os mapeamentos.

## Como navegar

- **Design e Arquitetura**: visão do sistema e padrões de documentação.
- **Coinsurance / SUSEP (BMG)**:
  - **Apólice**: PREMIT, PREMREC, PREMRECEB, RESPREM
  - **Sinistro**: SINAV, SINPEND, SINPAG
  - **Retornos / RO BMG**: arquivos gerados pela BMG (ex.: PREMRECEBC, RESPREMC)

## Execução local (MkDocs)

```bash
pip install mkdocs-material
mkdocs serve
```
