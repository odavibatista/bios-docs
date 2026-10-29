# Ficha Técnica — Tecnologias do BIOS

> Documento de referência técnica do projeto BIOS, cobrindo linguagens, frameworks,
> bibliotecas, ferramentas de teste e integrações externas. Complementa a Etapa 5
> (Projeto e Arquitetura) da documentação técnica.

## 1. Linguagens de Programação

- **TypeScript 6** — linguagem principal em toda a stack (back-end, front-end e
  módulos de ingestão), com `strict` habilitado no `tsconfig.json`. O back-end é compilado como
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
| Ingestão de fontes externas | Módulos do próprio `bios-core-api` (um por fonte), com consumidores de fila | Sem serviço à parte: quem pede e quem atende o job conversam só via fila (BullMQ), então um consumidor pode ser movido para um processo próprio sem mudar o contrato |

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
| Configuração | `@nestjs/config` + schema Zod | Variáveis de ambiente validadas no bootstrap (configuração inválida impede a aplicação de subir). A credencial `CGU_API_KEY` é opcional: sua ausência mantém CEIS/CNEP indisponíveis sem impedir o serviço, e aparece no `/health` sem expor o token |
| Documentação de API | `@nestjs/swagger`, `@scalar/nestjs-api-reference` | RF17 — API pública documentada |
| Cache | `ioredis` (mesmo cliente das filas) | NFDE01 — cache de consultas por CNPJ. A pilha `@nestjs/cache-manager` + `cache-manager` + `keyv` + `@keyv/redis`, prevista inicialmente, foi descartada: além de não resolver sozinha o requisito (degradação graciosa e orçamento de latência precisam de tratamento próprio em qualquer caso), ela traria o `@redis/client` como **segundo** driver de Redis ao lado do `ioredis` exigido pelo BullMQ, e as versões atuais não convivem — `cache-manager` 7 fixa `keyv` 5, enquanto o `@keyv/redis` 6 exige `keyv` 6. A superfície usada é `GET`/`SET EX`/`DEL`, que o `ioredis` já cobre |
| Filas / jobs assíncronos | `@nestjs/bullmq`, `bullmq` (série 6), `ioredis` | Comunicação interna entre quem pede e quem atende os jobs de ingestão (consumidores hospedados no core), ingestão periódica de evidências e scraping do GHG Protocol. A série 6 do BullMQ transformou o cliente de Redis em dependência *peer* opcional — ele passou a aceitar `ioredis`, `redis` ou até Postgres como backend —, de modo que o driver é escolhido explicitamente: `ioredis`, o de uso consolidado com BullMQ |
| Rate limiting | `@nestjs/throttler`, com contadores em Redis (`ioredis`) | NFSE04 — limite geral por IP, limite da API pública por chave de API e limite próprio, por IP e configurável (`THROTTLE_COMPANY_*_LIMIT`), das rotas de perfil, evidências, pesquisa e exportação. Os contadores ficam em Redis para que o limite valha por consumidor, e não por instância da API. O adaptador de armazenamento da comunidade (`@nest-lab/throttler-storage-redis`) não foi adotado porque declara compatibilidade só até o NestJS 11; a interface `ThrottlerStorage` é de um único método e foi implementada sobre o mesmo cliente do cache |
| Validação e serialização | `zod` (Standard Schema nativo do NestJS 12) | Schemas de request e response: validação de entrada (`StandardSchemaValidationPipe`), serialização de saída que descarta campos não declarados (`StandardSchemaSerializerInterceptor`) e geração do OpenAPI (`@nestjs/swagger`) a partir do mesmo schema |
| Autenticação | `jsonwebtoken`, `@types/jsonwebtoken`, `bcryptjs` | RF02, NFSE05 — login e hash de senha. O access token é um JWT assinado com HMAC-SHA256 (HS256), segredo de 32 bytes em `AUTH_ACCESS_TOKEN_SECRET`, vida curta (`AUTH_ACCESS_TOKEN_LIFETIME`, padrão 15 minutos) e afirmações mínimas — `sub` (usuário), `sid` (elo da sessão), `role` e emissor —, sem dado pessoal. O algoritmo é fixado na verificação, o que barra `alg: none` e a troca de algoritmo. O token apenas identifica a sessão; a validade vem da consulta ao elo em `user_sessions`, feita a cada requisição por um guard global (`AccessTokenGuard`) que torna toda rota privada por padrão — as públicas se declaram com `@Public()` — e roda depois do limite de requisições. O refresh token é opaco, de 256 bits, e só o resumo SHA-256 é persistido |
| Criptografia | `crypto` (nativo do Node) | NFSE03 — AES-256 em dados sensíveis (e-mail, endereço serializado em JSON) e sobre o hash bcrypt da senha. O documento do usuário registra **uma** versão de chave para todos os campos cifrados: toda escrita de campo cifrado passa por um único caso de uso que, se a chave ativa for mais nova, recifra os demais campos na mesma gravação |
| Tokens de uso único | `crypto` (nativo do Node) | RF25, RF26, RF28 — links de e-mail com token aleatório de 256 bits, do qual só o resumo SHA-256 é persistido (`user_tokens`). Validade por finalidade, em variáveis de ambiente: 24 horas na confirmação de cadastro e de novo e-mail (`AUTH_EMAIL_CONFIRMATION_TOKEN_LIFETIME`) e 2 horas na redefinição de senha (`AUTH_PASSWORD_RESET_TOKEN_LIFETIME`). O uso único é garantido por escrita condicional no banco, o que vale também sob requisições simultâneas, e emitir um novo token invalida o anterior da mesma finalidade. O mesmo mecanismo serve ao refresh token (RF32) |
| E-mail | `handlebars`, `nodemailer`, `@types/nodemailer` | RF18 — e-mails transacionais. O envio é assíncrono: o caso de uso registra o disparo em `email_dispatch_logs` e enfileira, e o consumidor da fila `email-dispatch` — hospedado no próprio core, por não haver serviço separado de envio — entrega a mensagem e fecha a auditoria. Nas suítes o transporte é substituído pelo `jsonTransport` do próprio Nodemailer, sem SMTP real |
| Dados sintéticos | `@faker-js/faker` | RF21 — honeypot: dependência de **produção**, que gera em pt-BR as empresas sintéticas das rotas isca (com instância própria do Faker e CNPJ válido calculado pelo value object de CNPJ); nos testes, gera os dados de todas as suítes |
| Identificadores | ObjectId nativo do MongoDB (`@default(auto())`) | Chaves primárias: geradas pelo próprio banco, compactas e ordenadas por tempo de criação |
| Datas | `Date` nativo, em UTC | Datas de evidência, janelas mensais da evolução setorial e prazos de token. A `date-fns`, prevista inicialmente, não foi adotada: as operações usadas (início do mês em UTC, soma de durações, comparação) são poucas e cobertas pelo `Date` |
| Planilha XLSX | `write-excel-file` | RF15, UC09 — exportação da lista de empresas. Monta a planilha em memória, com o teto de linhas de `EXPORT_MAX_COMPANIES` (padrão 5 000, máximo 20 000) |
| HTTP client | `fetch` nativo do Node | Consumo das APIs externas (Seção 6), com tempo máximo da chamada inteira, sem redirecionamentos e com corpo limitado; os dumps em lote são baixados em fluxo para o disco, com limite de tamanho. O `axios`, previsto inicialmente, não foi adotado: o `fetch` cobre a necessidade sem dependência extra |
| Leitura dos dumps do IBAMA | `csv-parse`, `yauzl` | UC14 — leitura em fluxo do CSV (separador `;`, texto com quebras de linha, BOM) e do `.zip` de CSVs, sem extrair para o disco nem carregar o arquivo na memória (o auto de infração passa de 700 MB descompactado). Bibliotecas de uso consolidado, sem dependências relevantes, em vez de um *parser* próprio de CSV e de `.zip` |

