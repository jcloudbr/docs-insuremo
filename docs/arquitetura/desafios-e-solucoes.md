# Principais Desafios e Soluções

## Visão Geral

**O que deve conter:**
Descreva de forma concisa os principais desafios técnicos, operacionais ou arquiteturais encontrados durante o desenvolvimento ou evolução do sistema. Explique como cada um foi resolvido, destacando decisões estratégicas e impactos positivos.

**Exemplo:**
Este documento aborda os desafios mais relevantes enfrentados no projeto de hospedagem de portais WordPress institucionais em nuvem. Cada desafio é acompanhado da solução adotada, com explicações técnicas e justificativas para a escolha realizada.

---

## Desafios Técnicos

### 1. Acesso Seguro ao Painel Administrativo do WordPress

**O que deve conter:**
- O problema enfrentado.
- Impacto potencial caso não resolvido.
- Solução implementada.
- Tecnologias ou práticas utilizadas.

**Exemplo:**
O painel administrativo (`/wp-admin`) do WordPress é um ponto vulnerável de ataques automatizados. Permitir acesso irrestrito poderia levar à violação de segurança por força bruta ou injeção.

**Solução Adotada:**
Implementamos uma camada de proteção baseada em regras de IP no WAF do Cloudflare, combinada com acesso restrito via VPN corporativa. Além disso, integramos o login com Azure AD para autenticação federada com MFA.

**Tecnologias Utilizadas:**
- Cloudflare WAF
- Regras de firewall na AWS
- Azure AD / EntraID

---

### 2. Escalabilidade Horizontal do WordPress

**O que deve conter:**
- O problema enfrentado.
- Requisitos não funcionais envolvidos.
- Solução implementada.
- Benefícios obtidos.

**Exemplo:**
O WordPress, por padrão, é uma aplicação stateful, o que dificulta sua execução em ambientes escaláveis e efêmeros, especialmente em containers.

**Solução Adotada:**
Utilizamos o Amazon ECS Fargate para orquestrar instâncias isoladas do WordPress, mantendo o estado persistente no Amazon EFS para uploads e plugins, e no Aurora MySQL para dados estruturados.

**Benefícios Obtidos:**
- Escalabilidade automática
- Isolamento entre ambientes
- Maior disponibilidade e recuperação rápida

---

### 3. Gerenciamento de Imagens Docker Personalizadas

**O que deve conter:**
- Complexidade identificada.
- Limitações existentes.
- Estratégia de resolução.
- Automação envolvida.

**Exemplo:**
Cada portal WordPress requer customizações específicas (plugins, temas, configurações), o que torna inviável o uso de uma única imagem Docker genérica.

**Solução Adotada:**
Criamos um pipeline de CI/CD no GitLab que gera imagens Docker personalizadas para cada cliente, versionadas e armazenadas no ECR da AWS.

**Automação Envolveda:**
- Pipeline GitLab CI
- Buildspec com AWS CLI
- Deploy automático no ECS

---

## Desafios Operacionais

### 4. Controle de Versão dos Sites

**O que deve conter:**
- Necessidade identificada.
- Dificuldade operacional sem solução.
- Implementação prática.
- Resultado alcançado.

**Exemplo:**
Sem versionamento claro dos conteúdos publicados, havia risco de perda de conteúdo ou dificuldade em rollback após erros.

**Solução Adotada:**
Implementamos backup diário automatizado do banco de dados Aurora e do diretório de uploads no S3. Além disso, usamos o Git para versionar temas e plugins customizados.

**Resultado Alcançado:**
- Rollback rápido e confiável
- Histórico completo de alterações
- Integração com processos de disaster recovery

---

### 5. Monitoramento e Logs Centralizados

**O que deve conter:**
- Problema de visibilidade no ambiente distribuído.
- Requisitos de monitoramento.
- Ferramentas ou serviços utilizados.
- Melhoria percebida após a implementação.

**Exemplo:**
Com múltiplos containers WordPress rodando no ECS, tornou-se difícil acompanhar logs de erro ou identificar falhas proativas.

**Solução Adotada:**
Configuramos coleta centralizada de logs usando o CloudWatch Logs da AWS, integrado ao ECS. Também adicionamos alertas por métricas críticas (CPU, memória, erros HTTP).

**Ferramentas Utilizadas:**
- AWS CloudWatch Logs
- CloudWatch Alarms
- Dashboard customizado no Grafana (opcional)

**Melhoria Percebida:**
- Detecção precoce de anomalias
- Redução de tempo de análise de incidentes
- Maior visibilidade operacional

---

## Considerações Finais

**O que deve conter:**
- Resumo dos principais aprendizados.
- Reflexão sobre decisões tomadas.
- Sugestões para futuras melhorias ou adaptações.

**Exemplo:**
A adoção de uma arquitetura modular e segura permitiu escalar rapidamente a infraestrutura para novos clientes. Os desafios técnicos foram superados com soluções nativas da nuvem e automação contínua. Futuramente, planejamos migrar parte do controle de acesso para uma solução zero-trust e aumentar a cobertura de testes automatizados nos pipelines.
