# 14. Projeto e Arquitetura — Fontes de Dados e Integrações

## 1. Fontes de dados externas

| Fonte | Acesso | Mecanismo | Autenticação | Padrão de ingestão |
| --- | --- | --- | --- | --- |
| Receita Federal (cadastro/CNAE) | Espelho comunitário dos dumps abertos da RFB, sem API institucional | `GET brasilapi.com.br/api/cnpj/v1/{cnpj}` (principal); `GET minhareceita.org/{cnpj}` (fallback) | Nenhuma | Síncrono sob demanda, com cache |
| IBGE (CNAE — apoio) | API pública oficial | `GET servicodados.ibge.gov.br/api/v2/cnae/...` | Nenhuma | Síncrono sob demanda, com cache |
| IBAMA (autuações e embargos) | Dados abertos (CKAN) | Dumps CSV/JSON/XML em URL fixa por recurso — **sem endpoint de consulta por CNPJ** | Nenhuma | Batch assíncrono (job diário) |
| CGU — Portal da Transparência (CEIS/CNEP) | API REST oficial (Swagger documentado) | `GET api.portaldatransparencia.gov.br/api-de-dados/ceis` e `/cnep`, paginado, filtro por CNPJ | Header `chave-api-dados`, token gratuito via conta gov.br | Síncrono sob demanda, com cache |
| GHG Protocol — Registro Público de Emissões | Sem API pública | Portal web (`registropublicodeemissoes.fgv.br`) | N/A | Fallback best-effort, isolado, não bloqueante |

Observações por fonte:

- **Receita Federal**: nenhum dos dois provedores é institucional — ambos são projetos comunitários que espelham o dump aberto da RFB. A cadeia de fallback (BrasilAPI → Minha Receita) é obrigatória, não redundância opcional. O retorno da BrasilAPI já cobre RF05 e RF06 numa única chamada (situação cadastral, porte, endereço, CNAE principal e secundários com descrição).
- **IBAMA**: não existe consulta por CNPJ ao vivo nessa fonte. O MVP ingere apenas os dois recursos principais — auto de infração e termo de embargo —, cada um já carregando o documento (CPF/CNPJ) do autuado/interessado na própria linha. Os recursos auxiliares (enquadramento legal, coordenadas, bioma, espécime, anexos) ficam fora do MVP: uma auditoria de dados feita por contribuição da sociedade civil ao 5º Plano de Ação de Governo Aberto documentou ausência de chave primária confiável para cruzar esses recursos entre si (contagens de registro divergentes entre tabelas relacionadas extraídas na mesma data, formatos de número de processo inconsistentes). Tentar reconstruir o quadro completo joinando tudo é risco de estouro de prazo, não de dificuldade de código.
- **CGU CEIS/CNEP**: aceito como sinal amplo de conduta administrativa, sem filtrar por órgão sancionador — decisão explícita (ver seção 4, categorização).
- **GHG Protocol**: cobertura estruturalmente baixa (~600 organizações reportando por ano, contra milhões de CNPJs ativos no Brasil). `UNKNOWN` é o resultado esperado na maioria das consultas — não é falha de coleta, é a realidade da adesão ao programa (RIN [IND02]).

## 2. Fila, agendamento e consulta sob demanda

Decisão: **BullMQ + Redis para tudo**, sem broker adicional. BullMQ já cobre nativamente os três elementos que a ingestão do IBAMA precisa — fila, repetição agendada (job repetível = cron) e retry com backoff —, o que torna um segundo broker (ex.: RabbitMQ) redundante com a stack já adotada (ver `overview.md`) e contrário à diretriz de viabilidade solo do projeto (ver `ways-of-working.md`).

Caso de CNPJ não indexado localmente (miss no índice do lote diário do IBAMA): reaproveita o mesmo adapter/parser do job diário, disparado sob demanda como job avulso do BullMQ para aquele CNPJ específico, contra o mesmo dump oficial. Isso **não** reintroduz webscraping nem um segundo mecanismo de ingestão — é o mesmo pipeline, com gatilho diferente.

## 3. Camadas de dados

Mantido conforme já esboçado no README (seção 4), sem alteração: três coleções Mongo — `evidence_raw` (bruto por fonte) → `evidence_normalized` → `company_profile` (agregado, pronto para busca/score/exportação) —, sem infraestrutura de streaming dedicada. Cada fonte isolada atrás de um adapter próprio (`DataSourceInterface` → `ReceitaDataSource`, `IbamaDataSource`, `CguDataSource`, `GhgDataSource`), conforme já previsto em [NFPD02].

