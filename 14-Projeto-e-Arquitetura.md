# 14. Projeto e Arquitetura — Fontes de Dados e Integrações

## 1. Fontes de dados externas

| Fonte | Acesso | Mecanismo | Autenticação | Padrão de ingestão |
| --- | --- | --- | --- | --- |
| Receita Federal (cadastro/CNAE) | Espelho comunitário dos dumps abertos da RFB, sem API institucional | `GET brasilapi.com.br/api/cnpj/v1/{cnpj}` (principal); `GET api.opencnpj.org/{cnpj}` (1º fallback); `GET minhareceita.org/{cnpj}` (fallback final) | Nenhuma | Síncrono sob demanda, com cache |
| IBGE (CNAE — apoio) | API pública oficial | `GET servicodados.ibge.gov.br/api/v2/cnae/...` | Nenhuma | Síncrono sob demanda, com cache |
| IBAMA (autuações e embargos) | Dados abertos (CKAN) | Dumps CSV/JSON/XML em URL fixa por recurso, hospedados no armazenamento do IBAMA (`.zip` de CSVs por ano para os autos; `.csv` para os embargos) — **sem endpoint de consulta por CNPJ** | Nenhuma | Batch assíncrono (job diário) |
| CGU — Portal da Transparência (CEIS/CNEP) | API REST oficial (Swagger documentado) | `GET api.portaldatransparencia.gov.br/api-de-dados/ceis` e `/cnep`, paginado, filtro por CNPJ | Header `chave-api-dados`, token gratuito via conta gov.br | Sob demanda pela fila ambiental, com cache e espera limitada |
| GHG Protocol — Registro Público de Emissões | Serviços públicos usados pelo portal, sem contrato REST oficial estável | Cadastro de organizações em `registropublicodeemissoesapi.fgv.br` (busca paginada por `POST`) e relatório PDF em `sistema-registropublicodeemissoesapi.fgv.br` | Nenhuma | Fallback best-effort na fila exclusiva `ghg-inventory`, com prazo próprio |

Observações por fonte:

- **Receita Federal**: nenhum dos três provedores é institucional — todos são projetos comunitários que espelham o dump aberto da RFB. A cadeia de fallback (BrasilAPI → OpenCNPJ → Minha Receita) é obrigatória, não redundância opcional: a Minha Receita só é consultada quando nem a BrasilAPI nem o OpenCNPJ retornam dados. O retorno da BrasilAPI já cobre RF05 e RF06 numa única chamada (situação cadastral, porte, endereço, CNAE principal e secundários com descrição).
- **IBAMA**: não existe consulta por CNPJ ao vivo nessa fonte. O MVP ingere apenas os dois recursos principais — auto de infração e termo de embargo —, cada um já carregando o documento (CPF/CNPJ) do autuado/interessado na própria linha. Os recursos auxiliares (enquadramento legal, coordenadas, bioma, espécime, anexos) ficam fora do MVP: uma auditoria de dados feita por contribuição da sociedade civil ao 5º Plano de Ação de Governo Aberto documentou ausência de chave primária confiável para cruzar esses recursos entre si (contagens de registro divergentes entre tabelas relacionadas extraídas na mesma data, formatos de número de processo inconsistentes). Tentar reconstruir o quadro completo joinando tudo é risco de estouro de prazo, não de dificuldade de código. Em outubro de 2026, o dump dos autos tem cerca de 712 mil linhas em 48 CSVs (720 MB descompactados) e o dos embargos, cerca de 117 mil (210 MB), em CSV com separador `;`, UTF-8 e campos de texto com quebras de linha. A maioria é de pessoa física (CPF): do total de linhas, só cerca de 31% dos autos e 15% dos embargos são de empresa. O rótulo de tipo de pessoa dos autos é inconsistente (há linhas `PJ` com CPF no documento) e os embargos não o trazem; por isso o descarte de pessoa física decide pelo documento (só CNPJ válido entra), e não pelo rótulo ([INL02]). O número público do documento (número e série) não é único no dump, e os documentos emitidos desde outubro de 2019 têm número alfanumérico e nenhuma série.
- **CGU CEIS/CNEP**: aceito como sinal amplo de conduta administrativa, sem filtrar por órgão sancionador — decisão explícita (ver seção 4, categorização).
- **GHG Protocol**: cobertura estruturalmente baixa (~600 organizações reportando por ano, contra milhões de CNPJs ativos no Brasil). `UNKNOWN` é o resultado esperado na maioria das consultas — não é falha de coleta, é a realidade da adesão ao programa (RIN [IND02]).

