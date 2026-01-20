# SINAVAC — Retorno de sinistro (BMG)

> TODO: detalhar.
>
> Documentação Técnica — Fluxo SINAVAC 


Visão Geral
Arquitetura de Dados
SINAVAC - Sistema de Aviso de Sinistros
Especificação de Campos
Fluxo de Processamento
Implementação Técnica
Casos de Uso
Troubleshooting


1. Visão Geral
1.1 Objetivo
Gerar arquivos CSV no formato SUSEP para reportar:

SINAVAC: Sinistros avisados (status "01")


1.2 Periodicidade

Mensal
Executado todo dia 5 do mês seguinte
Período: YYYYMM (ex: 202601 = Janeiro/2026)

1.3 Fontes de Dados
ÍndicePropósitoCampos PrincipaisClaimCaseDados do sinistroClaimNo, PolicyNo, ExtEndoNo, ExtClaimType, RiskStateEndorsementDados do endossoEndoType, LeaderPolicyNo, EndoNo
1.4 Arquivos Gerados
SINAVAC_YYYYMM_timestamp.csv
Formato timestamp: YYYYMMDDHHmmss
Exemplo: SINAVAC_202601_20260205103045.csv

2. Arquitetura de Dados
2.1 Modelo de Dados
┌─────────────────────────────────────────────────────────────┐
│                    ELASTICSEARCH                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────┐                  │
│  │        ClaimCase (Principal)         │                  │
│  ├──────────────────────────────────────┤                  │
│  │ CaseId: 992697123                   │                  │
│  │ CaseStatus: "01" ou "03"            │                  │
│  │ ClaimNo: "CBMGV93..."               │                  │
│  │ PolicyNo: "POBMGV93..."             │──┐               │
│  │ ExtPolicyNo: "789322900007004"      │  │               │
│  │ ExtEndoNo: "787722026011602"        │  │               │
│  │ ProductCode: "BMGV93"               │  │               │
│  │ InsertTime: "2026-01-16T14:25:28"   │  │               │
│  │ NoticeTime: "2026-01-01"            │  │               │
│  │ AccidentTime: "2026-01-01"          │  │               │
│  │ SettlementAmount: 0 ou 5000.00     │  │               │
│  │ ExtClaimType: "1"                   │  │               │
│  │ RiskState: "MA"                     │  │               │
│  └──────────────────────────────────────┘  │               │
│                                             │               │
│                                             │ JOIN          │
│                                    PolicyNo │               │
│                                             │               │
│  ┌──────────────────────────────────────┐  │               │
│  │       Endorsement (Secundário)       │  │               │
│  ├──────────────────────────────────────┤  │               │
│  │ EndoNo: "POBMGAP82-001"             │  │               │
│  │ EndoType: "3"                       │ ←┘               │
│  │ PolicyNo: "POBMGAP82..."            │                  │
│  │ LeaderPolicyNo: "788213400082352"   │                  │
│  │ LeaderEndorsementNo: ""             │                  │
│  │ ProductCode: "BMGAP82"              │                  │
│  │ EndoIssueDate: "2025-12-15..."      │                  │
│  └──────────────────────────────────────┘                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
2.2 Chave de JOIN
ClaimCase.PolicyNo = Endorsement.PolicyNo
Estratégia: Batch preload (1 query para todos os Endorsements)

3. SINAVAC - Sistema de Aviso de Sinistros
3.1 Definição
Sistema para reportar sinistros AVISADOS à SUSEP.
3.2 Critérios de Seleção
CritérioValorCaseStatus"01" (Avisado)Filtro de DataNoticeTime no período YYYYMMExemplo QueryNoticeTime >= "2026-01-01" AND NoticeTime <= "2026-01-31"
3.3 Layout do Arquivo
Total de Campos: 15
Separador: Pipe (|)
Encoding: UTF-8
Line Ending: \n


VR_MOV: Geralmente 0.00 (sinistro ainda não pago)
UF_RISCO: Obrigatório no SINAVAC
Data Base: Período do aviso (NoticeTime)