## 5. Bibliotecas de Teste

### 5.1 Back-end
- **Vitest 4** (test runner, padrão do NestJS 12) — API de mocks equivalente à do Jest
  (`vi.fn`, `vi.spyOn`), execução nativa de TypeScript/ESM sem transformador dedicado e
  cobertura via `@vitest/coverage-v8`, com limite mínimo de 75% configurado ([NFPD03]).
  Os testes rodam em três momentos: no commit, os afetados pelos arquivos em stage
  ([NFSE09]); no push, a verificação de tipos, o lint do projeto inteiro e os unitários
  afetados pelos arquivos enviados; no CI, a suíte inteira com cobertura e os e2e, que é a
  verificação exigida para o merge ([NFSE11]). Os e2e rodam em paralelo, com banco MongoDB,
  banco Redis e pasta temporária próprios para cada worker; com 4 workers, a suíte cai de
  642 s para 187–206 s, sem nenhum teste removido ou afrouxado.
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

### 5.3 Adapters de fontes externas e consumidores de fila
- **Vitest** — o mesmo test runner do back-end, pois os adapters e os consumidores são módulos
  do próprio `bios-core-api`;
- **Servidor HTTP local** (`node:http`, em porta livre) — simulação das APIs externas nos
  testes e2e de cada adapter: a URL base da fonte, que fica no catálogo `data_sources`, é
  apontada para ele, e o formato e o comportamento (sucesso, "não encontrado", erro, demora)
  são configuráveis por teste. Nenhum teste chama a internet, então o CI não depende da
  disponibilidade real da fonte;
