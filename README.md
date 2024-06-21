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

```sql
--Identificar qual a idade e sexo do outlier de último salário informado--

   WITH salary_stats AS (
  SELECT
    MAX(last_month_salary) AS max_salary,
    MIN(last_month_salary) AS min_salary,
    AVG(last_month_salary) AS avg_salary
  FROM
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info`
),
max_salary_details AS (
  SELECT
    age,
    sex,
    last_month_salary
  FROM
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info`
  WHERE
    last_month_salary = (SELECT max_salary FROM salary_stats)
)
SELECT
  max_salary_details.*,
  salary_stats.max_salary,
  salary_stats.min_salary,
  salary_stats.avg_salary
FROM
  max_salary_details,
  salary_stats;
```

E o resultado obtido foi:

![dados do outlier](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/285af331-196c-4369-b174-7b948eda296d)

Nesse caso essa informação pode estar com erro de digitação ou ter como representante da empresa uma mulher com idade de 44 anos, em qualquer que for o caso acrdito ser melhor deixar a variável e não alterar os dados, deixando apenas essa observação para uma análise mais detalhada do colaborador que for realiar a análise de liberação de crédito.

- Verificar e alterar o tipo de dados.

  ```sql
  SELECT
  CAST(last_month_salary AS INT64) AS last_month_salary_integer
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info`;
  ```
- Criar uma tabela com a nova coluna last_month_salary_integer

--Criando uma nova tabela com a coluna last_month_salary_integer--
--MAS POR QUE PRECISEI FAZER ESSA ALTERAÇÃO NESSA COLUNA ? FIZ NESSA POIS ERA A UNICA FLOAT QUE NÃO TEM PROBLEMA ARREDONDAR AS CASAS DECIMAIS--

CREATE TABLE `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info_new` AS
SELECT
  user_id,
  age,
  sex,
  number_dependents,
  CAST(last_month_salary AS INT64) AS last_month_salary_integer,
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info`;

- Criar novas variáveis.

Na nova tabela loans_outstanding_padronizada realizei um código para  somar os tipos de empréstimo de cada user_id, para que seja possível unir as tabelas posterioemente e totalizar as 36000 linhas, que refletem os 36000 clientes dessa base de dados.

```sql
--Na nova tabela loans_outstanding_padronizada somar os tipos de empréstimo de cada user_id, para que seja possível unir as tabelas posterioemente e totalizar as 36000 linhas, que refletem os 36000 clientes dessa base de dados--

SELECT
  user_id,
  SUM(CASE WHEN loan_type_padronizada = 'real estate' THEN 1 ELSE 0 END) AS loan_type_real_estate,
  SUM(CASE WHEN loan_type_padronizada = 'other' THEN 1 ELSE 0 END) AS loan_type_other,
  COUNT(*) AS total_loans
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding_padronizada`
GROUP BY
  user_id
ORDER BY
  user_id;

--Testando para verificar se vieram todos os empréstimos__

SELECT
  SUM(CASE WHEN loan_type_padronizada = 'real estate' THEN 1 ELSE 0 END) AS total_loan_type_real_estate,
  SUM(CASE WHEN loan_type_padronizada = 'other' THEN 1 ELSE 0 END) AS total_loan_type_other,
  COUNT(*) AS total_loans
FROM
  `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding_padronizada`;
```

Obtive o seguinte resultado:

![Criando novas variáveis](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/a36f057c-3dbe-4e6a-8321-62c37e9b230f)

A contagem totalizou os 305.335 empréstimos assim como na tabela loans_outstanding_padronizada

- Unir tabelas.

Para passar para a próxima fase de análise exploratória dos dados, é necessário unir as tabelas: user_info_new, loans_detail, loans_outstanding_new e default: 


```sql
--Unindo as tabelas--

SELECT 
    t1.*, 
    t2.more_90_days_overdue,
    t2.using_lines_not_secured_personal_assets,
    t2.number_times_delayed_payment_loan_30_59_days,
    t2.debt_ratio,
    t2.number_times_delayed_payment_loan_60_89_days,
    t3.loan_type_real_estate,
    t3.loan_type_other,
    t3.total_loans,
    t4.default_flag
FROM 
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info_new` t1
LEFT JOIN 
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_detail` t2 
ON 
    t1.user_id = t2.user_id
LEFT JOIN 
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding_new` t3 
ON 
    t1.user_id = t3.user_id
LEFT JOIN 
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.default` t4 
ON 
    t1.user_id = t4.user_id;
```

Verifiquei com ficaram as colunas, e identifiquei que as novas colunas: loan_type_real_estate, loan_type_other e total_loans, ficaram com alguns cmapos nullos, dessa forma entendo que alguns clientes não possuem empréstimo junto ao banco.

![Unindo tabelas](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/d6329194-9a04-454a-b46a-368415227522)

Em seguida criei a nova tabela com o nome uniao_tabelas:

```sql
--Criar nova tabela com nome uniao_tabelas--

  CREATE TABLE `euphoric-diode-426013-s0.Projeto_Risco_Relativo.uniao_tabelas` AS
SELECT 
    t1.*, 
    t2.more_90_days_overdue,
    t2.using_lines_not_secured_personal_assets,
    t2.number_times_delayed_payment_loan_30_59_days,
    t2.debt_ratio,
    t2.number_times_delayed_payment_loan_60_89_days,
    t3.loan_type_real_estate,
    t3.loan_type_other,
    t3.total_loans,
    t4.default_flag
FROM 
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.user_info_new` t1
LEFT JOIN 
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_detail` t2 
ON 
    t1.user_id = t2.user_id
LEFT JOIN 
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.loans_outstanding_new` t3 
ON 
    t1.user_id = t3.user_id
LEFT JOIN 
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.default` t4 
ON 
    t1.user_id = t4.user_id;  
```

## Fazer uma análise exploratória:

- Agrupar dados de acordo com variáveis ​​categóricas.

Use tabelas no Looker Studio para resumir dados em variáveis ​​categóricas.


- Ver variáveis ​​categóricas
  
Use gráficos de barras no Looker Studio para visualizar variáveis ​​categóricas.


- Aplicar medidas de tendência central (moda, média, mediana)
  
Use as opções da tabela para calcular estatísticas descritivas para ajudar a compreender a distribuição dos dados.


- Ver distribuição
  
Use histograma ou boxplot no LookerStudio para exibir variáveis ​​numéricas.


-Calcular quartis, decis ou percentis

Calcular quartis para variáveis ​​de risco relativo no BigQuery

-Calcular correlação entre variáveis ​​numéricas

Compreender a relação que existe entre variáveis ​​numéricas através de correlações. Use gráficos de dispersão e linhas de tendência. Você também pode usar o comando CORR no BigQuery

## Aplicar técnica de análise:

-Calcular risco relativo

Calcule o risco relativo em cada grupo (Mau pagador, Bom pagador) de cada variável em relação à variável “inadimplência”.

```sql
-- RISCO RELATIVO COM QUARTIL
-- Selecionar variáveis para calcular risco relativo
WITH risk_relative AS (
  SELECT
    quartil_debt_ratio, -- coluna de ntile
    COUNT(*) AS total_customers,  -- count de total user_id
    SUM(default_flag) AS total_default, -- soma de inadimplentes 
    AVG(default_flag) AS default_rate -- média de inadimplentes
  FROM
    `euphoric-diode-426013-s0.Projeto_Risco_Relativo.uniao_tabelas`
  GROUP BY
    quartil_debt_ratio
)

-- Cálculo do risco relativo
SELECT
  quartil_debt_ratio,
  total_customers,
  total_default,
  default_rate,
  default_rate / (SELECT AVG(default_flag) FROM `euphoric-diode-426013-s0.Projeto_Risco_Relativo.uniao_tabelas`) AS risk_relative
FROM
  risk_relative
ORDER BY
  quartil_debt_ratio;
----- resultado da consula: quartil 1 e 2 tem RR > 1
```
Obtive o seguinte resultado:

![Risco Relativo debt_ratio](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/3ce3cd50-20d2-453a-b1bf-edfd9d3a52dd)


Fiz esse mesmo comando para as variáveis: quartil_last_month_salary, quartil_age, quartil_more_90_days, quartil_using_lines, quartil_number_dependents e quartil_total_loans.
Conforme os resultadosm abaixo: 

![Risco Relativo Quartil Last month salary](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/77b7a470-e4b7-43c1-b5ca-caf0585c33ce)


![Risco Relarivo quartil age](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/aecf1e50-1055-4c47-9fb9-3a8da523f84f)


![Risco Relativo more 90 days](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/58308fe3-5cd7-4ca2-a521-d4e5c6b330b9)


![Risco Relativo Using Lines](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/a4cabcbb-a3c8-4ca3-82f1-aa78369c21cf)


![Risco Relativo number dependents](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/df624e69-9f3d-482a-b4da-e6df184fce43)


![Risco Relativo total loans](https://github.com/keiladelre/Projeto-Risco-Relativo/assets/171286176/ca581253-946e-40f7-bd78-7eaade8a1248)



-Validar hipótese

Nos grupos encontrados, valide a hipótese de quais apresentam risco relativo diferente.


1 - Os mais jovens correm um risco maior de não pagamento?

2 - Pessoas com mais empréstimos ativos correm maior risco de serem maus pagadores.?

3 - Pessoas que atrasaram seus pagamentos por mais de 90 dias correm maior risco de serem maus pagadores.

Após validar as hipóteses de acordo com o resultado do cálculo do risco relativo, construa uma tabela com os grupos de cada variável que apresenta maior risco de ser um mau pagador
