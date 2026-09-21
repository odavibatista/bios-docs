# Ficha Técnica — Tecnologias do BIOS

> Documento de referência técnica do projeto BIOS, cobrindo linguagens, frameworks,
> bibliotecas, ferramentas de teste e integrações externas. Complementa a Etapa 5
> (Projeto e Arquitetura) da documentação técnica.

## 1. Linguagens de Programação

- **TypeScript** — linguagem principal em toda a stack (back-end, front-end e
  microsserviços), com `strict` habilitado no `tsconfig.json`.
- **JavaScript** — restrito a arquivos de configuração/build quando exigido por alguma
  ferramenta, nunca como linguagem de lógica de negócio.

## 2. Frameworks

| Camada | Framework | Observação |
|---|---|---|
| Back-end | NestJS (v11.x) | Versão Major |
| Front-end | Angular (versão moderna, standalone components) | Decisão travada nesta ficha |
| Microsserviços de ingestão/scraping | NestJS *standalone* (sem camada HTTP exposta) | Comunicação com o core via fila (BullMQ) |

## 3. ORM e Persistência

- **Prisma ORM v6.19.x** — versão estável mais recente da série v6, compatível com
  MongoDB. **Decisão explícita:** o BIOS permanece nesta versão mesmo com a existência da
  Prisma ORM v7, porque a v7 **ainda não suporta MongoDB** (limitação declarada pela própria
  Prisma, decorrente de uma reescrita da camada de query engine que priorizou bancos SQL).
- **MongoDB** como banco de dados principal, modelado em três coleções lógicas
  (`evidence_raw`, `evidence_normalized`, `company_profile` — padrão Bronze/Silver/Gold).

## 4. Bibliotecas — Back-end

| Categoria | Biblioteca | Função no BIOS |
|---|---|---|
| Núcleo do framework | `@nestjs/common`, `@nestjs/core`, `@nestjs/platform-express` | Base do NestJS |
| Configuração | `@nestjs/config`, `dotenv` | Variáveis de ambiente |
| Documentação de API | `@nestjs/swagger`, `@scalar/nestjs-api-reference` | RF17 — API pública documentada |
| Cache | `@nestjs/cache-manager`, `cache-manager`, `keyv`, `@keyv/redis` | NFDE01 — cache de consultas |
| Filas / jobs assíncronos | `@nestjs/bullmq`, `bullmq` | Ingestão periódica de evidências, scraping do GHG Protocol |
| Rate limiting | `@nestjs/throttler` | NFSE04 — Throttler nos endpoints públicos |
| Validação | `nestjs-zod`, `zod`, `class-validator`, `class-transformer` | DTOs e validação de entrada |
| Autenticação | `jsonwebtoken`, `bcryptjs` | RF02, NFSE05 — login e hash de senha |
| Criptografia | `crypto` (nativo do Node) | NFSE03 — AES-256 em dados sensíveis |
| E-mail | `handlebars`, `nodemailer`, `@types/nodemailer` | RF18 — e-mails transacionais |
| Dados sintéticos | `@faker-js/faker` | RF21 — honeypot (dependência de **produção** no BIOS, onde é usada só em teste) |
| Identificadores | `@paralleldrive/cuid2` | Geração de IDs |
| Datas | `date-fns` | Manipulação de datas de evidência |
| HTTP client | `axios` | Consumo das APIs externas (Seção 6) |

## 5. Bibliotecas de Teste

### 5.1 Back-end
- **Jest** (test runner) + **ts-jest**;
- **@nestjs/testing** — `Test.createTestingModule` para injeção de dependência em teste,
  evitando o padrão de `jest.mock`;
- **supertest** + `@types/supertest` — testes e2e de endpoint HTTP;
- **@faker-js/faker** — fixtures de teste.

### 5.2 Front-end (Angular)
- **Jasmine + Karma** — testes unitários, padrão do Angular CLI;
- **@testing-library/angular** — testes de componente, com foco em comportamento observável
  pelo usuário em vez de detalhe de implementação interna;
- **Playwright** ou **Cypress** — testes e2e de fluxo completo (a decidir entre os dois na
  Etapa 6, sem impacto na Etapa 5).

### 5.3 Microsserviço (ingestão/scraping)
- **Jest** — mesmo test runner do back-end, por consistência;
- **nock** ou **MSW (Mock Service Worker)** — simulação de respostas das APIs externas nos
  testes de integração de cada adapter, sem depender da disponibilidade real da fonte durante
  o CI;
- **@faker-js/faker** — dados sintéticos de teste.

## 6. APIs Consumidas e Forma de Comunicação

| Fonte | Protocolo/Formato | Observação |
|---|---|---|
| BrasilAPI / OpenCNPJ | HTTPS + JSON | REST, sem autenticação |
| IBGE (`servicodados.ibge.gov.br`) | HTTPS + JSON | REST, sem autenticação |
| IBAMA — Dados Abertos | HTTPS + JSON | API CKAN (Action API), sem autenticação |
| CGU — Portal da Transparência (CEIS/CNEP) | HTTPS + JSON | REST, autenticação por chave de API em header |
| GHG Protocol — Registro Público de Emissões | HTTPS + HTML/CSV | **Exceção documentada** — sem API REST oficial; acesso via download estruturado, não é JSON |

A comunicação entre o BIOS e as fontes externas é majoritariamente **HTTPS com JSON**, com
única exceção no GHG Protocol, que não expõe API formal — desvio já tratado como excepcional
desde a definição das fontes de dados (hierarquia API First → dataset → download → scraping).

A comunicação **interna** entre os componentes do próprio BIOS (core API ↔ serviço de
ingestão ↔ serviço de scraping) também é HTTPS + JSON para chamadas síncronas, evoluindo para
mensageria assíncrona via BullMQ/Redis nos jobs de ingestão que não exigem resposta imediata
(ex.: atualização periódica de evidências, scraping do GHG Protocol).
