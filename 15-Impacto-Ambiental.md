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

## 15.3 O que o BIOS não mede

Por honestidade epistemológica (mesmo princípio das RIN, `07-Requisitos-Inversos.md`), ficam explicitamente fora do escopo de avaliação de impacto:

- Redução real de dano ambiental causada pelas empresas monitoradas — o sistema não tem visibilidade sobre causalidade entre consulta/exposição e mudança de comportamento;
- Qualquer alegação de que aumento do índice médio de um setor significa melhoria ambiental real, quando pode refletir apenas melhoria na cobertura/qualidade da coleta de evidências (viés de instrumentação);
- Impacto fora da plataforma (ex.: decisões de compra ou parceria efetivamente tomadas com base no BIOS) — fora do alcance de instrumentação do sistema em sua v1.

## 15.4 Dependência de arquitetura

O indicador [IA05] exige manter série histórica de scores, não apenas o valor mais recente. O modelo de camadas de dados definido em `14-Projeto-e-Arquitetura.md` (seção 3) prevê apenas `updatedAt` + hash da versão anterior (snapshot leve), suficiente para saber *que* um score mudou, mas não para reconstruir a série completa. Duas opções ficam registradas para quando [IA05] for implementado:

1. Adicionar uma coleção de snapshot histórico (`company_profile_history`), populada a cada recálculo de score;
2. Aceitar uma série truncada (apenas snapshot anterior + atual), suficiente para "score subiu/desceu" mas não para gráfico de série temporal completo;
3. **(adotada no modelo de dados)** Persistir apenas os agregados setoriais por janela de tempo na coleção `sector_score_snapshots` (média do índice e da confiança por ODS e divisão CNAE), calculados por job periódico a partir de `company_ods_scores`. Atende o [IA05] sem guardar série histórica por empresa individual — coerente com a restrição da nota do [IA05] — e com custo de armazenamento independente do número de empresas.

Não bloqueia o MVP (RF10/RF11 não exigem histórico; [RF30], que expõe [IA05] ao usuário, é Desejável), mas precisa ser decidido antes de implementar [IA05] em nível de código.

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
