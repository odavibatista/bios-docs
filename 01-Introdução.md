# 01. Introdução

## 1.1 Objetivo do Projeto

O objetivo deste projeto é oferecer um sistema de consulta e avaliação do comprometimento
ambiental de empresas brasileiras e exteriores, a partir da consolidação de evidências públicas associadas
ao seu CNPJ ou registro nacional (em caso de empresas operantes no Brasil). O BIOS busca ser uma solução capaz de casar uma série de dados e informações hoje dispersas e espalhadas acerca de como as empresas se portam diante da legislação ambiental (mas não somente limitada a esta, tendo a capacidade de atuar em outras frentes que não somente a ecológica), centralizando e refinando estes dados desestruturados. Tais dados brutos são convertidos em indicadores explicáveis de
aderência a dois Objetivos de Desenvolvimento Sustentável (ODS) de natureza ambiental — ODS 13
(Ação Contra a Mudança Global do Clima) e ODS 15 (Vida Terrestre / Vida Sobre a Terra). O projeto
visa ser um apoiador na tomada de decisões para profissionais de negócios, sustentabilidade,
pesquisadores, imprensa e órgãos públicos, utilizando única e exclusivamente de dados públicos hoje disponibilizados por entidades governamentais e provedores de processos internacionais.

O BIOS não visa não somente dar critérios de avaliação às empresas para que estas possam ser melhor escolhidas e analisadas por parte de funcionários públicos em licitações ou até mesmo em processos jurídicos, mas a mesma visa ser um meio de auxiliar profissionais do setor privado em processos de KYC (*Know Your Customer*) e KYE (*Know Your Employee*) (para mais informações, ler a [seção 2.2](02-Visão-Geral-do-Produto-e-Projeto.md#22-principais-stakeholders-e-usuários)), de modo que fornecedores possam saber o modo que empresas fazem uso de seus insumos e operam no mercado, e contratadores/terceirizadores dos setores público e privado sejam capazes de conhecer o modo que seus prestadores de serviço atuam.

Estes dados não se limitam somente a contratantes ou a contratadores, mas também a consultorias e órgãos responsáveis por concederem certificações ambientais de índole pública e privada. No setor público, concedendo licenças para atuações em frentes ambientais (licenças para remoção de material radioativo, hospitalar, entre outros) e no setor privado, fornecimento de certificações ISO por parte de consultorias com tal competência.

O BIOS apresenta as evidências de cada empresa, sem busca por CPF e sem transformar o índice em
decisão automática ou veredito sobre uma pessoa ou empresa ([IND01], [INL02]). A seção
[Avaliação de LGPD](16-Avaliação-LGPD.md) traz as regras desse uso.

O sistema possibilitará que qualquer usuário autenticado consulte uma empresa por CNPJ ou por
nome, obtendo um perfil consolidado com evidências categorizadas por fonte, um índice de
aderência ambiental e um nível de confiança associado, sempre rastreável até sua origem.
Desenvolvedores externos poderão consumir essas informações programaticamente, mediante chave
de API própria, para integração com sistemas de terceiros.

## 1.2 Convenções, Termos e Abreviações

A correta interpretação deste documento exige o conhecimento de algumas convenções, termos e
abreviações específicas, descritas a seguir em ordem alfabética.

***API***: Application Programming Interface — Interface de Programação de Aplicativos.

***BullMQ***: biblioteca de filas baseada em Redis, utilizada para orquestrar jobs
assíncronos de ingestão de dados.

***CEIS/CNEP***: Cadastro de Empresas Inidôneas e Suspensas / Cadastro Nacional de Empresas
Punidas, bases mantidas pela Controladoria-Geral da União (CGU).

***CI/CD***: Continuous Integration / Continuous Delivery — práticas combinadas de
integração e entrega contínua, com automação de build, teste e implantação.

***CNAE***: Classificação Nacional de Atividades Econômicas.

***CNPJ***: Cadastro Nacional da Pessoa Jurídica.

***Framework***: abstração que reúne código comum entre projetos de software, provendo
funcionalidade genérica configurável.

***GHG Protocol***: Greenhouse Gas Protocol — padrão internacional de contabilização de
emissões de gases de efeito estufa, com Registro Público de Emissões no Brasil.

***Honeypot***: mecanismo de segurança que expõe dados ou endpoints falsos para identificar
tentativas de acesso não autorizado.

***IBAMA***: Instituto Brasileiro do Meio Ambiente e dos Recursos Naturais Renováveis.

***JWT***: JSON Web Token — formato de token utilizado para autenticação e autorização.

***LGPD***: Lei Geral de Proteção de Dados.

***MongoDB***: sistema de banco de dados orientado a documentos, utilizado como
persistência principal do BIOS.

***ODS***: Objetivos de Desenvolvimento Sustentável da Organização das Nações Unidas. O MVP do
BIOS trabalha exclusivamente com dois ODS: o ODS 13 — Ação Contra a Mudança Global do Clima
(medidas urgentes para combater a mudança do clima e seus impactos) e o ODS 15 — Vida Terrestre
(proteger, recuperar e promover o uso sustentável dos ecossistemas terrestres, deter o
desmatamento e a perda de biodiversidade).

***ORM***: Object-Relational Mapping — camada de abstração entre código e banco de dados;
no BIOS, implementada via Prisma ORM.

***Rate limiting***: técnica de limitação do número de requisições permitidas em um
intervalo de tempo, usada para mitigar abuso de endpoints públicos.

***REST***: Representational State Transfer — estilo arquitetural para APIs baseadas em
HTTP.

***Token***: cadeia de caracteres com significado específico dentro de um protocolo ou
linguagem — no BIOS, usado tanto para autenticação (JWT) quanto para redefinição de senha.

***XLSX***: formato de arquivo de planilha do Microsoft Excel, utilizado na exportação de
listas de empresas.
