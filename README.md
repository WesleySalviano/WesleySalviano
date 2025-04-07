#Consultas SQL com funções de janela e análise temporal - desafio técnico para vaga de dados.#

Este projeto contém duas consultas SQL desenvolvidas para resolver um desafio envolvendo três tabelas relacionais: `students`, `courses` e `schools`.

O objetivo é extrair, por nome da escola e por dia, informações sobre alunos matriculados em cursos cujo nome começa com "data", além de calcular métricas temporais como soma acumulada e médias móveis.

---

## Estrutura das Tabelas

```sql
-- Exemplo de estrutura das tabelas
students (id INT, name TEXT, enrolled_at DATE, course_id TEXT)
courses  (id INT, name TEXT, price NUMERIC, school_id TEXT)
schools  (id INT, name TEXT)

---

### **2. consultas_postgresql.sql**

```sql
-- Consulta A: Alunos matriculados e valor total por escola e por dia
-- Foco em cursos que comecem com "data"

SELECT
    sc.name AS school_name,
    s.enrolled_at::date AS enrollment_date,
    COUNT(s.id) AS total_students,
    SUM(c.price) AS total_revenue
FROM
    students s
JOIN
    courses c ON s.course_id = c.id
JOIN
    schools sc ON c.school_id = sc.id
WHERE
    LOWER(c.name) LIKE 'data%' -- filtra cursos cujo nome começa com "data"
GROUP BY
    sc.name, s.enrolled_at::date
ORDER BY
    enrollment_date DESC;


-- Consulta B: Soma acumulada, média móvel de 7 e 30 dias
-- A partir da tabela da consulta anterior, calculamos estatísticas temporais

WITH daily_data AS (
    SELECT
        sc.name AS school_name,
        s.enrolled_at::date AS enrollment_date,
        COUNT(s.id) AS total_students
    FROM
        students s
    JOIN
        courses c ON s.course_id = c.id
    JOIN
        schools sc ON c.school_id = sc.id
    WHERE
        LOWER(c.name) LIKE 'data%' -- cursos que começam com "data"
    GROUP BY
        sc.name, s.enrolled_at::date
)

SELECT
    school_name,
    enrollment_date,
    total_students,
    
    -- Soma acumulada por escola
    SUM(total_students) OVER (
        PARTITION BY school_name
        ORDER BY enrollment_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_students,

    -- Média móvel de 7 dias
    ROUND(AVG(total_students) OVER (
        PARTITION BY school_name
        ORDER BY enrollment_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ), 2) AS moving_avg_7_days,

    -- Média móvel de 30 dias
    ROUND(AVG(total_students) OVER (
        PARTITION BY school_name
        ORDER BY enrollment_date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ), 2) AS moving_avg_30_days

FROM
    daily_data
ORDER BY
    school_name,
    enrollment_date DESC;
