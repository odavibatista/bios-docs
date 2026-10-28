# 06. Requisitos Não-Funcionais (RNF)

Os requisitos não funcionais descrevem restrições e qualidades — como o sistema deve se comportar,
e não o que ele faz — e seguem a mesma sintaxe ISO/IEC/IEEE 29148:2011 dos requisitos funcionais
([Condição], [Sujeito], [Ação], [Objeto] e [Restrição]; ver `05-Requisitos-Funcionais.md`, que
traz as heurísticas de redação). Aqui, a [Restrição] é mensurável sempre que a qualidade admite
medida (tempo, contagem, algoritmo, percentual): nenhum RNF usa termos subjetivos como "rápido"
ou "seguro" sem um limite verificável. Quando o requisito rege o
processo de desenvolvimento e não o comportamento do produto (ex.: [NFPD03], [NFSE08]–[NFSE11]),
o [Sujeito] é "o projeto".

As categorias abaixo seguem as nove características de qualidade da ISO/IEC 25010:2023
(SQuaRE — ver `12-Referências.md`), em substituição ao agrupamento ad-hoc usado em versões
anteriores deste documento (que incluía uma categoria "Padrões" sem correspondência na
norma). Os identificadores originais de cada RNF foram mantidos mesmo após a reclassificação,
para preservar rastreabilidade com as referências já feitas a eles em `13-Ficha-Técnica.md` e
`14-Projeto-e-Arquitetura.md`.

Correspondência com a edição de 2011 da norma, de oito características (ver `12-Referências.md`):
Adequação Funcional (*Functional Suitability*), Eficiência de
Desempenho, Compatibilidade, Confiabilidade, Segurança e Manutenibilidade mantêm-se; Usabilidade
passou a se chamar Capacidade de Interação; Portabilidade passou a se chamar Flexibilidade (e
ganhou a sub-característica Escalabilidade); e Segurança Operacional (*Safety*) é nova. O modelo
de qualidade em uso da edição de 2011 (efetividade, eficiência, satisfação, prevenção de
riscos e cobertura de contexto) só pode ser avaliado com o produto em uso real e, no BIOS,
depende da camada de front-end ainda não detalhada.

A **prioridade** de cada RNF segue os mesmos níveis do planejamento (`10-Gerenciamento-do-Projeto.md`,
seção "Prioridades"): *Essencial* quando a ausência do RNF inviabiliza ou invalida um requisito
Essencial do `05-Requisitos-Funcionais.md` (ex.: sessões revogáveis, criptografia de dados
sensíveis, respeito ao limite das fontes); *Importante* quando a ausência degrada a qualidade do
produto ou do processo sem inviabilizar um requisito Essencial (ex.: meta de tempo de resposta,
barreiras de qualidade do repositório); e *Desejável* quando a qualidade só sustenta um requisito
Desejável (ex.: honeypot).

Cada RNF indica a necessidade que o origina (história de usuário, em
`04-Atores-e-Histórias-dos-Usuários.md`) e traz a **verificação** prevista, conforme os métodos da ISO/IEC/IEEE 29148:2011:
*teste* (execução com entradas controladas), *inspeção* (exame de artefato ou configuração) ou
*análise* (avaliação de dados coletados, como relatórios).

## 6.1 Adequação Funcional

<p align="center"><b>[NFCO02] Rastreabilidade do score</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Adequação Funcional — Correção Funcional (*Functional Correctness*). |
| **Necessidade (história de usuário):** | HU17, HU21. |
| **Relacionados:** | UC14, UC15; [RF09], [RF10], [RF12]. |
| **Requisito:** | Quando o sistema calcular um score, ele deve vinculá-lo às evidências que o compuseram, de modo que toda contribuição seja derivável de uma evidência auditável, rastreável até o registro bruto mantido da fonte, e nenhum score seja gravado sem evidência rastreável. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Condição:** Quando o sistema calcular um score · **Sujeito:** ele [o sistema] · **Ação:** deve vinculá-lo · **Objeto:** às evidências que o compuseram · **Restrição:** de modo que toda contribuição seja derivável de uma evidência auditável, rastreável até o registro bruto mantido da fonte, e nenhum score seja gravado sem evidência rastreável |
| **Verificação:** | Teste. |


## 6.2 Eficiência de Desempenho

