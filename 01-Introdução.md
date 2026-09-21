# 01. Introdução

## 1.1 Objetivo do Projeto

O objetivo deste projeto é oferecer um mecanismo de consulta e avaliação do comprometimento
ambiental de empresas brasileiras, a partir da consolidação de evidências públicas associadas
ao CNPJ. Busca-se fornecer um ambiente que centralize informações hoje dispersas entre
diferentes órgãos governamentais, transformando dados brutos em indicadores explicáveis de
aderência aos Objetivos de Desenvolvimento Sustentável (ODS) de natureza ambiental. Com isso,
pretende-se apoiar a tomada de decisão de profissionais de negócios, de sustentabilidade,
pesquisadores, imprensa e órgãos públicos, sem incorrer nos riscos éticos e de privacidade
próprios de ferramentas de prospecção comercial tradicionais.

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

***ODS***: Objetivos de Desenvolvimento Sustentável da Organização das Nações Unidas.

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
