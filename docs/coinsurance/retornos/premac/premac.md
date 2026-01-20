# PREMAC — Visão geral (RO BMG)

> **O que é (negócio):** **PREMAC** é o **arquivo de retorno (RO)** que materializa, para fins regulatórios/contábeis, os **valores do cosseguro aceito** (BMG = 03417) para um conjunto de apólices (Master/Certificate) já **persistidas na estrutura definitiva (Policy/PolicyCore)**.
>
> **Como este projeto gera:** a exportação **não lê o PREMIT físico**; ela **consulta o índice de Policy** e carrega o **PolicyCore** para montar **1 linha por Policy**, aplicando regras de cálculo de valores aceitos (participação) e campos fixos (COD_CIA/COD_COSS).

---

## Escopo

- **Arquivo/RO:** `PREMAC`
- **Área:** **Retornos (ROs) → Apólice / Prêmio**
- **Entrada do motor:** **Policy index + PolicyCore** (estrutura definitiva)
- **Saída:** `PREMAC_YYYYMM_<timestamp>.csv` (delimitador `|`)
- **Regras SUSEP / espec:** usadas como **referência de campos e fórmulas**, mas a origem dos dados é a **estrutura definitiva**.

---

## Gatilho e periodicidade

- **Gatilho:** execução do job de export do **período (DT_BASE = YYYYMM)** no fechamento.
- **Período (DT_BASE):** normalmente vem do orquestrador como `YYYYMM` (ex.: `202512`).
- **Filtro por competência (BaseDate no índice):**
  - quando habilitado, o período `YYYYMM` é convertido para **`YYYY-MM-01`** para filtrar `BaseDate` no índice.

---

## Fonte de dados (estrutura definitiva)

### 1) Índice (Policy)
Consulta por lotes com paginação por `entity_id` (keyset):

- `FullOrgCode.multiLineExact = "bmg"`
- `CodCoss.multiLineExact = "05908"`
- `BaseDate = "YYYY-MM-01"` (quando habilitado)

> O resultado do índice entrega o documento mínimo (incluindo `PolicyId`/`entity_id`) para carregarmos o **PolicyCore**.

### 2) PolicyCore (load)
Para cada `policyId` retornado do índice, o serviço faz `load` do core e monta a linha PREMAC.

---

## Campos do PREMAC e origem no PolicyCore

### Identificação (regulatório)

| Campo PREMAC | Origem (Policy/PolicyCore) | Regra |
|---|---|---|
| `SEQUENCIA` | contador local | inicia em 1; formatado/padded |
| `COD_CIA` | constante | sempre `03417` |
| `NUM_PROC` | `NumProcLider` (ou líder em `CoInsuranceInsurerList`) | obrigatório |
| `DT_BASE` | período do job (`YYYYMM`) **ou** `DtBase` do core | padronizar para `YYYYMM` |
| `TIPO_MOV` | `TipoMov` | obrigatório |
| `UF_DEP` | `UFDep` | obrigatório |
| `UF_RISCO` | `UFRisco` | obrigatório |
| `COD_RAMO` | `CodRamoProduto` **(padrão)** | (alternativa: `PolicyCoverage[].CodRamo`) |
| `NUM_APOL` | `CoInsuranceInfoList[0].LeaderPolicyNo` | obrigatório |
| `NUM_END` | `LeaderEndorsementNo` (ou zeros se emissão) | se `TipoMov=101` → `000...0` |
| `COD_COSS` | constante | sempre `05908` (líder) |
| `DT_EMIS` | `IssueDate` | formato SUSEP `YYYYMMDD` |
| `DT_INI_VIG` | `EffectiveDate` | `YYYYMMDD` |
| `DT_FIM_VIG` | `ExpiryDate` | `YYYYMMDD` |

### Valores (regulatório + gerencial)

> O PolicyCore geralmente já carrega valores por cobertura com versões “Lider” e “Cosseguro (Ac)” (ex.: `PrCossAc`, `ComCosAc`, `AdFracCoss`, `LaborAmountCoss`).  
> O PREMAC exporta **totalizadores por Policy**, então faz **soma em `PolicyCoverageList`**.

| Campo PREMAC | Preferência (Core) | Fallback (regra) |
|---|---|---|
| `PR_EMIT` | `GrossPremium` ou `BeforeVatPremium` | (último recurso) somar `OverrideBeforeVatPremium` |
| `PR_COSS_AC` | soma `PolicyCoverage[].PrCossAc` | `PR_EMIT * 0,40` |
| `COM_COSS_AC` | soma `PolicyCoverage[].ComCosAc` | `Commission * 0,40` |
| `AD_FRAC` | soma `PolicyCoverage[].AdFracCoss` | soma `AdFracLider * 0,40` |
| `PRO_LAB` | soma `PolicyCoverage[].LaborAmountCoss` | soma `LaborAmountLider * 0,40` |
| `PRC_COMISS_COFEE` | `CoInsuranceInfoList[0].PrcComissCofee` | default `0,068` (6,8%) |
| `COFEE` | `PR_COSS_AC * PRC_COMISS_COFEE` | idem |

---



## Pontos de atenção (para ajuste do gerador)

1) **Filtro de BaseDate**
- Evite desabilitar `baseDateIndex` “na mão”, senão o export pode trazer **todo o tenant**.

2) **COD_RAMO**
- Defina qual é o “ramo do PREMAC”: `CodRamoProduto` (produto) vs `CodRamo` por cobertura.
- Se houver múltiplos ramos por cobertura, o “por Policy” vira uma decisão (ex.: pegar o principal ou quebrar por ramo).

3) **Movimentos 103–108 (cofee)**
- A regra de “buscar percentual da emissão original” pode ser feita via:
  - `PrcComissCofee` já persistido no core **ou**
  - busca em PREMAC anterior (se vocês decidirem persistir essa referência)
- Como combinamos: **a origem é Policy/PolicyCore**; a especificação é só referência.

---

## Referências

- **Especificação funcional:** `premac/premac-especificacao-funcional.md`
- **Especificação técnica:** `premac/premac-especificacao-tecnica.md`
- **Código (service):** `CoinsurancePremacService.groovy`
