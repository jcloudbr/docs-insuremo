# SINPENDAC — Retorno pendências (BMG)

> TODO: detalhar.
>
> Documentação Técnica — Fluxo SINPAGAC


Documentação Técnica — Fluxo SINPAGAC


. SINPAGAC - Sistema de Pagamento de Sinistros

1 Definição
Sistema para reportar sinistros PAGOS/LIQUIDADOS à SUSEP.
1.2 Critérios de Seleção
CritérioValorCaseStatus"03" (Pago/Settled)Filtro de DataInsertTime no período YYYYMMExemplo QueryInsertTime >= "2026-01-01" AND InsertTime <= "2026-01-31"
1.3 Layout do Arquivo
Total de Campos: 14
Separador: Pipe (|)
Encoding: UTF-8
Line Ending: \n
1.4 Estrutura de Campos

Campo SINAVAC	Origem	Campo JSON	Fallback	Formato
vr_mov	índice	SettlementAmount	TotalIncurred, RequestAmount	9999999999.99
tp_sin	default	-	fixo 01 (morte)	2 dígitos
uf_risco	default	-	vazio (não disponível)	2 caracteres
:
Campo	Default		Quando Implementar
TIPO_MOV	01		 ClaimCase.TipoMov
NUM_END	00000000000000000000		ClaimCase.EndorsementNo
TP_SIN	01		 ClaimCase.ClaimType
UF_RISCO	`` 		 ClaimCase.RiskState

1.5 Exemplo de Linha
csv0000000001|03417|202601|03|0993|05908|CBMGV93202600000062|789322900007004000|787722026011602000|16/01/2026|01/01/2026|01/01/2026|5000.00|01
1.6 Características

VR_MOV: Valor REAL pago (ex: 5000.00)
Data Base: Período do pagamento (InsertTime)


2. Especificação de Campos
2.1 Campos Calculados
2.1.1 SEQUENCIA
long sequenceCounter = 1L
String sequence = String.format("%010d", sequenceCounter++)

Contador sequencial
Zero-padded à esquerda (10 dígitos)
Reinicia a cada arquivo

2.1.2 COD_CIA
final String COD_CIA = "03417"

Código da companhia (constante)

2.1.3 DT_BASE
String dtBase = periodYYYYMM  // Ex: "202601"

Período do arquivo no formato YYYYMM
Vem do orquestrador

2.1.4 COD_COSS
final String COD_COSS = "05908"

Código do cosseguro (constante)


2.2 Campos com JOIN (Endorsement)
2.2.1 TIPO_MOV (Campo 4)
Fonte: Endorsement.EndoType

String tipoMovStr = String.format("%02d", tipoMov)
Valores:
EndoTypeTIPO_MOVDescrição"1""01"Emissão"2""02"Renovação"3""03"Alteração/Endosso"4""04"CancelamentoAusente"01"Default

2.3 Campos do ClaimCase
2.3.1 COD_RAMO (Campo 5)
Fonte: ClaimCase.ProductCode 

2.3.4 NUM_END (Campo 9)
Prioridades:

ClaimCase.ExtEndoNo 
Endorsement.LeaderEndorsementNo 
Endorsement.EndoNo 


DT_REG (Campo 10):
String insertTime = requireString(claimDoc, "InsertTime")
String dtReg = toSusepDate(insertTime)
// "2026-01-16T14:25:28" → "16/01/2026"
DT_AVISO (Campo 11):
String noticeTime = requireString(claimDoc, "NoticeTime")
String dtAviso = toSusepDate(noticeTime)
// "2026-01-01" → "01/01/2026"
DT_OCOR (Campo 12):
String accidentTime = requireString(claimDoc, "AccidentTime")
String dtOcor = toSusepDate(accidentTime)
// "2026-01-01" → "01/01/2026"



Valores:
ExtClaimTypeTP_SINDescrição"1""01"Morte"2""02"Invalidez"3""03"Incapacidade Temporária"4""04"Desemprego

2.3.8 UF_RISCO (Campo 15 - SINAVAC apenas)
Fonte: ClaimCase.RiskState
String validateUF(String uf) {
    if (uf == null || uf.isEmpty()) return ""
    String normalized = uf.toUpperCase().trim()
    if (!normalized.matches("[A-Z]{2}")) return ""
    return VALID_UFS.contains(normalized) ? normalized : ""
}

static final Set<String> VALID_UFS = [
    "AC", "AL", "AP", "AM", "BA", "CE", "DF", "ES", "GO", "MA",
    "MT", "MS", "MG", "PA", "PB", "PR", "PE", "PI", "RJ", "RN",
    "RS", "RO", "RR", "SC", "SP", "SE", "TO"
] as Set


**Formato:** 2 caracteres (sigla UF)
**Validação:** Contra lista de UFs brasileiras válidas

---

 3. Fluxo de Processamento

### 3.1 Visão Geral

┌─────────────────────────────────────────────────────────────┐
│ INÍCIO                                                      │
│ Job Scheduler: Todo dia 5 do mês                           │
│ Período: YYYYMM (ex: 202601)                               │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ FASE 1: PROCESSAMENTO SINAVAC                              │
├─────────────────────────────────────────────────────────────┤
│ 1.1 Query ClaimCases (Status "01")                         │
│ 1.2 Preload Endorsements (Batch)                           │
│ 1.3 Gerar CSV                                               │
│ 1.4 Salvar arquivo                                          │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ FASE 2: PROCESSAMENTO SINPAGAC                             │
├─────────────────────────────────────────────────────────────┤
│ 2.1 Query ClaimCases (Status "03")                         │
│ 2.2 Preload Endorsements (Batch)                           │
│ 2.3 Gerar CSV                                               │
│ 2.4 Salvar arquivo                                          │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ FASE 3: ENVIO                                               │
├─────────────────────────────────────────────────────────────┤
│ 3.1 Upload FTP SUSEP                                        │
│ 3.2 Log de auditoria                                        │
│ 3.3 Notificação (email/slack)                              │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ FIM                                                         │
│ Status: Sucesso/Erro                                        │
└─────────────────────────────────────────────────────────────┘        

14 campo
VR_MOV com valor real pago








