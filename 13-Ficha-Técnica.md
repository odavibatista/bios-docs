# Ficha Técnica — Tecnologias do BIOS

> Documento de referência técnica do projeto BIOS, cobrindo linguagens, frameworks,
> bibliotecas, ferramentas de teste e integrações externas. Complementa a Etapa 5
> (Projeto e Arquitetura) da documentação técnica.

## 1. Linguagens de Programação

- **TypeScript 6** — linguagem principal em toda a stack (back-end, front-end e
  microsserviços), com `strict` habilitado no `tsconfig.json`. O back-end é compilado como
  **ESM** (`"type": "module"`), padrão do NestJS 12. A série 6 é o teto porque duas
  ferramentas da stack ainda não aceitam a 7: `@nestjs/swagger` declara peer
  `^5.5.0 || ^6.0.0` e `typescript-eslint`, `>=4.8.4 <6.1.0`. Com a 7 instalada, a
  instalação limpa (`npm ci`) falha na resolução de peers e o CI não chega a rodar, de modo
  que a major fica bloqueada no Dependabot até as duas subirem o teto.
- **JavaScript** — restrito a arquivos de configuração/build quando exigido por alguma
  ferramenta, nunca como linguagem de lógica de negócio.

## 2. Frameworks

| Camada | Framework | Observação |
|---|---|---|
| Back-end | NestJS (v12.x) | Versão major atual; exige Node.js ≥ 22.22.3. Adotada pelo suporte nativo a Standard Schema (validação e serialização com Zod sem biblioteca intermediária) |
| Front-end | Angular (versão moderna, standalone components) | Decisão travada nesta ficha |
| Microsserviços de ingestão/scraping | NestJS *standalone* (sem camada HTTP exposta) | Comunicação com o core via fila (BullMQ) |

## 3. ORM e Persistência

- **Prisma ORM v6.19.x** — versão estável mais recente da série v6, compatível com
  MongoDB. **Decisão explícita:** o BIOS permanece nesta versão mesmo com a existência da
  Prisma ORM v7, porque a v7 **ainda não suporta MongoDB** (limitação declarada pela própria
  Prisma, decorrente de uma reescrita da camada de query engine que priorizou bancos SQL).
- **MongoDB 8** como banco de dados principal, executado como **replica set** (exigência do
  Prisma para transações e ações referenciais emuladas). As camadas de dados seguem o padrão
  Bronze/Silver/Gold em três coleções lógicas (`evidence_raw`, `evidence_normalized`,
  `company_profile`); o modelo completo está em `assets/database/bios-database.dbml.txt`.
- **Schema Prisma multiarquivo**, distribuído pelos módulos: cada módulo mantém os schemas
  das suas entidades em `src/modules/<módulo>/entity/*.prisma`, e apenas o `generator` e o
  `datasource` ficam na infraestrutura compartilhada. O `prisma.config.ts` aponta a raiz do
  schema para `src/`, de onde o Prisma reúne todos os arquivos recursivamente.

## 4. Bibliotecas — Back-end

| Categoria | Biblioteca | Função no BIOS |
|---|---|---|
| Núcleo do framework | `@nestjs/common`, `@nestjs/core`, `@nestjs/platform-express` | Base do NestJS |
| Configuração | `@nestjs/config` + schema Zod | Variáveis de ambiente validadas no bootstrap (configuração inválida impede a aplicação de subir) |
| Documentação de API | `@nestjs/swagger`, `@scalar/nestjs-api-reference` | RF17 — API pública documentada |
| Cache | `ioredis` (mesmo cliente das filas) | NFDE01 — cache de consultas por CNPJ. A pilha `@nestjs/cache-manager` + `cache-manager` + `keyv` + `@keyv/redis`, prevista inicialmente, foi descartada: além de não resolver sozinha o requisito (degradação graciosa e orçamento de latência precisam de tratamento próprio em qualquer caso), ela traria o `@redis/client` como **segundo** driver de Redis ao lado do `ioredis` exigido pelo BullMQ, e as versões atuais não convivem — `cache-manager` 7 fixa `keyv` 5, enquanto o `@keyv/redis` 6 exige `keyv` 6. A superfície usada é `GET`/`SET EX`/`DEL`, que o `ioredis` já cobre |
| Filas / jobs assíncronos | `@nestjs/bullmq`, `bullmq` (série 6), `ioredis` | Comunicação interna entre o core e os serviços de ingestão, ingestão periódica de evidências e scraping do GHG Protocol. A série 6 do BullMQ transformou o cliente de Redis em dependência *peer* opcional — ele passou a aceitar `ioredis`, `redis` ou até Postgres como backend —, de modo que o driver é escolhido explicitamente: `ioredis`, o de uso consolidado com BullMQ |
| Rate limiting | `@nestjs/throttler` | NFSE04 — Throttler nos endpoints públicos |
| Validação e serialização | `zod` (Standard Schema nativo do NestJS 12) | Schemas de request e response: validação de entrada (`StandardSchemaValidationPipe`), serialização de saída que descarta campos não declarados (`StandardSchemaSerializerInterceptor`) e geração do OpenAPI (`@nestjs/swagger`) a partir do mesmo schema |
| Autenticação | `jsonwebtoken`, `bcryptjs` | RF02, NFSE05 — login e hash de senha |
| Criptografia | `crypto` (nativo do Node) | NFSE03 — AES-256 em dados sensíveis e sobre o hash bcrypt da senha |
| E-mail | `handlebars`, `nodemailer`, `@types/nodemailer` | RF18 — e-mails transacionais |
| Dados sintéticos | `@faker-js/faker` | RF21 — honeypot (dependência de **produção** no BIOS, onde é usada só em teste) |
| Identificadores | ObjectId nativo do MongoDB (`@default(auto())`) | Chaves primárias: geradas pelo próprio banco, compactas e ordenadas por tempo de criação |
| Datas | `date-fns` | Manipulação de datas de evidência |
| HTTP client | `axios` | Consumo das APIs externas (Seção 6) |

