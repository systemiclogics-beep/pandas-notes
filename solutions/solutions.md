# SQL and Pandas Solutions

This file contains solutions for the SQL (20 questions) and Pandas (20 exercises) practice sets provided earlier. Use these as runnable examples — SQL statements for MySQL and Python/Pandas snippets for the sample DataFrames.

---

## SQL Solutions (assumed schemas)
-- student(std_id PK, std_name, city, marks, department_id)
-- course(course_id PK, course_name)
-- enrollment(enrollId PK, std_id FK, course_id FK, enroll_date)
-- customer(customerId PK, name, city, phone)
-- orders(orderId PK, customerId FK, orderDate, totalAmount)
-- orderitem(orderItemId PK, orderId FK, productId FK, quantity)
-- product(productId PK, productName, price, categoryId)
-- category(categoryId PK, categoryName)
-- employee(empId PK, name, department, salary, hire_date)

1) List names and marks of students who scored more than 75.
```sql
SELECT std_name, marks
FROM student
WHERE marks > 75;
```

2) Return the top 5 students by marks (name and marks).
```sql
SELECT std_name, marks
FROM student
ORDER BY marks DESC
LIMIT 5;
```

3) List all distinct cities from the student table.
```sql
SELECT DISTINCT city
FROM student;
```

4) For each city, show the number of students and the average marks. Order by average marks descending.
```sql
SELECT city, COUNT(*) AS student_count, AVG(marks) AS avg_marks
FROM student
GROUP BY city
ORDER BY avg_marks DESC;
```

5) Show cities that have more than 3 students (city and student_count).
```sql
SELECT city, COUNT(*) AS student_count
FROM student
GROUP BY city
HAVING COUNT(*) > 3;
```

6) List student name and course_name for every enrollment (only students who are enrolled in a course).
```sql
SELECT s.std_name, c.course_name
FROM student s
INNER JOIN enrollment e ON s.std_id = e.std_id
INNER JOIN course c ON e.course_id = c.course_id;
```

7) List all students and the course_name they are enrolled in, showing NULL if not enrolled.
```sql
SELECT s.std_name, c.course_name
FROM student s
LEFT JOIN enrollment e ON s.std_id = e.std_id
LEFT JOIN course c ON e.course_id = c.course_id;
```

8) Find student name, course_name, and enrollment id for all enrollments.
```sql
SELECT e.enrollid, s.std_name, c.course_name
FROM enrollment e
JOIN student s ON e.std_id = s.std_id
JOIN course c ON e.course_id = c.course_id;
```

9) Select students whose marks are greater than the overall average marks.
```sql
SELECT *
FROM student
WHERE marks > (
  SELECT AVG(marks) FROM student
);
```

10) Find students enrolled in any course taken by student with std_id = 10.
```sql
SELECT DISTINCT s2.*
FROM enrollment e1
JOIN enrollment e2 ON e1.course_id = e2.course_id
JOIN student s2 ON e2.std_id = s2.std_id
WHERE e1.std_id = 10 AND s2.std_id <> 10;
-- OR using IN
SELECT * FROM student WHERE std_id IN (
  SELECT std_id FROM enrollment WHERE course_id IN (
    SELECT course_id FROM enrollment WHERE std_id = 10
  )
);
```

11) Find orders where customerId is 1, 2, or 3 using IN; then rewrite using ORs.
```sql
-- Using IN
SELECT * FROM orders WHERE customerId IN (1,2,3);
-- Using OR
SELECT * FROM orders WHERE customerId = 1 OR customerId = 2 OR customerId = 3;
```

12) For each product, compute total quantity sold and total revenue (quantity * price). Show top 5 products by revenue.
```sql
SELECT p.productId, p.productName,
       SUM(oi.quantity) AS total_quantity,
       SUM(oi.quantity * p.price) AS total_revenue
FROM orderitem oi
JOIN product p ON oi.productId = p.productId
GROUP BY p.productId, p.productName
ORDER BY total_revenue DESC
LIMIT 5;
```

13) Find all products bought by customerId = 5: list productName and orderDate.
```sql
SELECT DISTINCT p.productName, o.orderDate
FROM orders o
JOIN orderitem oi ON o.orderId = oi.orderId
JOIN product p ON oi.productId = p.productId
WHERE o.customerId = 5;
```

14) For each customer and month (from orderDate), show total amount spent.
(MySQL: use DATE_FORMAT or YEAR()/MONTH())
```sql
SELECT o.customerId,
       YEAR(o.orderDate) AS yr,
       MONTH(o.orderDate) AS mth,
       SUM(o.totalAmount) AS total_spent
FROM orders o
GROUP BY o.customerId, YEAR(o.orderDate), MONTH(o.orderDate)
ORDER BY o.customerId, yr, mth;
```

15) Find customers who spent more than 1000 in total.
```sql
SELECT o.customerId, SUM(o.totalAmount) AS total_spent
FROM orders o
GROUP BY o.customerId
HAVING SUM(o.totalAmount) > 1000;
```

16) For each student, show std_name and whether their marks equal the maximum marks in the table.
```sql
SELECT std_name,
       CASE WHEN marks = (SELECT MAX(marks) FROM student) THEN 'Yes' ELSE 'No' END AS is_top
FROM student;
```

