# BIOS — Briefing de Sessão e Contexto de Projeto

## 1. Visão Geral do Projeto

**Nome:** BIOS (vida, em grego).
**Tema oficial (A3):** Inteligência Comercial de Agenda Verde: Prospecção Ativa e B2B Ambiental.
**Contexto acadêmico:** Avaliação 3 das disciplinas de Engenharia de Software e Gestão e
Qualidade de Software, curso de graduação em Ciência da Computação. Desenvolvimento
individual. Cronograma até novembro de 2026, com potencial de virar artigo acadêmico
(indicação do professor da disciplina).

**Definição resumida:** BIOS é uma plataforma que usa o CNPJ como eixo de integração de
dados públicos, consolida evidências ambientais de uma empresa a partir de múltiplas fontes
oficiais, relaciona essas evidências a ODS específicos, e gera um índice de aderência
ambiental explicável, acompanhado de um nível de confiança — permitindo busca, filtragem e
exportação de empresas segundo critérios socioambientais.

## 2. Escopo Confirmado

### O que o BIOS faz
- Consulta de empresa por CNPJ ou razão social.
- Exibição de dados cadastrais públicos e descrição de CNAE.
- Consolidação de evidências ambientais públicas por CNPJ (positivas e negativas).
- Cálculo de um índice de aderência ambiental por ODS, com nível de confiança separado.
- Explicabilidade do score (quais evidências, que peso, positivo/negativo).
- Filtros de busca (CNAE, UF, município, score mínimo, confiança mínima) e ranking.
- Exportação de listas filtradas em XLSX.
- Painel de conta com geração de chaves de API para consumo programático.

### O que o BIOS explicitamente NÃO faz (decisão de escopo, não limitação técnica)
- Não identifica, busca ou qualifica tomadores de decisão (pessoas físicas) nas empresas.
- Não coleta e-mail, telefone, WhatsApp ou perfil de LinkedIn de indivíduos.
- Não gera leads comerciais nem serve fluxos de outbound/pré-vendas/vendas.
- Não integra com CRMs para exportação de contatos.

Essa exclusão foi decidida cedo no projeto e é o principal ponto de diferenciação frente a
ferramentas de prospecção B2B genéricas — documentado à parte em "BIOS — Proposta e Nicho".

## 3. Fontes de Dados Validadas

| Fonte | Dado fornecido | Tipo de acesso | Observação |
|---|---|---|---|
| Receita Federal (via BrasilAPI, fallback Minha Receita) | Cadastro de pessoa jurídica | API pública, sem chave | Camada de identidade, não gera evidência ambiental por si só; nenhum dos dois provedores é institucional, cadeia de fallback é obrigatória |
| IBGE | Descrição de CNAE | API pública, sem chave (`servicodados.ibge.gov.br`) | Fonte de apoio para taxonomia do CNAE; BrasilAPI já retorna descrição do CNAE principal e secundários numa única chamada |
| IBAMA — Dados Abertos | Autuações e embargos ambientais | Dados abertos (CKAN) — **sem consulta por CNPJ ao vivo**, apenas dumps em lote | Ingestão batch assíncrona (job diário via BullMQ); MVP restrito aos dois recursos principais (auto de infração + embargo) — ver `Projeto-e-Arquitetura.md` |
| CGU — Portal da Transparência (CEIS/CNEP) | Sanções administrativas | API pública, chave via cadastro simples | Aceito como sinal amplo de conduta administrativa, sem filtrar por órgão sancionador — categorizado à parte de evidência ambiental (RF08) |
| GHG Protocol Brasil — Registro Público de Emissões | Inventários de emissões | Portal web, sem API REST documentada | Evidência positiva; fallback best-effort isolado e não bloqueante — cobertura estruturalmente baixa, `UNKNOWN` é o resultado esperado na maioria das consultas |

**Descartadas do MVP (documentado, não esquecido):** DataJud/CNJ (processos judiciais),
SICAR/CAR (dados geoespaciais rurais), LinkedIn e qualquer fonte de dado de contato pessoal.

## 4. Modelo Conceitual / Arquitetura de Dados

Princípio central do projeto: **"Evidence First"** — nenhum score existe sem evidência
rastreável.

DADO → EVIDÊNCIA → REGRA → INDICADOR → SCORE


- **Estados de evidência:** `CONFIRMED`, `NOT_FOUND`, `UNKNOWN`, `OUTDATED` — nunca booleano
  simples. Ausência de dado ≠ evidência negativa. (`NOT_APPLICABLE` e `CONFLICTING`, citados
  em versões anteriores deste briefing, foram descartados do MVP por decisão de escopo — RF08
  é a fonte de verdade formal; retomáveis como trabalho futuro.)
- **Score ≠ Confiança:** dois números separados. Score alto com confiança baixa significa
  poucas evidências disponíveis, não ausência de prática.
- **Score explicável:** toda pontuação deve ser derivável e exibível evidência por evidência
  (peso, sinal, fonte).
- **Camadas lógicas (Bronze/Silver/Gold), implementadas como coleções, não como
  infraestrutura separada:** `evidence_raw` (bruto por fonte) → `evidence_normalized`
  (normalizado) → `company_profile` (agregado, pronto para busca/score/exportação). Decisão
  tomada para conter o escopo de infraestrutura de um projeto solo.
- **Adapters por fonte:** cada fonte externa isolada atrás de uma interface
  (`DataSourceInterface` → `ReceitaDataSource`, `IbamaDataSource`, etc.), permitindo adicionar
  fontes sem alterar a lógica de negócio central.
- **Cache + snapshot leve:** evitar nova consulta à fonte externa dentro de uma janela de
  validade; versionamento simples via `updatedAt` + hash anterior, sem motor de diff dedicado.