<p align="center"><b>[NFDE01] Cache de consultas</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Eficiência de Desempenho — Comportamento Temporal (*Time Behaviour*). |
| **Necessidade (história de usuário):** | HU28. |
| **Relacionados:** | UC06; [RF03]. |
| **Requisito:** | (a) Quando uma consulta solicitar um CNPJ cujo cadastro esteja guardado dentro da janela de validade da fonte que o respondeu, o sistema deve servir o cadastro guardado, sem nova chamada à fonte externa.<br>(b) Quando uma consulta solicitar o perfil completo de uma empresa cujas evidências e índices por ODS estejam guardados há até 5 (cinco) minutos, o sistema deve servi-los sem nova coleta, descartando-os quando o índice da empresa for recalculado. |
| **Decomposição (ISO/IEC/IEEE 29148):** | (a) **Condição:** Quando uma consulta solicitar um CNPJ cujo cadastro esteja guardado dentro da janela de validade da fonte que o respondeu · **Sujeito:** o sistema · **Ação:** deve servir · **Objeto:** o cadastro guardado · **Restrição:** sem nova chamada à fonte externa<br>(b) **Condição:** Quando uma consulta solicitar o perfil completo de uma empresa cujas evidências e índices por ODS estejam guardados há até 5 minutos · **Sujeito:** o sistema · **Ação:** deve servi-los · **Objeto:** as evidências e os índices guardados · **Restrição:** sem nova coleta, descartando-os quando o índice da empresa for recalculado |
| **Verificação:** | Teste. |


<p align="center"><b>[NFDE02] Tempo de resposta do perfil completo</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Eficiência de Desempenho — Comportamento Temporal (*Time Behaviour*). |
| **Necessidade (história de usuário):** | HU28. |
| **Relacionados:** | UC06, UC07; [RF03], [RF05], [RF07]. |
| **Requisito:** | Quando o perfil completo (cadastro, evidências e índices por ODS) estiver em cache, o sistema deve responder em até 2 (dois) segundos. Sem cache completo, a espera conjunta por cadastro, coleta e recálculo deve obedecer ao orçamento configurado em `COMPANY_PROFILE_REQUEST_TIMEOUT_MS` (padrão 15 segundos), devolvendo o retrato persistido com indicação de pendência quando houver cadastro local, ou indisponibilidade temporária quando o cadastro não puder ser obtido. Cada espera por um job da fila fica limitada por `QUEUE_WAIT_TIMEOUT_MS` (padrão 10 segundos), sempre dentro desse orçamento. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Condição:** Quando o perfil completo estiver em cache, ou quando depender de atualização · **Sujeito:** o sistema · **Ação:** deve responder · **Restrição:** em até 2 segundos no cache; sem cache, com espera conjunta limitada por `COMPANY_PROFILE_REQUEST_TIMEOUT_MS`, padrão 15 segundos, e resposta parcial ou indisponibilidade temporária ao esgotar o prazo; cada espera por job limitada por `QUEUE_WAIT_TIMEOUT_MS`, padrão 10 segundos |
| **Verificação:** | Teste (medição do tempo de resposta). |


## 6.3 Compatibilidade

<p align="center"><b>[NFPD01] Respeito ao limite de requisições das fontes</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Compatibilidade — Interoperabilidade (*Interoperability*). |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | UC06, UC14; [RF03], [RF07]. |
| **Requisito:** | (a) Quando o sistema consultar uma fonte externa, ele deve respeitar o limite de requisições dessa fonte registrado no catálogo de fontes, compartilhando um único job entre as consultas simultâneas, ou repetidas em até 60 segundos, do mesmo CNPJ.<br>(b) Quando um sistema de terceiros consultar o perfil pela API pública, o sistema deve responder sem aguardar a coleta nas fontes externas, publicando a atualização do que falta e informando o que está pendente. |
| **Decomposição (ISO/IEC/IEEE 29148):** | (a) **Condição:** Quando o sistema consultar uma fonte externa · **Sujeito:** ele [o sistema] · **Ação:** deve respeitar · **Objeto:** o limite de requisições dessa fonte registrado no catálogo de fontes · **Restrição:** compartilhando um único job entre as consultas simultâneas, ou repetidas em até 60 segundos, do mesmo CNPJ<br>(b) **Condição:** Quando um sistema de terceiros consultar o perfil pela API pública · **Sujeito:** o sistema · **Ação:** deve responder · **Objeto:** a consulta · **Restrição:** sem aguardar a coleta nas fontes externas, publicando a atualização do que falta e informando o que está pendente |
| **Verificação:** | Teste. |


