> Documentação Técnica — Fluxo SINAVAC

Visão Geral
Arquitetura de Dados
SINAVAC - Sistema de Aviso de Sinistro
４ de Campos
Fluxo de Processamento
Implementação Técnica
Casos de Uso
Solução de problemas


1. Visão Geral
1.1 Objetivo
Gerar arquivos CSV no formato SUSEP para reportar:

SINAVAC: Sinistros avisados (status "01")


1.2 Periodicidade

Mensal
Executado todo dia 5 do mês seguinte
Período: AAAAMM (ex: 202601 = Janeiro/2026)

1.3 Fontes de Dados
ÍndicePropósitoCampos PrincipaisClaimCaseDados do sinistroClaimNo, PolicyNo, ExtEndoNo, ExtClaimType, RiskStateEndorsementDados do endossoEndoType, LeaderPolicyNo, EndoNo
1.4 Arquivos Gerados
SINAVAC_AAAAMM_timestamp.csv
Formato do carimbo de data/hora: AAAAMMDDDHHmmss
Exemplo: SINAVAC_202601_20260205103045.csv

2. Arquitetura de Dados
2.1 Modelo de Dados
┌───────────────────────────────────────────────────────────────────┐
│ ELASTICSEARCH │
├─ ...
│ │
│ ┌───────────────────────────────────────┐ │
│ │ ClaimCase (Principal) │ │
│ ├───────────────────────────────────────┤ │
│ │ CaseId: 992697123 │ │
│ │ Status do caso: "01" ou "03" │ │
│ │ Número da Reivindicação: "CBMGV93..." │ │
│ │ Apólice nº: "POBMGV93..." │──┐ │
│ │ ExtPolicyNo: "789322900007004" │ │ │
│ │ ExtEndoNo: "787722026011602" │ │ │
│ │ Código do Produto: "BMGV93" │ │ │
│ │ InsertTime: "2026-01-16T14:25:28" │ │ │
│ │ Horário de aviso: "2026-01-01" │ │ │
│ │ Hora do acidente: "2026-01-01" │ │ │
│ │ Valor do Acerto: 0 ou 5000,00 
│ │ ExtClaimType: "1"
│ │ Estado de Risco: "MA" 

│ │ PARTICIPE 
│ Número da Política 

│ │ Endosso (Secundário) 

│ │ EndoNo: "POBMGAP82-001" 
│ │ EndoType: "3"  ←┘ 
│ │ Apólice nº: "POBMGAP82..." 
│ │ Número da Política do Líder: "788213400082352" 
│ │ Número de Aprovação do Líder: "" 
│ │ Código do Produto: "BMGAP82" 
│ │ Data de encerramento: "2025-12-15..." 


2.2 Chave de JOIN
ClaimCase.PolicyNo = Endorsement.PolicyNo
Estratégia: Pré-carregamento em lote (1 consulta para todos os Endossos)

3 . SINAVAC - Sistema de Aviso de Sinistro
3.1
Sistema para reportar sinistros AVISADOS à SUSEP.
3.2 Critérios de Seleção
CritérioValorCaseStatus"01" (Avisado)Filtro de DataNoticeTime no período YYYYMMExemplo QueryNoticeTime >= "2026-01-01" AND NoticeTime <= "2026-01-31"
3.3 Layout do Arquivo
Total de Campos: 15
Separador: Pipe (|)
Codificação: UTF-8
Fim de linha: \n


VR_MOV: Geralmente 0,00 (sinistro ainda não pago)
UF_RISCO: Obrigatório no SINAVAC
Base de Dados: Período do aviso (NoticeTime)
