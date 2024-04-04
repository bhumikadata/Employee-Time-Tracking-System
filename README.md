# Employee-Time-Tracking-System

This repository contains SQL code for analyzing employee time spent in the office based on their movement records. The problem statement, SQL code, explanation, and its application are provided below.

# Problem Statement
A company records its employees' movements in and out of the office in a table with three columns: emp_id (employee ID), action (action taken, either "in" or "out"), and created_at (timestamp of the action).

The task is to measure the amount of time spent by each employee inside the office between the specified time range (2019-04-01 14:00:00 and 2019-04-02 10:00:00). The following conditions apply:

- The first entry for each employee is always "in".
- Every "in" action is succeeded by an "out".
- Employees can work across days.

![day 28](https://github.com/bhumikadata/Employee-Time-Tracking-System/assets/131578649/3fca857b-c373-4acc-9146-60a5e8a16375)

# SQL Code

```
-- Common Table Expressions (CTEs) to preprocess the data
WITH cte AS (
    -- Adding next created_at timestamp for each employee's action
    SELECT 
        *,
        LEAD(created_at) OVER(PARTITION BY emp_id ORDER BY created_at) AS next_created_at
    FROM 
        employee_record  
),
considered_time AS (
    -- Filtering records within the specified time range
    SELECT 
        emp_id,
        CASE 
            WHEN created_at < '2019-04-01 14:00:00' THEN '2019-04-01 14:00:00'
            ELSE created_at
        END AS in_time,
        CASE 
            WHEN next_created_at > '2019-04-02 10:00:00' THEN '2019-04-02 10:00:00' 
            ELSE next_created_at
        END AS out_time   
    FROM 
        cte  
    WHERE
        action = 'in'  
) 
-- Final query to calculate time spent by each employee
SELECT 
    emp_id,
    ROUND(SUM(CASE 
                WHEN in_time > out_time THEN 0 
                ELSE (julianday(out_time) - julianday(in_time)) * 24 * 60
            END), 2) AS time_spent
FROM 
    considered_time
GROUP BY 
    emp_id
```

# Explanation

#### CTE (Common Table Expression) Processing: 
The data is preprocessed using CTEs to add a next_created_at column indicating the timestamp of the next action for each employee.

#### Time Range Consideration: 
Records are filtered to include only those within the specified time range.

#### Calculation of Time Spent: 
The final query calculates the time spent by each employee inside the office. It considers cases where an employee might have stayed overnight.

# Application

This SQL script provides insights into employee time management. By analyzing the time spent by each employee in the office, the company can:

- Optimize work schedules based on peak working hours.
- Identify potential inefficiencies or anomalies in employee work patterns.
- Ensure compliance with labor laws regarding working hours.
