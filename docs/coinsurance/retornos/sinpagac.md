# SINPAGAC — Retorno pagamento (BMG)

SINPAGAC — Visão geral (Sinistros Pagos BMG)

O que é (negócio)SINPAGAC é o arquivo de retorno que materializa, para fins regulatórios/contábeis, os sinistros pagos/liquidados do cosseguro aceito (BMG = 03417) para um conjunto de sinistros já persistidos na estrutura definitiva (ClaimCase).Como este projeto gera: a exportação consulta o índice de ClaimCase (com status "03" - Pago) e faz JOIN com o índice de Endorsement para montar 1 linha por sinistro liquidado, aplicando regras de mapeamento de campos e validações SUSEP.
Escopo	
	
Aspecto	Descrição
Arquivo/RO	SINPAGAC
Área	Retornos (ROs) → Sinistros / Pagamento
Entrada do motor	ClaimCase index + Endorsement index
Saída	SINPAGAC_YYYYMM_<timestamp>.csv (delimitador |)
Regras SUSEP / espec	Usadas como referência de campos obrigatórios e formatos

Gatilho e periodicidadeGatilho:
Execução do job de export do período (DT_BASE = YYYYMM) no fechamento mensal.Período (DT_BASE):
Normalmente vem do orquestrador como YYYYMM (ex.: 202601).Filtro por competência (InsertTime):

Quando habilitado, o período YYYYMM é convertido para range de timestamps:

Início: YYYY-MM-01T00:00:00
Fim: YYYY-MM-DDT23:59:59 (último dia do mês)


Campo filtrado: InsertTime (data/hora em que o pagamento foi registrado no sistema)
Fonte de dados (estrutura definitiva)1) Índice ClaimCase (Principal)Consulta por lotes com paginação por entity_id (keyset):Filtros:

CaseStatus = "03" (Sinistro Pago/Liquidado)
InsertTime >= "YYYY-MM-01T00:00:00" (início do período)
InsertTime <= "YYYY-MM-DDT23:59:59" (fim do período)
Ordenação:

entity_id ASC (keyset pagination)
 resultado do índice entrega

Documento completo do ClaimCase com todos os campos necessários
PolicyNo (chave para JOIN com Endorsement)
2) Índice Endorsement (JOIN)Para cada batch de ClaimCases:

Extrai PolicyNo únicos
Faz query batch no índice Endorsement
Cria Map de lookup PolicyNo → Endorsement

Filtros:
PolicyNo IN [lista de PolicyNos únicos

Objetivo:
Obter EndoType (campo crítico para TIPO_MOV)
Obter campos backup: LeaderPolicyNo, LeaderEndorsementNo, EndoNo
Campos do SINPAGAC


Identificação		
			
#	Campo SUSEP	Origem	Regra/Observação
1	SEQUENCIA	Contador local	Inicia em 1, zero-padded
2	COD_CIA	Constante	Sempre 03417
3	DT_BASE	Parâmetro	YYYYMM do período
4	TIPO_MOV	Endorsement.EndoType	JOIN obrigatório; default 01 se ausente
5	COD_RAMO	ClaimCase.ProductCode (derivado)	Mapeamento: BMGAP82→0993, BMGAP83→0118, etc
6	COD_COSS	Constante	Sempre 05908 (líder)
7	NUM_SIN	ClaimCase.ClaimNo	Número do sinistro (obrigatório)
8	NUM_APOL	ClaimCase.ExtPolicyNo / Endorsement.LeaderPolicyNo / ClaimCase.PolicyNo	Prioridade: ExtPolicyNo → LeaderPolicyNo → PolicyNo
9	NUM_END	ClaimCase.ExtEndoNo / Endorsement.LeaderEndorsementNo / Endorsement.EndoNo	Prioridade: ExtEndoNo → LeaderEndorsementNo → EndoNo → Zeros
10	DT_REG	ClaimCase.InsertTime	Formato dd/MM/yyyy
11	DT_AVISO	ClaimCase.NoticeTime	Data FNOL original (dd/MM/yyyy)
12	DT_OCOR	ClaimCase.AccidentTime	Data ocorrência (dd/MM/yyyy)
13	VR_MOV	ClaimCase.SettlementAmount	Valor REAL pago (não zero)
14	TP_SIN	ClaimCase.ExtClaimType	Tipo sinistro: 01=Morte, 02=Invalidez, 03=Incapacidade, 04=Desemprego


Mapeamentos específicos
1. TIPO_MOV (Campo 4)
Fonte: Endorsement.EndoType
EndoTypeTIPO_MOVDescrição"1""01"Emissão"2""02"Renovação"3""03"Alteração/Endosso"4""04"CancelamentoAusente"01"Default

2. COD_RAMO (Campo 5)
Fonte: ClaimCase.ProductCode 
ProductCodeCOD_RAMO Descrição SUSEPBMGAP82*0993Vida em GrupoBMGAP83*0118PrestamistaBMGV93*0993Vida*AP*0161Acidentes PessoaisOutrosPrimeiros 4 dígitosGenérico

3. NUM_APOL (Campo 8) 

ClaimCase.ExtPolicyNo 
Endorsement.LeaderPolicyNo 
ClaimCase.PolicyNo 
Padding: 20 caracteres 

4. NUM_END (Campo 9) 

ClaimCase.ExtEndoNo 
Endorsement.LeaderEndorsementNo 
Endorsement.EndoNo 






