# 05. Requisitos Funcionais (RF)

## 5.1 Requisitos Funcionais do Sistema BIOS

<p align="center"><b>[RF01] Cadastrar usuário</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF02], [RF22], [RF28]. |
| **Objetivo:** | O sistema deve permitir que um usuário crie uma conta para acessar as funcionalidades do BIOS. |


<p align="center"><b>[RF02] Autenticar usuário</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF01], [RF19], [RF20], [RF32]. |
| **Objetivo:** | O sistema deve permitir que um usuário se autentique para acessar informações de empresas, exigindo login válido para qualquer consulta. |


<p align="center"><b>[RF03] Buscar empresa por CNPJ</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF02], [RF05], [RF07]. |
| **Objetivo:** | O sistema deve permitir buscar uma empresa a partir do seu CNPJ. |


<p align="center"><b>[RF04] Buscar empresa por razão social/nome fantasia</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF03]. |
| **Objetivo:** | O sistema deve permitir buscar empresas por aproximação de nome quando o CNPJ não é conhecido. |


<p align="center"><b>[RF05] Exibir dados cadastrais da empresa</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF03], [RF06]. |
| **Objetivo:** | O sistema deve exibir razão social, situação cadastral, porte, endereço e CNAE (principal e secundários) da empresa consultada. |


<p align="center"><b>[RF06] Exibir descrição textual do CNAE</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF05]. |
| **Objetivo:** | O sistema deve traduzir o código CNAE em sua descrição de atividade econômica. |


<p align="center"><b>[RF07] Consolidar evidências ambientais</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF03], [RF08], [RF10]. |
| **Objetivo:** | O sistema deve consolidar, por CNPJ, evidências ambientais provenientes de múltiplas fontes públicas (IBAMA, CEIS/CNEP, GHG Protocol — Registro Público de Emissões). |


<p align="center"><b>[RF08] Registrar metadados da evidência</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Requisitos associados:** | [RF07]. |
| **Objetivo:** | Cada evidência deve registrar fonte, categoria, data de coleta, data de publicação e estado (CONFIRMED, NOT_FOUND, UNKNOWN, OUTDATED). |


<p align="center"><b>[RF09] Exibir referência à fonte original</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF07], [RF08]. |
| **Objetivo:** | O sistema deve exibir, para cada evidência, o link ou identificador do órgão de origem, permitindo auditoria. |


<p align="center"><b>[RF10] Calcular índice de aderência ambiental</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Requisitos associados:** | [RF07], [RF11]. |
| **Objetivo:** | O sistema deve calcular, por empresa, um índice de aderência ambiental para cada ODS de referência do MVP — ODS 13 (Ação Contra a Mudança Global do Clima) e ODS 15 (Vida Terrestre) —, a partir das evidências vinculadas a cada ODS. Os ODS de referência são entidades próprias do banco de dados, populadas via seed (ver `14-Projeto-e-Arquitetura.md`, seção 3.1). |


<p align="center"><b>[RF11] Calcular nível de confiança</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Requisitos associados:** | [RF10]. |
| **Objetivo:** | O sistema deve calcular, para cada índice por ODS, um nível de confiança separado do índice, refletindo quantidade e qualidade das evidências disponíveis para aquele ODS. |


<p align="center"><b>[RF12] Explicar composição do score</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF10]. |
| **Objetivo:** | O sistema deve exibir, separadamente para cada ODS, quais evidências contribuíram para o índice, com peso e sinal (positivo/negativo). |


<p align="center"><b>[RF13] Filtrar empresas</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF10]. |
| **Objetivo:** | O sistema deve permitir filtrar empresas por CNAE, UF, município e, para um ODS selecionado, índice mínimo e confiança mínima. |


<p align="center"><b>[RF14] Ordenar resultados por score</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF13]. |
| **Objetivo:** | O sistema deve permitir ordenar os resultados de uma busca pelo índice de um ODS selecionado, ascendente ou descendente. |


<p align="center"><b>[RF15] Exportar lista em XLSX</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF13]. |
| **Objetivo:** | O sistema deve permitir exportar uma lista de empresas filtrada em formato XLSX, com índice e nível de confiança em colunas separadas por ODS. |


<p align="center"><b>[RF16] Gerenciar chaves de API</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF01], [RF17]. |
| **Objetivo:** | O sistema deve permitir que o usuário gere e revogue suas próprias chaves de API. |


<p align="center"><b>[RF17] Expor API pública documentada</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Desenvolvedor externo. |
| **Requisitos associados:** | [RF16], [RF05], [RF10]. |
| **Objetivo:** | O sistema deve expor endpoints documentados (OpenAPI/Swagger) para consulta de perfil de empresa via API. |


<p align="center"><b>[RF18] Disparar e-mails transacionais</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Requisitos associados:** | [RF01], [RF16], [RF25], [RF28], [RF32]. |
| **Objetivo:** | O sistema deve disparar e-mails transacionais (confirmação de cadastro, recuperação de senha, alerta de login suspeito) usando templates Handlebars via Nodemailer. |


<p align="center"><b>[RF19] Registrar tentativas de login</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Requisitos associados:** | [RF02], [RF20]. |
| **Objetivo:** | O sistema deve registrar IP de origem, data/hora e resultado (sucesso/falha) de cada tentativa de autenticação. |


