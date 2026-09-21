# 02. Visão Geral do Produto e Projeto

O objetivo central deste projeto é oferecer um mecanismo eficiente e confiável para a consulta
e avaliação do comprometimento ambiental de empresas brasileiras, centralizando evidências
hoje dispersas entre diferentes órgãos públicos. Busca-se oferecer um ambiente que transforme
dados brutos, dispersos e de difícil interpretação em indicadores explicáveis e auditáveis de
aderência aos Objetivos de Desenvolvimento Sustentável (ODS) de natureza ambiental, apoiando a
tomada de decisão de profissionais de negócios, de sustentabilidade, pesquisadores, imprensa e
órgãos públicos.

Os usuários terão acesso ao sistema para consultar empresas por CNPJ ou nome, visualizar
evidências categorizadas por fonte e por estado de confirmação, e compreender a composição de
um índice de aderência ambiental por meio de sua explicação evidência a evidência. Essa
abordagem visa proporcionar aos usuários uma compreensão rastreável e auditável do perfil
socioambiental consultado, evitando vereditos absolutos não sustentados por evidência.

As metas do projeto incluem o desenvolvimento de uma interface simples e direta, que permita a
qualquer usuário consultar uma empresa sem precisar compreender a estrutura das bases
governamentais subjacentes. Além disso, o sistema deverá garantir a privacidade dos dados de
seus próprios usuários e tratar exclusivamente dados públicos de pessoa jurídica, sem qualquer
coleta de dado pessoal de indivíduo vinculado à empresa consultada.

## 2.1 Problema

O problema principal que este projeto busca solucionar é a dispersão de informações sobre o
comprometimento ambiental de empresas brasileiras entre diferentes órgãos públicos — Receita
Federal, IBAMA, CGU, entre outros — sem consolidação, sem índice comparável e sem explicação de
como cada indicador foi composto. Esse problema afeta tanto os profissionais que precisam
avaliar empresas sob critério socioambiental quanto a sociedade em geral, que perde
transparência sobre como as empresas atuam em relação às pautas ambientais.

Profissionais de negócios e de sustentabilidade são responsáveis por avaliar empresas segundo
critérios socioambientais para fins de parceria, fornecimento ou relacionamento comercial.

Órgãos governamentais, imprensa, pesquisadores e estudantes precisam consultar o histórico
ambiental de empresas para fins de fiscalização, apuração jornalística, pesquisa acadêmica ou
benchmarking de mercado.

O impacto esperado no setor é que, com a solução em funcionamento, espera-se maior
transparência sobre o comprometimento ambiental de empresas brasileiras, reduzindo a assimetria
de informação entre quem regula, quem fiscaliza, quem compra e quem presta contas à sociedade.
A consolidação de evidências rastreáveis contribui para que empresas sejam avaliadas com base
em dados verificáveis, não em reputação informal ou desinformação.

A solução permitirá que qualquer usuário consulte, de forma centralizada, evidências ambientais
públicas associadas a uma empresa, sem precisar visitar múltiplos portais governamentais.

Usuários poderão comparar empresas entre si por meio de um índice de aderência ambiental
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
| Usuários | Consultar empresas por CNPJ ou nome, interpretar evidências e índices de aderência ambiental, filtrar e exportar listas de empresas. | Profissionais de negócios e de sustentabilidade/ESG; órgãos e agentes governamentais; imprensa; pesquisadores e estudantes; empresas interessadas em adequação às pautas ambientais. |
| Desenvolvedores Externos | Consumir a API pública do sistema para integração com sistemas próprios, mediante chave de API. | Desenvolvedores e sistemas de terceiros. |

## 2.3 Documento de Visão

Essa seção resume as necessidades do dono do produto e as características esperadas do produto
de software a ser desenvolvido. O quadro 1 representa o modelo do documento de visão com suas
categorias subdivididas.

**Quadro 1 – Modelo do Documento de Visão.**

| | |
|---|---|
| **Nome do Produto:** BIOS — Inteligência Comercial de Agenda Verde: Prospecção Ativa e B2B Ambiental | **Público-alvo:** Profissionais de negócios e de sustentabilidade/ESG, órgãos governamentais, imprensa, pesquisadores e estudantes interessados em avaliar o comprometimento ambiental de empresas brasileiras. |
| **Problema/Necessidade:** O problema principal é a dispersão de informações sobre o comprometimento ambiental de empresas brasileiras entre diferentes órgãos públicos, sem consolidação, sem índice comparável e sem explicação de como cada indicador foi composto. A demanda consiste em um sistema que centralize essas evidências por CNPJ, gerando um índice de aderência ambiental explicável e auditável, sem incorrer em coleta de dado pessoal ou geração de leads comerciais. | **Características-chave:** Consulta de empresa por CNPJ ou nome; consolidação de evidências ambientais de múltiplas fontes públicas; cálculo de índice de aderência ambiental com nível de confiança explicável; filtros de busca por CNAE, UF, município, índice e confiança; exportação de listas em XLSX; API pública documentada para consumo programático. |
| **Condições Gerais de Satisfação (Critérios para aceitação):** A solução prevê autenticação obrigatória para consulta de informações. Toda evidência exibida deve ser rastreável até sua fonte pública original. O sistema não deve afirmar veredito absoluto sobre a sustentabilidade de uma empresa, apenas reportar evidências e índices derivados delas. Dados sensíveis de usuários devem ser armazenados com criptografia AES-256, e o sistema deve manter cobertura mínima de 75% de testes automatizados. | **Diferenciais:** Modelo de evidências com estado explícito (confirmado, não encontrado, desconhecido, conflitante, desatualizado), evitando que ausência de dado seja interpretada como evidência negativa; separação estrutural entre índice de aderência e nível de confiança; ausência deliberada de qualquer funcionalidade de identificação ou contato de tomador de decisão, mantendo o escopo estritamente voltado à pessoa jurídica. |
| **Premissas, Riscos e Impedimentos:** Premissas: as fontes de dados públicas consumidas permanecerão disponíveis e gratuitas durante o desenvolvimento. Riscos: indisponibilidade ou instabilidade de APIs públicas externas; ausência de API REST oficial para uma das fontes de evidência positiva; restrição de tempo e de mão de obra por se tratar de desenvolvimento individual. | |
| **Expectativas de Prazo:** Novembro/2026 (MVP); Dezembro/2026 (apresentação final). | **Expectativas de Custo:** R$0,00 (uso exclusivo de fontes de dados públicas e gratuitas, sem contratação de serviços pagos). |
| **Elaborado por:** Davi Deosmar Batista Oliveira Miranda. | **Aprovado por (Dono do Produto):** Sidny Oliveira. |