## 6.4 Capacidade de Interação

Nenhum RNF formalizado nesta característica até o momento. A explicabilidade de resultado
(RF11, RF12) e a proibição de veredito absoluto (RIN [IND01]) endereçam parte do espírito
desta característica em nível funcional, mas nenhuma meta não-funcional de interação (ex.:
tempo de aprendizado, taxa de erro do usuário) foi definida. Pendência a ser tratada quando a
camada de front-end for detalhada.

## 6.5 Confiabilidade

<p align="center"><b>[NFCO01] Continuidade com dados parciais</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Confiabilidade — Tolerância a Falhas (*Fault Tolerance*). |
| **Necessidade (história de usuário):** | HU27. |
| **Relacionados:** | UC06, UC07; [RF07]. |
| **Requisito:** | Quando uma fonte externa estiver indisponível, o sistema deve concluir a consulta com as evidências das demais fontes, registrando `UNKNOWN` para a fonte afetada, ou preservando a resposta anterior dela (como `OUTDATED` após o vencimento), sem falhar a consulta inteira. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Condição:** Quando uma fonte externa estiver indisponível · **Sujeito:** o sistema · **Ação:** deve concluir · **Objeto:** a consulta · **Restrição:** com as evidências das demais fontes; registrando `UNKNOWN` para a fonte afetada, ou preservando a resposta anterior dela (como `OUTDATED` após o vencimento); sem falhar a consulta inteira |
| **Verificação:** | Teste. |


## 6.6 Segurança

<p align="center"><b>[NFSE01] Armazenamento de chaves de API com hash</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Segurança — Confidencialidade (*Confidentiality*). |
| **Necessidade (história de usuário):** | HU30. |
| **Relacionados:** | UC11; [RF16]. |
| **Requisito:** | (a) Quando o sistema gerar uma chave de API, ele deve armazenar somente o hash SHA-256 do seu valor completo, nunca o texto plano.<br>(b) Após a geração de uma chave de API, o sistema não deve reexibir o valor completo da chave. |
| **Decomposição (ISO/IEC/IEEE 29148):** | (a) **Condição:** Quando o sistema gerar uma chave de API · **Sujeito:** ele [o sistema] · **Ação:** deve armazenar · **Objeto:** somente o hash SHA-256 do seu valor completo · **Restrição:** nunca o texto plano<br>(b) **Condição:** Após a geração de uma chave de API · **Sujeito:** o sistema · **Ação:** não deve reexibir · **Objeto:** o valor completo da chave |
| **Verificação:** | Teste e inspeção do esquema de dados. |


<p align="center"><b>[NFSE02] Conformidade com a LGPD</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Segurança — Confidencialidade (*Confidentiality*). Formalmente uma exigência regulatória, sem sub-característica dedicada na norma — enquadrada aqui por ser o item mais próximo do modelo. |
| **Necessidade (história de usuário):** | HU08, HU29. |
| **Relacionados:** | UC05, UC06, UC14; [RF27]; [INL02]. |
| **Requisito:** | (a) Quando o sistema coletar, armazenar ou exibir dados de uma empresa, ele deve limitá-los a dados públicos da pessoa jurídica, sem dado pessoal de indivíduo.<br>(b) Quando o sistema guardar registros operacionais (tokens de e-mail, sessões, bloqueios de acesso, tentativas de login, acessos ao honeypot, envios de e-mail e requisições às fontes), ele deve eliminá-los ao fim do prazo de retenção de cada tipo, configurável por `DATA_RETENTION_*` (padrões de 7 a 180 dias). |
| **Decomposição (ISO/IEC/IEEE 29148):** | (a) **Condição:** Quando o sistema coletar, armazenar ou exibir dados de uma empresa · **Sujeito:** ele [o sistema] · **Ação:** deve limitá-los · **Objeto:** a dados públicos da pessoa jurídica · **Restrição:** sem dado pessoal de indivíduo<br>(b) **Condição:** Quando o sistema guardar registros operacionais · **Sujeito:** ele [o sistema] · **Ação:** deve eliminá-los · **Objeto:** os tokens de e-mail, as sessões, os bloqueios, as tentativas de login, os acessos ao honeypot, os envios de e-mail e as requisições às fontes · **Restrição:** ao fim do prazo de retenção de cada tipo, configurável por `DATA_RETENTION_*` |
| **Verificação:** | Inspeção (modelo de dados e respostas da API) e teste. |


