# 04. Atores e Histórias dos Usuários

Essa seção apresenta todos os atores da aplicação, bem como as principais histórias dos
usuários. Cada ator representa um papel particular de usuário da aplicação. Porém, além de
representar pessoas, os atores também podem ser processos automáticos ou outras aplicações que
troquem informações com o BIOS. O quadro 2 descreve brevemente cada ator da aplicação e o quadro
3 expõe as histórias dos usuários.

## 4.1 Atores

<p align="center"><b>Quadro 2 - Atores</b></p>

| Ator | Descrição |
| ------------ | ------------ |
| Usuário | Representa qualquer pessoa com conta autenticada no BIOS (papel `USER`). Pode consultar, filtrar, ordenar e exportar empresas, interpretar índices e evidências, e gerenciar a própria conta e chaves de API. É a generalização dos perfis de público descritos na seção 2.2 (ver seção 4.2). |
| Administrador | Representa um usuário com papel `ADMIN` — mesma entidade de usuário, diferenciada apenas pelo valor do enum de papel. Além de tudo o que o Usuário faz, mantém a blacklist de domínios de e-mail. |
| Desenvolvedor Externo | Representa um usuário que consome os dados do BIOS de forma programática, por meio da API pública e de chave de API gerada no próprio painel de conta. Não é um papel distinto no enum: é um Usuário atuando via API. |
| Sistema | Representa os processos automáticos do BIOS: ingestão de dados das fontes externas, cálculo de índice e de nível de confiança, disparo de e-mails transacionais, registro e bloqueio de tentativas de login e honeypot. |
| Fontes Externas | Representa as aplicações públicas das quais o BIOS consome dados: Receita Federal (via BrasilAPI e Minha Receita), IBGE, IBAMA, CGU (CEIS/CNEP) e GHG Protocol — Registro Público de Emissões (ver `14-Projeto-e-Arquitetura.md`, seção 1). |

## 4.2 Perfis de Usuário (Personas)

O papel Usuário agrupa públicos com objetivos distintos, todos com as mesmas permissões no
sistema. Para que as histórias reflitam necessidades concretas, a coluna "Eu como..." do quadro
3 usa os perfis abaixo, derivados dos stakeholders da seção 2.2.

| Perfil | Necessidade principal |
| ------------ | ------------ |
| Analista de compras/suprimentos | Avaliar o comprometimento ambiental de fornecedores, prestadores de serviço e clientes antes de firmar relacionamento comercial. |
| Profissional de sustentabilidade/ESG | Embasar avaliações socioambientais de parceiros com evidências rastreáveis e auditáveis. |
| Agente de órgão governamental | Consultar, de forma centralizada, o histórico público de autuações e sanções de uma empresa. |
| Jornalista | Apurar o histórico ambiental de uma empresa citada em uma matéria, com fontes verificáveis. |
| Pesquisador ou estudante | Estudar o comportamento ambiental de empresas e setores com base em dados reproduzíveis. |
| Representante de empresa em adequação | Encontrar empresas do próprio setor com bom histórico, para referência e parceria. |

## 4.3 Histórias dos Usuários

A coluna "Requisitos relacionados" acrescenta ao modelo de referência a rastreabilidade entre
cada história e os requisitos funcionais (`05-Requisitos-Funcionais.md`), não-funcionais
(`06-Requisitos-Nao-Funcionais.md`) e inversos (`07-Requisitos-Inversos.md`) que a atendem.

<p align="center"><b>Quadro 3 - Histórias dos Usuários</b></p>

