# Loan-Insigths
## Practical Exam: Loan Insights
<p>EasyLoan offers a wide range of loan services, including personal loans, car loans, and mortgages.</p>
<p>EasyLoan offers loans to clients from Canada, United Kingdom and United States.</p>
<p>The analytics team wants to report performance across different geographic areas. They aim to identify areas of strength and weakness for the business strategy team.</p>
<p>They need your help to ensure the data is accessible and reliable before they start reporting.</p>
<p>The data you need is in the database named <code>lending</code>.</p>
<p><strong>Database Schema</strong></p>
<p><img src="https://assets.datacamp.com/production/project_1748/img/lending_schema.png" alt="database schema"></p>
<h2 id="task1">Task 1</h2>
<p>The analytics team wants to use the <code>client</code> table to create a dashboard for client details. For them to proceed, they need to be sure the data is clean enough to use.</p>
<p>The <code>client</code> table below illustrates what the analytics team expects the data types and format to be.</p>
<table>
<thead>
<tr>
<th>Column Name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>client_id</td>
<td>Unique integer (set by the database, can’t take any other value)</td>
</tr>
<tr>
<td>date_of_birth</td>
<td>Date of birth of the client, as a date  (format: YYYY-MM-DD)</td>
</tr>
<tr>
<td>employment_status</td>
<td>Current employment status of the client, either employed or unemployed, as a lower case string</td>
</tr>
<tr>
<td>country</td>
<td>The country where the client resides, either USA, UK or CA, as an upper case string</td>
</tr>
</tbody>
</table>
<p>Write a query to ensure that the <code>client</code> table matches the description provided. Your query should not update the <code>client</code> table.</p>

%%sql 
postgresql:///lending

-- Keep the two lines above
-- Start your answers here..
SELECT 
    client_id, 
    TO_CHAR(CASE
        WHEN date_of_birth SIMILAR TO '\\d{{4}}-\\d{{2}}-\\d{{2}}' THEN date_of_birth::DATE
        ELSE TO_DATE(date_of_birth, 'FMMonth DD, YYYY')
    END, 'YYYY-MM-DD') AS date_of_birth,
    CASE 
        WHEN LOWER(employment_status) SIMILAR TO 'employed|unemployed' THEN LOWER(employment_status)
        ELSE 'employed'
    END AS employment_status,
    UPPER(country) AS country
FROM client
LIMIT 10;

%%nose

import pandas as pd
import numpy as np

last_output = _

# load the dataset
test_solution = pd.read_csv('datasets/task1.csv')

student_result = last_output.DataFrame().sort_values(by='client_id')
test_solution['date_of_birth'] = test_solution['date_of_birth'].astype('string')
student_result['date_of_birth'] = student_result['date_of_birth'].astype('string')

def test_ksa1():  
    assert student_result.shape == (300, 4), \
    "Clean categorical and text data by manipulating strings. "
    assert (test_solution['employment_status'].str.strip().values == student_result['employment_status'].str.strip().values).all(), \
    "Clean categorical and text data by manipulating strings. "
         
def test_ksa2():
    assert student_result.shape == (300, 4), \
    "Convert values between data types."
    assert (test_solution['date_of_birth'].values == student_result['date_of_birth'].values).all(), \
    "Convert values between data types. "

## Task 2
<p>You have been informed that there was a problem in the backend system as some of the <code>repayment_channel</code> values are missing. </p>
<p>The missing values are critical to the analysis so they need to be filled in before proceeding.</p>
<p>Luckily, they have discovered a pattern in the missing values:</p>
<ul>
<li>Repayment higher than 4000 dollars should be made via <code>bank account</code>.</li>
<li>Repayment lower than 1000 dollars should be made via <code>mail</code>.</li>
</ul>
<p>Return the corrected <code>repayment</code> table.</p>

%%sql 
postgresql:///lending

-- Keep the two lines above
-- Start your answers here..

UPDATE repayment
SET repayment_channel = CASE
    WHEN repayment_amount > 4000 THEN 'bank account'
    WHEN repayment_amount < 1000 THEN 'mail'
    ELSE repayment_channel
END
WHERE repayment_channel = '-';


%%sql 
postgresql:///lending

SELECT *
FROM repayment
LIMIT 10;

%%nose

import pandas as pd
import numpy as np

last_output = _

