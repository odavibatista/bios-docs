# 05. Requisitos Funcionais (RF)

Os requisitos deste documento são redigidos conforme a sintaxe da ISO/IEC/IEEE 29148:2011
(Systems and software engineering — Life cycle processes — Requirements engineering; ver
`12-Referências.md`). Um requisito não é uma funcionalidade nem uma solução: ele traduz uma
necessidade de um stakeholder (ver `04-Atores-e-Histórias-dos-Usuários.md`) em uma declaração
clara, verificável e rastreável. Cada declaração é composta por até cinco elementos:

| Elemento | Responde a | Exemplo (RF20) |
| :----------- | :----------- | :----------- |
| **[Condição]** | Quando ou se o requisito se aplica. | Quando um mesmo IP ou usuário acumular 5 tentativas consecutivas de autenticação malsucedidas, |
| **[Sujeito]** | Quem realiza a ação (o sistema ou um de seus módulos). | o sistema |
| **[Ação]** | O comportamento esperado, em verbo verificável. | deve bloquear |
| **[Objeto]** | O que é manipulado ou produzido pela ação. | novas tentativas de autenticação desse IP ou usuário |
| **[Restrição]** | Limite de desempenho, qualidade, formato ou tempo. | por 15 (quinze) minutos. |

Heurísticas de redação aplicadas a todos os RF:

