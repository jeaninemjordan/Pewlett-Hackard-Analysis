# Pewlett-Hackard-Analysis

## An analysis of an employee database using PostgreSQL.

### Project Overview

This project intended on examining the personnel rosters for a large corporation anticipating an influx in employees nearing retirement with a specific focus on the effects of the sales and development teams. A need to designate current, qualified employees as mentors to replacement staff is recognized and reviewed during this analysis. 

#### Resources: 

* PostgreSQL 11.16 
* pgAdmin 4: version 6.8

#### Results

Six supplied CSV files containing various employee data were used to map the ERD diagram pictured in the schema below and supply this project with the information used to conduct this analysis.

SCHEMA PIC

A query was ran that joined the Employees table (listing the employee number, first name, and last name) and the Titles table (listing the title, start date and end date). The employee birth date was filtered to only include employees nearing retirement age (1952 – 1955 in this analysis). A new table was subsequentially created  (Retirement Titles). 

```
SELECT e.emp_no, 
        e.first_name,
        e.last_name,
        ti.title,
        ti.from_date,
        ti.to_date
INTO retirement_titles
FROM employees as e
INNER JOIN titles as ti
ON e.emp_no = ti.emp_no
WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31')
ORDER BY emp_no
;
PIC
The Retirement Titles table was then used to glean the employee number, first name, last name and title and was filtered to only include the current titles, thus removing the duplicates and creating the unique titles table. 
SELECT DISTINCT ON (emp_no) emp_no, first_name, last_name, title
INTO unique_titles
FROM retirement_titles
ORDER BY emp_no, from_date DESC
;
```

PIC

Another query was written that retrieved the number of employees by their most recent job titles approaching potential retirement from the Unique Titles table. A new table, the Retiring Titles table, was created that groups the retiring employees by title in descending order to ascertain what departments and positions will incur the biggest loss from a mass retirement. It is observed that the sales and development departments will be the most affected by the loss of employees.
SELECT COUNT(title) count, title
INTO retiring_titles
FROM unique_titles
GROUP BY (title) 
ORDER BY count DESC
;
'''

PIC

To assess which retiring employees were qualified for the organization’s mentorship program, a new analysis was conducted that used the employees’ ages and employment duration as determining metrics. A new table was created, the Mentorship Eligibility table, using the query below. 

```
SELECT DISTINCT ON (emp_no)
        e.emp_no, 
        e.first_name,
        e.last_name,
        e.birth_date,
        de.from_date,
        de.to_date,
        ti.title
INTO mentorship_eligibility
FROM employees as e
INNER JOIN dept_emp as de
ON e.emp_no = de.emp_no
INNER JOIN titles as ti
ON e.emp_no = ti.emp_no
WHERE (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31') AND de.to_date = '9999-01-01'
ORDER BY emp_no, ti.from_date DESC
;
```

PIC 

Further examination of the organization’s eligible mentors was conducted to determine that most of the retirees eligible to mentor are Senior Engineers (529 at 34%) and Senior Staff agents (569 at 37%). The remaining 29% of eligible mentors are Engineers, General Staff, Technique Leaders and Assistant Engineers.

```
ELECT COUNT(title) count, title
INTO eligible_mentors_titles
FROM mentorship_eligibility
GROUP BY (title) 
ORDER BY count DESC
```

PIC

Additional analysis was conducted by grouping the positions in demand by department as well as by title to determine where further mentorship and coverage would be needed. Two queries were written that create the Unique Titles Dept table as well as the Position Demand table. 

```
INTO unique_titles_dept
FROM retirement_titles as rt	
INNER JOIN dept_emp as de
ON (rt.emp_no = de.emp_no)
INNER JOIN departments as d 
ON (d.dept_no = de.dept_no)
ORDER BY rt.emp_no, rt.to_date DESC;
```
```
SELECT ut.dept_name, ut.title, COUNT(ut.title) 
INTO position_demand
FROM (SELECT title, dept_name from unique_titles_dept) as ut
GROUP BY ut.dept_name, ut.title
ORDER BY ut.dept_name DESC;
```

PIC

#### Summary

90,398 (30%) of 300,024 employees are nearing retirement eligibility for the Pewitt Hackard organization. Using the information provided by the organization alongside the PostgreSQL analysis conducted during this project, it can be determined that most of the potential retirees are working as Senior Engineers (29,415 at 33%) or Senior Staff agents (28,255 at 31%). Engineers, General Staff, Technique Leaders, Assistant Engineers and Managers make up the remaining 36% of retiring staff. There are 1549 employees (17%) who are qualified for the mentorship program (determined by their ages and employment duration); however, this equates to a 58:1 employee to mentor ratio if all vacancies are anticipated to be filled. The additional analysis conducted to determine position demand by department is helpful to pinpoint how many mentors will be allotted to which department. 
