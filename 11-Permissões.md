# 11. Permissões

## 11.1 Cargos e Permissões Relacionadas

O BIOS possui um único tipo de conta, diferenciado pelo enum de papel (`USER` ou `ADMIN` —
ver Quadro 2, `04-Atores-e-Histórias-dos-Usuários.md`). Além dos papéis, há dois contextos de
acesso sem sessão: o **Visitante**, que ainda não está autenticado, e a **Chave de API**,
usada por sistemas de terceiros. A chave de API não é um papel: ela age em nome do usuário que
a gerou, mas restrita aos endpoints da API pública.

De acordo com a regra de negócio adotada, algumas permissões podem estar incluídas em outras.
Isso significa que, para que uma permissão seja totalmente revogada, deve-se retirar também as
permissões nas quais ela está incluída.

| Permissões | Incluso em | Visitante | Usuário (`USER`) | Administrador (`ADMIN`) | Chave de API |
|---|---|---|---|---|---|
| `[P01]` Cadastrar Conta | | ✅ | | | |
| `[P02]` Efetuar Login e Recuperar Senha | | ✅ | | | |
| `[P03]` Manter Própria Conta e Sessão | | | ✅ | ✅ | |
| `[P04]` Consultar Empresa | | | ✅ | ✅ | |
| `[P05]` Visualizar Perfil Ambiental | `[P04]` | | ✅ | ✅ | |
| `[P06]` Pesquisar Empresas | | | ✅ | ✅ | |
| `[P07]` Exportar Lista XLSX | `[P06]` | | ✅ | ✅ | |
| `[P08]` Consultar Evolução Setorial | | | ✅ | ✅ | |
| `[P09]` Manter Chaves de API Próprias | `[P03]` | | ✅ | ✅ | |
| `[P10]` Consultar Perfil via API | `[P04][P05]` | | | | ✅ |
| `[P11]` Integrar com CRM (bônus) | `[P10]` | | | | ✅ |
| `[P12]` Manter Blacklist de Domínios | | | | ✅ | |
| `[P13]` Renovar Sessão | | | ✅ | ✅ | |

## 11.2 Correspondência com Casos de Uso

| Permissão | Caso(s) de uso (`08-Artefatos-de-Análise.md`) | Requisitos |
|---|---|---|
| `[P01]` | UC01 | [RF01], [RF22], [RF28] |
| `[P02]` | UC02, UC04 (recuperação) | [RF02], [RF19], [RF20], [RF25] |
| `[P03]` | UC03, UC04 (alteração), UC05 | [RF23], [RF24], [RF26], [RF27] |
| `[P04]` | UC06 | [RF03], [RF04], [RF05], [RF06] |
| `[P05]` | UC07 | [RF07]–[RF12] |
| `[P06]` | UC08 | [RF13], [RF14] |
| `[P07]` | UC09 | [RF15] |
| `[P08]` | UC10 | [RF30] |
| `[P09]` | UC11 | [RF16] |
| `[P10]` | UC12 | [RF17] |
| `[P11]` | UC16 | [RF31] |
| `[P12]` | UC13 | [RF29] |
| `[P13]` | UC17 | [RF32] |

## 11.3 Regras Complementares

- **Sem acesso anônimo a dados de empresa.** Toda consulta exige sessão ou chave de API
  válida; o Visitante acessa apenas cadastro, login e recuperação de senha.
- **Escopo próprio.** `[P03]` e `[P09]` operam somente sobre a própria conta e as próprias
  chaves; nenhum papel, inclusive `ADMIN`, lê ou altera dados de conta ou chaves de outro
  usuário.
- **Administrador não é superusuário de dados.** O papel `ADMIN` acrescenta apenas `[P12]`.
  A promoção de um usuário a `ADMIN` é feita por seed/script de banco, sem interface no MVP.
- **Chave de API sem acesso à conta.** Uma chave não autentica no front-end nem acessa
  `[P03]`, `[P09]` ou `[P12]`; se a conta do titular for excluída, todas as suas chaves são
  revogadas (UC05).
- **Renovação de sessão pelo refresh token.** `[P13]` autentica pelo refresh token, e não
  pelo access token (que, no momento da renovação, normalmente já expirou).
- **Processos automáticos fora da matriz.** Os casos de uso do ator Sistema (UC14, UC15) e os
  mecanismos de defesa (bloqueio de login, honeypot) são executados pelo próprio BIOS e não
  são concedidos a nenhum papel.
