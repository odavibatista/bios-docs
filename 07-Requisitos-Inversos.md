# 07. Requisitos Inversos (RIN)

## 7.1 Dados

| [IND01] Não afirmar veredito absoluto de sustentabilidade |
| ----------- |
| **Objetivo:** O sistema não deve afirmar que uma empresa "é" ou "não é" sustentável, apenas reportar evidências e índices derivados delas. |

| [IND02] Não converter ausência de dado em evidência negativa |
| ----------- |
| **Objetivo:** O sistema não deve tratar "não encontramos informação" (UNKNOWN) como equivalente a "a empresa não pratica" (NOT_FOUND confirmado). |

## 7.2 Privacidade e Legal

| [INL01] Não inferir conteúdo de registro sigiloso |
| ----------- |
| **Objetivo:** O sistema não deve inferir ou supor o conteúdo de um processo/registro identificado publicamente, mas cujo teor não esteja disponível na fonte de origem. |

| [INL02] Não coletar dado pessoal de indivíduo vinculado à empresa |
| ----------- |
| **Objetivo:** O sistema não deve buscar, armazenar ou exibir contato, perfil de rede social ou qualquer dado pessoal de sócio, administrador ou colaborador da empresa consultada — apenas dados públicos da pessoa jurídica. |