- **Cortado do MVP por decisão de escopo:** Opportunity Score (separado do Environmental
  Score), motor de regras configurável (pesos hardcoded na v1), exportação em PDF (XLSX
  primeiro).

## 5. Stack Técnica

- **Back-end:** NestJS + Prisma + **MongoDB** (dado não estruturado, disperso, sem grandes
  relações — decisão tomada e mantida.
- **Front-end:** Angular moderno como primeira escolha, aberto a Next.js/React+TanStack Query
  por velocidade de entrega solo.
- **Microsserviços leves:** `bios-core-api` (orquestração, API pública, contas/chaves),
  módulo/serviço de ingestão CNPJ, módulo/serviço de ingestão ambiental (score), serviço
  isolado de fallback para o GHG Protocol rodando à parte para não derrubar a API principal
  em caso de indisponibilidade ou falha da fonte (SICAR já descartado do MVP — ver seção 2).

## 6. Padrão de Documentação Técnica Adotado

Referência principal: wiki de um projeto de referência usado como modelo (template rigoroso,
com RF/RNF/RIN por ID, prioridade e ator). O projeto **Coyote** (ERP incompleto do próprio
autor) foi analisado como contraexemplo — mesma base de template, mas que degenerou em
especificação de tela em prosa livre a partir do `Home.md`; decisão explícita de **não
repetir esse padrão** no BIOS.

Mapeamento da estrutura de documentação adotada → estrutura exigida pela A3 (item 8 da
proposta da disciplina):

| Seção da documentação técnica | Seção A3 correspondente |
|---|---|
| 01 Introdução | Introdução |
| 02 Visão Geral do Produto e Projeto | Problema, Contexto, Justificativa, Objetivos |
| 03 Sistemas Relacionados | (sem equivalente direto — contexto de mercado) |
| 04 (a escrever) | Modelagem — casos de uso, classes, sequência |
| 05/06/07 RF/RNF/RIN | Engenharia de Requisitos |
| 08 Impacto Ambiental | Impacto Ambiental |
| 10 Gerenciamento do Projeto | Gerenciamento do Projeto |
| 11 (a escrever) | Permissões (adaptado, simplificado) |
| 12 Referências | Referências |
| 13 Ficha Técnica | (sem equivalente direto — complementa Projeto e Arquitetura) |
| 14 Projeto e Arquitetura | Projeto e Arquitetura |

## 7. Lições de Engenharia Aplicáveis ao BIOS

Os seguintes pontos foram identificados como práticas a **adotar desde o início no BIOS**, para
não repetir no BIOS problemas encontrados lá:

- Injetar o cliente Prisma via `PrismaService` do próprio Nest, nunca como singleton global
  importado diretamente (`export const prisma = new PrismaClient()`), para preservar
  testabilidade via `overrideProvider` em vez de `jest.mock` de módulo inteiro.
- Tipar/injetar usecases contra a **interface** do repositório, nunca contra a classe
  concreta — mesmo quando a interface já existe, é fácil injetar a implementação por engano
  e anular o propósito da inversão de dependência.
- Dar generics reais à interface base de usecase (`UseCaseInterface<TParams, TReturn>`),
  nunca `(...params: any) => any`.
- Não escrever teste de especificação para interfaces puras (não têm comportamento em
  runtime); testar comportamento real (DTOs/schemas de validação, lógica de usecase).
- Evitar `@Res()` do Express nos controllers do Nest — usar o retorno automático do
  framework para manter interceptors, filtros de exceção e serialização padrão ativos.
- Não deixar o módulo raiz (`AppModule`) crescer como lista plana de dezenas de módulos;
  agrupar por domínio em módulos agregadores desde cedo.
- Cuidado com nomes de módulo quase-idênticos e semanticamente ambíguos (ex.: módulos
  chamados com as mesmas palavras em ordem invertida) — nomear pelo conceito de negócio, não
  pela combinação de substantivos.
- Manter o `README.md` raiz atualizado desde o início — não deixar o boilerplate padrão do
  `nest new` em um projeto que crescerá em escopo.

## 8. Requisitos de Segurança e Engenharia — Referência

Lista completa de RF/RNF definidos, com status de formalização, está no documento separado
`BIOS_Status_Requisitos.md`. Resumo dos temas cobertos: criptografia AES-256 de dados
sensíveis, rate limiting (Throttler), honeypot de nível mediano (Faker.js + gerador de CNPJ,
baixo volume, IP bloqueado via decorator), blacklist de domínios de e-mail (JSON seed +
entidade em banco), tracking e bloqueio de tentativas de login (5 tentativas / 15 min),
sessões JWT revogáveis, cobertura de teste ≥75% (unitário, integração, componente, e2e,
funcional — front e back), Husky, Dependabot, ESLint, branches `main`/`pre-prod`/`develop`
como convenção (sem hospedagem tríplice), pipelines YAML no CI apenas para lint+teste,
templates de PR (.md) e Issue (.yaml), GitHub Projects em Kanban de fluxo com labels por tipo
de teste, modelo de desenvolvimento incremental, e-mails via Handlebars+Nodemailer, Clean
Architecture/SOLID/OO como princípio arquitetural.

## 9. Pendências em Aberto

- **Etapa 4 (Modelagem):** diagrama de classes, casos de uso e sequência — não iniciados.
- **Documento técnico consolidado:** ainda não existe um único arquivo unindo todas as seções
  na ordem exigida pelo item 8 da proposta da A3 (Introdução → Referências).
- **Diagramas de arquitetura (C4/componentes):** não
  produzidos.