| ID | Eu como... | Gostaria de... | Para... | Requisitos relacionados |
| ------------ | ------------ | ------------ | ------------ | ------------ |
| [HU01] | Usuário, | criar uma conta com meu e-mail e uma senha, | acessar as funcionalidades de consulta do BIOS. | [RF01], [RF22] |
| [HU02] | Usuário, | confirmar meu e-mail por meio de um link enviado no cadastro, | ativar minha conta e garantir que ninguém se cadastre com um e-mail que não é seu. | [RF28], [RF18] |
| [HU03] | Usuário, | me autenticar com e-mail e senha, | consultar informações de empresas com acesso restrito a usuários cadastrados. | [RF02] |
| [HU04] | Usuário, | encerrar minha sessão a qualquer momento, | impedir que outra pessoa use minha conta em um dispositivo compartilhado. | [RF23], [NFSE05] |
| [HU05] | Usuário, | alterar minha senha informando a senha atual, com as demais sessões encerradas ao final, | manter a segurança do acesso à minha conta. | [RF24], [NFSE05] |
| [HU06] | Usuário, | recuperar minha senha esquecida por meio do e-mail cadastrado, | retomar o acesso à minha conta sem depender de suporte. | [RF25], [RF18] |
| [HU07] | Usuário, | editar meus dados de conta, | manter meu cadastro atualizado. | [RF26], [RF28] |
| [HU08] | Usuário, | excluir minha conta e meus dados pessoais, | exercer meu direito de eliminação como titular de dados, conforme a LGPD. | [RF27], [NFSE02] |
| [HU09] | Usuário, | que tentativas repetidas de login malsucedidas sejam bloqueadas temporariamente e que eu seja avisado por e-mail de um login suspeito, | impedir que minha conta seja invadida por tentativa e erro. | [RF19], [RF20], [RF18], [NFSE06] |
| [HU10] | Usuário, | que meus dados pessoais sejam armazenados de forma criptografada, | ter minha privacidade preservada mesmo em caso de vazamento do banco de dados. | [NFSE03] |
| [HU11] | Analista de compras/suprimentos, | buscar um fornecedor pelo CNPJ, | conhecer seu perfil ambiental antes de contratá-lo. | [RF03], [RF05] |
| [HU12] | Jornalista, | buscar uma empresa pela razão social ou nome fantasia, | apurar uma matéria mesmo sem conhecer o CNPJ da empresa citada. | [RF04] |
| [HU13] | Usuário, | ver razão social, situação cadastral, porte, endereço e CNAE da empresa consultada, | confirmar que encontrei a empresa certa antes de analisar suas evidências. | [RF05] |
| [HU14] | Pesquisador ou estudante, | ver a descrição textual da atividade econômica (CNAE) da empresa, | entender o setor de atuação sem consultar a tabela de classificação do IBGE. | [RF06] |
| [HU15] | Profissional de sustentabilidade/ESG, | ver em uma única tela as evidências ambientais da empresa consolidadas de diferentes órgãos públicos, | não precisar visitar vários portais governamentais para montar o histórico. | [RF07] |
| [HU16] | Profissional de sustentabilidade/ESG, | ver o estado de cada evidência (confirmada, não encontrada, desconhecida ou desatualizada) e as datas de publicação e de coleta, | distinguir falta de informação de evidência negativa e saber quão recente é cada dado. | [RF08], [IND02] |
| [HU17] | Agente de órgão governamental, | acessar o link ou identificador da fonte original de cada evidência, | auditar a informação diretamente no órgão que a publicou. | [RF09], [NFCO02] |
| [HU18] | Jornalista, | ver sanções administrativas (CEIS/CNEP) separadas de infrações ambientais (IBAMA), | não noticiar uma sanção sem motivação ambiental como se fosse dano ambiental. | [RF08], [IND01] |
| [HU19] | Analista de compras/suprimentos, | ver o índice de aderência ambiental da empresa para cada ODS (13 — Ação Climática e 15 — Vida Terrestre), | comparar fornecedores com um critério objetivo e padronizado. | [RF10] |
| [HU20] | Profissional de sustentabilidade/ESG, | ver, para cada ODS, o nível de confiança separado do índice, | saber se o índice se apoia em muitas ou em poucas evidências antes de tomar uma decisão. | [RF11], [IND02] |
| [HU21] | Pesquisador ou estudante, | ver quais evidências compuseram o índice de cada ODS, com seus pesos e sinais (positivo ou negativo), | entender e reproduzir o cálculo do índice. | [RF12], [NFCO02] |
| [HU22] | Usuário, | que o sistema não declare que uma empresa "é" ou "não é" sustentável, | tirar minhas próprias conclusões a partir das evidências apresentadas. | [IND01] |
| [HU23] | Analista de compras/suprimentos, | filtrar empresas por CNAE, UF, município e por índice e confiança mínimos em um ODS escolhido, | montar uma lista de potenciais fornecedores do meu setor e da minha região. | [RF13] |
| [HU24] | Representante de empresa em adequação, | ordenar os resultados de uma busca pelo índice de aderência de um ODS escolhido, | encontrar rapidamente empresas do meu setor com melhor histórico, para referência e parceria. | [RF13], [RF14] |
| [HU25] | Analista de compras/suprimentos, | exportar a lista de empresas filtrada em formato XLSX, | compartilhá-la com minha equipe e trabalhar com ela em planilhas. | [RF15] |
| [HU26] | Pesquisador ou estudante, | acompanhar a evolução do índice médio de aderência de cada ODS por setor (CNAE) ao longo do tempo, | estudar tendências setoriais sem expor empresas individuais a um ranking punitivo. | [RF30] |
| [HU27] | Usuário, | que a consulta continue respondendo com os dados disponíveis quando uma das fontes externas estiver fora do ar, | não ficar sem nenhuma informação por falha de um único órgão. | [NFCO01] |
| [HU28] | Usuário, | que a consulta de uma empresa já consultada recentemente responda em até 2 segundos, | ter uma experiência de uso fluida. | [NFDE01], [NFDE02] |
| [HU29] | Agente de órgão governamental, | que o sistema não exiba nenhum dado pessoal de sócios, administradores ou colaboradores da empresa consultada, | ter a garantia de que a plataforma não serve para expor ou contatar indivíduos. | [INL02], [NFSE02] |
| [HU30] | Desenvolvedor Externo, | gerar e revogar minhas próprias chaves de API pelo painel de conta, | integrar meu sistema ao BIOS e cortar o acesso imediatamente caso uma chave vaze. | [RF16], [NFSE01] |
| [HU31] | Desenvolvedor Externo, | consultar o perfil de uma empresa por meio de uma API documentada em OpenAPI/Swagger, | integrar os dados do BIOS ao meu sistema sem precisar de engenharia reversa. | [RF17], [NFSE04] |
| [HU32] | Desenvolvedor Externo de um CRM, | importar o perfil ambiental de pessoas jurídicas para o CRM (requisito bônus, fora do MVP), | enriquecer o cadastro de contas corporativas com dados ambientais, sem dados de contato de indivíduos. | [RF31], [INL02] |
| [HU33] | Administrador, | adicionar e remover domínios da blacklist de e-mails sem novo deploy, | impedir cadastros com e-mails descartáveis assim que um novo domínio for identificado. | [RF22], [RF29] |
| [HU34] | Administrador, | que qualquer chamador de um endpoint isca tenha seu IP registrado e bloqueado automaticamente, | identificar e conter robôs de raspagem sem afetar os usuários legítimos. | [RF21], [NFSE07] |