## 5. Bibliotecas de Teste

### 5.1 Back-end
- **Vitest 4** (test runner, padrão do NestJS 12) — API de mocks equivalente à do Jest
  (`vi.fn`, `vi.spyOn`), execução nativa de TypeScript/ESM sem transformador dedicado e
  cobertura via `@vitest/coverage-v8`, com limite mínimo de 75% configurado ([NFPD03]).
  A série 5 fica adiada: ela exige o binário nativo do Rolldown já na inicialização, e esse
  binário é declarado como dependência opcional *e* peer — combinação que o npm não instala
  a partir do lockfile, deixando o runner sem como subir.
  Ficam fora da medição, pelo sufixo do nome, o bootstrap da aplicação, os seeders
  (`seeder.ts`), a configuração (`config.ts`), as exceções (`.exception.ts`), os protocolos
  (`.protocol.ts`), os decorators (`.decorator.ts`) e os módulos do Nest (`.module.ts`); os
  testes desses arquivos continuam sendo executados, apenas não entram no cálculo;
- **@nestjs/testing** — `Test.createTestingModule` com `overrideProvider` para substituir
  dependências pelo container de injeção, evitando mock de módulo inteiro;
- **supertest** + `@types/supertest` — testes e2e de endpoint HTTP;
- **@faker-js/faker** — dados de **todas** as suítes de teste, unitárias e e2e: nenhum valor
  arbitrário é fixado à mão. Os registros são gerados por factories (`test/factories/`) e,
  quando um teste falha, a semente usada é exibida para reproduzir exatamente os mesmos
  dados.

### 5.2 Front-end (Angular)
- **Jasmine + Karma** — testes unitários, padrão do Angular CLI;
- **@testing-library/angular** — testes de componente, com foco em comportamento observável
  pelo usuário em vez de detalhe de implementação interna;
- **Playwright** ou **Cypress** — testes e2e de fluxo completo (a decidir entre os dois na
  Etapa 6, sem impacto na Etapa 5).

### 5.3 Microsserviço (ingestão/scraping)
- **Vitest** — mesmo test runner do back-end, por consistência;
- **nock** ou **MSW (Mock Service Worker)** — simulação de respostas das APIs externas nos
  testes de integração de cada adapter, sem depender da disponibilidade real da fonte durante
  o CI;
- **@faker-js/faker** — dados de todas as suítes de teste, com a mesma regra do back-end.

## 6. APIs Consumidas e Forma de Comunicação

| Fonte | Protocolo/Formato | Observação |
|---|---|---|
| BrasilAPI / OpenCNPJ / Minha Receita | HTTPS + JSON | REST, sem autenticação; cadeia de fallback nessa ordem |
| IBGE (`servicodados.ibge.gov.br`) | HTTPS + JSON | REST, sem autenticação |
| IBAMA — Dados Abertos | HTTPS + JSON | API CKAN (Action API), sem autenticação |
| CGU — Portal da Transparência (CEIS/CNEP) | HTTPS + JSON | REST, autenticação por chave de API em header |
| GHG Protocol — Registro Público de Emissões | HTTPS + HTML/CSV | **Exceção documentada** — sem API REST oficial; acesso via download estruturado, não é JSON |

A comunicação entre o BIOS e as fontes externas é majoritariamente **HTTPS com JSON**, com
única exceção no GHG Protocol, que não expõe API formal — desvio já tratado como excepcional
desde a definição das fontes de dados (hierarquia API First → dataset → download → scraping).

A comunicação **interna** entre os componentes do próprio BIOS (core API ↔ serviços de
ingestão ↔ serviço de fallback do GHG Protocol) ocorre **exclusivamente via filas BullMQ sobre
o Redis**: os serviços de ingestão são aplicações NestJS *standalone*, sem camada HTTP exposta.
Quando uma consulta exige resposta imediata (ex.: perfil de CNPJ ainda não presente em cache),
o core enfileira o job e aguarda sua conclusão com timeout; jobs que não exigem resposta
imediata (ex.: atualização periódica de evidências) são apenas enfileirados.
