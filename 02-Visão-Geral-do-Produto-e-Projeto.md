# 02. Visão Geral do Produto e Projeto

O objetivo central deste projeto é oferecer um mecanismo eficiente e confiável para a consulta
e avaliação do comprometimento ambiental de empresas brasileiras, centralizando evidências
hoje dispersas entre diferentes fontes públicas de dados, mesclando-as e juntando-as em uma equação para cálculo dos valores indicatórios. 
Hoje, o sistema opera sobre dois Objetivos de Desenvolvimento Sustentável (ODS) de natureza ambiental — ODS 13
(Ação Contra a Mudança Global do Clima) e ODS 15 (Vida Terrestre), mas sua modelagem de dados permite a inserção dos demais ODS's e a adição de novas fontes de dados nas entidades do banco de dados e capacidade do sistema se integrar com outros provedores públicos.

Os usuários podem consultar empresas por CNPJ ou nome, visualizar
evidências categorizadas por fonte e por estado de confirmação, compreender a composição de
um índice de aderência ambiental por meio de sua explicação analisando cada evidência disponibilizada, mas sem que o BIOS dê um vereditos absoluto não sustentados por nenhuma fonte (cada dado possui sua origem rastreável e indicada no perfil e no score).

As metas do projeto incluem o desenvolvimento de uma interface simples e direta, que permita a
qualquer usuário consultar uma empresa sem precisar compreender a estrutura das bases
governamentais subjacentes, de modo que os dados sejam acessíveis de modo mais conveniente, intuitivo e confortável do que nas fontes de consulta, que oferecem os mesmos via longos PDFs, JSONs desestruturados e até mesmo arquivos ZIPs de mais de 700MB com arquivos completamente dispersos e difíceis de serem filtrados.

A razão social de empresário individual pode conter o nome do titular, e para proteger estes dados de exposição maior, sequências de
CPF são mascaradas, e contatos e perfis de indivíduos não são buscados ([INL02], [NFSE02]).
Manter os cadastros de empresários individuais na consulta e na pesquisa é uma decisão de produto
(UC06, regra 11), e o titular pode pedir a remoção pelo sistema de DPO do BIOS ([Avaliação de LGPD](16-Avaliação-LGPD.md)).

## 2.1 Problema

Não existe nenhuma plataforma que centralize dados ambientais de modo concatenado no mercado e até mesmo nas entidades públicas. A crescente preocupação de entidades públicas e privadas em aderir aos ODS's aumenta o investimento destas nessas pautas, mas o rastreio de todas as informações convenientes a esse trabalho ainda é muito penoso pela falta de uma centralização destas: no Brasil, querer consultar dados de uma empresa exigem múltiplas consultas demoradas em diferentes sistemas de órgãos como  Receita Federal, IBAMA, CGU, e outros tantos. E, ainda que a consulta seja feita, a junção dessas informações obtidas para se construir um perfil ambiental de uma empresa ou entidade exige muito esforço humano e nenhuma padronização existente no mercado para tal, ainda exigindo uma elaboração própria de um modo de cálculo ou avaliação.

O impacto do BIOS nos diversos setores envolvidos nas ODS's é que, com a solução em funcionamento, este traga maior facilidade e
transparência sobre o comprometimento ambiental e social de empresas e entidades, reduzindo a assimetria
de informação entre quem regula, quem fiscaliza, quem compra e quem presta contas à sociedade.
A consolidação de evidências rastreáveis contribui para que empresas sejam avaliadas com base
em dados verificáveis, não em reputação informal ou desinformação, tendo o princípio evidence-first para repugnar quaisquer incertezas ou falsidades na composição dos índices.

A solução permitirá que qualquer usuário consulte, de forma centralizada, evidências ambientais
públicas associadas a uma empresa, sem precisar visitar múltiplos portais ou serviços instáveis, dispersos e desorganizados. Estes poderão comparar empresas entre si por meio de um índice de aderência ambiental
explicável, filtrando por setor, localização e nível de confiança das evidências disponíveis.

Desenvolvedores externos terão acesso a essas informações de forma programática, por meio de
uma API pública documentada, podendo integrá-las a sistemas próprios.

## 2.2 Principais Stakeholders e Usuários

Essa seção será detalhada no documento de termo de atribuição de papéis e responsabilidades no
projeto/software.

| Papel | Responsabilidades | Stakeholders |
|---|---|---|
| Responsável pelo Projeto | Supervisionar o desenvolvimento do projeto, garantir o cumprimento de prazos e requisitos, tomar decisões de escopo e arquitetura. | Davi Deosmar Batista Oliveira Miranda. |
| Desenvolvedor | Implementar as funcionalidades do sistema (back-end, front-end, integrações e testes). | Davi Deosmar Batista Oliveira Miranda. |
| Usuários | Consultar empresas por CNPJ ou nome, interpretar evidências e índices de aderência ambiental, filtrar e exportar listas de empresas, inclusive para apoiar verificações de cadastro com CNPJ em KYC e KYE. | Profissionais de negócios e de sustentabilidade/ESG; equipes que verificam cadastros com CNPJ; órgãos e agentes governamentais; imprensa; pesquisadores e estudantes; empresas interessadas em adequação às pautas ambientais. |
| Desenvolvedores Externos | Consumir a API pública do sistema para integração com sistemas próprios, mediante chave de API. | Desenvolvedores e sistemas de terceiros. |