- **Obrigação em "deve".** Não se usam "poderá", "deveria", "tentará" nem equivalentes: o que
  é opcional para o usuário é descrito na condição (ex.: "se o usuário escolher sair de todos os
  dispositivos"), e o que é opcional para a entrega é sinalizado na prioridade.
- **Sem termos subjetivos.** "Rápido", "fácil", "adequado" e "simples" não aparecem; todo
  limite é numérico (tempo, tamanho, contagem) ou remete a uma regra enumerada.
- **Sujeito explícito.** A ação é sempre atribuída a "o sistema" ou a um módulo identificado.
- **Uma sentença por comportamento.** O RF que reúne mais de um comportamento do mesmo caso de
  uso traz uma sentença completa por comportamento, numerada (a), (b)..., cada uma com seus
  próprios elementos. Condição, objeto e restrição são omitidos quando não se aplicam.
- **Texto normativo separado das notas.** A linha "Requisito" contém só o que é exigido e
  verificável; justificativas, dependências e decisões de escopo ficam na linha "Observações".
- **O quê, não o como.** Tecnologias só aparecem como restrição de projeto definida pela
  equipe (ex.: o motor de templates de e-mail do [RF18]); o detalhamento fica em
  `13-Ficha-Técnica.md` e `14-Projeto-e-Arquitetura.md`.
- **Rastreável.** Cada RF indica a necessidade que o origina (história de usuário, em
  `04-Atores-e-Histórias-dos-Usuários.md`), os casos de uso (`08-Artefatos-de-Análise.md`,
  seção 8.2) que o realizam e os requisitos associados. Qualidades mensuráveis (tempo, disponibilidade,
  segurança) ficam nos RNF (`06-Requisitos-Nao-Funcionais.md`), e o que o sistema não deve fazer
  fica nos RIN (`07-Requisitos-Inversos.md`).

## 5.1 Requisitos Funcionais do Sistema BIOS

<p align="center"><b>[RF01] Cadastrar usuário</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC01. |
| **Necessidade (história de usuário):** | HU01. |
| **Requisitos associados:** | [RF02], [RF22], [RF28]. |
| **Requisito:** | (a) Quando uma pessoa informar nome (de 2 a 100 caracteres), e-mail e senha (de 8 a 128 caracteres, com ao menos uma letra e um número), o sistema deve criar uma conta inativa, com papel `USER`.<br>(b) Quando o e-mail informado já estiver cadastrado, o sistema deve exibir a mesma resposta dada a um e-mail novo, sem revelar a existência da conta. |


<p align="center"><b>[RF02] Autenticar usuário</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC02. |
| **Necessidade (história de usuário):** | HU03. |
| **Requisitos associados:** | [RF01], [RF19], [RF20], [RF32]. |
| **Requisito:** | (a) Quando um usuário com conta ativa informar e-mail e senha válidos, o sistema deve autenticá-lo e abrir uma sessão própria do dispositivo.<br>(b) Quando o e-mail ou a senha informados forem inválidos, o sistema deve exibir uma mensagem de credenciais inválidas que não distingue e-mail inexistente de senha incorreta.<br>(c) Quando uma requisição não trouxer sessão válida, o sistema deve recusá-la, exceto nas rotas de acesso anônimo (cadastro, confirmação de e-mail, login, renovação de sessão, recuperação de senha e health check). |


<p align="center"><b>[RF03] Buscar empresa por CNPJ</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC06. |
| **Necessidade (história de usuário):** | HU11. |
| **Requisitos associados:** | [RF02], [RF05], [RF07]. |
| **Requisito:** | (a) Quando o usuário autenticado informar um CNPJ (numérico ou alfanumérico) com formato e dígitos verificadores válidos, o sistema deve localizar a empresa correspondente, consultando as fontes cadastrais na ordem de prioridade do catálogo de fontes.<br>(b) Quando o CNPJ informado for inválido, o sistema deve exibir erro de validação.<br>(c) Quando nenhuma fonte cadastral consultada encontrar o CNPJ, o sistema deve informar que a empresa não foi encontrada. |


<p align="center"><b>[RF04] Buscar empresa por razão social/nome fantasia</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC06. |
| **Necessidade (história de usuário):** | HU12. |
| **Requisitos associados:** | [RF03]. |
| **Requisito:** | Quando o usuário autenticado informar um texto de até 100 caracteres, com ao menos uma palavra de 3 ou mais caracteres, o sistema deve listar, de forma paginada, as empresas da base local em que cada palavra digitada inicie alguma palavra da razão social ou do nome fantasia, em qualquer ordem e ignorando caixa, acentos e pontuação. |


<p align="center"><b>[RF05] Exibir dados cadastrais da empresa</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC06. |
| **Necessidade (história de usuário):** | HU11, HU13. |
| **Requisitos associados:** | [RF03], [RF06]. |
| **Requisito:** | Quando o perfil cadastral de uma empresa for localizado, o sistema deve exibir ao usuário a razão social, a situação cadastral, o porte, o endereço e o CNAE (principal e secundários) da empresa consultada, sem dado pessoal de sócio ou administrador ([INL02]). |


<p align="center"><b>[RF06] Exibir descrição textual do CNAE</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC06. |
| **Necessidade (história de usuário):** | HU14. |
| **Requisitos associados:** | [RF05]. |
| **Requisito:** | Quando o sistema exibir um código CNAE, ele deve exibir junto a descrição da atividade econômica correspondente, adotando a descrição oficial do IBGE e, na sua ausência, a do provedor cadastral. |


<p align="center"><b>[RF07] Consolidar evidências ambientais</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC07, UC14. |
| **Necessidade (história de usuário):** | HU15. |
| **Requisitos associados:** | [RF03], [RF08], [RF10]. |
| **Requisito:** | Quando o perfil ambiental de uma empresa com perfil cadastral local for solicitado, o sistema deve consolidar, por CNPJ, as evidências ambientais das fontes públicas IBAMA, CEIS/CNEP (CGU) e GHG Protocol — Registro Público de Emissões, mantendo as evidências das demais fontes quando uma delas estiver indisponível ([NFCO01]). |


<p align="center"><b>[RF08] Registrar metadados da evidência</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Casos de uso:** | UC07, UC14. |
| **Necessidade (história de usuário):** | HU16, HU18. |
| **Requisitos associados:** | [RF07]. |
| **Requisito:** | Quando o sistema registrar uma evidência, ele deve gravar a fonte, a categoria, a data de coleta, a data de publicação e o estado (`CONFIRMED`, `NOT_FOUND`, `UNKNOWN` ou `OUTDATED`) da evidência, sem duplicá-la quando a coleta for repetida. |


<p align="center"><b>[RF09] Exibir referência à fonte original</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC07. |
| **Necessidade (história de usuário):** | HU17. |
| **Requisitos associados:** | [RF07], [RF08]. |
| **Requisito:** | (a) Quando o sistema exibir uma evidência, ele deve exibir o nome público da fonte e o link ou identificador do órgão de origem, para auditoria, com link em URL HTTPS absoluta e sem credenciais.<br>(b) Quando uma evidência não tiver identificador não vazio nem link válido, o sistema deve exibi-la como `UNKNOWN`, sem sinal, resumo nem detalhes confirmatórios. |


<p align="center"><b>[RF10] Calcular índice de aderência ambiental</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Casos de uso:** | UC07, UC15. |
| **Necessidade (história de usuário):** | HU19. |
| **Requisitos associados:** | [RF07], [RF11]. |
| **Requisito:** | Quando as evidências normalizadas de uma empresa forem criadas ou atualizadas, o sistema deve calcular, por empresa, um índice de aderência ambiental para cada ODS de referência do MVP — ODS 13 (Ação Contra a Mudança Global do Clima) e ODS 15 (Vida Terrestre) —, a partir das evidências vinculadas a cada ODS, sem combiná-los em índice geral. |
| **Observações:** | Os ODS de referência são entidades próprias do banco de dados, populadas via seed (ver `14-Projeto-e-Arquitetura.md`, seção 3.1). |


<p align="center"><b>[RF11] Calcular nível de confiança</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Casos de uso:** | UC07, UC15. |
| **Necessidade (história de usuário):** | HU20. |
| **Requisitos associados:** | [RF10]. |
| **Requisito:** | (a) Quando o sistema calcular o índice de um ODS, ele deve calcular, para esse índice, um nível de confiança separado, a partir da quantidade e da qualidade das evidências disponíveis para o ODS, e exibi-los lado a lado, nunca combinados.<br>(b) Quando não houver evidência conclusiva para um ODS, o sistema deve registrar o índice como indisponível, com a confiança mínima. |


<p align="center"><b>[RF12] Explicar composição do score</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC07, UC15. |
| **Necessidade (história de usuário):** | HU21. |
| **Requisitos associados:** | [RF10]. |
| **Requisito:** | Quando o usuário selecionar a explicação do score de um ODS, o sistema deve listar, separadamente para esse ODS, as evidências que contribuíram para o índice, com o peso e o sinal (positivo ou negativo) de cada uma. |


<p align="center"><b>[RF13] Filtrar empresas</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC08. |
| **Necessidade (história de usuário):** | HU23, HU24. |
| **Requisitos associados:** | [RF10]. |
| **Requisito:** | Quando o usuário informar critérios de pesquisa, o sistema deve listar, de forma paginada, as empresas da base local que atendam aos filtros de CNAE, UF e município e, para um único ODS selecionado, ao índice mínimo e à confiança mínima informados. |


<p align="center"><b>[RF14] Ordenar resultados por score</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC08. |
| **Necessidade (história de usuário):** | HU24. |
| **Requisitos associados:** | [RF13]. |
| **Requisito:** | Quando o usuário escolher a ordenação de uma pesquisa, o sistema deve ordenar os resultados pelo índice do ODS selecionado, em ordem ascendente ou descendente, conforme a escolha. |


<p align="center"><b>[RF15] Exportar lista em XLSX</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC09. |
| **Necessidade (história de usuário):** | HU25. |
| **Requisitos associados:** | [RF13]. |
| **Requisito:** | Quando o usuário solicitar a exportação de uma pesquisa com ao menos um resultado, o sistema deve gerar um arquivo em formato XLSX com a lista filtrada e ordenada, com o índice e o nível de confiança em colunas separadas por ODS, sem dado pessoal ou de contato de indivíduo ([INL02]). |


<p align="center"><b>[RF16] Gerenciar chaves de API</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC11. |
| **Necessidade (história de usuário):** | HU30. |
| **Requisitos associados:** | [RF01], [RF17]. |
| **Requisito:** | (a) Quando o usuário autenticado informar um nome de identificação (de 2 a 60 caracteres), o sistema deve gerar uma chave de API e exibir o valor completo uma única vez, até o limite de 10 chaves ativas por usuário.<br>(b) Quando o usuário autenticado solicitar a listagem, o sistema deve exibir, para cada chave ativa, apenas o nome, o prefixo, a data de criação e a data do último uso.<br>(c) Quando o usuário autenticado confirmar a revogação de uma chave própria, o sistema deve invalidá-la imediatamente. |


<p align="center"><b>[RF17] Expor API pública documentada</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Desenvolvedor externo. |
| **Casos de uso:** | UC12. |
| **Necessidade (história de usuário):** | HU31. |
| **Requisitos associados:** | [RF16], [RF05], [RF10]. |
| **Requisito:** | (a) Quando um sistema de terceiros enviar uma requisição com chave de API ativa ao endpoint de perfil, o sistema deve retornar o perfil da empresa em JSON (dados cadastrais, evidências e, por ODS, índice, confiança e explicação), dentro do limite de requisições por chave.<br>(b) Quando a chave for ausente, inválida ou revogada, o sistema deve responder HTTP 401, sem indicar o motivo.<br>(c) Quando o limite de requisições por chave for excedido, o sistema deve responder HTTP 429, informando em `Retry-After` quando a chamada pode ser repetida.<br>(d) O sistema deve documentar os endpoints públicos em OpenAPI/Swagger. |


<p align="center"><b>[RF18] Disparar e-mails transacionais</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Casos de uso:** | UC01, UC02, UC04, UC05, UC17. |
| **Necessidade (história de usuário):** | HU02, HU06, HU09, HU35. |
| **Requisitos associados:** | [RF01], [RF16], [RF25], [RF28], [RF32]. |
| **Requisito:** | (a) Quando uma conta for criada ou o e-mail de uma conta for alterado, o sistema deve enviar ao endereço informado um e-mail transacional de confirmação.<br>(b) Quando um usuário solicitar a redefinição de senha de uma conta ativa, o sistema deve enviar ao e-mail cadastrado um e-mail transacional com o link de redefinição.<br>(c) Quando uma conta for bloqueada por tentativas de login malsucedidas ou o reuso de um refresh token for detectado, o sistema deve enviar ao titular da conta um e-mail transacional de alerta de login suspeito.<br>(d) O sistema deve compor os e-mails transacionais a partir de templates Handlebars, enviados via Nodemailer. |


<p align="center"><b>[RF19] Registrar tentativas de login</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Casos de uso:** | UC02. |
| **Necessidade (história de usuário):** | HU09. |
| **Requisitos associados:** | [RF02], [RF20]. |
| **Requisito:** | Quando uma tentativa de autenticação for concluída, o sistema deve registrar o IP de origem, a data e a hora e o resultado (sucesso ou falha) da tentativa. |


<p align="center"><b>[RF20] Bloquear temporariamente após tentativas de login malsucedidas</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Casos de uso:** | UC02. |
| **Necessidade (história de usuário):** | HU09. |
| **Requisitos associados:** | [RF19]. |
| **Requisito:** | Quando um mesmo IP ou um mesmo usuário acumular 5 (cinco) tentativas consecutivas de autenticação malsucedidas, o sistema deve bloquear novas tentativas de autenticação desse IP ou usuário por 15 (quinze) minutos, informando o tempo restante do bloqueio. |


<p align="center"><b>[RF21] Expor endpoints/tabelas isca (honeypot)</b></p>

| Prioridade: | ☐ Essencial / ☐ Importante / ☑ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Casos de uso:** | Não há (processo automático, fora dos casos de uso — ver `08-Artefatos-de-Análise.md`, seção 8.1). |
| **Necessidade (história de usuário):** | HU34. |
| **Requisitos associados:** | [RF17], [RF20]. |
| **Requisito:** | (a) Quando um chamador acessar um endpoint isca, o sistema deve retornar dados sintéticos de empresas, com CNPJs válidos, gerados com Faker.js e um gerador de CNPJ válido, em no máximo 10 registros por página.<br>(b) Quando um chamador acessar um endpoint isca, o sistema deve registrar e bloquear o IP do chamador, por meio de um decorator dedicado. |


<p align="center"><b>[RF22] Manter blacklist de domínios de e-mail</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático); Administrador. |
| **Casos de uso:** | UC01, UC13. |
| **Necessidade (história de usuário):** | HU01, HU33. |
| **Requisitos associados:** | [RF01], [RF29]. |
| **Requisito:** | (a) Quando um cadastro ou uma troca de e-mail for solicitado, o sistema deve recusar o e-mail cujo domínio conste na lista de domínios bloqueados.<br>(b) O sistema deve manter a lista de domínios bloqueados como entidade persistida em banco de dados, inicializada a partir de um arquivo JSON de sementes (seed) e atualizável sem novo deploy. |