17) Add a column 'email' VARCHAR(100) to customer, and then rename it to 'contact_email' (MySQL syntax).
```sql
ALTER TABLE customer ADD COLUMN email VARCHAR(100);
ALTER TABLE customer CHANGE COLUMN email contact_email VARCHAR(100);
```

18) Increase salary by 10% for employees in the 'Sales' department.
```sql
UPDATE employee
SET salary = salary * 1.10
WHERE department = 'Sales' AND salary IS NOT NULL;
```

19) Delete orderitems whose quantity is 0.
```sql
DELETE FROM orderitem WHERE quantity = 0;
```

20) Create orderitem table with PK and FKs including ON DELETE CASCADE on orderId.
```sql
CREATE TABLE orderitem (
  orderItemId INT AUTO_INCREMENT PRIMARY KEY,
  orderId INT NOT NULL,
  productId INT NOT NULL,
  quantity INT NOT NULL DEFAULT 1,
  FOREIGN KEY (orderId) REFERENCES orders(orderId) ON DELETE CASCADE,
  FOREIGN KEY (productId) REFERENCES product(productId) ON DELETE RESTRICT
) ENGINE=InnoDB;
```

---

## Pandas Solutions (assumes the sample DataFrames provided)
# Use the sample DataFrames: students, courses, enrollment, products, customers, orders, orderitem, employees

1) Select: show std_name and marks for students with marks > 75.
```python
students.loc[students['marks'] > 75, ['std_name','marks']]
```

2) List students with missing marks.
```python
students[students['marks'].isna()]
```

3) Students in "Delhi" with marks >= 75.
```python
students[(students['city'] == 'Delhi') & (students['marks'] >= 75)][['std_name','marks']]
```

4) Add grade column: marks >=85 -> 'A', >=70 -> 'B', else 'C'.
```python
import numpy as np
conds = [students['marks'] >= 85, students['marks'] >= 70]
choices = ['A','B']
students['grade'] = np.select(conds, choices, default='C')
```

5) Fill missing student marks with department median.
```python
dept_median = students.groupby('department')['marks'].transform('median')
mask = students['marks'].isna()
students.loc[mask, 'marks'] = dept_median[mask]
```

6) For each city, compute count of students and average marks.
```python
students.groupby('city')['marks'].agg(student_count='count', avg_marks='mean').reset_index()
```

7) For each department show average, min, and max marks.
```python
students.groupby('department')['marks'].agg(['mean','min','max']).reset_index()
```

8) Add column dept_mean that contains the mean marks for that student’s department.
```python
students['dept_mean'] = students.groupby('department')['marks'].transform('mean')
```

9) Merge students, enrollment, and courses to get student names and course_name.
```python
students.merge(enrollment, on='std_id', how='inner').merge(courses, left_on='course_id', right_on='course_id')[['std_name','course_name']]
```

10) Left merge: show all students and their course_name (NaN if none).
```python
students.merge(enrollment, on='std_id', how='left').merge(courses, on='course_id', how='left')[['std_name','course_name']]
```

11) Distinct cities and counts.
```python
students['city'].value_counts()
# or
students['city'].unique()
```

12) Top 3 students by marks.
```python
students.sort_values('marks', ascending=False).head(3)[['std_name','marks']]
```

13) Find the month for each order and show totalAmount per month.
```python
orders['order_month'] = orders['orderDate'].dt.to_period('M')
orders.groupby('order_month')['totalAmount'].sum().reset_index()
```

14) Compute total revenue per product (quantity * price) and top product by revenue.
```python
oi = orderitem.merge(products, on='productId', how='left')
oi['revenue'] = oi['quantity'] * oi['price']
rev = oi.groupby(['productId','productName'])['revenue'].sum().reset_index()
rev.sort_values('revenue', ascending=False)
```

15) Create salary_band column mapping salary to Low/Medium/High and handle NaN as 'Unknown'.
```python
def band(x):
    if pd.isna(x):
        return 'Unknown'
    if x < 55000:
        return 'Low'
    if x < 70000:
        return 'Medium'
    return 'High'

employees['salary_band'] = employees['salary'].apply(band)
```

16) Fill employee salary NaNs with department median salary.
```python
emp_median = employees.groupby('department')['salary'].transform('median')
mask = employees['salary'].isna()
employees.loc[mask, 'salary'] = emp_median[mask]
```

17) Create student_code = std_id (string) + '_' + department (uppercased).
```python
students['student_code'] = students['std_id'].astype(str) + '_' + students['department'].str.upper()
```

18) Create boolean column student_above_dept_mean comparing marks with dept_mean (#8).
```python
students['above_dept_mean'] = students['marks'] > students['dept_mean']
```

19) Pivot: average marks by department (rows) and city (columns).
```python
pivot = students.pivot_table(values='marks', index='department', columns='city', aggfunc='mean')
```

20) Concat new students rows with students and reset index.
```python
new_students = pd.DataFrame([
    {'std_id': 201, 'std_name': 'Zara', 'city':'Delhi','marks':82,'department':'CS'},
    {'std_id': 202, 'std_name': 'Liam', 'city':'Mumbai','marks':71,'department':'EE'}
])
combined = pd.concat([students, new_students], ignore_index=True)
```

---

If you want any edits (different filenames, additional comments, or a different format), tell me and I will update the file.
