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

Cada RNF indica a necessidade que o origina (história de usuário, em
`04-Atores-e-Histórias-dos-Usuários.md`) e traz a **verificação** prevista, conforme os métodos da ISO/IEC/IEEE 29148:2011:
*teste* (execução com entradas controladas), *inspeção* (exame de artefato ou configuração) ou
*análise* (avaliação de dados coletados, como relatórios).

## 6.1 Adequação Funcional

<p align="center"><b>[NFCO02] Rastreabilidade do score</b></p>

| Característica: | Adequação Funcional — Correção Funcional (*Functional Correctness*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU17, HU21. |
| **Relacionados:** | UC14, UC15; [RF09], [RF10], [RF12]. |
| **Requisito:** | Quando o sistema calcular um score, ele deve vinculá-lo às evidências que o compuseram, de modo que toda contribuição seja derivável de uma evidência auditável, rastreável até o registro bruto mantido da fonte, e nenhum score seja gravado sem evidência rastreável. |
| **Verificação:** | Teste. |


## 6.2 Eficiência de Desempenho

<p align="center"><b>[NFDE01] Cache de consultas</b></p>

| Característica: | Eficiência de Desempenho — Comportamento Temporal (*Time Behaviour*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU28. |
| **Relacionados:** | UC06; [RF03]. |
| **Requisito:** | Quando uma consulta solicitar um CNPJ cujo cadastro esteja guardado dentro da janela de validade da fonte que o respondeu, o sistema deve servir o cadastro guardado, sem nova chamada à fonte externa. |
| **Verificação:** | Teste. |


<p align="center"><b>[NFDE02] Tempo de resposta do perfil em cache</b></p>

| Característica: | Eficiência de Desempenho — Comportamento Temporal (*Time Behaviour*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU28. |
| **Relacionados:** | UC06; [RF03], [RF05]. |
| **Requisito:** | Quando a consulta de perfil de uma empresa for atendida a partir do cache, o sistema deve responder em até 2 (dois) segundos. |
| **Verificação:** | Teste (medição do tempo de resposta). |


## 6.3 Compatibilidade

<p align="center"><b>[NFPD01] Respeito ao limite de requisições das fontes</b></p>

| Característica: | Compatibilidade — Interoperabilidade (*Interoperability*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | UC06, UC14; [RF03], [RF07]. |
| **Requisito:** | Quando o sistema consultar uma fonte externa, ele deve respeitar o limite de requisições dessa fonte registrado no catálogo de fontes, compartilhando um único job entre as consultas simultâneas, ou repetidas em até 60 segundos, do mesmo CNPJ. |
| **Verificação:** | Teste. |


## 6.4 Capacidade de Interação

Nenhum RNF formalizado nesta característica até o momento. A explicabilidade de resultado
(RF11, RF12) e a proibição de veredito absoluto (RIN [IND01]) endereçam parte do espírito
desta característica em nível funcional, mas nenhuma meta não-funcional de interação (ex.:
tempo de aprendizado, taxa de erro do usuário) foi definida. Pendência a ser tratada quando a
camada de front-end for detalhada.

## 6.5 Confiabilidade

<p align="center"><b>[NFCO01] Continuidade com dados parciais</b></p>

| Característica: | Confiabilidade — Tolerância a Falhas (*Fault Tolerance*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU27. |
| **Relacionados:** | UC06, UC07; [RF07]. |
| **Requisito:** | Quando uma fonte externa estiver indisponível, o sistema deve concluir a consulta com as evidências das demais fontes, registrando `UNKNOWN` para a fonte afetada, ou preservando a resposta anterior dela (como `OUTDATED` após o vencimento), sem falhar a consulta inteira. |
| **Verificação:** | Teste. |


## 6.6 Segurança

<p align="center"><b>[NFSE01] Armazenamento de chaves de API com hash</b></p>

| Característica: | Segurança — Confidencialidade (*Confidentiality*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU30. |
| **Relacionados:** | UC11; [RF16]. |
| **Requisito:** | Quando o sistema gerar uma chave de API, ele deve armazenar somente o hash SHA-256 do seu valor completo, nunca o texto plano, e não deve reexibir o valor completo após a geração. |
| **Verificação:** | Teste e inspeção do esquema de dados. |


<p align="center"><b>[NFSE02] Conformidade com a LGPD</b></p>

| Característica: | Segurança — Confidencialidade (*Confidentiality*). Formalmente uma exigência regulatória, sem sub-característica dedicada na norma — enquadrada aqui por ser o item mais próximo do modelo. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU08, HU29. |
| **Relacionados:** | UC05, UC06, UC14; [RF27]; [INL02]. |
| **Requisito:** | Quando o sistema coletar, armazenar ou exibir dados de uma empresa, ele deve limitá-los a dados públicos da pessoa jurídica, sem dado pessoal de indivíduo. |
| **Verificação:** | Inspeção (modelo de dados e respostas da API) e teste. |


<p align="center"><b>[NFSE03] Criptografia de dados sensíveis</b></p>

| Característica: | Segurança — Confidencialidade (*Confidentiality*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU10. |
| **Relacionados:** | UC01, UC05; [RF01], [RF26]. |
| **Requisito:** | Quando o sistema persistir o e-mail e o endereço de um usuário, ele deve cifrá-los com AES-256 e, quando persistir a senha, deve armazenar o hash bcrypt dela, cifrado em seguida com AES-256. |
| **Verificação:** | Teste e inspeção do esquema de dados. |


<p align="center"><b>[NFSE04] Rate limiting em endpoints públicos</b></p>

| Característica: | Segurança — Resistência (*Resistance*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU31. |
| **Relacionados:** | UC07, UC12; [RF17]. |
| **Requisito:** | Quando um IP ou uma chave de API exceder o limite de requisições de um endpoint público, o sistema (ThrottlerModule do NestJS) deve recusar as requisições excedentes com HTTP 429, informando em `Retry-After` quando repeti-las. O limite é contado por chave de API na consulta de perfil via API e é de 20 consultas por minuto por IP na rota de evidências. |
| **Verificação:** | Teste. |


<p align="center"><b>[NFSE05] Sessões revogáveis com rotação de credenciais</b></p>

| Característica: | Segurança — Autenticidade (*Authenticity*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU04, HU05, HU35. |
| **Relacionados:** | UC02, UC03, UC17; [RF02], [RF23], [RF32]. |
| **Requisito:** | (a) Quando o sistema emitir um access token JWT, ele deve fixar sua validade em 15 minutos por padrão (configurável, no máximo 60) e validá-lo contra a sessão a cada requisição, de modo que a revogação (logout, troca de senha) tenha efeito imediato.<br>(b) Quando o sistema emitir um refresh token, ele deve gerá-lo opaco e de uso único, com validade de 7 dias por padrão (configurável), armazenar somente o seu hash e rotacioná-lo a cada renovação.<br>(c) Quando um refresh token já rotacionado for reapresentado, o sistema deve detectar o reuso e revogar as sessões do usuário. |
| **Verificação:** | Teste. |


<p align="center"><b>[NFSE06] Bloqueio de login</b></p>

| Característica: | Segurança — Resistência (*Resistance*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU09. |
| **Relacionados:** | UC02; [RF19], [RF20]. |
| **Requisito:** | Quando um mesmo IP ou usuário acumular 5 (cinco) tentativas consecutivas de login malsucedidas, o sistema deve bloqueá-lo por 15 (quinze) minutos. |
| **Verificação:** | Teste. |


<p align="center"><b>[NFSE07] Honeypot com baixa disponibilidade</b></p>

| Característica: | Segurança — Resistência (*Resistance*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU34. |
| **Relacionados:** | [RF21]. |
| **Requisito:** | O sistema deve manter os endpoints isca isolados dos endpoints reais, de modo que a indisponibilidade dos endpoints isca, mantida de propósito, não reduza a disponibilidade dos endpoints reais. |
| **Verificação:** | Teste. |


<p align="center"><b>[NFSE08] Monitoramento de vulnerabilidades de dependências</b></p>

| Característica: | Segurança — Resistência (*Resistance*). Detalhe operacional na Etapa de Gerência de Configuração. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | O projeto deve monitorar as vulnerabilidades de suas dependências por meio do GitHub Dependabot. |
| **Verificação:** | Inspeção da configuração do repositório. |


## 6.7 Manutenibilidade

<p align="center"><b>[NFPD02] Isolamento por adapter</b></p>

| Característica: | Manutenibilidade — Modularidade (*Modularity*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | UC14; [RF07]; `14-Projeto-e-Arquitetura.md`. |
| **Requisito:** | O sistema deve isolar cada fonte externa em um módulo próprio (adapter), acessando-a exclusivamente por ele. |
| **Verificação:** | Inspeção da estrutura de módulos. |


<p align="center"><b>[NFPD03] Cobertura mínima de testes</b></p>

| Característica: | Manutenibilidade — Testabilidade (*Testability*). |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | O projeto deve manter cobertura mínima de 75% (setenta e cinco por cento) entre testes unitários, de integração, de componente, e2e e funcionais, no back-end e no front-end. |
| **Verificação:** | Análise do relatório de cobertura. |


<p align="center"><b>[NFSE09] Barreira de qualidade pré-commit</b></p>

| Característica: | Manutenibilidade — Modificabilidade (*Modifiability*). Detalhe operacional na Etapa de Gerência de Configuração. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | Antes de cada commit, o projeto deve executar a barreira de qualidade configurada no Husky. |
| **Verificação:** | Inspeção da configuração do repositório. |


<p align="center"><b>[NFSE10] Análise estática de código</b></p>

| Característica: | Manutenibilidade — Analisabilidade (*Analysability*). Detalhe operacional na Etapa de Gerência de Configuração. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | O projeto deve submeter o código-fonte à análise estática do ESLint. |
| **Verificação:** | Inspeção da configuração do repositório. |


<p align="center"><b>[NFSE11] Proteção das branches principais</b></p>

| Característica: | Manutenibilidade — Modificabilidade (*Modifiability*). Detalhe operacional na Etapa de Gerência de Configuração. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Relacionados:** | `10-Gerenciamento-do-Projeto.md`; `13-Ficha-Técnica.md`. |
| **Requisito:** | O projeto deve manter protegidas as branches `main` e `pre-prod`. |
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