<p align="center"><b>[NFSE03] Criptografia de dados sensíveis</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Segurança — Confidencialidade (*Confidentiality*). |
| **Necessidade (história de usuário):** | HU10. |
| **Relacionados:** | UC01, UC05; [RF01], [RF26]. |
| **Requisito:** | (a) Quando o sistema persistir o e-mail e o endereço de um usuário, ele deve cifrá-los com AES-256.<br>(b) Quando o sistema persistir a senha de um usuário, ele deve armazenar o hash bcrypt dela, cifrado em seguida com AES-256. |
| **Decomposição (ISO/IEC/IEEE 29148):** | (a) **Condição:** Quando o sistema persistir o e-mail e o endereço de um usuário · **Sujeito:** ele [o sistema] · **Ação:** deve cifrá-los · **Objeto:** [o e-mail e o endereço] · **Restrição:** com AES-256<br>(b) **Condição:** Quando o sistema persistir a senha de um usuário · **Sujeito:** ele [o sistema] · **Ação:** deve armazenar · **Objeto:** o hash bcrypt dela · **Restrição:** cifrado em seguida com AES-256 |
| **Verificação:** | Teste e inspeção do esquema de dados. |


<p align="center"><b>[NFSE04] Rate limiting em endpoints públicos</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Segurança — Resistência (*Resistance*). |
| **Necessidade (história de usuário):** | HU31. |
| **Relacionados:** | UC07, UC12; [RF17]. |
| **Requisito:** | (a) Quando um IP ou uma chave de API exceder o limite de requisições de um endpoint público, o sistema (ThrottlerModule do NestJS) deve recusar as requisições excedentes com HTTP 429, informando em `Retry-After` quando repeti-las.<br>(b) O sistema deve limitar a rota de evidências a 20 consultas por minuto por IP.<br>(c) O sistema deve contar por chave de API o limite de requisições da consulta de perfil via API.<br>(d) O sistema deve limitar a consulta de perfil completo do painel a 60 consultas por minuto por IP, por padrão.<br>(e) O sistema deve limitar a exportação da lista a 10 exportações por minuto por IP, por padrão.<br>(f) O sistema deve limitar a pesquisa de empresas a 60 pesquisas por minuto por IP, por padrão.<br>(g) O sistema deve ler de configuração o limite de cada rota dos itens (b), (d), (e) e (f), sem valor fixo no código.<br>(h) O sistema deve recusar com HTTP 422 a exportação da lista que passe de 5000 empresas por arquivo, por padrão (configurável até 20000). |
| **Decomposição (ISO/IEC/IEEE 29148):** | (a) **Condição:** Quando um IP ou uma chave de API exceder o limite de requisições de um endpoint público · **Sujeito:** o sistema · **Ação:** deve recusar · **Objeto:** as requisições excedentes · **Restrição:** com HTTP 429, informando em `Retry-After` quando repeti-las<br>(b) **Sujeito:** o sistema · **Ação:** deve limitar · **Objeto:** a rota de evidências · **Restrição:** a 20 consultas por minuto por IP<br>(c) **Sujeito:** o sistema · **Ação:** deve contar · **Objeto:** o limite de requisições da consulta de perfil via API · **Restrição:** por chave de API<br>(d) **Sujeito:** o sistema · **Ação:** deve limitar · **Objeto:** a consulta de perfil completo do painel · **Restrição:** a 60 consultas por minuto por IP, por padrão<br>(e) **Sujeito:** o sistema · **Ação:** deve limitar · **Objeto:** a exportação da lista · **Restrição:** a 10 exportações por minuto por IP, por padrão<br>(f) **Sujeito:** o sistema · **Ação:** deve limitar · **Objeto:** a pesquisa de empresas · **Restrição:** a 60 pesquisas por minuto por IP, por padrão<br>(g) **Sujeito:** o sistema · **Ação:** deve ler · **Objeto:** o limite de cada rota dos itens (b), (d), (e) e (f) · **Restrição:** de configuração, sem valor fixo no código<br>(h) **Sujeito:** o sistema · **Ação:** deve recusar com HTTP 422 · **Objeto:** a exportação da lista · **Restrição:** que passe de 5000 empresas por arquivo, por padrão (configurável até 20000) |
| **Verificação:** | Teste. |


