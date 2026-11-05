# 16. Avaliação de LGPD

Esta avaliação registra como o BIOS trata dados pessoais diante da Lei nº 13.709/2018 (LGPD), inclusive nos usos em KYC e KYE. É uma análise técnica do projeto, feita em novembro de 2026, e não substitui parecer jurídico. O uso real por terceiros depende das pendências da seção 16.7.

## 16.1 Dado público continua protegido

Os dados do BIOS vêm de bases públicas, mas isso não os tira da LGPD. A lei protege a informação sobre pessoa natural identificada ou identificável (art. 5º, I), e o tratamento de dado de acesso público deve considerar a finalidade, a boa-fé e o interesse público que justificaram sua disponibilização (art. 7º, § 3º). Um novo uso desses dados é possível, desde que tenha propósito legítimo e específico e preserve os direitos do titular (art. 7º, § 7º).

O enquadramento acadêmico (art. 4º, II, *b*) cobre o trabalho universitário. Ele não alcança uma empresa que use o BIOS em KYC ou KYE.

## 16.2 O que é dado pessoal no BIOS

| Dado | É dado pessoal? | Tratamento |
| --- | --- | --- |
| Razão social e nome fantasia de empresário individual | Sim, quando trazem o nome do titular | Mantidos por decisão de produto. A natureza jurídica `2135` identifica esses cadastros |
| CPF dentro da razão social | Sim | Mascarado na ingestão (`***`). Digitar um CPF não encontra o titular |
| Endereço de empresário individual | Sim, e costuma ser o da residência | Fica no perfil por decisão de produto, por ser público na Receita. Sai da exportação em XLSX e da importação por CRM, e o titular pode pedir a remoção |
| Evidências ligadas ao CNPJ de empresário individual | Sim, porque se relacionam a pessoa identificável | Exibidas com a fonte e a data, como evidência da inscrição |
| Sócios, administradores, e-mails e telefones | Sim | Não são lidos nem guardados ([INL02]) |
| Conta de usuário do BIOS | Sim | Campos cifrados e exclusão pelo titular ([NFSE02], [NFSE03]) |

O BIOS não coleta dado pessoal sensível (art. 5º, II).

**Verificação em 05/11/2026:** uma varredura das 110.372 empresas e 580.166 evidências da base de apresentação `v3` não encontrou padrões de CPF, e-mail ou telefone em nenhum campo de texto. A varredura procura esses formatos e não prova a ausência de nomes de pessoas em textos livres.

## 16.3 Regras para KYC e KYE

Em KYC e KYE, quem decide consultar é a organização usuária. Ela atua como controladora do próprio processo e precisa ter base legal, finalidade definida e transparência com a pessoa avaliada. O BIOS limita o que oferece a esse uso:

1. **O objeto da consulta é o cadastro com CNPJ.** O KYE serve principalmente à contratação de prestadores PJ e alcança a pessoa somente quando ela tem cadastro próprio com CNPJ, como empresário individual. Não há busca por CPF nem dado de quem não tem CNPJ.
2. **Ausência de cadastro não significa nada.** Não encontrar uma pessoa no BIOS não é evidência sobre ela ([IND02]).
3. **O índice não decide sobre pessoas.** Índice e confiança apoiam a análise humana e nunca fundamentam decisão automática. O titular tem direito de pedir a revisão de decisão tomada unicamente por tratamento automatizado (art. 20).
4. **A consulta deve ser proporcional ao cargo.** A Lei nº 9.029/1995 proíbe prática discriminatória e limitativa para acesso à relação de trabalho. Por cautela, o BIOS adota o critério da tese do TST no Tema 1 de recursos repetitivos (IRR-243000-58.2013.5.13.0023), que trata de certidão de antecedentes criminais: previsão legal, natureza do ofício ou grau especial de fidúcia. A tese não trata do BIOS, e aplicá-la aqui é uma analogia a confirmar na revisão jurídica.
5. **Só o necessário sai do sistema.** Integrações e planilhas levam o mínimo para a finalidade (art. 6º, III), sem contato nem endereço de rua.
6. **Sem curva do índice ligada a uma pessoa.** O perfil de empresário individual não mostra a evolução do índice ([RF36]). A linha do tempo mostra as mesmas evidências, sem transformá-las em uma nota ao longo do tempo. O art. 20 alcança decisões que definem o perfil pessoal ou profissional do titular, e uma curva por pessoa aproximaria o BIOS desse uso.

Em licitações, terceirizações e privatizações, o órgão público é o controlador da própria consulta e segue as regras do tratamento pelo poder público (LGPD, arts. 23 e seguintes). Quando o participante é empresário individual, valem as mesmas regras acima.

## 16.4 Medidas já aplicadas

| Medida | Onde |
| --- | --- |
| Sócios e contatos fora da ingestão | Adapters das fontes ([INL02]) |
| Linhas de pessoa física descartadas pelo documento, e não pelo rótulo da fonte | Ingestão do IBAMA |
| CPF mascarado na razão social e no nome fantasia | UC06, regra 11 |
| Conferência de CPF e e-mail nos dados publicados | Gerador da base de apresentação e testes das capturas |
| XLSX e CRM sem logradouro, número, complemento e CEP | UC09 e UC16 |
| Índice sem veredito, com confiança separada | [IND01], [IND02] |
| Sem histórico nem variação por empresa | [IND03], [IA05] |
| Retenção limitada dos registros operacionais | [NFSE02], item (b) |