# load the dataset
test_solution = pd.read_csv('datasets/task2.csv')
student_result = last_output.DataFrame().sort_values(by='repayment_id')

def test_ksa1():  
    assert student_result.shape == (1500, 5), \
    "Identify and replace missing values."
    assert (test_solution['repayment_channel'].str.strip().values == student_result['repayment_channel'].str.strip().values).all(), \
    "Identify and replace missing values."

## Task 3
<p>Starting on January 1st, 2022, all US clients started to use an online signing system.</p>
<p>The analytics team wants to analyze the loan portfolio for the US clients via the new online signing system.</p>
<p><img src="https://assets.datacamp.com/production/project_1748/img/lending_schema.png" alt="database schema"></p>
<p>Write a query that returns the data for the analytics team. Your output should include <code>client_id</code>,<code>contract_date</code>, <code>principal_amount</code> and <code>loan_type</code> columns.</p>

%%sql 
postgresql:///lending

-- Keep the two lines above
-- Start your answers here..
SELECT l.client_id, c.contract_date, l.principal_amount, l.loan_type
FROM loan AS l
LEFT JOIN client AS cl ON l.client_id = cl.client_id
LEFT JOIN contract AS c ON l.contract_id = c.contract_id
WHERE cl.country = 'USA'
AND c.contract_date >= '2022-01-01'
ORDER BY c.contract_date
LIMIT 10;



%%nose

import pandas as pd
import numpy as np

last_output = _

# load the dataset
test_solution = pd.read_csv('datasets/task3.csv')

student_result = last_output.DataFrame().sort_values(by=['client_id','contract_date','principal_amount'])


def test_ksa1():  
    assert student_result.shape == (94, 4), \
    "Extract data based on different conditions using PostgreSQL."
    assert (test_solution['client_id'].values == student_result['client_id'].values).all(), \
    "Extract data based on different conditions using PostgreSQL."

def test_ksa2():
    assert student_result.shape == (94, 4), \
        "Interpret a database schema and combine multiple tables by rows or columns using PostgreSQL."
    assert (test_solution['principal_amount'].values == student_result['principal_amount'].values).all(), \
        "Interpret a database schema and combine multiple tables by rows or columns using PostgreSQL."

## Task 4
<p>The business strategy team is considering offering a more competitive rate to the US market. </p>
<p>The analytic team want to compare the average interest rates offered by the company for the same loan type in different countries to determine if there are significant differences.</p>
<p><img src="https://assets.datacamp.com/production/project_1748/img/lending_schema.png" alt="database schema"></p>
<p>Write a query that returns the data for the analytics team. Your output should include <code>loan_type</code>, <code>country</code> and <code>avg_rate</code> columns.</p>

%%sql 
postgresql:///lending
    
-- Keep the two lines above
-- Start your answers here..
SELECT loan_type, country, ROUND(AVG(interest_rate),2) AS avg_rate
FROM loan AS l
INNER JOIN client AS c
ON l.client_id = c.client_id
GROUP BY loan_type, country
ORDER BY avg_rate DESC
LIMIT 10;

%%nose

import pandas as pd
import numpy as np

last_output = _

# load the dataset
test_solution = pd.read_csv('datasets/task4.csv')

student_result = last_output.DataFrame().sort_values(by=['loan_type','country','avg_rate'])
student_result['avg_rate'] = student_result['avg_rate'].astype('float')
test_solution['avg_rate'] = test_solution['avg_rate'].astype('float')
student_result['avg_rate'] = student_result['avg_rate'].round(2)
test_solution['avg_rate'] = test_solution['avg_rate'].round(2)
student_result = student_result.reset_index(drop=True)

def test_ksa1():  
    assert student_result.shape == (9, 3), \
    "Aggregate numeric, categorical variables and dates by groups using PostgreSQL."
    assert (test_solution['avg_rate'] == student_result['avg_rate']).all(), \
    "Aggregate numeric, categorical variables and dates by groups using PostgreSQL."

def test_ksa2():  
    assert student_result.shape == (9, 3), \
    "Interpret a database schema and combine multiple tables by rows or columns using PostgreSQL."
    assert (test_solution['country'].str.strip().values == student_result['country'].str.strip().values).all(),\
    "Interpret a database schema and combine multiple tables by rows or columns using PostgreSQL."
