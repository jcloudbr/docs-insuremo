# SINAVAC — Retorno de sinistro (BMG)

> TODO: detalhar.
>
SINAVAC — Visão geral (Sinistros Avisados BMG)
O que é (negócio)
SINAVAC é o arquivo de retorno que materializa, para fins regulatórios/contábeis, os sinistros avisados (FNOL - First Notice of Loss) do cosseguro aceito (BMG = 03417) para um conjunto de sinistros já persistidos na estrutura definitiva (ClaimCase).
Como este projeto gera: a exportação consulta o índice de ClaimCase e faz JOIN com o índice de Endorsement para montar 1 linha por sinistro avisado, aplicando regras de mapeamento de campos e validações SUSEP.

Escopo
AspectoDescriçãoArquivo/ROSINAVACÁreaRetornos (ROs) → Sinistros / AvisoEntrada do motorClaimCase index + Endorsement indexSaídaSINAVAC_YYYYMM_<timestamp>.csv (delimitador |)Regras SUSEP / especUsadas como referência de campos obrigatórios e formatos

Gatilho e periodicidade
Gatilho:
Execução do job de export do período (DT_BASE = YYYYMM) no fechamento mensal.
Período (DT_BASE):
Normalmente vem do orquestrador como YYYYMM (ex.: 202601).
Filtro por competência (NoticeTime):
Quando habilitado, o período YYYYMM é convertido para range de datas:

Início: YYYY-MM-01
Fim: último dia do mês (ex.: 2026-01-31)
Campo filtrado: NoticeTime (data em que o sinistro foi avisado)
Fonte de dados (estrutura definitiva)

1) Índice ClaimCase (Principal)
Consulta por lotes com paginação por entity_id (keyset):
Filtros:

CaseStatus = "01" (Sinistro Avisado)
NoticeTime >= "YYYY-MM-01" (início do período)
NoticeTime <= "YYYY-MM-DD" (último dia do período)

Ordenação:

entity_id ASC (keyset pagination)
O resultado do índice entrega:
Documento completo do ClaimCase com todos os campos necessários
PolicyNo (chave para JOIN com Endorsement)

2) Índice Endorsement (JOIN)
Para cada batch de ClaimCases:
Extrai PolicyNo únicos
Faz query batch no índice Endorsement
Cria Map de lookup PolicyNo → Endorsement

Filtros:
PolicyNo IN [lista de PolicyNos únicos]
Objetivo:
Obter EndoType (campo crítico para TIPO_MOV)
Obter campos backup: LeaderPolicyNo, LeaderEndorsementNo, EndoNo

Campos do SINAVAC 
Identificação 
Documentação	Código
"Query ClaimCase com Status '01'"	queryClaimCaseIndexBatch(periodYYYYMM, "01", cursorEntityId)
"Preload Endorsements (batch)"	preloadEndorsements(claimCaseIndexDocs)
"JOIN via PolicyNo"	Map<String, Map<String, Object>> endorsementMap
"15 campos SUSEP"	Consts.HEADER = ["SEQUENCIA","COD_CIA",...,"UF_RISCO"]
TIPO_MOV: Endorsement.EndoType	dto.TipoMov = extractTipoMov(endorsementDoc)
COD_RAMO: ProductCode derivado	dto.CodRamo = deriveBranchCode(getString(claimCaseDoc, "ProductCode"))
NUM_SIN: ClaimCase.ClaimNo	dto.NumSin = requireString(claimCaseDoc, "ClaimNo")
NUM_APOL: Prioridade em cascata	dto.NumApol = extractNumApol(claimCaseDoc, endorsementDoc)
DT_AVISO: ClaimCase.NoticeTime	dto.DtAviso = DateUtils.toSusepDateDdMmYyyy(requireString(claimCaseDoc, "NoticeTime"))


Resumo da Correspondência:	
	
Seção da Documentação	Código Correspondente
Visão Geral	// SINAVAC (SUSEP) - Sistema de Aviso de Sinistros
Gatilho e Periodicidade	void process(), filtros de data
Fonte de Dados	queryClaimCaseIndexBatch(), preloadEndorsements()
Campos e Origens	extractSinavacRow(), todos os extract*()
Mapeamentos	deriveBranchCode(), mapClaimTypeToCode(), etc
Validações	validateUF(), requireString()
Conversão de Datas	DateUtils.toSusepDateDdMmYyyy()
Formatação CSV	buildCsvLine()
Fluxo de Processamento	exportSinavac()
Performance	Batch queries, keyset pagination
Error Logging	ErrorLogger class



Mapeamentos específicos
1. TIPO_MOV (Campo 4)
Fonte: Endorsement.EndoType
EndoTypeTIPO_MOVDescrição"1""01"Emissão"2""02"Renovação"3""03"Alteração/Endosso"4""04"CancelamentoAusente"01"Default

2. COD_RAMO (Campo 5)
Fonte: ClaimCase.ProductCode (derivado)
ProductCodeCOD_RAMODescrição SUSEPBMGAP82*0993Vida em GrupoBMGAP83*0118PrestamistaBMGV93*0993Vida*AP*0161Acidentes PessoaisOutrosPrimeiros 4 dígitosGenérico

3. NUM_APOL (Campo 8) - Prioridade em Cascata

ClaimCase.ExtPolicyNo (preferencial)
Endorsement.LeaderPolicyNo (backup)
ClaimCase.PolicyNo (fallback)

Padding: 20 caracteres (zeros à direita)

4. NUM_END (Campo 9) - Prioridade em Cascata

