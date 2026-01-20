# Design Técnico de Baixo Nível

## Objetivo

**O que deve conter:**

Descreve os detalhes técnicos da implementação da arquitetura. Deve ser voltado para operadores de infraestrutura, DevOps e desenvolvedores que precisam compreender o comportamento interno da solução, configurações, automações, segurança e práticas operacionais.

**Exemplo:**

Esta seção detalha o comportamento interno do ambiente WordPress implantado em containers ECS, incluindo fluxo de requisição, variáveis de configuração, processos de deploy, estratégias de segurança e procedimentos de backup. Ela serve como referência técnica para manutenção e evolução da solução.

---

## Fluxo de Requisição

**O que deve conter:**

Descreva o caminho completo de uma requisição típica do usuário até o backend. Inclua componentes intermediários (CDN, balanceadores, serviços internos) e explique como o conteúdo é entregue.

**Exemplo:**

1. O usuário acessa `https://xpto.com.br`.
2. A requisição é inspecionada pelo WAF da Cloudflare, que aplica políticas de segurança e cache.
3. Se permitida, a requisição é encaminhada ao Application Load Balancer (ALB) da AWS.
4. O ALB distribui o tráfego para uma task ECS em execução, onde o WordPress está rodando em um container.
5. O WordPress acessa o banco de dados Amazon Aurora para recuperar o conteúdo e carrega arquivos do Amazon EFS.
6. A resposta é enviada de volta ao usuário pelo mesmo caminho reverso.

---

## Configurações de Ambiente

**O que deve conter:**

Liste as variáveis de ambiente, configurações de runtime, dependências externas, mapeamentos de volumes e recursos consumidos (memória, CPU etc.).

**Exemplo:**

| Variável / Configuração      | Valor / Descrição                                                                 |
|-----------------------------|------------------------------------------------------------------------------------|
| `WORDPRESS_DB_HOST`         | aurora-cluster.cluster-abc.us-east-1.rds.amazonaws.com                             |
| `WORDPRESS_DB_USER`         | admin                                                                              |
| `WORDPRESS_DB_PASSWORD`     | Armazenado como secret no ECS                                                     |
| `WORDPRESS_DB_NAME`         | wp_prod                                                                            |
| `WP_HOME`                   | https://portal.exemplo.gov.br                                                     |
| `WP_SITEURL`                | https://portal.exemplo.gov.br                                                     |
| Montagem de volume EFS      | Montado no container em `/var/www/html/wp-content`                                |

---

## Requisitos de Infraestrutura

**O que deve conter:**

Detalhe os recursos mínimos necessários para execução da aplicação. Isso inclui CPU, memória, política de rede, sistema operacional ou imagem base, além de configurações específicas de execução. Essas informações são importantes para sizing, escalabilidade e custo.

**Exemplo:**

| Requisito                    | Valor / Descrição                                                                 |
|-----------------------------|------------------------------------------------------------------------------------|
| CPU                         | 0.5 vCPU (por container ECS)                                                      |
| Memória                     | 1 GB (por container ECS)                                                          |
| Rede                        | Sub-redes privadas, sem IP público                                                |
| Sistema Operacional         | Amazon Linux 2 (base da imagem Fargate)                                           |
| Imagem Docker               | `registry.gitlab.com/exemplo/wordpress-custom:latest`                             |
| Montagem de volume (EFS)    | `/var/www/html/wp-content`                                                       |
| Logging                     | Logs enviados para AWS CloudWatch Logs com retenção de 30 dias                    |

Essas configurações foram definidas para balancear custo e desempenho, considerando o perfil de uso dos portais e a arquitetura baseada em Fargate. Em projetos com maior volume de tráfego ou processamento, os valores de CPU e memória podem ser ajustados conforme métricas reais de uso.

## Pipeline de Deploy

**O que deve conter:**

Descreva como a aplicação é implantada: processo de build da imagem, testes, publicação e atualização do serviço. Se for gerenciado por pipeline CI/CD, documente os estágios e ferramentas utilizadas.

**Exemplo:**

1. Desenvolvedores versionam o código e as configurações da aplicação no GitLab.
2. O GitLab CI é responsável por:
   - Executar testes automatizados
   - Construir a imagem Docker do WordPress personalizada
   - Publicar a imagem no Amazon ECR
   - Aplicar atualizações no ECS com `deploy rolling` controlado
3. As configurações de infraestrutura (ALB, ECS, Aurora, EFS) são gerenciadas via Terraform, com repositório separado, seguindo práticas de GitOps.

---

## Estratégias de Segurança

**O que deve conter:**

Explique os mecanismos de segurança adotados para proteger o ambiente, como controle de acesso, uso de VPN, autenticação, criptografia, e proteção contra ataques externos.

**Exemplo:**

- A URL `/wp-admin` é protegida por uma regra personalizada de WAF na Cloudflare que bloqueia todo tráfego externo não proveniente de IPs da VPN.
- A autenticação no painel do WordPress é federada via EntraID (Azure AD), com autenticação multifator ativada.
- Containers ECS não têm IP público e comunicam-se exclusivamente em sub-redes privadas.
- Todos os segredos (senhas, tokens) são armazenados como `secrets` no ECS, com permissões controladas via IAM.
- Logs de acesso e erros são auditados via CloudWatch Logs.

---

## Estratégia de Backup e Recuperação

**O que deve conter:**

Explique como backups são realizados, onde são armazenados, frequência, retenção e como ocorre a recuperação em caso de falha.

**Exemplo:**

- O Amazon Aurora realiza backups automáticos com retenção de 7 dias e suporta Point-in-Time Recovery (PITR).
- O Amazon EFS é protegido por políticas do AWS Backup, com execução semanal e retenção de 30 dias.
- Restauração do banco e do EFS pode ser feita manualmente por operadores com permissão, via console AWS ou CLI.
- Os dados críticos são armazenados em múltiplas zonas de disponibilidade (Multi-AZ) para resiliência adicional.