## 2. Fila, agendamento e consulta sob demanda

Decisão: **BullMQ + Redis para tudo**, sem broker adicional. BullMQ já cobre nativamente os três elementos que a ingestão do IBAMA precisa — fila, repetição agendada (job repetível = cron) e retry com backoff —, o que torna um segundo broker (ex.: RabbitMQ) redundante com a stack já adotada (ver `overview.md`) e contrário à diretriz de viabilidade solo do projeto (ver `ways-of-working.md`).

A mesma decisão vale para a comunicação **interna**: a ingestão (cadastro por CNPJ, evidências ambientais e inventário GHG) **não é um conjunto de serviços à parte**, e sim de módulos do próprio `bios-core-api` — um módulo por fonte externa (`brasil-api`, `opencnpj`, `minha-receita`, `ibge-cnae`, e os que virão para IBAMA, CGU e GHG), mais um consumidor de fila por assunto (por exemplo, o `CompanyProfileProcessor`, que atende a fila `company-profile`). Quem pede e quem atende conversam **exclusivamente via filas BullMQ**, sem chamar um ao outro diretamente: o contrato do job (nome, payload e resultado) é o único acoplamento. Isso mantém o ganho que a separação em serviços traria — o consumidor pode ser movido para um processo próprio sem tocar em quem publica o job — sem o custo de operar vários deployáveis, o que seria contrário à diretriz de viabilidade solo do projeto. Quando a consulta do usuário exige resposta imediata, o core enfileira o job e aguarda sua conclusão com timeout (sequência em `08-Artefatos-de-Análise.md`, seção 8.4.1); estourado o timeout de uma fonte não bloqueante (GHG), a evidência correspondente é exibida como `UNKNOWN`.

O *retry* com *backoff* é configurado **por fila** e, quando necessário, **por agendamento**: a fila `ibama-ingestion` é compartilhada pelo lote diário do IBAMA (minutos por tentativa, fonte que costuma ficar horas fora do ar) e pelos jobs por CNPJ (que erram rápido, pois há quem espere a resposta). O lote diário declara a própria política — 5 tentativas com *backoff* exponencial de base 5 minutos — sem alterar a dos demais jobs da fila. A execução de cada job agendado é registrada em `ingestion_runs` e só é marcada como falha quando as tentativas se esgotam; as evidências anteriores seguem valendo até o fim da janela de validade (UC14, fluxo 2.1).

O isolamento de cada fonte, que o [NFPD02] exige, vem do **adapter próprio** (`DataSourceInterface`), do **limite de requisições e do tempo máximo por fonte** (catálogo `data_sources`, alterável sem deploy) e do **resultado de falha esperada** (a fonte indisponível volta como resultado, não como exceção que derrube a consulta). No caso do GHG Protocol, a indisponibilidade da fonte nunca bloqueia a consulta porque a evidência é best-effort, com prazo próprio, e não porque roda em outro processo.

O consumidor GHG tem fila própria (`ghg-inventory`) e concorrência 1, sem competir
com o consumidor ambiental. O produtor reutiliza a coleta local válida e, quando
necessário, publica um job por CNPJ/janela de 15 minutos, aguardando com o prazo do
catálogo. Timeout, falha do job ou indisponibilidade da fila devolvem `UNKNOWN`
quando não há resposta anterior; respostas vencidas permanecem `OUTDATED`. O job
pode concluir depois do timeout de quem espera. Dentro do adapter, um orçamento
único cobre as chamadas HTTP e o parser: cada etapa usa somente o prazo restante.
O reaproveitamento do cache não renova a data nem a validade da coleta original.

O catálogo público GHG não informa CNPJ: razão social/nome fantasia do cadastro
local servem apenas para selecionar candidatos. O cadastro de organizações do portal
é sincronizado uma vez por dia (job agendado na fila `ghg-inventory`, guardado no cache
por sete dias, nunca substituído por um cadastro incompleto) e lido localmente pela
coleta de cada CNPJ; somente a falta do cadastro o sincroniza na hora. O relatório do ano mais recente
precisa conter o CNPJ exato e o mesmo ano listado no catálogo. Não se atribui
inventário de matriz, grupo ou outra empresa por semelhança do nome. Sem vínculo
verificável, o resultado é `UNKNOWN`, nunca `NOT_FOUND`. A data de publicação fica
nula quando não informada: ano inventariado e data de geração do PDF não a
substituem. Guarda-se somente o retrato reduzido aos campos permitidos, com
referência organização/ciclo e URL oficial do relatório; o PDF temporário é
removido após a leitura. A consolidação já reúne essa evidência; a exposição no perfil é uma etapa posterior.