<p align="center"><b>[RF20] Bloquear temporariamente após tentativas de login malsucedidas</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Requisitos associados:** | [RF19]. |
| **Objetivo:** | O sistema deve bloquear temporariamente novas tentativas de autenticação de um mesmo IP e/ou usuário após 5 (cinco) tentativas consecutivas malsucedidas, mantendo o bloqueio por 15 (quinze) minutos. |


<p align="center"><b>[RF21] Expor endpoints/tabelas isca (honeypot)</b></p>

| Prioridade: | ☐ Essencial / ☐ Importante / ☑ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Requisitos associados:** | [RF17], [RF20]. |
| **Objetivo:** | O sistema deve disponibilizar endpoints isca que retornam dados sintéticos e plausíveis (empresas e CNPJs gerados via Faker.js e gerador de CNPJ válido), em baixo volume (máximo de 10 registros por página) e com baixa disponibilidade proposital, registrando e bloqueando via decorator dedicado o IP de qualquer chamador desses endpoints. |


<p align="center"><b>[RF22] Manter blacklist de domínios de e-mail</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático); Administrador. |
| **Requisitos associados:** | [RF01], [RF29]. |
| **Objetivo:** | O sistema deve manter uma lista de domínios de e-mail bloqueados, inicializada a partir de um arquivo JSON de sementes (seed) e persistida como entidade em banco de dados, permitindo consulta em tempo de cadastro/autenticação e atualização posterior sem necessidade de novo deploy. |


<p align="center"><b>[RF23] Encerrar sessão</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF02], [RF32]. |
| **Objetivo:** | O sistema deve permitir que o usuário encerre, a qualquer momento, a sessão do dispositivo atual ou — opcionalmente — de todos os seus dispositivos, com efeito imediato sobre os tokens já emitidos. |


<p align="center"><b>[RF24] Alterar senha</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF02], [RF23]. |
| **Objetivo:** | O sistema deve permitir que o usuário autenticado altere sua senha, exigindo a senha atual como confirmação e revogando sessões ativas ao concluir a troca. |


<p align="center"><b>[RF25] Recuperar senha esquecida</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF01], [RF18]. |
| **Objetivo:** | O sistema deve permitir que um usuário solicite redefinição de senha via e-mail cadastrado, através de um token de uso único e validade limitada, enviado por e-mail transacional. |


<p align="center"><b>[RF26] Editar dados da conta</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF02], [RF28]. |
| **Objetivo:** | O sistema deve permitir que o usuário autenticado edite seus próprios dados cadastrais (nome e endereço), exigindo nova confirmação ([RF28]) caso o e-mail seja alterado. |


<p align="center"><b>[RF27] Excluir conta</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF02], [RF16], [RF23]. |
| **Objetivo:** | O sistema deve permitir que o usuário autenticado exclua a própria conta, mediante confirmação de senha, removendo seus dados pessoais, revogando todas as suas chaves de API e encerrando todas as suas sessões ativas — atendendo ao direito de eliminação do titular previsto na LGPD ([NFSE02]). |


<p align="center"><b>[RF28] Confirmar e-mail de cadastro</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário; Sistema (processo automático). |
| **Requisitos associados:** | [RF01], [RF18]. |
| **Objetivo:** | O sistema deve manter a conta recém-criada inativa até que o usuário confirme o e-mail informado, por meio de um token de uso único e validade limitada enviado por e-mail transacional ([RF18]). |


<p align="center"><b>[RF29] Gerenciar blacklist de domínios de e-mail</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Administrador. |
| **Requisitos associados:** | [RF22]. |
| **Objetivo:** | O sistema deve permitir que um usuário com papel de administrador consulte, adicione e remova domínios da blacklist mantida pelo [RF22], com efeito imediato e sem necessidade de novo deploy. |


<p align="center"><b>[RF30] Consultar evolução do índice médio por setor</b></p>

| Prioridade: | ☐ Essencial / ☐ Importante / ☑ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF06], [RF10]. |
| **Objetivo:** | O sistema deve exibir, para cada ODS, a evolução do índice de aderência ambiental médio agregado por CNAE entre janelas de tempo sucessivas, nunca por empresa individual — conforme o indicador [IA05] e sua nota de restrição em `15-Impacto-Ambiental.md`. Depende da decisão de armazenamento de série histórica registrada na seção 15.4. |


<p align="center"><b>[RF31] Disponibilizar integração com CRMs (bônus — fora do MVP)</b></p>

| Prioridade: | ☐ Essencial / ☐ Importante / ☑ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Desenvolvedor externo. |
| **Requisitos associados:** | [RF16], [RF17]. |
| **Objetivo:** | O sistema deve permitir que CRMs de terceiros importem o perfil ambiental de **pessoas jurídicas** (dados cadastrais, índice, confiança e evidências) a partir da API pública, sem exportar qualquer dado de contato ou dado pessoal de indivíduo ([INL02]). Requisito bônus: entra apenas se houver folga no cronograma após a entrega dos requisitos essenciais e importantes, e não compõe o escopo mínimo do MVP. |


<p align="center"><b>[RF32] Renovar sessão</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF02], [RF18], [RF23]. |
| **Objetivo:** | O sistema deve permitir renovar a sessão sem novo login, trocando um refresh token válido por um novo par de access token e refresh token. Cada refresh token é de uso único: a renovação o invalida (rotação), e a reapresentação de um refresh token já rotacionado deve revogar todas as sessões do usuário e disparar alerta de login suspeito por e-mail ([RF18]). |
