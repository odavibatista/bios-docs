# 06. Requisitos Não-Funcionais (RNF)

## 6.2 Confiabilidade

[NFCO01] Continuidade com dados parciais — degradação graciosa se uma fonte externa estiver
indisponível, sem falhar a consulta inteira.

[NFCO02] Rastreabilidade — todo score deve ser derivável de evidências auditáveis.

## 6.3 Desempenho

[NFDE01] Cache de consultas — evitar nova chamada à fonte externa para CNPJ já consultado
dentro da validade.

[NFDE02] Tempo de resposta — consulta de perfil em cache deve responder em até 2s.

## 6.4 Segurança

[NFSE01] Chaves de API armazenadas com hash, nunca em texto plano.

[NFSE02] Conformidade com LGPD — apenas dados públicos de pessoa jurídica, sem dado pessoal
de indivíduo.

[NFSE03] Criptografia AES-256 de dados sensíveis (e-mail, senha, endereço de usuário).

[NFSE04] Rate limiting em endpoints públicos (ThrottlerModule do NestJS).

[NFSE05] Sessões JWT revogáveis (logout forçado, troca de senha).

[NFSE06] Bloqueio de login — 5 tentativas malsucedidas resultam em bloqueio de 15 minutos por
IP e/ou usuário.

[NFSE07] Honeypot com baixa disponibilidade — endpoints isca isolados, sem impacto na
disponibilidade dos endpoints reais.

[NFSE08] Monitoramento de vulnerabilidades de dependências via GitHub Dependabot (detalhe
operacional na Etapa de Gerência de Configuração).

[NFSE09] Barreira de qualidade pré-commit via Husky (detalhe operacional na Etapa de
Gerência de Configuração).

[NFSE10] Análise estática de código via ESLint (detalhe operacional na Etapa de Gerência de
Configuração).

[NFSE11] Proteção das branches `main` e `pre-prod` (detalhe operacional na Etapa de Gerência
de Configuração).

## 6.6 Padrões

[NFPD01] Respeitar o rate limit de cada fonte externa consumida.

[NFPD02] Isolamento por adapter — cada fonte externa isolada em módulo próprio.

[NFPD03] Cobertura mínima de 75% entre testes unitários, de integração, de componente, e2e e
funcionais (back-end e front-end).
