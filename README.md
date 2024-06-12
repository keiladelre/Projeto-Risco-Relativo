# DOCUMENTAÇÃO

## Objetivo:

Crir um modelo de análise de dados que faça a classificação dos clientes, por meio de um score de crédito, a partir da análise de dados e a avaliação do rsico relativo que possa classificar os solicitantes em diferentes categorias de risco com base em sua probabilidade de inadimplência. utilize a mattriz de confusão e a realização de consultas complexas no BigQuery.

## Case

Com a diminuição das taxas de juros, aumentou-se a demanda por crédito e o Banco "Super Caja" esta enferentando dificuldades para realizar a análise de crédito pois seu processo de análise pe manual.
Diante disto, propõe-se uma solução de automação do processo de análise por meio de técnicas avançadas de análise de dados. O objetivo é melhorar a eficiência e a precisão na avaliação de risco de crédito e reduzir o risco de empréstimos não reembolsáveis.
Esta proposta também destaca a integração de uma métrica existente de pagamento em atraso, fortalecendo assim a capacidade do modelo.

## Ferramentas utilizadas:

Google BigQuery: Data warehouse que permite o processamento de grandes volumes de dados.
Google Colab: Plataforma para trabalhar com a linguagem de programação Python em Notebooks.
Apresentações Google: ferramenta para  criação e edição de apresnetações.
Google Looker Studio: ferramenta para criação e edição de painéis e relatórios de dados.

## Linguagens utilizadas:

SQL no BigQuery e Python no Google Colab.

## Base de dados utilizadas:

Os dados estão divididos em 4 tabelas:

![Arquivos_base de dados](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/b2e593c7-0ebf-4d30-91e4-1b6ceb5bc896)

- Dados do usuário/ cliente.
  
  ![Campos_tabela 1](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/581dd14a-ada9-41f6-8911-f25de92c6ddf)

- Dados do tipo de empréstimo.
  
![Campos_tabela 2](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/e73b37cc-f617-4129-9c75-deb353990504)
 
- Comportamento de pagamento desses empréstimos.
  
![Campos_tabela 3](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/c5262b36-fbd1-460c-a906-b8b0f544b161)

- Informação dos clientes já identificados como inadimplentes.
  
![Campos tabela 4](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/fc871396-2233-42c1-bccd-ac6b024f5142)


## Processar e preparar a base de dados:

- Conectar/ importar dados para a ferramenta.

Após baixar os 4 arquivos .csv, importei os arquivos para o Google BigQuery, criando uma tabela para cada arquivo, sem alterar os nomes dos arquivos:

![Tabelas BigQuery](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/39a23ffa-100c-45ac-a6bd-b50ca8e0e7b4)

- Identificar e tratar nulos.

```sql
--Não foram encontrados valores nulos na tabela default, para os campos user_id e default_flag--

SELECT *
FROM euphoric-diode-426013-s0.Projeto_Risco_Relativo.default
 
WHERE
  default_flag IS NULL
````

Para consultar as outras 3 tabelas alterei o comando para trazer a consulta de todas as colunas da tabela de uma única vez:

```sql
-- Não foram encontrados valores nulos para a tabela loans_detail--

  SELECT 
  SUM(CASE WHEN user_id IS NULL THEN 1 ELSE 0 END) AS user_id_nulls,
  SUM(CASE WHEN more_90_days_overdue IS NULL THEN 1 ELSE 0 END) AS more_90_days_overdue_nulls,
  SUM(CASE WHEN using_lines_not_secured_personal_assets IS NULL THEN 1 ELSE 0 END) AS using_lines_not_secured_personal_assets_nulls,
  SUM(CASE WHEN number_times_delayed_payment_loan_30_59_days IS NULL THEN 1 ELSE 0 END) AS number_times_delayed_payment_loan_30_59_days_nulls,
  SUM(CASE WHEN debt_ratio IS NULL THEN 1 ELSE 0 END) AS debt_ratio_nulls,
  SUM(CASE WHEN number_times_delayed_payment_loan_60_89_days IS NULL THEN 1 ELSE 0 END) AS number_times_delayed_payment_loan_60_89_days_nulls
FROM 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_detail`;
```
![Consulta Nulos](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/910bac76-6399-4446-904b-070c2381bc32)

```sql
 -- Consulta valores nulos para a tabela loans_outstanding--

SELECT 
  SUM(CASE WHEN loan_id IS NULL THEN 1 ELSE 0 END) AS loan_id_nulls,
  SUM(CASE WHEN user_id IS NULL THEN 1 ELSE 0 END) AS user_id_nulls,
  SUM(CASE WHEN loan_type IS NULL THEN 1 ELSE 0 END) AS loan_type_nulls
FROM 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding`;
```
![Consulta nulos 2](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/695c7b66-dbf9-41f6-9411-a41152b0a46d)

```sql
 -- Consulta valores nulos para a tabela user_info, encontrados 7199 campos nulos na coluna último mês de salário e 943 campos nulos na coluna número de dependentes--

