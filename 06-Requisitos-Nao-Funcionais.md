# 06. Requisitos Não-Funcionais (RNF)

As categorias abaixo seguem as nove características de qualidade da ISO/IEC 25010:2023
(SQuaRE — ver `12-Referências.md`), em substituição ao agrupamento ad-hoc usado em versões
anteriores deste documento (que incluía uma categoria "Padrões" sem correspondência na
norma). Os identificadores originais de cada RNF foram mantidos mesmo após a reclassificação,
para preservar rastreabilidade com as referências já feitas a eles em `13-Ficha-Técnica.md` e
`14-Projeto-e-Arquitetura.md`.

## 6.1 Adequação Funcional

[NFCO02] Rastreabilidade (Functional Correctness) — todo score deve ser derivável de
evidências auditáveis.

## 6.2 Eficiência de Desempenho

[NFDE01] Cache de consultas (Time Behaviour) — evitar nova chamada à fonte externa para CNPJ
já consultado dentro da validade.

[NFDE02] Tempo de resposta (Time Behaviour) — consulta de perfil em cache deve responder em
até 2s.

## 6.3 Compatibilidade

[NFPD01] Respeitar o rate limit de cada fonte externa consumida (Interoperability).

## 6.4 Capacidade de Interação

Nenhum RNF formalizado nesta característica até o momento. A explicabilidade de resultado
(RF11, RF12) e a proibição de veredito absoluto (RIN [IND01]) endereçam parte do espírito
desta característica em nível funcional, mas nenhuma meta não-funcional de interação (ex.:
tempo de aprendizado, taxa de erro do usuário) foi definida. Pendência a ser tratada quando a
camada de front-end for detalhada.

## 6.5 Confiabilidade

[NFCO01] Continuidade com dados parciais (Fault Tolerance) — degradação graciosa se uma fonte
externa estiver indisponível, sem falhar a consulta inteira.

## 6.6 Segurança

[NFSE01] Chaves de API armazenadas com hash, nunca em texto plano (Confidentiality).

[NFSE02] Conformidade com LGPD — apenas dados públicos de pessoa jurídica, sem dado pessoal
de indivíduo (Confidentiality; formalmente uma exigência regulatória, sem sub-característica
dedicada na norma — enquadrada aqui por ser o item mais próximo do modelo).

[NFSE03] Criptografia AES-256 de dados sensíveis (e-mail e endereço de usuário). A senha recebe
duas camadas: hash bcrypt, cifrado em seguida com AES-256
(Confidentiality).

[NFSE04] Rate limiting em endpoints públicos (ThrottlerModule do NestJS) (Resistance).

[NFSE05] Sessões revogáveis com rotação de credenciais (Authenticity): access token JWT de
vida curta (padrão de 15 minutos), validado contra a sessão a cada requisição — revogação com
efeito imediato (logout, troca de senha) —, e refresh token opaco de uso único, rotacionado a
cada renovação, armazenado apenas como hash e com detecção de reuso.

[NFSE06] Bloqueio de login — 5 tentativas malsucedidas resultam em bloqueio de 15 minutos por
IP e/ou usuário (Resistance).

[NFSE07] Honeypot com baixa disponibilidade — endpoints isca isolados, sem impacto na
disponibilidade dos endpoints reais (Resistance).

[NFSE08] Monitoramento de vulnerabilidades de dependências via GitHub Dependabot (detalhe
operacional na Etapa de Gerência de Configuração) (Resistance).

## 6.7 Manutenibilidade

[NFPD02] Isolamento por adapter — cada fonte externa isolada em módulo próprio (Modularity).

[NFPD03] Cobertura mínima de 75% entre testes unitários, de integração, de componente, e2e e
funcionais (back-end e front-end) (Testability).

[NFSE09] Barreira de qualidade pré-commit via Husky (detalhe operacional na Etapa de
Gerência de Configuração) (Modifiability).

[NFSE10] Análise estática de código via ESLint (detalhe operacional na Etapa de Gerência de
Configuração) (Analysability).

[NFSE11] Proteção das branches `main` e `pre-prod` (detalhe operacional na Etapa de Gerência
de Configuração) (Modifiability).

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