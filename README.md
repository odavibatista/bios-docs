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