<p align="center"><b>[NFSE05] Sessões revogáveis com rotação de credenciais</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Segurança — Autenticidade (*Authenticity*). |
| **Necessidade (história de usuário):** | HU04, HU05, HU35. |
| **Relacionados:** | UC02, UC03, UC17; [RF02], [RF23], [RF32]. |
| **Requisito:** | (a) Quando o sistema emitir um access token JWT, ele deve fixar sua validade em 15 minutos por padrão (configurável, no máximo 60).<br>(b) Quando o sistema receber uma requisição com access token, ele deve validá-lo contra a sessão, de modo que a revogação (logout, troca de senha) tenha efeito imediato.<br>(c) Quando o sistema emitir um refresh token, ele deve gerá-lo opaco e de uso único, com validade de 7 dias por padrão (configurável).<br>(d) Quando o sistema emitir um refresh token, ele deve armazenar somente o hash do refresh token.<br>(e) Quando o sistema renovar uma sessão, ele deve rotacionar o refresh token, emitindo um novo.<br>(f) Quando um refresh token já rotacionado for reapresentado, o sistema deve revogar as sessões do usuário. |
| **Decomposição (ISO/IEC/IEEE 29148):** | (a) **Condição:** Quando o sistema emitir um access token JWT · **Sujeito:** ele [o sistema] · **Ação:** deve fixar · **Objeto:** sua validade · **Restrição:** em 15 minutos por padrão (configurável, no máximo 60)<br>(b) **Condição:** Quando o sistema receber uma requisição com access token · **Sujeito:** ele [o sistema] · **Ação:** deve validá-lo · **Objeto:** [o access token] · **Restrição:** contra a sessão, de modo que a revogação (logout, troca de senha) tenha efeito imediato<br>(c) **Condição:** Quando o sistema emitir um refresh token · **Sujeito:** ele [o sistema] · **Ação:** deve gerá-lo · **Objeto:** [o refresh token] opaco e de uso único · **Restrição:** com validade de 7 dias por padrão (configurável)<br>(d) **Condição:** Quando o sistema emitir um refresh token · **Sujeito:** ele [o sistema] · **Ação:** deve armazenar · **Objeto:** somente o hash do refresh token<br>(e) **Condição:** Quando o sistema renovar uma sessão · **Sujeito:** ele [o sistema] · **Ação:** deve rotacionar · **Objeto:** o refresh token · **Restrição:** emitindo um novo<br>(f) **Condição:** Quando um refresh token já rotacionado for reapresentado · **Sujeito:** o sistema · **Ação:** deve revogar · **Objeto:** as sessões do usuário |
| **Verificação:** | Teste. |


<p align="center"><b>[NFSE06] Bloqueio de login</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Segurança — Resistência (*Resistance*). |
| **Necessidade (história de usuário):** | HU09. |
| **Relacionados:** | UC02; [RF19], [RF20]. |
| **Requisito:** | Quando um mesmo IP ou usuário acumular 5 (cinco) tentativas consecutivas de login malsucedidas, o sistema deve bloqueá-lo por 15 (quinze) minutos. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Condição:** Quando um mesmo IP ou usuário acumular 5 (cinco) tentativas consecutivas de login malsucedidas · **Sujeito:** o sistema · **Ação:** deve bloqueá-lo · **Objeto:** [o IP ou usuário] · **Restrição:** por 15 (quinze) minutos |
| **Verificação:** | Teste. |


<p align="center"><b>[NFSE07] Honeypot com baixa disponibilidade</b></p>

| Prioridade: | ☐ Essencial / ☐ Importante / ☑ Desejável |
| :----------- | :----------- |
| **Característica:** | Segurança — Resistência (*Resistance*). |
| **Necessidade (história de usuário):** | HU34. |
| **Relacionados:** | [RF21]. |
| **Requisito:** | O sistema deve manter os endpoints isca isolados dos endpoints reais, de modo que a indisponibilidade dos endpoints isca, mantida de propósito, não reduza a disponibilidade dos endpoints reais. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Sujeito:** o sistema · **Ação:** deve manter · **Objeto:** os endpoints isca isolados dos endpoints reais · **Restrição:** de modo que a indisponibilidade dos endpoints isca, mantida de propósito, não reduza a disponibilidade dos endpoints reais |
| **Verificação:** | Teste. |


