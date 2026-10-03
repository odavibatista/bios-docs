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

### 3.1 ODS como entidade de referência

Os ODS aos quais os índices de aderência (RF10) se referem são modelados como entidade própria — coleção `ods`, com número, título e descrição —, populada via seed na inicialização do banco, no mesmo padrão adotado para a blacklist de domínios (RF22). No MVP, a seed contém dois registros:

| ODS | Título | Fonte(s) de evidência | Sinal disponível |
| --- | --- | --- | --- |
| 13 | Ação Contra a Mudança Global do Clima | GHG Protocol — Registro Público de Emissões | Apenas positivo (publicação de inventário de emissões) |
| 15 | Vida Terrestre | IBAMA (autos de infração e termos de embargo); CGU — CEIS/CNEP como sinal complementar de peso reduzido (ver seção 4) | Apenas negativo (autuação/embargo; sanção administrativa) |

Cada evidência normalizada referencia o ODS ao qual se vincula, e o `company_profile` armazena uma lista de resultados por ODS (`{ odsId, indice, confianca }`) em vez de um par único índice/confiança. Nenhum ODS é assumido de forma implícita no código: incluir um novo ODS passa a ser uma nova entrada de seed somada às regras de pontuação correspondentes, sem alteração de esquema — o que preserva a escalabilidade a outros ODS prevista no objetivo geral do projeto.

Limitações assumidas e registradas:

- **ODS 13 com cobertura estruturalmente baixa.** A única fonte é o GHG Protocol (~600 organizações reportando por ano — ver seção 1), sem API e tratada, em tempo de execução, como fallback best-effort isolado (sua ingestão é prioridade Essencial de entrega — ver `10-Gerenciamento-do-Projeto.md` —, mas sua indisponibilidade nunca bloqueia a consulta). Para a grande maioria das empresas, o índice do ODS 13 terá estado `UNKNOWN` e confiança baixa. Isso não é falha de coleta e não pode ser tratado como evidência negativa (RIN [IND02]); é exatamente o cenário que a separação índice/confiança (RF11) existe para comunicar.
- **Sinais unilaterais.** Nas fontes do MVP, o ODS 13 dispõe apenas de sinal positivo e o ODS 15 apenas de sinal negativo. Os dois índices, portanto, não são comparáveis entre si nem combinados em um índice geral único; filtros e ordenação (RF13, RF14) operam sempre sobre um ODS selecionado.

## 4. Categorização de evidências (campo `categoria` do RF08)

| Categoria | Fonte(s) | ODS vinculado | Natureza do sinal |
| --- | --- | --- | --- |
| `INFRACAO_AMBIENTAL` | IBAMA | 15 | Evidência ambiental direta |
| `CONDUTA_ADMINISTRATIVA` | CGU (CEIS/CNEP) | 15 (sinal complementar, peso reduzido — ver nota abaixo) | Sinal amplo de conduta, não necessariamente de origem ambiental |
| `EMISSAO_GEE` | GHG Protocol | 13 | Evidência positiva de transparência/gestão de emissões; confiança sempre baixa por padrão, dada a cobertura reduzida da fonte |

Essa separação existe para que uma sanção administrativa genérica do CEIS/CNEP (que pode não ter motivação ambiental nenhuma) não seja apresentada, na explicação do score (RF12), como se fosse evidência de infração ambiental — mantendo o princípio "Evidence First" e o RIN [IND01] (não afirmar veredito absoluto a partir de sinal de natureza diferente).

**Decisão — vínculo de `CONDUTA_ADMINISTRATIVA` ao ODS 15 como sinal complementar.** Sanções do CEIS/CNEP não se enquadram diretamente em nenhum dos dois ODS do MVP; tematicamente, estariam mais próximas do ODS 16 (Paz, Justiça e Instituições Eficazes), fora do escopo. Ainda assim, a categoria compõe o índice do ODS 15, com as seguintes salvaguardas:

- **Peso reduzido:** o peso de uma evidência `CONDUTA_ADMINISTRATIVA` é sempre inferior ao de uma evidência `INFRACAO_AMBIENTAL`, de modo que uma sanção administrativa genérica nunca pese tanto quanto uma autuação ou embargo ambiental (pesos hardcoded na v1 — ver README, seção 4);
- **Categoria preservada na explicação:** na explicação do score (RF12), a evidência continua rotulada como conduta administrativa, nunca como infração ambiental — a separação de categorias descrita acima permanece válida;
- **Apenas sinal negativo confirmado:** pontua somente com estado `CONFIRMED`; a ausência de sanção (`NOT_FOUND`) não gera bônus, por não constituir evidência de boa conduta ambiental.

Justificativa da escolha pelo ODS 15 e não pelo ODS 13: tanto o IBAMA quanto o CEIS/CNEP registram sanções administrativas aplicadas por órgãos públicos, ou seja, sinais de mesma natureza (negativa, sancionatória), o que mantém o índice do ODS 15 internamente coerente. Vinculá-la ao ODS 13 misturaria um sinal sancionatório sem relação com o clima a uma fonte de natureza oposta (publicação voluntária de inventário de emissões).

## 5. Estados de evidência — decisão

O README (seção 4) e o RF08 formal (`05-Requisitos-Funcionais.md`) chegaram a listar números diferentes de estados (seis vs. quatro). Decisão: os quatro estados do RF08 (`CONFIRMED`, `NOT_FOUND`, `UNKNOWN`, `OUTDATED`) são o modelo definitivo do MVP. `NOT_APPLICABLE` e `CONFLICTING` ficam descartados do escopo atual e registrados aqui como trabalho futuro, não perdidos. O README foi atualizado (seção 4) para refletir essa decisão.

## 6. Justificativa da arquitetura conforme ISO/IEC 25010:2023

A arquitetura descrita nas seções 1 a 4 é justificada abaixo pelas nove características de
qualidade da ISO/IEC 25010:2023 (referenciada em `12-Referências.md`), amarrada aos RNFs já
formalizados e reclassificados em `06-Requisitos-Nao-Funcionais.md`.

| Característica ISO/IEC 25010:2023 | Como a arquitetura do BIOS a endereça | RNF/decisão relacionada |
| --- | --- | --- |
| Adequação Funcional | Modelo evidência-primeiro (dado → evidência → regra → indicador → score) com estado explícito por evidência (RF08) garante que cada fonte contribua de forma isolada e auditável para o score, sem sobreposição de responsabilidade entre adapters | RF07, RF08, RF10; NFCO02 |
| Eficiência de Desempenho | Cache de consulta evita nova chamada síncrona à fonte externa dentro da janela de validade; ingestão pesada do IBAMA roda como job assíncrono em lote (BullMQ), fora do caminho crítico da consulta em tempo real | NFDE01, NFDE02 |
| Compatibilidade | Cada fonte isolada atrás de `DataSourceInterface`; a cadeia de fallback do cadastro (BrasilAPI → Minha Receita) troca de provedor sem alterar contrato interno; rate limit de cada fonte externa respeitado | NFPD01; seção 1 |
| Capacidade de Interação | Não endereçada nesta camada de arquitetura (back-end/dados) — depende de decisões de front-end ainda não formalizadas | — (ver `06-Requisitos-Nao-Funcionais.md`, seção 6.4) |
| Confiabilidade | Degradação graciosa por fonte — indisponibilidade de uma fonte (ex.: GHG Protocol) não bloqueia a consulta inteira; job de ingestão com retry/backoff nativo do BullMQ | NFCO01 |
| Segurança | Criptografia AES-256, hash de chave de API, sessão JWT revogável, rate limiting, bloqueio de login, honeypot, monitoramento de dependências | NFSE01–NFSE08 |
| Manutenibilidade | Isolamento por adapter (um módulo por fonte), camadas lógicas `evidence_raw` → `evidence_normalized` → `company_profile`, cobertura mínima de teste, barreira pré-commit e análise estática | NFPD02, NFPD03, NFSE09–NFSE11; seção 3 |
| Flexibilidade | Mesmo isolamento por adapter também sustenta adição de novas fontes sem alterar o núcleo, ainda que formalizado sob Manutenibilidade | NFPD02 (referência cruzada) |
| Segurança Operacional (Safety) | Não aplicável — o BIOS não atua sobre processo físico ou operacional de terceiros | — |

Capacidade de Interação depende de decisões de front-end ainda não formalizadas; a tabela
cobre apenas a parcela resolvida pela arquitetura de back-end/dados deste documento.
