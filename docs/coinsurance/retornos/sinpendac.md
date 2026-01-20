Documentação Técnica: Fluxo SINPENDAC
1. Visão Geral do Processo (ETL)
O processo de geração do arquivo SINPENDAC tem como objetivo reportar à SUSEP o controle de sinistros pendentes relacionados a operações de cosseguro cedido, com base exclusiva nos dados de sinistro recebidos da Generali (GBS) e internalizados no InsureMO por meio da entidade ClaimCase.
O arquivo SINPENDAC deve refletir fielmente as informações originalmente recebidas no arquivo SINPEND da Generali, sem qualquer recalculo atuarial, enriquecimento sistêmico ou derivação a partir da apólice.
________________________________________
2. Fluxo Geral do Processo
2.1 Extração
•	Os dados são extraídos da estrutura ClaimCase do InsureMO.
•	O ClaimCase representa o sinistro conforme recebido da Generali (GBS).
•	Apenas sinistros pendentes devem ser considerados.
•	A extração pode ocorrer via índice ClaimCase, respeitando os campos disponíveis, ou via core, quando necessário para campos não indexados.
________________________________________
2.2 Transformação
•	Os campos do ClaimCase são mapeados diretamente para o layout SINPENDAC.
•	Campos regulatórios fixos (cod_cia, cod_coss) são aplicados conforme premissas SUSEP.
•	A participação de cosseguro da BMG (40%) é aplicada exclusivamente nos campos exigidos pelo layout.
•	Não é permitido:
o	Recalcular valores
o	Ajustar datas
o	Alterar códigos recebidos da Generali
________________________________________
2.3 Carga
•	O arquivo SINPENDAC é gerado de forma consolidada.
•	A numeração de registros (sequencia) reinicia a cada arquivo.
•	O arquivo final é persistido conforme o fluxo padrão do projeto (S3).
________________________________________
3. Origem dos Dados (Data Mapping)
Os dados do SINPENDAC têm como fonte única a entidade ClaimCase, que espelha os dados originalmente informados pela Generali (GBS) no arquivo SINPEND.
Não há dependência de dados da apólice ou de cálculos internos da seguradora.
________________________________________
4. Campos de Identificação (06.01)
Todos os campos abaixo devem possuir valores idênticos aos registrados no ClaimCase.
Campo SINPENDAC	Origem (ClaimCase)	Regra
sequencia	Controle interno	Sequencial, iniciando em 1
cod_cia	Fixo	Sempre "03417"
dt_base	Parâmetro	Formato YYYYMM
tipo_mov	Endorsement.EndoType	Mesmo valor recebido
cod_ramo	ProductCode	Conforme informado
cod_coss	Fixo	Sempre "05908"
num_sin	ClaimNo	Número do sinistro
num_apol	ExtPolicyNo / LeaderPolicyNo	Número da apólice
num_end	Endorsement.EndoNo / ExtEndoNo	Número do endosso
dt_reg	InsertTime	Data de registro
dt_aviso	NoticeTime	Data de aviso
dt_ocor	AccidentTime	Data de ocorrência
tp_sin	ExtClaimType	Tipo de sinistro
________________________________________
5. Campos de Valores (06.02)
Os valores financeiros devem refletir exatamente os dados informados pela Generali e registrados no ClaimCase.
________________________________________
5.1 vr_pendente
•	Origem: ClaimCase.vr_cos_ced
•	Regra:
O valor deve ser utilizado sem qualquer ajuste ou arredondamento.
vr_pendente = ClaimCase.vr_cos_ced
________________________________________
5.2 vr_tot
•	Origem: ClaimCase.vr_tot
•	Regra:
O valor total deve ser multiplicado pela participação fixa de cosseguro da BMG (40%).
vr_tot = ClaimCase.vr_tot × 0,40
________________________________________
6. Regras Fixas e Premissas Regulatórias
•	cod_cia = 03417
•	cod_coss = 05908
•	Participação de cosseguro BMG = 40%
•	Não é permitido enriquecer ou recalcular dados
•	O arquivo deve refletir 1:1 o conteúdo do ClaimCase
•	A sequência reinicia a cada geração
________________________________________
7. Consistência com SINPEND / ClaimCase
O SINPENDAC é um reflexo regulatório direto do arquivo SINPEND recebido da Generali.
Premissas de Consistência
•	Deve existir correspondência 1:1 entre registros do ClaimCase e registros do SINPENDAC
•	Campos de identificação devem ser idênticos
•	Valores financeiros devem respeitar estritamente os valores registrados no ClaimCase
•	A única transformação permitida é a aplicação do percentual de cosseguro onde exigido
________________________________________
8. Mapeamento Técnico Consolidado – SINPENDAC
SEQUENCIA     -> calculado (sequencial)
COD_CIA       -> constante "03417"
DT_BASE       -> parâmetro (YYYYMM)
TIPO_MOV      -> Endorsement.EndoType
COD_RAMO      -> ClaimCase.ProductCode
COD_COSS      -> constante "05908"
NUM_SIN       -> ClaimCase.ClaimNo
NUM_APOL      -> ClaimCase.ExtPolicyNo | Endorsement.LeaderPolicyNo
NUM_END       -> Endorsement.EndoNo | ClaimCase.ExtEndoNo
DT_REG        -> ClaimCase.InsertTime
DT_AVISO      -> ClaimCase.NoticeTime
DT_OCOR       -> ClaimCase.AccidentTime
VR_PENDENTE   -> ClaimCase.SettlementAmount
VR_TOT        -> ClaimCase.TotalIncurred
TP_SIN        -> ClaimCase.ExtClaimType
________________________________________
9. Conclusão
O arquivo SINPENDAC deve ser tratado como um espelho regulatório dos dados de sinistro informados pela Generali (GBS), respeitando integralmente:
•	A estrutura do ClaimCase
•	As regras SUSEP
•	As premissas de cosseguro da BMG
Qualquer divergência entre SINPENDAC, SINPEND e ClaimCase caracteriza inconsistência regulatória.
________________________________________
Documentação Técnica: Fluxo Integrado SINAVAC, SINPENDAC e SINPAGAC
1. Visão Geral do Fluxo Regulatório
Os arquivos SINAVAC, SINPENDAC e SINPAGAC representam etapas distintas e complementares do ciclo de vida de um sinistro reportado à SUSEP, no contexto de cosseguro cedido da BMG Seguridade.
Cada arquivo possui função regulatória específica e não se sobrepõe aos demais.
________________________________________
2. Papel de Cada Arquivo no Ciclo do Sinistro
Arquivo	Função	Momento do Sinistro
SINAVAC	Aviso de sinistro	Registro inicial do sinistro
SINPENDAC	Controle de sinistros pendentes	Sinistro em aberto
SINPAGAC	Pagamento / liquidação	Encerramento financeiro
________________________________________
3. Fluxo Lógico Integrado (Visão de Processo)
┌───────────────┐
│ Generali (GBS)│
│  SINAVAC      │  → Aviso de sinistro
│  SINPEND      │  → Sinistro pendente
│  SINPAG       │  → Pagamento
└───────┬───────┘
        │
        ▼
