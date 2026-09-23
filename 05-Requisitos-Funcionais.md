# 05. Requisitos Funcionais (RF)

## 5.1 Requisitos Funcionais do Sistema BIOS

<p align="center"><b>[RF01] Cadastrar usuário</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF02], [RF22]. |
| **Objetivo:** | O sistema deve permitir que um usuário crie uma conta para acessar as funcionalidades do BIOS. |


<p align="center"><b>[RF02] Autenticar usuário</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF01], [RF19], [RF20]. |
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
| **Objetivo:** | O sistema deve calcular um índice de aderência ambiental por empresa a partir das evidências coletadas. |


<p align="center"><b>[RF11] Calcular nível de confiança</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Sistema (processo automático). |
| **Requisitos associados:** | [RF10]. |
| **Objetivo:** | O sistema deve calcular um nível de confiança, separado do índice, refletindo quantidade e qualidade das evidências disponíveis. |


<p align="center"><b>[RF12] Explicar composição do score</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF10]. |
| **Objetivo:** | O sistema deve exibir quais evidências contribuíram para o índice, com peso e sinal (positivo/negativo). |


<p align="center"><b>[RF13] Filtrar empresas</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF10]. |
| **Objetivo:** | O sistema deve permitir filtrar empresas por CNAE, UF, município, índice mínimo e confiança mínima. |


<p align="center"><b>[RF14] Ordenar resultados por score</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF13]. |
| **Objetivo:** | O sistema deve permitir ordenar os resultados de uma busca por índice, ascendente ou descendente. |


<p align="center"><b>[RF15] Exportar lista em XLSX</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF13]. |
| **Objetivo:** | O sistema deve permitir exportar uma lista de empresas filtrada em formato XLSX. |


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
| **Requisitos associados:** | [RF01], [RF16], [RF25]. |
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
| **Requisitos associados:** | [RF01]. |
| **Objetivo:** | O sistema deve manter uma lista de domínios de e-mail bloqueados, inicializada a partir de um arquivo JSON de sementes (seed) e persistida como entidade em banco de dados, permitindo consulta em tempo de cadastro/autenticação e atualização posterior sem necessidade de novo deploy. |


<p align="center"><b>[RF23] Encerrar sessão</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Ator(es):** | Usuário. |
| **Requisitos associados:** | [RF02]. |
| **Objetivo:** | O sistema deve permitir que o usuário encerre sua sessão a qualquer momento, revogando o token JWT ativo. |


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
