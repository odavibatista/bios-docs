# 15. Impacto Ambiental

## 15.1 Objetivo da Avaliação

O BIOS não atua diretamente sobre o comportamento ambiental das empresas consultadas — ele consolida e explica evidências públicas já existentes. A avaliação de impacto, portanto, mede o alcance e a qualidade da transparência gerada pela plataforma, não uma melhoria causal no comportamento ambiental das empresas monitoradas. Essa distinção é deliberada e segue o mesmo princípio epistemológico dos Requisitos Inversos (RIN [IND01], [IND02]): o sistema não pode reivindicar impacto que não é capaz de comprovar causalmente.

## 15.2 Indicadores de Alcance e Uso

| ID | Indicador | Forma de cálculo | Fonte |
| --- | --- | --- | --- |
| [IA01] | Empresas com perfil ambiental consultado | Contagem de CNPJs distintos consultados via RF03 em um período | Logs de consulta / `company_profile` |
| [IA02] | Volume de evidências consolidadas por categoria | Contagem de registros em `evidence_normalized`, segmentado por `categoria` (`INFRACAO_AMBIENTAL`, `CONDUTA_ADMINISTRATIVA`, `EMISSAO_GEE` — ver `14-Projeto-e-Arquitetura.md`, seção 4) | `evidence_normalized` |
| [IA03] | Concentração geográfica de evidências | Contagem de evidências por UF/município, reaproveitando os filtros de RF13 | `company_profile` + endereço (RF05) |
| [IA04] | Cobertura de evidência por fonte | Proporção de consultas com estado `CONFIRMED`/`NOT_FOUND`/`OUTDATED` vs. `UNKNOWN`, por fonte (IBAMA, CEIS/CNEP, GHG Protocol) | `evidence_normalized`, campo `estado` (RF08) |
| [IA05] | Evolução do índice de aderência médio por setor, por ODS | Média do índice de cada ODS (RF10 — ODS 13 e ODS 15) agregada por CNAE, comparada entre janelas de tempo sucessivas | `sector_score_snapshots` (ver 15.4) |

**Nota sobre [IA05]:** reportado **por setor (CNAE)**, nunca por empresa individual isolada — inclusive na
exposição ao usuário, formalizada como [RF30] (Desejável). Reportar evolução de score por empresa única, sem controle de causalidade sobre por que o score mudou (nova evidência ingerida vs. mudança real de conduta), geraria um ranking punitivo não sustentado por evidência suficiente — o mesmo risco que a separação score/confiança (RF11) já existe para mitigar.

**Exceção delimitada à nota ([RF33], [RF34], [IND03]):** a tela principal mostra, por empresa, a **última** mudança do índice de cada ODS. Isso não reporta evolução, porque o risco acima vem de dois pontos que a tela trata: (a) a falta de controle sobre a causa, já que cada mudança aparece com a evidência que a provocou, e uma mudança nas regras de pontuação nem é registrada; (b) o ranking, já que a lista é só cronológica. O sistema não guarda nem exibe o índice anterior, o tamanho da variação ou uma série por empresa, e não apresenta a mudança como melhora ou piora de conduta. O [IA05] continua sendo reportado apenas por setor.

**Segunda exceção delimitada à nota ([RF36], [IND03]):** o perfil de uma empresa mostra a evolução do índice em degraus. O risco da nota continua tratado pelos mesmos dois pontos: (a) a causa, porque cada degrau mostra a evidência que o provocou e a data; (b) o ranking, porque a evolução só aparece no perfil daquela empresa, sem número de variação nem comparação com outras. A série é reconstruída com as regras vigentes a partir das evidências atuais, e a tela informa isso. Ela é calculada na leitura e não é gravada, e o modelo de dados segue sem histórico por empresa. Cadastros de empresário individual ficam de fora, porque a curva ficaria ligada a uma pessoa. O [IA05] segue reportado por setor.

## 15.3 O que o BIOS não mede

Por honestidade epistemológica (mesmo princípio das RIN, `07-Requisitos-Inversos.md`), ficam explicitamente fora do escopo de avaliação de impacto:

- Dano real causado pelas empresas monitoradas — o sistema não calcula um total de estragos ou prejuízos ambientais que uma empresa possa ter causado a nível quantitativo ou monetário, apenas traz delitos ou processos que as mesmas possam ter cometido nestas esferas legais;
- Qualquer alegação de que uma movimentação do índice de uma empresa ([RF33]) significa mudança real de conduta, quando ela reflete apenas a chegada, a saída ou a alteração de uma evidência pública;
- Impacto fora da plataforma (ex.: decisões de compra ou parceria efetivamente tomadas com base no BIOS) — fora do alcance de instrumentação do sistema em sua v1.

## 15.4 Dependência de arquitetura

O indicador [IA05] exige manter série histórica de scores, não apenas o valor mais recente. O modelo de camadas de dados definido em `14-Projeto-e-Arquitetura.md` (seção 3) prevê apenas `updatedAt` + hash da versão anterior (snapshot leve), suficiente para saber *que* um score mudou, mas não para reconstruir a série completa. Duas opções ficam registradas para quando [IA05] for implementado:

1. Adicionar uma coleção de snapshot histórico (`company_profile_history`), populada a cada recálculo de score;
2. Aceitar uma série truncada (apenas snapshot anterior + atual), suficiente para "score subiu/desceu" mas não para gráfico de série temporal completo;
3. **(adotada no modelo de dados)** Persistir apenas os agregados setoriais por janela de tempo na coleção `sector_score_snapshots` (média do índice e da confiança por ODS e divisão CNAE), calculados por job periódico a partir de `company_ods_scores`. Atende o [IA05] sem guardar série histórica por empresa individual — coerente com a restrição da nota do [IA05] — e com custo de armazenamento independente do número de empresas.

A mesma restrição vale para a coleção `company_movements` ([RF33]): ela guarda só a última movimentação de cada ODS por empresa, sem índice anterior, e não forma série histórica.

**Implementação da opção 3** (bios-core-api, #89 e #90):

- **Janela mensal** (mês civil, em UTC). Um job diário regrava o agregado do mês corrente a partir dos
  índices atuais; virado o mês, o anterior fica como estava. Como `company_ods_scores` guarda só o índice
  atual de cada empresa, a série começa no primeiro mês em que o job roda e cresce daí em diante.
- **Setor = divisão CNAE do CNAE principal** do cadastro (2 dígitos). Empresas sem perfil cadastral não têm
  setor e ficam de fora; só entram os índices calculados pela versão de regras vigente, porque médias de
  versões diferentes não se comparam.
- **Amostra mínima** (padrão: 5 empresas, configurável). Abaixo dela, o índice médio e a confiança média do
  setor ficam nulos, e só as contagens aparecem: com poucas empresas, a média revelaria o índice de uma
  empresa individual, o que a nota do [IA05] proíbe.
- **Consulta:** a evolução de um ODS num setor lista as janelas da mais antiga à mais recente; com menos de
  duas janelas, informa que ainda não há histórico suficiente.

**Série da versão de apresentação.** A versão publicada do front-end não tem a API em execução, e o job acima
só registra o mês em que roda. Para mostrar o [IA05] nessa versão, os 23 meses anteriores ao mês registrado são
reconstruídos fora da API: as regras de pontuação do próprio código são reaplicadas às evidências atuais de cada
empresa, considerando só o que existia no fim de cada mês (data de publicação, início da sanção ou fim do ano do
inventário) e a situação da época (sanção vigente, embargo não baixado). A amostra mínima também vale aqui.

A reconstrução foi validada reproduzindo os 220.744 índices atuais das 110.372 empresas da base, sem nenhuma
diferença. Ela tem um limite assumido: registros que a fonte já retirou não entram, então a série mostra o passado
visto pelas evidências de hoje, e não o que teria sido medido na época. Por isso, a tela avisa que os meses
anteriores são reconstruídos, e a ressalva do viés de instrumentação (seção 15.3) vale com mais força para eles.
O ODS 13 fica estável por construção: cada inventário soma o mesmo peso, e o índice médio das empresas com
inventário não muda.