A consolidação por CNPJ inicia em paralelo a consulta local IBAMA e os produtores
CGU/GHG. Esse despachante mora num módulo de nível mais alto, `environmental-intelligence`, que
depende das fontes (`ibama`, `cgu`, `ghg`) e do armazenamento (`evidences`), sem dependência
inversa: o armazenamento não conhece as fontes que o alimentam. O consumidor de
`environmental-evidences`, `EnvironmentalEvidencesProcessor`, atende só `fetch-cgu-evidences`, com
concorrência 3.
Lote diário e coleta avulsa do IBAMA vão para a fila própria `ibama-ingestion`, com
um consumidor de concorrência 1 (um download de dumps por vez) e reaproveitamento
dos dumps por 10 minutos, para que avulsos de CNPJs distintos não façam um download
cada nem disputem workers com a CGU (#262). CEIS e CNEP são
consultados em paralelo dentro do job CGU e persistidos independentemente; esse
job não altera execuções de ingestão IBAMA. A espera CGU respeita
`QUEUE_WAIT_TIMEOUT_MS`; o job pode concluir depois. As duas fontes CGU válidas
na persistência dispensam publicação; a validade é verificada por cadastro e o
reaproveitamento não renova as datas. A publicação CGU é deduplicada por CNPJ e
janela de 15 minutos. O produtor prepara desconhecimento antes de publicar e o
worker não confunde esse marcador com resposta já coletada.

Após as esperas limitadas, a consolidação relê a persistência e devolve as três
categorias, com fonte, ODS, estado, publicação, coleta, validade e referência/link
original. Falha de uma fonte preserva as outras e solicita `UNKNOWN` quando não
há resposta anterior; respostas anteriores conservam suas datas e vencem como
`OUTDATED`. Somente `CONFIRMED` traz sinal na resposta agrupada; os demais
estados têm sinal nulo, sem apagar o sinal histórico da persistência. Não há
veredito de sustentabilidade ou cálculo de índice nesse contrato. Falha global
de leitura/gravação do banco continua sendo erro de infraestrutura: não prova
ausência de evidências.

Caso de CNPJ não indexado localmente (miss no índice do lote diário do IBAMA): reaproveita o mesmo adapter/parser do job diário, disparado sob demanda como job avulso do BullMQ para aquele CNPJ específico, contra o mesmo dump oficial. Isso **não** reintroduz webscraping nem um segundo mecanismo de ingestão — é o mesmo pipeline, com gatilho diferente.

A consulta autenticada `GET /companies/{cnpj}/evidences` expõe as categorias e a
proveniência das evidências (RF09), sem calcular score. O perfil completo (cadastro, evidências e,
por ODS, índice, confiança e explicação) é composto pelo módulo de orquestração ambiental e servido
por `GET /companies/{cnpj}` e `GET /v1/companies/{cnpj}` (UC06 + UC07, #81): cadastro primeiro, depois
evidências e recálculo, depois a leitura dos índices gravados; as partes ambientais ficam em cache por
cinco minutos, apagadas pelo recálculo do índice.
O campo `source` traz o nome público atual do catálogo, acompanhado de
`source_slug`, `source_reference` e `source_url`. O schema de resposta exige URL
absoluta HTTPS quando informada; links com credenciais embutidas são rejeitados.
O endereço original é preservado, inclusive o dataset oficial do IBAMA quando
não há link individual. Uma referência sem URL continua utilizável para auditoria;
o sistema não fabrica um endereço. Recursos da CGU podem exigir autenticação no
próprio órgão, sem que a chave do BIOS seja incluída na resposta.

Na projeção, uma evidência guardada como `CONFIRMED` sem identificador não vazio
nem link HTTPS válido é exibida como `UNKNOWN`, sem sinal, resumo ou detalhes
confirmatórios. A regra também é validada pelo schema de resposta. O registro
persistido e suas datas permanecem intactos para futura atualização pela fonte.
`UNKNOWN`, `NOT_FOUND` e `OUTDATED` conservam seus estados e podem ter referências
nulas; isso não autoriza inventar registros ou converter falta de informação em
ausência confirmada. A rota exige sessão JWT e documenta resposta e erros no
OpenAPI. O contrato publicado em OpenAPI (RF17) é feito para gerar cliente: cada modelo de requisição e de resposta é um componente nomeado (os de erro, um só, `ErrorResponse`), cada operação tem `operationId` explícito e estável, e o documento traz a URL base (`API_BASE_URL`) e a versão do pacote. O CI o publica como artefato para o front gerar o cliente sem subir a API.

## 3. Camadas de dados

Mantido conforme já esboçado no README (seção 4), sem alteração: três coleções Mongo — `evidence_raw` (bruto por fonte) → `evidence_normalized` → `company_profile` (agregado, pronto para busca/score/exportação) —, sem infraestrutura de streaming dedicada. Cada fonte isolada atrás de um adapter próprio (`DataSourceInterface` → `ReceitaDataSource`, `IbamaDataSource`, `CguDataSource`, `GhgDataSource`), conforme já previsto em [NFPD02].

### 3.1 ODS como entidade de referência

Os ODS aos quais os índices de aderência (RF10) se referem são modelados como entidade própria — coleção `ods`, com número, título e descrição —, populada via seed na inicialização do banco, no mesmo padrão adotado para a blacklist de domínios (RF22). No MVP, a seed contém dois registros:

| ODS | Título | Fonte(s) de evidência | Sinal disponível |
| --- | --- | --- | --- |
| 13 | Ação Contra a Mudança Global do Clima | GHG Protocol — Registro Público de Emissões | Apenas positivo (publicação de inventário de emissões) |
| 15 | Vida Terrestre | IBAMA (autos de infração e termos de embargo); CGU — CEIS/CNEP como sinal complementar de peso reduzido (ver seção 4) | Apenas negativo (autuação/embargo; sanção administrativa) |

Cada evidência normalizada referencia o ODS ao qual se vincula, e os resultados por ODS (`{ odsId, indice, confianca }`) ficam na coleção `company_ods_scores`, um documento por empresa e ODS, em vez de um par único índice/confiança no `company_profile` — o que permite filtrar e ordenar pelo índice de um ODS selecionado com um único índice composto (modelo completo em `08-Artefatos-de-Análise.md`, seção 8.5). Nenhum ODS é assumido de forma implícita no código: incluir um novo ODS passa a ser uma nova entrada de seed somada às regras de pontuação correspondentes, sem alteração de esquema — o que preserva a escalabilidade a outros ODS prevista no objetivo geral do projeto.

Limitações assumidas e registradas:

- **ODS 13 com cobertura estruturalmente baixa.** A única fonte é o GHG Protocol (~600 organizações reportando por ano — ver seção 1), sem API e tratada, em tempo de execução, como fallback best-effort isolado (sua ingestão é prioridade Essencial de entrega — ver `10-Gerenciamento-do-Projeto.md` —, mas sua indisponibilidade nunca bloqueia a consulta). Para a grande maioria das empresas, o índice do ODS 13 terá estado `UNKNOWN` e confiança baixa. Isso não é falha de coleta e não pode ser tratado como evidência negativa (RIN [IND02]); é exatamente o cenário que a separação índice/confiança (RF11) existe para comunicar.
- **Sinais unilaterais.** Nas fontes do MVP, o ODS 13 dispõe apenas de sinal positivo e o ODS 15 apenas de sinal negativo. Os dois índices, portanto, não são comparáveis entre si nem combinados em um índice geral único; filtros e ordenação (RF13, RF14) operam sempre sobre um ODS selecionado. A pesquisa filtra `company_profile` por nome, UF, município e CNAE e, com um ODS selecionado, junta o documento daquele ODS em `company_ods_scores` pelo CNPJ para devolver o índice e a confiança e aplicar os mínimos; a lista traz só o resumo, e as contribuições continuam no perfil. A ordenação pelo índice (RF14) lê primeiro as empresas com nota direto do índice composto de `company_ods_scores` (campos `uf`, `city` e `cnae_codes` desnormalizados), paginando antes de juntar o perfil, e só depois as sem nota, por razão social: empresa sem índice vai ao fim nos dois sentidos, e a maior confiança desempata índices iguais. O índice de uma empresa sem perfil (calculado pela ingestão do IBAMA) não entra na pesquisa, o que o campo `uf` não nulo garante. Com nome, a pesquisa segue pelo perfil. A exportação em XLSX (RF15) reaproveita a mesma pesquisa, sem paginação, e junta os índices de todos os ODS ativos em colunas separadas; é gerada em memória, no limite de 5000 empresas, a partir da base local e sem chamar fontes.

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

### 4.1 Regras fixas do índice por ODS (RF10)

O motor interno carrega os ODS ativos da coleção `ods`. Cada ODS ativo precisa de
regras explícitas no código; incluir uma seed sem regras interrompe o cálculo
antes de gravar. A escala é **assinada de −100 a 100**: pontos positivos indicam
evidências de transparência climática no ODS 13, e negativos indicam autuações
ambientais ou sanções complementares no ODS 15. Não existe índice geral, nem
comparação direta entre ODS 13 e 15. O zero não é usado como substituto de falta
de dados: sem contribuição conclusiva, `score_value` é nulo.

| ODS | Categoria | Sinal | Peso por evidência | Teto absoluto da categoria |
| --- | --- | --- | ---: | ---: |
| 13 | `EMISSAO_GEE` | positivo | 60 | 100 |
| 15 | `INFRACAO_AMBIENTAL` | negativo | 40 | 80 |
| 15 | `CONDUTA_ADMINISTRATIVA` | negativo | 5 | 20 |

Uma evidência só pontua quando está `CONFIRMED`, dentro da validade e é
rastreável por identificador não vazio ou URL HTTPS original válida. A categoria
e o sinal devem corresponder à regra do ODS. `NOT_FOUND` não dá bônus;
`UNKNOWN`, `OUTDATED` e confirmação sem rastreabilidade não pontuam. Em ordem
determinística por categoria e chave, cada evidência elegível grava uma
contribuição com ID, categoria, sinal, peso fixo e pontos efetivos. Ao atingir o
teto da categoria, contribuições posteriores continuam auditáveis com zero ponto.
Assim, a soma exata dos pontos reproduz o índice, limitado a −100…100, e o total
de sanções administrativas nunca supera 20 pontos negativos.

Sanção CEIS/CNEP cujo `details.endDate` precede o dia civil da apuração em
`America/Sao_Paulo` recebe 25% do peso (−1,25). Termo de embargo IBAMA com
`details.resource = TERMO_EMBARGO` e `details.lifted = true` também recebe
25% (−10). A sanção com fim no dia da apuração, o embargo ativo, o auto de
infração e a sanção sem data de fim mantêm o peso integral. Esses registros
continuam `CONFIRMED` se a coleta estiver vigente: fim da sanção e levantamento
do embargo não são vencimento da coleta. O teto de cada categoria é aplicado
depois do redutor, e cada contribuição guarda o peso efetivo, os pontos e
`weight_reason` (`CGU_SANCTION_EXPIRED`, `IBAMA_EMBARGO_LIFTED` ou nulo).
A confiança RF11 continua independente da magnitude dos pontos.

`company_ods_scores` contém um documento por empresa e ODS ativo, atualizado
atomicamente por CNPJ. O recálculo mantém o ID/data de criação e remove scores
de ODS desativados; `rules_version` identifica a regra usada. UF, cidade e CNAEs
vêm do perfil, quando ele existe, ou ficam nulos/vazios. A confiança fica em campo
separado, conforme RF11 abaixo. A versão conjunta das regras do cálculo é
`rf10-rf11-v3`; os pesos integrais da versão `rf10-rf11-v2` permanecem iguais. O motor
é um caso de uso interno, executado pelo job `recalculate-scores(cnpj)` da fila `ods-scoring`
(UC15, #80), que o fim da ingestão do IBAMA (um job por CNPJ afetado, em lotes), a consulta sob
demanda (aguardável com prazo, diagrama 3) e a atualização do perfil cadastral (reescrita dos campos
desnormalizados) acionam. Pedidos simultâneos do mesmo CNPJ viram um job só enquanto ele não
começou, e um recálculo mais antigo nunca sobrescreve um mais novo. Alterar a versão das regras
dispara um recálculo em lote, por varredura agendada; ao final de cada recálculo, o cache derivado
do CNPJ (perfil, evidências e notas) é invalidado, e os retratos das fontes, não. O perfil completo
(#81) lê esses retratos e não recalcula; ODS ainda sem retrato aparece como pendente, sem número.

### 4.2 Nível de confiança por ODS (RF11, versão `rf10-rf11-v2`)

Para cada ODS, a confiança é calculada independentemente do valor e da magnitude
do índice, entre 0 e 1. Sem achado `CONFIRMED` vigente, rastreável e com sinal
compatível, vale **0**, inclusive quando todas as fontes responderam
`NOT_FOUND`. Com ao menos um achado, a fórmula é:

`confiança = teto × (S/M) × (N/(N+1)) × ((1+Q)/(2+Q)) × (S/(S+U+O))`

| Símbolo | Definição |
| --- | --- |
| `M` | Fontes esperadas: 1 para ODS 13 (`GHG_PROTOCOL`); 3 para ODS 15 (`IBAMA`, `CGU_CEIS`, `CGU_CNEP`). |
| `S` | Fontes distintas com resposta vigente: `CONFIRMED` elegível ao índice ou `NOT_FOUND`. A ausência confirmada cobre a fonte, mas não cria contribuição nem índice. |
| `N` | Número de contribuições confirmadas, incluindo as de zero ponto após o teto da categoria; não depende da magnitude do índice. |
| `Q` | Soma de qualidade das contribuições: IBAMA 1, CGU CEIS/CNEP 0,25 por achado, GHG 1. A sanção genérica tem qualidade menor que a infração ambiental direta. |
| `U` e `O` | Número de fontes distintas com `UNKNOWN` e `OUTDATED`, respectivamente. Uma confirmação/ausência com validade vencida conta em `O`; `UNKNOWN` não envelhece. |
| `teto` | 0,25 para ODS 13, pela cobertura estruturalmente pequena do GHG Protocol; 1 para ODS 15. |

Os fatores medem, respectivamente, cobertura das fontes, quantidade de achados,
qualidade dos sinais e penalidade por fonte inconclusiva ou desatualizada. Fontes
sem evidência não ampliam `S`; uma fonte pode ter resposta válida e registro
desatualizado, caso em que ambos os estados entram nos fatores. Registros de fonte
ou categoria incompatível com o ODS são ignorados na confiança. Adicionar um
achado elegível não reduz a confiança; adicionar `UNKNOWN` ou `OUTDATED` não a
aumenta. Esses estados nunca alteram o índice. Índice e confiança são persistidos
lado a lado em `company_ods_scores`, sem fusão em um número único.

**Cobertura das fontes (decisão #253).** A ausência confirmada em todas as fontes
continua sem índice e com confiança 0: `NOT_FOUND` não é evidência de boa conduta
(IND02) e não ganha pontos nem confiança própria. Para o usuário distinguir "consultado
e sem registros" de "sem dados", o retrato guarda à parte um `coverage_status` e a lista
`sources_without_records`, calculados no mesmo recálculo e sem efeito sobre índice ou
confiança (a fórmula e o RF11 não mudam). Uma fonte conta como sem registros quando sua
ausência está dentro da validade e ela não tem achado confirmado vigente no ODS.

| `coverage_status` | Significado |
| --- | --- |
| `HAS_RECORDS` | Há ao menos uma contribuição ao índice. |
| `NO_RECORDS_IN_ALL_SOURCES` | Sem contribuição e todas as fontes esperadas do ODS responderam `NOT_FOUND` vigente. |
| `NO_RECORDS_IN_SOME_SOURCES` | Sem contribuição; só parte das fontes tem ausência vigente, as demais estão `UNKNOWN`, `OUTDATED` ou sem coleta. |
| `NO_DATA` | Sem contribuição e sem ausência vigente em nenhuma fonte. |

Retratos gravados antes desta regra têm `coverage_status` nulo até o próximo recálculo (#80).
A mensagem ao usuário deve dizer "sem registros nas fontes consultadas", nunca "empresa
sem infrações" ou veredito de conduta (IND01).

### 4.3 Explicação auditável do índice (RF12)

A consulta autenticada `GET /companies/{cnpj}/scores/{odsNumber}/explanation`
lê o retrato já calculado, sem acionar coleta ou recálculo. Para cada item de
`contributions`, expõe a evidência referenciada, sua categoria e fonte pública,
referência ou link HTTPS original, sinal, peso e pontos efetivos. A soma de
`points` é o `score_value`; o `confidence_value` permanece separado. O campo
`weight_reason` explicita o redutor histórico quando aplicado, sem apagar a
evidência. A categoria `CONDUTA_ADMINISTRATIVA` não é rotulada como infração
ambiental.

`state = CONFIRMED` descreve o estado da evidência quando ela compôs o índice;
`current_state` pode ser `OUTDATED` após a validade da coleta. O retorno informa
também a data e a versão do cálculo. Um índice gravado sem contribuição responde
com `index_status = UNAVAILABLE`, valor nulo e lista vazia, acompanhado de
`coverage_status` e `sources_without_records` (seção 4.2). Sem score gravado
para empresa/ODS, a rota responde 404. Se a contribuição aponta a uma evidência
que foi removida, mudou de categoria/sinal ou perdeu a proveniência, responde
409 em vez de omitir a linha; o recálculo é tarefa da etapa UC15/#80.
Nenhum desses números constitui veredito de sustentabilidade (IND01).

## 5. Estados de evidência — decisão

O README (seção 4) e o RF08 formal (`05-Requisitos-Funcionais.md`) chegaram a listar números diferentes de estados (seis vs. quatro). Decisão: os quatro estados do RF08 (`CONFIRMED`, `NOT_FOUND`, `UNKNOWN`, `OUTDATED`) são o modelo definitivo do MVP. `NOT_APPLICABLE` e `CONFLICTING` ficam descartados do escopo atual e registrados aqui como trabalho futuro, não perdidos. O README foi atualizado (seção 4) para refletir essa decisão.

Os diagramas C4 de contexto (nível 1) e de contêineres (nível 2) que materializam as decisões
desta seção estão em `08-Artefatos-de-Análise.md`, seção 8.6; o modelo de dados das coleções,
na seção 8.5.

## 6. Justificativa da arquitetura conforme ISO/IEC 25010:2023

A arquitetura descrita nas seções 1 a 4 é justificada abaixo pelas nove características de
qualidade da ISO/IEC 25010:2023 (referenciada em `12-Referências.md`), amarrada aos RNFs já
formalizados e reclassificados em `06-Requisitos-Nao-Funcionais.md`.

| Característica ISO/IEC 25010:2023 | Como a arquitetura do BIOS a endereça | RNF/decisão relacionada |
| --- | --- | --- |
| Adequação Funcional | Modelo evidência-primeiro (dado → evidência → regra → indicador → score) com estado explícito por evidência (RF08) garante que cada fonte contribua de forma isolada e auditável para o score, sem sobreposição de responsabilidade entre adapters | RF07, RF08, RF10; NFCO02 |
| Eficiência de Desempenho | Cache de consulta evita nova chamada síncrona à fonte externa dentro da janela de validade; ingestão pesada do IBAMA roda como job assíncrono em lote (BullMQ), fora do caminho crítico da consulta em tempo real | NFDE01, NFDE02 |
| Compatibilidade | Cada fonte isolada atrás de `DataSourceInterface`; a cadeia de fallback do cadastro (BrasilAPI → OpenCNPJ → Minha Receita) troca de provedor sem alterar contrato interno; rate limit de cada fonte externa respeitado | NFPD01; seção 1 |
| Capacidade de Interação | Não endereçada nesta camada de arquitetura (back-end/dados) — depende de decisões de front-end ainda não formalizadas | — (ver `06-Requisitos-Nao-Funcionais.md`, seção 6.4) |
| Confiabilidade | Degradação graciosa por fonte — indisponibilidade de uma fonte (ex.: GHG Protocol) não bloqueia a consulta inteira; job de ingestão com retry/backoff nativo do BullMQ | NFCO01 |
| Segurança | Criptografia AES-256, hash de chave de API, sessão JWT revogável, rate limiting, bloqueio de login, honeypot, monitoramento de dependências | NFSE01–NFSE08 |
| Manutenibilidade | Isolamento por adapter (um módulo por fonte), camadas lógicas `evidence_raw` → `evidence_normalized` → `company_profile`, cobertura mínima de teste, barreira pré-commit e análise estática | NFPD02, NFPD03, NFSE09–NFSE11; seção 3 |
| Flexibilidade | Mesmo isolamento por adapter também sustenta adição de novas fontes sem alterar o núcleo, ainda que formalizado sob Manutenibilidade | NFPD02 (referência cruzada) |
| Segurança Operacional (Safety) | Não aplicável — o BIOS não atua sobre processo físico ou operacional de terceiros | — |

Capacidade de Interação depende de decisões de front-end ainda não formalizadas; a tabela
cobre apenas a parcela resolvida pela arquitetura de back-end/dados deste documento.