┌─────────────────────────────┐
│ InsureMO                    │
│ ClaimCase                   │
│ (espelho dos dados GBS)     │
└───────┬─────────┬───────────┘
        │         │
        │         │
        ▼         ▼
┌────────────┐ ┌──────────────┐
│ SINAVAC    │ │ SINPENDAC    │
│ (Aviso)   │ │ (Pendente)   │
└───────┬────┘ └───────┬──────┘
        │              │
        ▼              ▼
        └──────────┬──────────┘
                   ▼
            ┌──────────────┐
            │ SINPAGAC     │
            │ (Pagamento) │
            └──────────────┘
________________________________________
4. Regras de Sequência e Dependência
4.1 SINAVAC
•	Primeiro arquivo do ciclo.
•	Representa o registro do aviso do sinistro.
•	Um sinistro só pode existir em SINPENDAC ou SINPAGAC se já tiver passado pelo SINAVAC.
________________________________________
4.2 SINPENDAC
•	Representa o estado pendente do sinistro.
•	Pode aparecer em vários meses consecutivos, enquanto o sinistro não for liquidado.
•	Sempre reflete o estado atual do sinistro naquele mês-base.
________________________________________
4.3 SINPAGAC
•	Representa o pagamento ou liquidação do sinistro.
•	Pode ocorrer:
o	Em parcela única
o	Em múltiplos pagamentos
•	Após o SINPAGAC final, o sinistro não deve mais aparecer no SINPENDAC.
________________________________________
5. Exemplo Prático Completo (Ciclo de Vida)
Cenário
•	Sinistro: SIN123456
•	Apólice: APOL78910
•	Cosseguro BMG: 40%
•	Valor total do sinistro: R$ 100.000,00
________________________________________
5.1 Mês 01 – Aviso do Sinistro
Evento
•	Sinistro avisado à Generali em 10/01/2026
•	Nenhum pagamento realizado
Arquivo Gerado
SINAVAC – Janeiro/2026
NUM_SIN    = SIN123456
DT_AVISO   = 10/01/2026
VR_MOV     = 0,00
➡ O sinistro passa a existir oficialmente na SUSEP
________________________________________
5.2 Mês 02 – Sinistro Pendente
Estado
•	Sinistro ainda em análise
•	Valor estimado total: R$ 100.000,00
•	Parcela BMG (40%): R$ 40.000,00
Arquivo Gerado
SINPENDAC – Fevereiro/2026
NUM_SIN      = SIN123456
VR_PENDENTE  = 40.000,00
VR_TOT       = 40.000,00
➡ Sinistro permanece em aberto
________________________________________
5.3 Mês 03 – Pagamento Parcial
Evento
•	Pagamento parcial de R$ 60.000,00
•	Parcela BMG (40%): R$ 24.000,00
Arquivos Gerados
SINPAGAC – Março/2026
NUM_SIN   = SIN123456
VR_PAG    = 24.000,00
SINPENDAC – Março/2026
NUM_SIN      = SIN123456
VR_PENDENTE  = 16.000,00
VR_TOT       = 40.000,00
➡ Sinistro continua pendente, mas com saldo reduzido
________________________________________
5.4 Mês 04 – Liquidação Final
Evento
•	Pagamento final de R$ 40.000,00
•	Parcela BMG (40%): R$ 16.000,00
Arquivo Gerado
SINPAGAC – Abril/2026
NUM_SIN   = SIN123456
VR_PAG    = 16.000,00
➡ Sinistro encerrado
________________________________________
5.5 Mês 05 – Pós-Liquidação
Regra
•	Sinistro não aparece mais no SINPENDAC
•	Último registro foi o SINPAGAC
________________________________________
6. Regras de Consistência entre Arquivos
Regra	Descrição
Sequência lógica	SINAVAC → SINPENDAC → SINPAGAC
Dependência	SINPENDAC e SINPAGAC exigem SINAVAC prévio
Persistência	SINPENDAC repete mensalmente enquanto pendente
Encerramento	Após SINPAGAC final, não gerar SINPENDAC
Fonte	Sempre ClaimCase
Cosseguro	Aplicar 40% apenas onde exigido
________________________________________
7. Conclusão
O fluxo SINAVAC → SINPENDAC → SINPAGAC representa o ciclo regulatório completo do sinistro, permitindo à SUSEP acompanhar:
•	Aviso
•	Evolução
•	Pagamento
•	Encerramento
A correta aplicação desse fluxo garante consistência regulatória, rastreabilidade e aderência total às normas SUSEP.