ClaimCase.ExtEndoNo (preferencial)
Endorsement.LeaderEndorsementNo (backup)
Endorsement.EndoNo (fallback)
Zeros (se todos ausentes)

Padding: 20 caracteres (zeros à direita)

5. VR_MOV (Campo 13) - Prioridade em Cascata

ClaimCase.SettlementAmount (preferencial)
ClaimCase.TotalIncurred (backup)
ClaimCase.RequestAmount (fallback)
0.00 (default)

Observação: Para SINAVAC (status "01"), geralmente é 0.00 pois o sinistro foi apenas avisado, não liquidado.

6. TP_SIN (Campo 14)
Fonte: ClaimCase.ExtClaimType
ExtClaimTypeTP_SINDescrição"1" ou "DEATH" ou "MORTE""01"Morte"2" ou "DISABILITY" ou "INVALIDEZ""02"Invalidez"3" ou "TEMPORARY_DISABILITY" ou "INCAPACIDADE""03"Incapacidade Temporária"4" ou "UNEMPLOYMENT" ou "DESEMPREGO""04"DesempregoAusente"01"Default

7. UF_RISCO (Campo 15)
Fonte: ClaimCase.RiskState
Validação: Contra lista de UFs válidas (27 estados brasileiros)
Formato: 2 caracteres maiúsculos
Lista válida:
AC, AL, AP, AM, BA, CE, DF, ES, GO, MA,
MT, MS, MG, PA, PB, PR, PE, PI, RJ, RN,
RS, RO, RR, SC, SP, SE, TO

Conversão de datas
Formato de entrada (ISO 8601):

"2026-01-16T14:25:28" (com timestamp)
"2026-01-01" (apenas data)

Formato de saída (SUSEP):

"16/01/2026" (dd/MM/yyyy)

Regra:

Remove timestamp (se existir)
Converte YYYY-MM-DD → dd/MM/yyyy
Pontos de atenção (para ajuste do gerador)
1. Filtro de NoticeTime

Evite desabilitar o filtro de data, senão o export pode trazer todos os sinistros avisados do tenant
Use range de datas (gte/lte) para evitar problemas com diferentes fusos horários

2. JOIN com Endorsement

Batch preload é essencial para performance (evita N+1 queries)
Map lookup O(1) garante processamento rápido
Fallback para default se Endorsement não for encontrado (TIPO_MOV = "01")

3. Campos obrigatórios

ClaimNo, PolicyNo, InsertTime, NoticeTime, AccidentTime são obrigatórios
Validar presença antes de processar linha
Logar erro se campo obrigatório estiver ausente

4. UF_RISCO

Validar contra lista de UFs válidas
Retornar vazio se inválido (não bloquear a linha)
Normalizar para maiúsculas

5. VR_MOV para SINAVAC

Geralmente 0.00 (sinistro avisado, não pago)
Aceitar valores não-zero se houver (reserva inicial)
Formato decimal: 0.00 (ponto, Locale.US)


Fluxo de processamento

│ PASSO 1: Query ClaimCase (Batch)                           │

│ - Filtro: CaseStatus = "01"                                │
│ - Filtro: NoticeTime no período YYYYMM                     │
│ - Paginação: entity_id (keyset)                            │
│ - Retorna: Lista de ClaimCases (1000 por batch)            │

                        ↓

│ PASSO 2: Preload Endorsements (Batch)                      │

│ - Extrai: PolicyNos únicos do batch                        │
│ - Query: Endorsement IN [PolicyNos]                        │
│ - Cria: Map<PolicyNo, Endorsement>                         │
│ - Performance: 1 query para todo o batch                   │

                        ↓

│ PASSO 3: Processar Cada ClaimCase                          │

│ Para cada ClaimCase:                                        │
│   1. Busca Endorsement no Map (O(1))                       │
│   2. Extrai campos do ClaimCase                            │
│   3. Extrai campos do Endorsement (se existe)              │
│   4. Aplica prioridades em cascata                         │
│   5. Valida campos obrigatórios                            │
│   6. Converte formatos (datas, valores)                    │
│   7. Gera linha CSV                                         │

                        ↓

│ PASSO 4: Atualizar Cursor e Continuar                      │

│ - Cursor = last(entity_id) + 1                             │
│ - Continua até: batch vazio ou < PAGE_SIZE                 │

                        ↓

│ PASSO 5: Salvar Arquivo e Logs                             │

│ - Arquivo CSV: S3 bucket "export"                          │
│ - Log de erros: S3 bucket "error" (se houver)              │
│ - Estatísticas: Total lido, OK, Erros                      │


Performance
Queries por execução:

2 queries por batch (ClaimCase + Endorsement)
Batch size: 1000 registros
Paginação: Keyset (entity_id) - evita problemas com PageNo > 100

Tempo estimado:
VolumeTempo Estimado100 sinistros< 10 segundos1.000 sinistros< 1 minuto10.000 sinistros< 5 minutos

Error logging
Arquivo de log: LOG_SINAVAC_YYYYMM_<timestamp>.csv
Formato:
csvTimestamp;CaseId;ClaimNo;PolicyNo;Error
2026-01-22 14:30:15;992697123;CBMGV93...;POBMGV93...;Campo obrigatório ausente: NoticeTime
Campos:

Timestamp: Data/hora do erro
CaseId: ID do sinistro
ClaimNo: Número do sinistro
PolicyNo: Número da apólice
Error: Mensagem de erro

Arquivo: SINAVAC_202601_20260122