## 4. Categorização de evidências (campo `categoria` do RF08)

| Categoria | Fonte(s) | Natureza do sinal |
| --- | --- | --- |
| `INFRACAO_AMBIENTAL` | IBAMA | Evidência ambiental direta |
| `CONDUTA_ADMINISTRATIVA` | CGU (CEIS/CNEP) | Sinal amplo de conduta, não necessariamente de origem ambiental |
| `EMISSAO_GEE` | GHG Protocol | Evidência positiva de transparência/gestão de emissões; confiança sempre baixa por padrão, dada a cobertura reduzida da fonte |

Essa separação existe para que uma sanção administrativa genérica do CEIS/CNEP (que pode não ter motivação ambiental nenhuma) não seja apresentada, na explicação do score (RF12), como se fosse evidência de infração ambiental — mantendo o princípio "Evidence First" e o RIN [IND01] (não afirmar veredito absoluto a partir de sinal de natureza diferente).

## 5. Estados de evidência — decisão

O README (seção 4) e o RF08 formal (`05-Requisitos-Funcionais.md`) chegaram a listar números diferentes de estados (seis vs. quatro). Decisão: os quatro estados do RF08 (`CONFIRMED`, `NOT_FOUND`, `UNKNOWN`, `OUTDATED`) são o modelo definitivo do MVP. `NOT_APPLICABLE` e `CONFLICTING` ficam descartados do escopo atual e registrados aqui como trabalho futuro, não perdidos. O README foi atualizado (seção 4) para refletir essa decisão.

## 6. Justificativa da arquitetura conforme ISO/IEC 25010

A arquitetura descrita nas seções 1 a 4 é justificada abaixo por característica de qualidade da ISO/IEC 25010:2011 (referenciada em `12-Referências.md`), amarrada aos RNFs já formalizados em `06-Requisitos-Nao-Funcionais.md` sempre que exista requisito correspondente.

| Característica ISO/IEC 25010 | Como a arquitetura do BIOS a endereça | RNF/decisão relacionada |
| --- | --- | --- |
| Adequação Funcional | Modelo evidência-primeiro (dado → evidência → regra → indicador → score) com estado explícito por evidência (RF08) garante que cada fonte contribua de forma isolada e auditável para o score, sem sobreposição de responsabilidade entre adapters | RF07, RF08, RF10 |
| Eficiência de Desempenho | Cache de consulta evita nova chamada síncrona à fonte externa dentro da janela de validade; ingestão pesada do IBAMA roda como job assíncrono em lote (BullMQ), fora do caminho crítico da consulta em tempo real | NFDE01, NFDE02 |
| Compatibilidade | Cada fonte isolada atrás de `DataSourceInterface`; a cadeia de fallback do cadastro (BrasilAPI → Minha Receita) troca de provedor sem alterar contrato interno nem lógica de negócio dependente | NFPD02; seção 1 |
| Usabilidade | Endereçada apenas parcialmente nesta camada (o grosso é front-end); a separação estrutural entre score e confiança, e a explicação evidência a evidência (RF12), existem para que o resultado seja interpretável sem exigir conhecimento prévio das bases governamentais de origem | RF11, RF12 |
| Confiabilidade | Degradação graciosa por fonte — indisponibilidade de uma fonte (ex.: GHG Protocol) não bloqueia a consulta inteira; job de ingestão com retry/backoff nativo do BullMQ | NFCO01, NFCO02 |
| Segurança | Criptografia AES-256, hash de chave de API, sessão JWT revogável, rate limiting, bloqueio de login, honeypot — já formalizados como bloco dedicado de RNF | NFSE01–NFSE11 |
| Manutenibilidade | Isolamento por adapter (um módulo por fonte) e camadas lógicas `evidence_raw` → `evidence_normalized` → `company_profile` reduzem acoplamento entre ingestão, normalização e agregação — alterar uma fonte não exige tocar nas demais | NFPD02; seção 3 |
| Portabilidade | Stack inteiramente Node.js/TypeScript, com Prisma ORM como camada de abstração sobre MongoDB; permanência deliberada na Prisma v6 (em vez da v7) preserva justamente essa portabilidade, já que a v7 não suporta MongoDB | `13-Ficha-Técnica.md`, seção 3 |

Usabilidade e Compatibilidade dependem em parte de decisões de front-end ainda não formalizadas; a tabela cobre apenas a parcela resolvida pela arquitetura de back-end/dados deste documento.
