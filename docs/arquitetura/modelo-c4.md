# Modelo C4

## Objetivo

**O que deve conter:**

Explique o propósito da adoção do Modelo C4. Destaque que ele permite visualizar a arquitetura do sistema em diferentes níveis de abstração, facilitando o entendimento técnico por públicos distintos (devs, ops, gestão).

**Exemplo:**

O Modelo C4 é utilizado para representar visualmente a arquitetura do sistema em quatro níveis progressivos de detalhamento: contexto, containers, componentes e, quando aplicável, código. Esta abordagem fornece uma linguagem visual padronizada que facilita a comunicação entre times técnicos e gestores, garantindo clareza na modelagem da solução.

---

## Nível 1 — Diagrama de Contexto

**O que deve conter:**

Represente como o sistema se relaciona com usuários externos, atores institucionais, integrações externas e ambientes. Este diagrama não precisa mostrar detalhes internos — o foco é a visão macro.

**Exemplo:**

Este nível mostra que o sistema:

- É acessado por usuários externos via navegador.
- Está protegido por Cloudflare, que atua como proxy reverso e WAF.
- Interage com sistemas externos como provedores de identidade (EntraID) e serviços de DNS.

**Imagem esperada:**

```markdown
![Diagrama de Contexto](../img/contexto.png)
```

## Nível 2 — Diagrama de Containers

**O que deve conter:**

Descreva os principais containers, serviços ou aplicações que compõem o sistema. Indique suas responsabilidades, tecnologias utilizadas, como se comunicam entre si e com o exterior. Este nível é fundamental para equipes técnicas compreenderem a divisão funcional da aplicação.

**Exemplo:**

A arquitetura é composta pelos seguintes containers e serviços:

- **Cloudflare**: Atua como camada de proteção (WAF), entrega de conteúdo (CDN) e gerenciamento de DNS.
- **Application Load Balancer (ALB)**: Responsável por receber as requisições HTTP da internet e distribuí-las para os containers do WordPress.
- **Amazon ECS (Fargate)**: Executa containers isolados do WordPress, sem necessidade de provisionamento de servidores.
- **Amazon Aurora**: Banco de dados relacional altamente disponível, compatível com MySQL.
- **Amazon EFS**: Sistema de arquivos montado em rede para armazenar uploads, plugins e configurações do WordPress.

O diagrama a seguir mostra a comunicação entre esses containers e serviços:

```markdown
![Diagrama de Containers](../img/containers.png)
```

## Nível 3 — Diagrama de Componentes (opcional)

**O que deve conter:**

Inclua este nível somente quando houver lógica específica implementada dentro de um container ou aplicação que justifique o detalhamento. Use este diagrama para mostrar como diferentes módulos internos se organizam e interagem dentro de uma aplicação, especialmente se houver desenvolvimento próprio ou integração com sistemas internos.

**Exemplo:**

No caso de um ambiente WordPress com personalizações, o container pode incluir os seguintes componentes:

- **Autenticação Federada (SSO com EntraID)**: Plugin customizado ou configurado que permite login com contas institucionais via protocolo SAML ou OpenID Connect.
- **Gerenciador de Cache Interno**: Componente que realiza cache de páginas e objetos usando plugins como W3 Total Cache ou Redis Object Cache.
- **Módulo de Integração com APIs Institucionais**: Responsável por consumir APIs REST externas, como sistemas acadêmicos, diretórios ou portais da organização.

Estes componentes são estruturados dentro do próprio container do WordPress e configurados via arquivos de tema, plugin ou scripts adicionais embarcados na imagem Docker.

Se a aplicação utiliza WordPress sem personalizações relevantes, este nível pode ser omitido da documentação.

```markdown
![Diagrama de Componentes](../img/componentes.png)
```

## Nível 4 — Diagrama de Código (opcional)

**O que deve conter:**

Este nível só deve ser incluído quando houver desenvolvimento interno que justifique o detalhamento da estrutura de código. O foco é mostrar como o código-fonte está organizado internamente, seja por classes, funções, pacotes ou módulos. Não é necessário para aplicações que utilizam software de prateleira sem personalizações relevantes.

**Exemplo:**

No contexto deste projeto, este nível pode ser utilizado quando há desenvolvimento de plugins ou middlewares personalizados, como:

- Plugin de autenticação com EntraID (Azure AD)
- Middleware de auditoria e rastreamento de acessos administrativos
- Integrações com APIs REST de sistemas internos (ex: matrícula, eventos, diretórios)

Um diagrama de código pode representar:

- A estrutura de classes PHP utilizadas na lógica do plugin
- Fluxos de execução entre controladores, manipuladores de eventos e funções auxiliares
- O uso de padrões como MVC, Hook-based architecture ou Injeção de Dependência

**Quando não incluir:**

Se o WordPress for utilizado com temas e plugins públicos, sem desenvolvimento próprio ou lógica customizada, este nível deve ser omitido para manter a documentação objetiva e enxuta.

**Imagem esperada (se aplicável):**

```markdown
![Diagrama de Código](../img/codigo.png)
```

## Armazenamento e Referência de Imagens

**O que deve conter:**

Defina uma convenção de armazenamento e nomenclatura para os diagramas utilizados na documentação. Essa padronização garante organização, facilita a manutenção ao longo do tempo e assegura consistência entre projetos que utilizam este template.

**Exemplo:**

Todos os diagramas utilizados nesta documentação devem ser armazenados na pasta `docs/img/` com os seguintes nomes padronizados:
```
docs/img/
├── contexto.png # Diagrama de Nível 1 (Contexto)
├── containers.png # Diagrama de Nível 2 (Containers)
├── componentes.png # Diagrama de Nível 3 (Componentes)
└── codigo.png # Diagrama de Nível 4 (Código)

```

Esses arquivos devem ser mantidos em formato `.png` ou `.svg` exportados diretamente das ferramentas de modelagem utilizadas.

### Ferramentas recomendadas

- **draw.io (diagrams.net)** – ferramenta visual, amplamente adotada, com suporte a versionamento por arquivo `.drawio`.
- **Structurizr DSL** – ideal para quem deseja gerar os diagramas a partir de arquivos de texto e manter como código.
- **MermaidJS ou PlantUML** – úteis quando os diagramas precisam ser incluídos diretamente no Markdown e renderizados dinamicamente.

### Como incluir imagens no Markdown

Para referenciar as imagens nos arquivos `.md`, utilizar o seguinte padrão:

```markdown
![Diagrama de Containers](../img/containers.png)
```