<p align="center"><b>[RF23] Encerrar sessão</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC03. |
| **Necessidade (história de usuário):** | HU04. |
| **Requisitos associados:** | [RF02], [RF32]. |
| **Requisito:** | (a) Quando o usuário autenticado solicitar sair, o sistema deve encerrar a sessão do dispositivo atual, com efeito imediato sobre os tokens já emitidos.<br>(b) Se o usuário escolher sair de todos os dispositivos, o sistema deve encerrar todas as suas sessões ativas, com efeito imediato sobre os tokens já emitidos. |


<p align="center"><b>[RF24] Alterar senha</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC04. |
| **Necessidade (história de usuário):** | HU05. |
| **Requisitos associados:** | [RF02], [RF23]. |
| **Requisito:** | Quando o usuário autenticado informar a senha atual correta e uma nova senha válida (mesma política do cadastro e diferente da atual), o sistema deve gravar a nova senha e revogar todas as sessões ativas do usuário ao concluir a troca. |


<p align="center"><b>[RF25] Recuperar senha esquecida</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC04. |
| **Necessidade (história de usuário):** | HU06. |
| **Requisitos associados:** | [RF01], [RF18]. |
| **Requisito:** | (a) Quando um usuário solicitar a redefinição de senha informando o e-mail de uma conta ativa, o sistema deve enviar a esse e-mail um link com token de uso único, válido por 2 horas por padrão (configurável), e exibir sempre a mesma mensagem de confirmação, exista ou não conta com o e-mail informado.<br>(b) Quando o usuário acessar um link válido e informar uma nova senha válida, o sistema deve gravar a nova senha e revogar todas as sessões ativas do usuário. |