## 16.5 Remoção a pedido do titular

O titular pede a remoção por e-mail, em odavibatista@hotmail.com, informado no rodapé do site, nos READMEs e na página de privacidade.

1. **Conferência.** O pedido traz o CNPJ e a comprovação da titularidade. A natureza jurídica `2135` (empresário individual) ajuda a conferir. Pede-se só o necessário para confirmar o nome, e o documento é descartado depois da conferência.
2. **Retirada na API.** O administrador usa `POST /admin/company-removals`. O sistema apaga cadastro, evidências, índices e movimentações e guarda só o CNPJ e a data, para nenhuma coleta trazer o cadastro de volta.
3. **Retirada da base pública.** O CNPJ entra em `removed-cnpjs.json`, no dump local, e uma nova versão da base é publicada. A lista nunca vai ao repositório, porque identificaria quem pediu.
4. **Resposta.** O titular recebe a resposta em até 15 dias, sem custo.

**Limites.** O pedido não alcança a Receita Federal nem outros sites. As médias de setor não mudam, porque não identificam ninguém. Versões antigas da base continuam no histórico do repositório público até que ele seja reescrito, o que deve ser avaliado caso a caso. Registros de sociedades não têm titular pessoa natural e não são retirados a pedido, para a ferramenta não virar um meio de esconder sanções; dado errado é corrigido a partir da fonte.

## 16.6 Teste de legítimo interesse

Segue o modelo simplificado do Guia Orientativo da ANPD de fevereiro de 2024 (Anexo II), que o guia recomenda antes de usar essa hipótese legal. Foi preenchido em novembro de 2026 pelo responsável pelo projeto, sem revisão jurídica. Deve ser refeito quando a finalidade, as fontes ou os dados mudarem.

**Tratamento avaliado:** reunir dados públicos de cadastros com CNPJ e evidências ambientais de fontes oficiais, calcular índices e oferecer a consulta a usuários autenticados, inclusive em KYC e KYE.

### Parte 1: finalidade

| Pergunta | Resposta |
| --- | --- |
| Qual é o interesse? | Permitir a avaliação ambiental de empresas e cadastros com CNPJ, com fonte verificável. É um interesse do BIOS e de terceiros, ligado à proteção ambiental e à diligência em relações comerciais |
| É legítimo e concreto? | Sim. As fontes são públicas e têm finalidade de transparência e controle social |
| Trata dado sensível? | Não (art. 5º, II) |

### Parte 2: necessidade

| Pergunta | Resposta |
| --- | --- |
| Os dados são os necessários? | Nome e CNPJ identificam o cadastro; evidências, cidade e atividade sustentam o índice |
| O endereço completo é necessário? | Não para o índice. Fica no perfil porque serve à verificação cadastral e já é público na Receita. Sai da exportação e da importação por CRM, e o titular pode pedir a remoção |
| Há forma menos intrusiva? | Em parte. Sócios, contatos e CPF ficam de fora. Reduzir o endereço de empresários individuais continua possível, porque a natureza jurídica agora é conhecida |

### Parte 3: balanceamento e salvaguardas

| Pergunta | Resposta |
| --- | --- |
| O tratamento é esperado? | O registro no CNPJ é público, e a consulta por terceiros é comum. O uso em KYE é menos esperado, por isso tem regras próprias (seção 16.3) |
| Qual é o risco ao titular? | Exposição do nome e do endereço de casa em conjunto com evidências, e uso do índice para negar emprego ou contrato |
| Quais salvaguardas existem? | Índice sem veredito, confiança separada, CPF mascarado, ausência de busca por CPF, regras de uso, retenção limitada e consulta só com login, salvo a base pública da demonstração |
| Há transparência? | Sim: página de privacidade, aviso no rodapé e README |
| Há canal de oposição? | Sim: e-mail no rodapé, com remoção apagando os dados e impedindo nova coleta |

**Conclusão:** para o uso acadêmico e a consulta sobre empresas, o interesse legítimo se sustenta com as salvaguardas acima. O ponto mais frágil é o KYE sobre pessoas: o relatório de impacto (seção 16.7) deve vir antes de qualquer uso real. Se o resultado for inconclusivo, o guia orienta adotar outra hipótese legal.

## 16.7 Pendências antes do uso real

| Nº | Pendência | Fundamento | Situação |
| --- | --- | --- | --- |
| 1 | Revisão jurídica desta avaliação | Art. 50 | Sem advogado no projeto. Uma opção é o núcleo de prática jurídica da universidade |
| 2 | Relatório de impacto para KYE | Art. 38 | Não feito |
| 3 | Prazo de exibição de evidências antigas e encerradas ligadas a empresário individual | Arts. 6º, III, e 15 | Em aberto. A base traz movimentações desde 2008 |
| 4 | Indicação formal do encarregado | Art. 41 | O contato é o responsável pelo projeto. Definir antes do uso comercial |