- **Testes de contrato** (Vitest, `test/contract/`) — como a simulação reproduz a forma que o
  código supõe, uma mudança de formato da fonte real passaria despercebida (um cadastro de
  1029 organizações lido como uma amostra de 10 escapou assim). Um workflow agendado e manual
  do GitHub Actions, **fora do CI de pull request** e não bloqueante, consulta IBAMA, CGU e GHG
  com os mesmos leitores da produção e confere campos, paginação, tamanho e cabeçalhos; a
  credencial da CGU vem só de secret do repositório. As imagens do MongoDB e do Redis do CI
  são fixadas por digest;
- **@faker-js/faker** — dados de todas as suítes de teste, com a mesma regra do back-end.

## 6. APIs Consumidas e Forma de Comunicação

| Fonte | Protocolo/Formato | Observação |
|---|---|---|
| BrasilAPI / OpenCNPJ / Minha Receita | HTTPS + JSON | REST, sem autenticação; cadeia de fallback nessa ordem |
| IBGE (`servicodados.ibge.gov.br`) | HTTPS + JSON | REST, sem autenticação |
| IBAMA — Dados Abertos | HTTPS + JSON | API CKAN (Action API), sem autenticação |
| CGU — Portal da Transparência (CEIS/CNEP) | HTTPS + JSON | REST, autenticação por chave de API em header |
| GHG Protocol — Registro Público de Emissões | HTTPS + JSON/PDF | **Exceção documentada** — catálogo público usado pelo portal (sem contrato REST oficial estável) e download do relatório PDF; o CNPJ e o ano do relatório confirmam a identidade |

CEIS/CNEP usam o token `CGU_API_KEY`, obrigatório e validado no ambiente da API,
enviado somente no header `chave-api-dados`. A chave real não integra o repositório
nem os logs; o cadastro ocorre pela conta gov.br no Portal da Transparência.

O adapter GHG lista o cadastro de organizações do portal pela busca paginada
(`SearchAllOrganizations`, até 1000 por página) e baixa o relatório PDF do
inventário mais recente de cada candidato (até três candidatos pelo nome cadastral
local). A rota `GetAllOrganizations` não serve de cadastro: devolve só uma amostra
de 10 organizações. A listagem é sincronizada uma vez por dia por um job agendado na fila
`ghg-inventory` e guardada no cache (Redis) por sete dias; a coleta de cada CNPJ a lê
dali, sem chamada externa para localizar candidatos. O nome apenas localiza: só o CNPJ exato e o ano inventariado do PDF permitem
confirmar a evidência. A extração usa `pdfjs-dist` em thread descartável, com prazo e
memória limitados; somente CNPJ, identificadores da organização/ciclo e ano são
mantidos, sem contatos nem texto integral do PDF. Ausência de correspondência,
formato inválido e indisponibilidade resultam em `UNKNOWN`.

A comunicação entre o BIOS e as fontes externas é majoritariamente **HTTPS com JSON**, com
única exceção no GHG Protocol, que não expõe API formal — desvio já tratado como excepcional
desde a definição das fontes de dados (hierarquia API First → dataset → download → scraping).

A comunicação **interna** entre quem pede e quem atende os jobs de ingestão (cadastro por CNPJ,
evidências ambientais, inventário do GHG Protocol) ocorre **exclusivamente via filas BullMQ
sobre o Redis**. Os consumidores são hospedados no próprio `bios-core-api`, sem camada HTTP
entre as partes; como o contrato do job é o único acoplamento, um consumidor pode ser movido
para um processo próprio quando a carga pedir.
Quando uma consulta exige resposta imediata (ex.: perfil de CNPJ ainda não presente em cache),
o core enfileira o job e aguarda sua conclusão com timeout; jobs que não exigem resposta
imediata (ex.: atualização periódica de evidências) são apenas enfileirados.
