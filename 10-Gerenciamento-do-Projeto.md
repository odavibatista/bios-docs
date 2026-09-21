# 10. Planejamento do Projeto

## Escopo

O MVP do BIOS consolidará, a partir do CNPJ, dados cadastrais de empresas (Receita Federal e
CNAE/IBGE) e evidências públicas de comprometimento ambiental (autuações e embargos do IBAMA;
sanções administrativas do CEIS/CNEP — Portal da Transparência), gerando um índice de
aderência ambiental explicável e um nível de confiança associado. O sistema permitirá busca
por CNPJ ou razão social, filtragem por CNAE/UF/porte/score, ranqueamento de resultados e
exportação de listas em XLSX, além de um painel de conta com geração de chaves de API para
consumo externo dos dados. Ficam fora do escopo do MVP, por decisão deliberada: identificação
de tomadores de decisão, coleta de contatos pessoais (e-mail, telefone, WhatsApp, LinkedIn) e
qualquer funcionalidade de geração de leads comerciais — o sistema trabalha exclusivamente com
dados públicos de pessoa jurídica de natureza ambiental/regulatória. Também ficam fora do MVP,
podendo compor trabalho futuro: integração com fontes de licenciamento estadual,
geoprocessamento (SICAR/MapBiomas), motor de regras configurável e exportação em PDF.

## Processo de Software

O desenvolvimento seguirá um modelo incremental de desenvolvimento de software, adequado a
uma equipe de um único integrante: cada incremento corresponde a um conjunto coeso de
funcionalidades (por exemplo, consulta por CNPJ, motor de evidências, motor de score,
exportação), entregue, testado e integrado antes do início do incremento seguinte. Não será
adotado um framework ágil formal com ritos de squad (daily, planning, retro), já que estes
perdem sentido para um integrante único; os princípios de entrega incremental, priorização
contínua e adaptação a mudanças de requisito são mantidos.

## Equipe

O projeto é de desenvolvimento individual. Davi Deosmar Batista Oliveira Miranda (RA
823.212.382) acumula a responsabilidade única pelo projeto, com atuação nas seguintes frentes:

- Gestão de Sprint: planejamento e definição de tarefas, acompanhamento de progresso, gestão
  de prazos e entregas, revisão e retrospectiva;
- Engenharia de Requisitos e Documentação Sistêmica: levantamento e documentação de
  requisitos, documentação técnica, manuais e guias de uso, manutenção e atualização contínua
  da documentação;
- Desenvolvimento Back-end: arquitetura e desenvolvimento de APIs, implementação de regras de
  negócio (motor de evidências e de score), integrações com fontes de dados externas e banco
  de dados, testes unitários e de integração;
- Desenvolvimento Front-end: desenvolvimento de interfaces e componentes, integração com a
  API, responsividade e performance, testes de componente e end-to-end;
- QA & Testes: planejamento de testes, criação e execução de testes automatizados e manuais,
  garantia de qualidade do produto, acompanhamento de bugs;
- DevOps (Pipelines CI): configuração e manutenção de pipeline de lint e testes (GitHub
  Actions), gerenciamento de branches, monitoramento de vulnerabilidades de dependências
  (Dependabot).

## Responsabilidades

Por se tratar de desenvolvimento individual, todas as responsabilidades acima são exercidas
pela mesma pessoa, sem prejuízo do rastreamento formal de cada atividade por etapa e por tipo
de teste no quadro de gerenciamento do projeto do [GitHub Projects](https://github.com/users/odavibatista/projects/15).

## Atividades

- Formação do tema e identificação do problema ambiental (concluído);
- Gerenciamento do projeto: escopo, cronograma, riscos e ferramenta de gestão (concluído);
- Engenharia de requisitos: levantamento e documentação de requisitos funcionais, não
  funcionais e requisitos inversos (concluído);
- Modelagem: diagramas de classes, casos de uso e sequência;
- Projeto e arquitetura: definição e justificativa da arquitetura — NestJS, Prisma, MongoDB,
  padrão de evidências rastreáveis, adapters por fonte de dados (concluído);
- Desenvolvimento do MVP: implementação incremental das funcionalidades essenciais;
- Gerência de configuração: versionamento, integração contínua, qualidade de commit
  (concluído);
- Avaliação de impacto ambiental: definição de indicadores e métricas de resultado;
- Apresentação final do projeto.

## Cronograma

- Agosto: Formação do tema, identificação do problema (concluído);
- Setembro: Gerenciamento do projeto (escopo, riscos, ferramenta) e Engenharia de Requisitos
  (concluído);
- Outubro: Modelagem e Projeto/Arquitetura; início do desenvolvimento do MVP;
- Novembro: Continuidade e finalização do desenvolvimento do MVP; Gerência de Configuração;
  Avaliação de Impacto Ambiental;
- Dezembro: Apresentação final (Expo/Laboratório) e entrega do produto.

## Prioridades

- Essencial: consulta de empresa por CNPJ, consolidação de evidências ambientais (IBAMA,
  CEIS/CNEP), cálculo do índice de aderência ambiental com nível de confiança explicável,
  autenticação de usuário;
- Importante: filtros de busca, ranqueamento, exportação em XLSX, painel de conta e API
  pública documentada;
- Desejável: evidências positivas via GHG Protocol (Registro Público de Emissões), mecanismos
  adicionais de segurança de borda (ex.: honeypot).

## Riscos

- Dependência de disponibilidade e estabilidade das APIs públicas externas consumidas
  (Receita Federal/BrasilAPI, IBGE, IBAMA, CGU), fora do controle do projeto — mitigação via
  cache local e adapters isolados por fonte;
- Ausência de API REST oficial para o Registro Público de Emissões do GHG Protocol, exigindo
  coleta manual/estruturada como exceção documentada à estratégia API First — mitigação via
  tratamento dessa fonte como atualização de baixa frequência;
- Restrição de tempo e de mão de obra por se tratar de desenvolvimento individual — mitigação
  via modelo incremental, escopo de MVP deliberadamente contido e corte explícito de
  funcionalidades não essenciais (ver Escopo);
- Risco de ambiguidade em score sem evidência suficiente, mitigado pela separação estrutural
  entre índice de aderência e nível de confiança, evitando que ausência de dado seja
  interpretada como evidência negativa.

## Ferramenta de Gerenciamento

Será utilizado o GitHub Projects, com quadro Kanban de fluxo (Backlog → Doing → Review →
Testing → Done). Cada tarefa recebe etiquetas indicando o tipo de teste aplicável (unitário,
integração, componente, end-to-end e funcional, nas camadas de back-end e front-end),
permitindo rastrear a cobertura de teste por funcionalidade diretamente no quadro.