### Usos em decisões de contratação

Em todos os casos abaixo, o BIOS reúne evidências públicas para a análise humana. Ele não substitui as
consultas oficiais, não emite certidão e não decide quem contratar ([IND01]).

| Uso | Quem usa | Como o BIOS ajuda |
| --- | --- | --- |
| Contratação de prestadores PJ (KYE) | Empresas que contratam pessoas com CNPJ próprio | Mostra autuações, sanções e situação cadastral do CNPJ do prestador |
| Licitações e contratos públicos | Órgãos e entidades públicas, na habilitação e antes de contratar | Reúne CEIS, CNEP e autuações do IBAMA em uma consulta por CNPJ, como apoio à verificação nas fontes oficiais |
| Terceirizações | Órgãos públicos e empresas | Avalia a empresa que assumirá um serviço, no início do contrato e nas prorrogações |
| Privatizações, concessões e parcerias | Órgãos públicos, na diligência sobre participantes | Levanta o histórico ambiental público de quem disputa o ativo ou o serviço |
| Alinhamento à cultura ambiental | Empresas que querem parceiros compatíveis com sua agenda verde | Permite comparar fornecedores e prestadores por índice e confiança, com a evidência de cada resultado |

A Lei nº 14.133/2021 tem o desenvolvimento nacional sustentável entre seus princípios (art. 5º) e entre
os objetivos da licitação (art. 11, IV), e o estudo técnico preliminar descreve os possíveis impactos
ambientais da contratação (art. 18, § 1º, XII). A consulta ao CEIS e ao CNEP no momento de contratar
continua sendo feita nas fontes oficiais (art. 91, § 4º): o BIOS mostra a última coleta, com a data, e
não exclui ninguém de um certame. O índice não é critério de habilitação nem de julgamento: esses
critérios vêm da lei e do edital.

## 2.3 Documento de Visão

Essa seção resume as necessidades do dono do produto e as características esperadas do produto
de software a ser desenvolvido. O quadro 1 representa o modelo do documento de visão com suas
categorias subdivididas.

**Quadro 1 – Modelo do Documento de Visão.**

| | |
|---|---|
| **Nome do Produto:** BIOS — Business Intelligence On Sustainability | **Público-alvo:** Profissionais de negócios e de sustentabilidade/ESG, órgãos governamentais, imprensa, pesquisadores e estudantes interessados em avaliar o comprometimento ambiental de empresas brasileiras. |
| **Problema/Necessidade:** Dados ambientais e autuais de compromentimento em pautas sociais hoje são muito dispersos e muitas vezes sujeitos a erros ou indisponibilidade ou grande burocracia para serem obtidos. Além disso, são fornecidos de modo difratado: sem consolidação, sem índice comparável e sem explicação de como cada indicador ou problema foi composto. A demanda consiste em um sistema que centralize essas evidências por CNPJ, gerando um índice de aderência ambiental explicável e auditável, sem incorrer em coleta de dado pessoal ou geração de leads comerciais. O BIOS não possui fins lucrativos, não visa ser um sistema de consulta de dados pessoais para fins puramente econômicos e não visa ser um SaaS fechado a um único propósito: foi desenhado, sistemica e socialmente, visando se expandir e dar insumos de diversos fatores e demandas sociais. | **Características-chave:** Consulta de empresa por CNPJ ou nome; consolidação de evidências ambientais de múltiplas fontes públicas; cálculo de índice de aderência ambiental com nível de confiança explicável; filtros de busca por CNAE, UF, município, índice e confiança; exportação de listas em XLSX; API pública documentada para consumo programático. |
| **Condições Gerais de Satisfação (Critérios para aceitação):** A solução prevê autenticação obrigatória para consulta de informações. Toda evidência exibida deve ser rastreável até sua fonte pública original. O sistema não deve afirmar veredito absoluto sobre a sustentabilidade de uma empresa, apenas reportar evidências e índices derivados delas. Dados sensíveis de usuários devem ser armazenados com criptografia AES-256, e o sistema deve manter cobertura mínima de 75% de testes automatizados. | **Diferenciais:** Modelo de evidências com estado explícito (confirmado, não encontrado, desconhecido, desatualizado), evitando que ausência de dado seja interpretada como evidência negativa; separação estrutural entre índice de aderência e nível de confiança; ausência deliberada de qualquer funcionalidade de identificação ou contato de tomador de decisão, mantendo o escopo estritamente voltado à pessoa jurídica. |
| **Premissas, Riscos e Impedimentos:** Premissas: as fontes de dados públicas consumidas permanecerão disponíveis e gratuitas durante o desenvolvimento. Riscos: Restrição de tempo e de mão de obra por se tratar de desenvolvimento individual. | |
| **Expectativas de Prazo:** Novembro/2026 (MVP); Dezembro/2026 (apresentação final). | **Expectativas de Custo:** R$0,00 (uso exclusivo de fontes de dados públicas e gratuitas, sem contratação de serviços pagos). |
| **Elaborado por:** Davi Deosmar Batista Oliveira Miranda. | **Aprovado por (Dono do Produto):** Sidny Oliveira. |
