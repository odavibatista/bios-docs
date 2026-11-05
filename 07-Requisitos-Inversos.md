# 07. Requisitos Inversos (RIN)

Os requisitos inversos descrevem o que o sistema **não** deve fazer. Seguem a sintaxe
ISO/IEC/IEEE 29148:2011 dos demais requisitos ([Condição], [Sujeito], [Ação], [Objeto] e
[Restrição]; ver `05-Requisitos-Funcionais.md`), com a forma verbal negativa "não deve" no lugar
de "deve". Como a ação proibida precisa ser verificável, cada RIN nomeia o comportamento vedado
e, quando há alternativa permitida, a indica na mesma declaração. Cada RIN traz a decomposição
nos cinco elementos, como nos requisitos funcionais.

## 7.1 Dados

<p align="center"><b>[IND01] Não afirmar veredito absoluto de sustentabilidade</b></p>

| Relacionados: | UC07, UC18; [RF10], [RF12], [RF34]. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU18, HU22. |
| **Requisito:** | Quando o sistema exibir o resultado da avaliação de uma empresa, ele não deve afirmar que a empresa "é" ou "não é" sustentável, limitando-se a reportar as evidências e os índices derivados delas. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Condição:** Quando o sistema exibir o resultado da avaliação de uma empresa · **Sujeito:** ele [o sistema] · **Ação:** não deve afirmar · **Objeto:** que a empresa "é" ou "não é" sustentável · **Restrição:** limitando-se a reportar as evidências e os índices derivados delas |


<p align="center"><b>[IND02] Não converter ausência de dado em evidência negativa</b></p>

| Relacionados: | UC07, UC15, UC18; [RF08], [RF11], [RF33], [RF34]. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU16, HU20. |
| **Requisito:** | (a) Quando uma fonte não retornar informação sobre uma empresa (`UNKNOWN`), o sistema não deve tratar essa ausência como equivalente a "a empresa não pratica" (`NOT_FOUND` confirmado).<br>(b) Quando uma fonte não retornar informação sobre uma empresa (`UNKNOWN`), o sistema não deve pontuar essa ausência como evidência negativa, reduzindo apenas o nível de confiança. |
| **Decomposição (ISO/IEC/IEEE 29148):** | (a) **Condição:** Quando uma fonte não retornar informação sobre uma empresa (`UNKNOWN`) · **Sujeito:** o sistema · **Ação:** não deve tratar · **Objeto:** essa ausência como equivalente a "a empresa não pratica" (`NOT_FOUND` confirmado)<br>(b) **Condição:** Quando uma fonte não retornar informação sobre uma empresa (`UNKNOWN`) · **Sujeito:** o sistema · **Ação:** não deve pontuar · **Objeto:** essa ausência como evidência negativa · **Restrição:** reduzindo apenas o nível de confiança |


<p align="center"><b>[IND03] Não apresentar movimentação como ranking nem como mudança de conduta</b></p>

| Relacionados: | UC07, UC15, UC18, UC19; [RF33], [RF34], [RF36], [RF37]; [IA05]. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU22, HU36, HU38. |
| **Requisito:** | (a) Quando o sistema exibir a movimentação do índice de uma empresa na lista da tela principal, ele não deve exibir o valor anterior, o tamanho da variação nem a série de índices da empresa, limitando-se ao sentido, à causa e à data.<br>(b) Quando o sistema listar empresas com movimentação, ele não deve ordená-las pelo sentido nem pelo tamanho da mudança, limitando-se à ordem cronológica.<br>(c) Quando o sistema exibir a movimentação do índice de uma empresa, ele não deve apresentá-la como melhora ou piora da conduta ambiental da empresa, limitando-se a informar a evidência que mudou o índice.<br>(d) Quando o sistema exibir a evolução do índice no perfil de uma empresa ([RF36]), ele não deve exibir o tamanho da variação nem comparar a empresa com outras, limitando-se ao índice em cada data e à evidência que o causou. |
| **Decomposição (ISO/IEC/IEEE 29148):** | (a) **Condição:** Quando o sistema exibir a movimentação do índice de uma empresa na lista da tela principal · **Sujeito:** ele [o sistema] · **Ação:** não deve exibir · **Objeto:** o valor anterior, o tamanho da variação nem a série de índices da empresa · **Restrição:** limitando-se ao sentido, à causa e à data<br>(b) **Condição:** Quando o sistema listar empresas com movimentação · **Sujeito:** ele [o sistema] · **Ação:** não deve ordená-las · **Objeto:** pelo sentido nem pelo tamanho da mudança · **Restrição:** limitando-se à ordem cronológica<br>(c) **Condição:** Quando o sistema exibir a movimentação do índice de uma empresa · **Sujeito:** ele [o sistema] · **Ação:** não deve apresentá-la · **Objeto:** como melhora ou piora da conduta ambiental da empresa · **Restrição:** limitando-se a informar a evidência que mudou o índice<br>(d) **Condição:** Quando o sistema exibir a evolução do índice no perfil de uma empresa · **Sujeito:** ele [o sistema] · **Ação:** não deve exibir · **Objeto:** o tamanho da variação nem a comparação da empresa com outras · **Restrição:** limitando-se ao índice em cada data e à evidência que o causou |


## 7.2 Privacidade e Legal

<p align="center"><b>[INL01] Não inferir conteúdo de registro sigiloso</b></p>

| Relacionados: | UC07; [RF07], [RF09]. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | Não há (requisito derivado de decisão de projeto). |
| **Requisito:** | Quando um processo ou registro for identificado publicamente, mas seu teor não estiver disponível na fonte de origem, o sistema não deve inferir nem supor o conteúdo desse registro. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Condição:** Quando um processo ou registro for identificado publicamente, mas seu teor não estiver disponível na fonte de origem · **Sujeito:** o sistema · **Ação:** não deve inferir nem supor · **Objeto:** o conteúdo desse registro |


<p align="center"><b>[INL02] Não coletar dado pessoal de indivíduo vinculado à empresa</b></p>

| Relacionados: | UC06, UC07, UC09, UC14, UC16, UC18; [RF05], [RF15], [RF31], [RF34]; [NFSE02]. |
| :----------- | :----------- |
| **Necessidade (história de usuário):** | HU29, HU32. |
| **Requisito:** | O sistema não deve buscar, armazenar nem exibir contato, perfil de rede social ou outro dado pessoal de sócio, administrador ou colaborador da empresa consultada, restringindo-se aos dados públicos vinculados ao CNPJ e às evidências da empresa. A razão social pública de empresário individual pode conter o nome do titular; sequências de CPF nela são mascaradas. |
| **Decomposição (ISO/IEC/IEEE 29148):** | **Sujeito:** o sistema · **Ação:** não deve buscar, armazenar nem exibir · **Objeto:** contato, perfil de rede social ou outro dado pessoal de sócio, administrador ou colaborador da empresa consultada · **Restrição:** dados públicos vinculados ao CNPJ e evidências da empresa; razão social pública de empresário individual permitida, com sequências de CPF mascaradas |