SELECT 
  SUM(CASE WHEN user_id IS NULL THEN 1 ELSE 0 END) AS user_id_nulls,
  SUM(CASE WHEN age IS NULL THEN 1 ELSE 0 END) AS age_nulls,
  SUM(CASE WHEN sex IS NULL THEN 1 ELSE 0 END) AS sex_nulls,
  SUM(CASE WHEN last_month_salary IS NULL THEN 1 ELSE 0 END) AS last_month_salary_nulls,
  SUM(CASE WHEN number_dependents IS NULL THEN 1 ELSE 0 END) AS number_dependents_nulls
FROM 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info`;
```
![Consulta nulos 3](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/4ba7b6de-6c40-45a0-8dda-620982b49e3a)

- Identificar duplicados.

```sql
--Identificar valores duplicados na tabela default, para os campos user_id, não foram encontrados valores duplicados__

SELECT
  user_id,
  COUNT (*) AS quantidade
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.default`
GROUP BY
  user_id
HAVING
  COUNT (*)>1


  --Identificar valores duplicados na tabela loans_detail, para os campos user_id, não foram encontrados valores duplicados__

SELECT
  user_id,
  COUNT (*) AS quantidade
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_detail`
GROUP BY
  user_id
HAVING
  COUNT (*)>1

  
  --Identificar valores duplicados na tabela loans_outstanding, para os campos loan_id, pois o user_id irá se repetir para clientes que já pagaram mais de uma parcela do empréstimo, e a verificação de duplicados na coluna loan_id é para garantir que um mesmo empréstimo não tenha sido computado duas vezes, não foram encontrados valores duplicados__

SELECT
  loan_id,
  COUNT (*) AS quantidade
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding`
GROUP BY
  loan_id
HAVING
  COUNT (*)>1

    --Identificar valores duplicados na tabela luser_info, não foram encontrados valores duplicados__

SELECT
  user_id,
  COUNT (*) AS quantidade
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info`
GROUP BY
  user_id
HAVING
  COUNT (*)>1
```

- Analisado os 7199 campos nulos da coluna last_month_salary para entender quantos deles possui flag de mal pagador, por meio de um LEFT JOIN das tabelas: user.info e default:

```sql
-- verificar a flag para os casos de valor nulo no campo last_month_salary___
SELECT 
  t1.*, 
  t2.default_flag
FROM 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info` t1
LEFT JOIN 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.default` t2
ON 
  t1.user_id = t2.user_id;


--Contar quantos campos com valor nulo no campo last_month_salary possui flag 0 (bom pagador e flag 1 mal pagador)--
SELECT 
  SUM(CASE WHEN d.default_flag = 1 THEN 1 ELSE 0 END) AS count_default_flag_1,
  SUM(CASE WHEN d.default_flag = 0 THEN 1 ELSE 0 END) AS count_default_flag_0
FROM 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.default` d
LEFT JOIN 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info` ui
ON 
  ui.user_id = d.user_id
WHERE 
  ui.last_month_salary IS NULL;
```
Encontrado o seguinte resultado:

![Análise campos nulos e flag de mau pagador](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/8a1f9a5b-6e13-4b62-9295-53d408a6b706)

- Identificar e gerenciar dados fora do escopo da análise.

Realizado o teste de correlação entre as variáveis da tabela loans_detail que possui as informações de atraso no pagamento de parcelas com período entre 30 e 59 dias, 60 a 89 dias e 90 dias de atraso. Foi encontrada uma alta correlação entre essas três variáveis, por esse motivo utilizarei apenas a variável 90 dias de atraso.

```sql
--Testar as correlações entre as variáveis de tempo de atraso no pagamento das parcelas de empréstimo--
SELECT
CORR(more_90_days_overdue,number_times_delayed_payment_loan_60_89_days) AS corr_90_days_60_89_day,
CORR(more_90_days_overdue,number_times_delayed_payment_loan_30_59_days) AS corr_90_days_30_59_day,
CORR(number_times_delayed_payment_loan_60_89_days,number_times_delayed_payment_loan_30_59_days) AS corr_89_days_30_59_day
FROM `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_detail`
```
Resultado da Consulta:

![Teste de correlação](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/c4b1e04b-0307-4ce7-9c48-8460e7fb541a)

Realizado teste de correlação das variáveis de pagamento de parcela do empréstimo com período entre 30 e 59 dias, 60 a 89 dias e 90 dias de atraso e a defaul_flag que indica se um ciente já este na lista de inadimplentes ou não, e a correlação foi baixa.

