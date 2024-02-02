---
sidebar_position: 1
---

# Salário

## Glossário

- Salario: Todos os valores monetários recebidos dentro de um periodo de tempo (mensal, semanal, etc), exemplos:
  - O salário em si
  - VA, VR, VT, cartao multibeneficios (cartao ifood)
  - Auxilio home office

## Linha de raciocinio

- O salário será algo separado das transacoes recorrentes, (inclusive, estou pensando em remover por completo as "transacoes recorrentes" e separa-las em multiplas coisas especificasa pra cada caso, assim temos mais controle sobre como aquilo é feito, mas isso é conversa pra outra hora)
  - Junto das recurrent transactions? Podemos ver q tem praticamente tudo em comum, a unica coisa q muda é o tipo de contrato (CLT/PJ) e o startDate.
  - Podemos criar a tabela salary e a salary_recurrent_transactions, para vincular a transaction ao salary.
- O salário ele tem as seguintes caracteristicas:
  - A pessoa pode ter várias modalidades de contratacao:
    - CLT
    - PJ
  - Ele pode ser pago parcelado, dividido em multiplas parcelas (maximo 3? 5?)
    - As parcelas devem ser de valores fixos
    - Caso a pessoa seja CLT, o recebimento do salario pode acontecer de SEGUNDA a SABADO
  - Caso a pessoa seja CLT, ela recebe 13
    - O 13 varia de acordo com a quantidade de tempo que a pessoa ficou na empresa
    - O 13 pode ser pago parcelado em até 2 parcelas
  - Caso a pessoa seja CLT, ela pode receber Abono Salarial
    - CLT que receberam em media até 2 salarios minimos durante o ano anterior
    - O saque pode ser feito a partir de 15 de fevereiro
  - Caso a pessoa seja CLT, fazer o calculo de ferias bonitinho
    - Recebe +1 salario (ou parte do salario referente a quantidade de dias de folga?) ao entrar de ferias
    - O salario do mes em q ela voltar de ferias é alterado baseado na quantidade de dias trabalhados
  - A pessoa pode receber beneficios nos cartoes de VA, VR e VT, oq tbm sao incluidos como "salario"
  - ~~Tabela de salario:~~
    - ~~id~~
    - ~~accountId~~
    - ~~type: CLT, PJ~~
    - ~~startDate? (Para calcular 13, abono, etc)~~
    - ~~salary_installments (tabela auxiliar, one-to-many)~~
      - ~~id~~
      - ~~salaryId~~
      - ~~amount~~
      - ~~day~~
        - ~~1-31 (dias 1 a 31 do mes)~~
          - ~~Caso o dia seja maior q 28 e seja em fevereiro, será ajustado para o ultimo dia do mes~~
        - ~~FIRST_DAY_OF_MONTH (primeiro dia do mes)~~
        - ~~LAST_DAY_OF_MONTH (ultimo dia do mes)~~
        - ~~FIFTH_BUSSNIESS_DAY (quinto dia util)~~
      - ~~paymentType: BANK_ACCOUNT, PREPAID_CREDIT (para deposito de VA, VR, VT)~~
      - ~~bankAccountId?~~
        - ~~onlyIf paymentType = BANK_ACCOUNT~~
      - ~~cardId?~~
        - ~~onlyIf paymentType = PREPAID_CREDIT~~
      - ~~conditions (array)~~
        - ~~Mesmos valores de RecurrenceConditionsEnum~~
  - Tabela de salario:
    - id
    - accountId
    - type: CLT, PJ
    - startDate? (Para calcular 13, abono, etc)
    - salary_recurrent_transactions (tabela auxiliar, one-to-many)
      - salaryId
      - recurrent_transaction_id

## Referencias

- [Adiantamento Salárial](https://blog.caju.com.br/leis-trabalhistas/adiantamento-salarial/)

## Conclusao

- Usaremos os recurrent transactions para criar essas transactions
- Teremos uma tabela de salario, para salvar os dados gerais de salario (inclusive, com isso vamos poder ter dados de tipo, quantos usuarios do nosso app trabalham como CLT, qual a renda media deles, etc)
- Tabelas
  - salary:
    - id
    - accountId
    - type: CLT, PJ
    - startDate? (Para calcular 13, abono, etc)
  - salary_recurrent_transactions (tabela auxiliar, one-to-many):
    - salaryId
    - recurrent_transaction_id
    - type (SALARY, VA, VR, VT, THIRTEENTH, ALLOWANCE)
- Devemos adicionar os valores MONTH\_$\{month} ao enum RecurrenceCreateEnum e usar a seguinte logica no recurrent transactions:
  - Apenas recurrent transaction ANUAIS podem (e devem!) ter os valores MONTH\_$\{month} na frequency
  - Quando um valor MONTH*$\{month} estiver presente, deve ser setada esse mes para a data, e podem ter qualquer outro valor de DAY*$\{day} funcionam normalmente (caso + de 1 valor desses esteja presente, multiplas transactions serao criadas naquele mes)