<p align="center"><b>[RF26] Editar dados da conta</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC05. |
| **Necessidade (história de usuário):** | HU07. |
| **Requisitos associados:** | [RF02], [RF28]. |
| **Requisito:** | (a) Quando o usuário autenticado alterar o nome ou o endereço, o sistema deve gravar os dados, com o endereço armazenado de forma cifrada.<br>(b) Quando o usuário autenticado alterar o e-mail informando a senha atual, o sistema deve enviar um link de confirmação ao novo endereço e só efetivar a troca após a confirmação ([RF28]). |


<p align="center"><b>[RF27] Excluir conta</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC05. |
| **Necessidade (história de usuário):** | HU08. |
| **Requisitos associados:** | [RF02], [RF16], [RF23]. |
| **Requisito:** | Quando o usuário autenticado confirmar a exclusão da própria conta com sua senha, o sistema deve remover seus dados pessoais, revogar todas as suas chaves de API e encerrar todas as suas sessões ativas, atendendo ao direito de eliminação do titular previsto na LGPD ([NFSE02]). |


<p align="center"><b>[RF28] Confirmar e-mail de cadastro</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário; Sistema (processo automático). |
| **Casos de uso:** | UC01, UC05. |
| **Necessidade (história de usuário):** | HU02, HU07. |
| **Requisitos associados:** | [RF01], [RF18]. |
| **Requisito:** | (a) Enquanto o e-mail informado no cadastro não for confirmado, o sistema deve manter a conta inativa, recusando seu login.<br>(b) Quando o usuário acessar o link de confirmação, enviado por e-mail transacional ([RF18]) com token de uso único válido por 24 horas por padrão (configurável), o sistema deve ativar a conta. |