```sql
--Testar as correlações entre as variáveis de tempo de atraso no pagamento das parcelas de empréstimo e a flag de cliete inadimplente--

SELECT
  CORR(t1.more_90_days_overdue, t2.default_flag) AS corr_default_flag_90_days,
  CORR(t1.number_times_delayed_payment_loan_30_59_days, t2.default_flag) AS corr_default_flag_30_59_days,
  CORR(t1.number_times_delayed_payment_loan_60_89_days, t2.default_flag) AS corr_default_flag_60_89_days
FROM 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_detail` t1
JOIN 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.default` t2
ON 
  t1.user_id = t2.user_id;
```

Resultado da Consulta:

![Teste de correlação 2](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/b13adeb8-5d70-4e8c-bd4d-355b12784de1)

- Identificar e tratar dados inconsistentes em variáveis ​​categóricas.

Dentre as 4 tabelas utilizadas no projeto temos apenas a tabela loans_outstanding com a variável string loan_type e a tabela user_info com a variável string sex.
Verifiquei com o comando abaixo a contagem dos tipos de cada uma delas.

```sql
--Verificar se existe alguma inconsistência em cariável categórica--
SELECT 
  loan_type, 
  COUNT(*)
FROM 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding`
GROUP BY 
  loan_type
HAVING 
  COUNT(*) > 1
ORDER BY 
  loan_type;
```
E obtive o seguinte resultado:

![Variáveis categóricas](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/9c4daf13-d553-4309-a447-9f228b8cfa26)

Totalizou 305.330 linhas, mas a tabela possui 305.335, faltando 5 linhas.

```sql
--Verificar se existe alguma inconsistência em variável categórica--
SELECT 
  sex, 
  COUNT(*)
FROM 
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info`
GROUP BY 
  sex
HAVING 
  COUNT(*) > 1
ORDER BY 
  sex;
```
E obtive o seguinte resultado:

![Variáveis categóricas 2](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/c46188ed-3c18-4fcc-84df-6ea5a437c083)

Totalizando as 36000 linhas da tabela original.

Para resolver os 5 valores inconsistentes utilizei o comando abaixo para padronizar os campos com letras minúsculas, e  contar novamente a coluna loan_type.

```sql
--Padronizar a coluna loan_type para que todos os campos contenham letras minúsculas, e contar novamente a coluna  loan_type pra verificar se totaliza as 305.335 linhas da tabela__

  SELECT 
  loan_type_padronizada,
  COUNT(*) AS quantidade
FROM 
  (SELECT
    loan_id,
    user_id,
    loan_type,
    LOWER(loan_type) AS loan_type_padronizada
  FROM
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding`) AS subconsulta
GROUP BY 
  loan_type_padronizada
ORDER BY 
  quantidade DESC;
  ```

E obtive esse resultado:

![Variáveis categóricas alterando para letra minúscula](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/2919c78b-cdfb-4cbe-b7ed-bc8d58ce0c17)

Apareceram as 305.335 linhas mas uma inconsistência com a palavra others, alterei para others com o comando abaixo:

```sql
--Alterar a palavra others para other__

SELECT 
  CASE 
    WHEN loan_type_padronizada = 'others' THEN 'other'
    ELSE loan_type_padronizada 
  END AS loan_type_padronizada_corrigida,
  COUNT(*) AS quantidade
FROM 
  (SELECT
    loan_id,
    user_id,
    loan_type,
    LOWER(loan_type) AS loan_type_padronizada
  FROM
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding`) AS subconsulta
GROUP BY 
  loan_type_padronizada_corrigida
ORDER BY 
  quantidade DESC;
```

E obtive o resultado com as 305.335 linhas:

![Alterando others para other](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/11801cb7-4e54-4d5e-a4b3-2a164f9ad2aa)


Crie a nova tabela loans_outstanding_padronizada com os dados padronizados.


```sql
--Criar tabela loans_outstanding com essas dados da coluna loan_type corrigidos--

CREATE TABLE
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding_padronizada` AS
SELECT
  loan_id,
  user_id,
  CASE 
    WHEN LOWER(loan_type) = 'others' THEN 'other'
    ELSE LOWER(loan_type) 
  END AS loan_type_padronizada
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding`;
```

-  Identificar e tratar dados discrepantes em variáveis ​​numéricas

Utilizei o comando abaixo para verificar outliers nas variáveis idade e último salário informado.

```sql
--Identificar e tratar dados discrepantes em variáveis ​​numéricas--

SELECT
  MAX (age),
  MIN (age),
  AVG (age),
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info`

```

Resultado obtido:

![Identificando Outliers](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/89025c12-7f46-4501-bc68-dce62e950383)


```sql
-- Identificando outiliers para o último salário informado--

  SELECT
  MAX (last_month_salary),
  MIN (last_month_salary),
  AVG (last_month_salary),
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info`
```

Resultado obtido:

![Identificando Outliers 2](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/954db46b-f269-41cb-a4f6-b18797b51ffe)

A partir desse resultado, onde aparece um salário de R$ 1.560.100 pensei sobre a possibilidade de se tratar de uma empresa e não pessoa física.

(verificar os outros dados desse outlier como idade e sexo)