<p align="center"><b>[NFSE08] Monitoramento de vulnerabilidades de dependências</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Segurança — Resistência (*Resistance*). Detalhe operacional na Etapa de Gerência de Configuração. |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | O projeto deve monitorar as vulnerabilidades de suas dependências por meio do GitHub Dependabot. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Sujeito:** o projeto · **Ação:** deve monitorar · **Objeto:** as vulnerabilidades de suas dependências · **Restrição:** por meio do GitHub Dependabot |
| **Verificação:** | Inspeção da configuração do repositório. |


## 6.7 Manutenibilidade

<p align="center"><b>[NFPD02] Isolamento por adapter</b></p>

| Prioridade: | ☑ Essencial / ☐ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Manutenibilidade — Modularidade (*Modularity*). |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | UC14; [RF07]; `14-Projeto-e-Arquitetura.md`. |
| **Requisito:** | O sistema deve isolar cada fonte externa em um módulo próprio (adapter), acessando-a exclusivamente por ele. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Sujeito:** o sistema · **Ação:** deve isolar · **Objeto:** cada fonte externa · **Restrição:** em um módulo próprio (adapter); acessando-a exclusivamente por ele |
| **Verificação:** | Inspeção da estrutura de módulos. |


<p align="center"><b>[NFPD03] Cobertura mínima de testes</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Manutenibilidade — Testabilidade (*Testability*). |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | O projeto deve manter cobertura mínima de 75% (setenta e cinco por cento) entre testes unitários, de integração, de componente, e2e e funcionais, no back-end e no front-end. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Sujeito:** o projeto · **Ação:** deve manter · **Objeto:** cobertura mínima de 75% (setenta e cinco por cento) · **Restrição:** entre testes unitários, de integração, de componente, e2e e funcionais, no back-end e no front-end |
| **Verificação:** | Análise do relatório de cobertura. |


<p align="center"><b>[NFSE09] Barreira de qualidade pré-commit</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Manutenibilidade — Modificabilidade (*Modifiability*). Detalhe operacional na Etapa de Gerência de Configuração. |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | Antes de cada commit, o projeto deve executar a barreira de qualidade configurada no Husky. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Condição:** Antes de cada commit · **Sujeito:** o projeto · **Ação:** deve executar · **Objeto:** a barreira de qualidade configurada no Husky |
| **Verificação:** | Inspeção da configuração do repositório. |


<p align="center"><b>[NFSE10] Análise estática de código</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Manutenibilidade — Analisabilidade (*Analysability*). Detalhe operacional na Etapa de Gerência de Configuração. |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | O projeto deve submeter o código-fonte à análise estática do ESLint. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Sujeito:** o projeto · **Ação:** deve submeter · **Objeto:** o código-fonte · **Restrição:** à análise estática do ESLint |
| **Verificação:** | Inspeção da configuração do repositório. |


<p align="center"><b>[NFSE11] Proteção das branches principais</b></p>

| Prioridade: | ☐ Essencial / ☑ Importante / ☐ Desejável |
| :----------- | :----------- |
| **Característica:** | Manutenibilidade — Modificabilidade (*Modifiability*). Detalhe operacional na Etapa de Gerência de Configuração. |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | O projeto deve manter protegidas as branches `main` e `pre-prod`. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Sujeito:** o projeto · **Ação:** deve manter protegidas · **Objeto:** as branches `main` e `pre-prod` |
| **Verificação:** | Inspeção da configuração do repositório. |


## 6.8 Flexibilidade

Nenhum RNF dedicado a esta característica. O isolamento por adapter ([NFPD02], seção 6.7)
também sustenta Adaptability — novas fontes de dado podem ser adicionadas sem alterar a
lógica central —, mas está formalizado sob Manutenibilidade por ser, em primeiro lugar, uma
decisão de baixo acoplamento entre módulos internos, não de adaptação a ambiente externo
variável.

## 6.9 Segurança Operacional (Safety)

Não aplicável ao BIOS. A característica Safety da ISO/IEC 25010:2023 trata de risco
operacional/físico (ex.: sistemas embarcados, controle industrial, dispositivos médicos) —
fora do escopo de uma plataforma de consulta e agregação de dados via API, sem atuação sobre
processo físico ou operacional de terceiros. Categoria mantida na estrutura do documento por
completude frente à norma, sem RNF associado.
