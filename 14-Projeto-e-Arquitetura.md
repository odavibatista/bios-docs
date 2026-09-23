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

## 5. Pendência aberta — estados de evidência

O README (seção 4) lista seis estados de evidência (`CONFIRMED`, `NOT_FOUND`, `UNKNOWN`, `NOT_APPLICABLE`, `CONFLICTING`, `OUTDATED`); o RF08 formal, em `05-Requisitos-Funcionais.md`, lista quatro (`CONFIRMED`, `NOT_FOUND`, `UNKNOWN`, `OUTDATED`). Esta versão do documento segue o RF08 como fonte de verdade (é o requisito formal, o README é a briefing informal ainda pendente de consolidação — ver README, seção 9). Se `NOT_APPLICABLE` e `CONFLICTING` forem necessários, é decisão de escopo a confirmar antes de detalhar o RF08 em nível de implementação.
