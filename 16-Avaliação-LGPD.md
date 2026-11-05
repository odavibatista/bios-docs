# 16. Avaliação de LGPD

Esta avaliação registra como o BIOS trata dados pessoais diante da Lei nº 13.709/2018 (LGPD), inclusive nos usos em KYC e KYE. É uma análise técnica do projeto, feita em novembro de 2026, e não substitui parecer jurídico. O uso real por terceiros depende de revisão jurídica e das pendências da seção 16.5.

## 16.1 Dado público continua protegido

Os dados do BIOS vêm de bases públicas, mas isso não os tira da LGPD. A lei protege a informação sobre pessoa natural identificada ou identificável (art. 5º, I), e o tratamento de dado de acesso público deve considerar a finalidade, a boa-fé e o interesse público que justificaram sua disponibilização (art. 7º, § 3º). Um novo uso desses dados é possível, desde que tenha propósito legítimo e específico e preserve os direitos do titular (art. 7º, § 7º).

O enquadramento acadêmico (art. 4º, II, *b*) cobre o trabalho universitário. Ele não alcança uma empresa que use o BIOS em KYC ou KYE.

## 16.2 O que é dado pessoal no BIOS

| Dado | É dado pessoal? | Tratamento |
| --- | --- | --- |
| Razão social e nome fantasia de empresário individual | Sim, quando trazem o nome do titular | Mantidos por decisão de produto. A base não distingue a natureza jurídica do cadastro |
| CPF dentro da razão social | Sim | Mascarado na ingestão (`***`). Digitar um CPF não encontra o titular |
| Endereço de empresário individual | Sim, e costuma ser o da residência | Fica no perfil. Sai da exportação em XLSX e da importação por CRM |
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

## 16.5 Pendências antes do uso real

| Nº | Pendência | Fundamento | Situação |
| --- | --- | --- | --- |
| 1 | Canal para o titular que não tem conta pedir acesso, correção ou se opor ao tratamento | Art. 18 e § 2º | Não existe. A exclusão de conta atende só o usuário |
| 2 | Decidir se o perfil e a base de apresentação mantêm número, complemento e CEP. Os arquivos em `public/presentation` são públicos, sem autenticação real | Art. 6º, III | Em aberto. A base não distingue empresário individual, então a regra teria de valer para todos ou exigir a natureza jurídica |
| 3 | Aviso de privacidade e indicação do encarregado | Arts. 9º e 41 | Não existem |
| 4 | Avaliação de legítimo interesse e relatório de impacto para KYE | Arts. 7º, IX, 10 e 38 | Não feitos. O Guia da ANPD de 2024 traz o modelo de teste de balanceamento |
| 5 | Prazo de exibição de evidências antigas e encerradas ligadas a empresário individual | Arts. 6º, III, e 15 | Em aberto. A base traz movimentações desde 2008 |