<p align="center"><b>[RF29] Gerenciar blacklist de domínios de e-mail</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Administrador. |
| **Casos de uso:** | UC13. |
| **Necessidade (história de usuário):** | HU33. |
| **Requisitos associados:** | [RF22]. |
| **Requisito:** | Quando um usuário com papel de administrador solicitar, o sistema deve consultar, adicionar e remover domínios da blacklist mantida pelo [RF22], recusando domínio inválido ou já cadastrado, com efeito imediato e sem novo deploy. |


<p align="center"><b>[RF30] Consultar evolução do índice médio por setor</b></p>

| Prioridade: | ☐ Essencial / ☐ Importante / ☑ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC10. |
| **Necessidade (história de usuário):** | HU26. |
| **Requisitos associados:** | [RF06], [RF10]. |
| **Requisito:** | Quando o usuário selecionar um CNAE e um ODS, o sistema deve exibir a evolução do índice de aderência ambiental médio do setor entre janelas de tempo sucessivas, agregada por CNAE e nunca por empresa individual. |
| **Observações:** | Conforme o indicador [IA05] e sua nota de restrição em `15-Impacto-Ambiental.md`. Depende da decisão de armazenamento de série histórica registrada na seção 15.4. |


<p align="center"><b>[RF31] Disponibilizar integração com CRMs (bônus — fora do MVP)</b></p>

| Prioridade: | ☐ Essencial / ☐ Importante / ☑ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Desenvolvedor externo. |
| **Casos de uso:** | UC16. |
| **Necessidade (história de usuário):** | HU32. |
| **Requisitos associados:** | [RF16], [RF17]. |
| **Requisito:** | Quando um CRM de terceiros enviar uma lista de CNPJs com chave de API ativa, o sistema deve retornar o perfil ambiental de cada pessoa jurídica (dados cadastrais, índice, confiança e evidências) em formato de importação de contas, sem exportar dado de contato nem dado pessoal de indivíduo ([INL02]). |
| **Observações:** | Requisito bônus: entra apenas se houver folga no cronograma após a entrega dos requisitos essenciais e importantes, e não compõe o escopo mínimo do MVP. |


<p align="center"><b>[RF32] Renovar sessão</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Casos de uso:** | UC17. |
| **Necessidade (história de usuário):** | HU35. |
| **Requisitos associados:** | [RF02], [RF18], [RF23]. |
| **Requisito:** | (a) Quando o usuário apresentar um refresh token válido, o sistema deve trocá-lo por um novo par de access token e refresh token, invalidando o refresh token apresentado (uso único, com rotação).<br>(b) Quando for apresentado um refresh token já rotacionado, o sistema deve revogar todas as sessões do usuário e disparar o alerta de login suspeito por e-mail ([RF18]). |
