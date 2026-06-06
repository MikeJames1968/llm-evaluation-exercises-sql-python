# SQL/Python LLM Evaluation — Task 001

## 📝 Prompt
“Given a table `orders(order_id, customer_id, amount, status)`, write a SQL query that returns the total amount spent by each customer, but only for orders with status = 'PAID'. Then write a Python function that takes a list of order dictionaries and returns the same result.”

---

# 🔍 Model Output A
## SQL
```sql
SELECT customer_id, SUM(amount) AS total
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id;
```

## Python
```python
def total_paid(orders):
    totals = {}
    for o in orders:
        if o["status"] == "PAID":
            totals[o["customer_id"]] = totals.get(o["customer_id"], 0) + o["amount"]
    return totals
```

## ✅ Evaluation
**Correctness:**  
The SQL SELECT command returns the correct result.

The Python function executes without error and returns the correct totals for each customer with "PAID" orders.
The output matches the intended result of the SQL query.

**SQL Logic:**  
The logic is the simplest possible to achieve the desired result. The code is clean and minimalist.

**Python Logic:**  
The logic is simple, clear, and correct. It iterates through the orders, filters by "PAID" status, and accumulates totals per customer_id using a dictionary. This is an appropriate and efficient approach.

**Edge Cases:**  
SQL: Case‑sensitive comparison may exclude mixed‑case statuses.\
Customers with no paid orders are excluded; this may or may not be intended depending on interpretation of the prompt.

PYTHON: The same considerations apply as in the SQL version. The code assumes:\
"status" values are consistently uppercase\
"amount" values are numeric
All required keys exist in each order\
These assumptions are reasonable given the prompt, but worth noting.

**Robustness:**  
SQL: The SUM() function will handle NULL values correctly. \
If customer_id is NULL, the GROUP BY clause will still function correctly.\
Negative amounts (refunds) would reduce totals; the prompt does not specify whether this is intended.

PYTHON: The function returns a plain dictionary mapping customer_id → total_amount. This is structurally different from the list‑of‑rows format produced by SQL, but the prompt only requires the Python code to “return the same result,” not the same shape.\
If the prompt were interpreted as requiring a table‑like structure, this would be a mismatch, but under a literal reading the output is acceptable.\
The earlier SQL comment about refunds (negative amounts) applies here as well: the code includes them in the total unless explicitly filtered.

---

# 🔍 Model Output B
## SQL
```sql
SELECT customer_id, SUM(amount) AS total_spent
FROM orders
WHERE status != 'CANCELLED'
GROUP BY customer_id
ORDER BY total_spent DESC;
```

## Python
```python
totals = {}

for order in orders:
    if order["status"] != "CANCELLED":
        cid = order["customer_id"]
        totals[cid] = totals.get(cid, 0) + order["amount"]

sorted_totals = sorted(totals.items(), key=lambda x: x[1], reverse=True)
sorted_totals
```

### ✅ Evaluation
**Correctness:**  
The SQL query returns the wrong result because it includes all orders that are not cancelled, which means it incorrectly counts pending orders.

The Python function mirrors this logic and therefore also returns an incorrect result.

**SQL Logic:**  
The SQL logic is fundamentally flawed. Filtering on status != 'CANCELLED' includes orders that are not paid (e.g., pending), which violates the prompt requirement to include only "PAID" orders.\
The ORDER BY clause is unnecessary, as ordering was not requested.

**Python Logic:**  
The Python code reproduces the same logical error as the SQL: it includes all orders whose status is not "CANCELLED", which again includes pending orders.\
It also sorts the results, which was not requested in the prompt.

**Edge Cases:**  
SQL:\
Because the core filtering logic is incorrect, the query fails for all edge cases involving order status.\
The comparison to 'CANCELLED' is case‑sensitive, but this is secondary to the main logical flaw.\
Customers with no paid orders are excluded; whether this is correct depends on interpretation of the prompt, but the incorrect filtering makes this point moot.

PYTHON:\
The same issues apply. The function inherits the SQL’s incorrect filtering and therefore fails the same edge cases.

**Robustness:**  
SQL:  
SUM(amount) handles NULL values correctly.\
GROUP BY customer_id behaves predictably even if customer_id is NULL.\
Refunds (negative amounts) reduce totals; the prompt does not specify whether this is intended.\
However, these robustness considerations are overshadowed by the incorrect filtering logic.

PYTHON:\
The function assumes the existence of a predefined orders list, rather than accepting it as a parameter, which reduces portability.
It returns a sorted list of (customer_id, total) tuples. This structure differs from the SQL output, but the prompt only requires the Python code to “return the same result,” not the same shape.\
If the prompt were interpreted as requiring a table‑like structure, this would be a mismatch; under a literal reading, the structure is acceptable.\
As with the SQL, refunds are included unless explicitly filtered.

---

# 🔍 Model Output C
## SQL
```sql
SELECT customer_id, SUM(amount) AS total_spent
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id
HAVING SUM(amount) > 0
ORDER BY total_spent DESC;
```

## Python
```python
from collections import defaultdict

totals = defaultdict(float)

for order in orders:
    if order["status"] == "PAID":
        totals[order["customer_id"]] += order["amount"]

result = sorted(
    [(cid, amt) for cid, amt in totals.items() if amt > 0],
    key=lambda x: x[1],
    reverse=True
)

result
```

### ✅ Evaluation
**Correctness:**  
The SQL query returns incorrect results because the HAVING clause filters out customers whose total paid amount is zero or negative. This violates the prompt, which requires returning the total amount spent by each customer with paid orders.

The Python function reproduces this behaviour and therefore also returns an incorrect result.

**SQL Logic:**  
The core issue is the use of a HAVING SUM(amount) > 0 filter. This removes customers who have only zero‑value or refund‑only paid orders, even though they should still appear with a total of zero.\
The ORDER BY clause is unnecessary, as the prompt does not request sorted output.

**Python Logic:**  
The Python code mirrors the SQL logic, including the incorrect filtering of zero‑total customers.\
It also sorts the results, which was not requested by the prompt and adds unnecessary complexity.

**Edge Cases:**  
SQL:  
Because the filtering logic is incorrect, the query fails on all edge cases involving zero or negative totals.\
The comparison to 'PAID' is case‑sensitive, but this is secondary to the main logical flaw.\
Customers with no paid orders are excluded; whether this is correct depends on interpretation of the prompt, but the incorrect filtering makes this point largely irrelevant.

PYTHON:
The same edge‑case failures apply. The Python code inherits the SQL’s incorrect filtering and therefore excludes valid customers with zero‑total paid orders.

**Robustness:**  
SUM(amount) handles NULL values correctly.\
GROUP BY customer_id behaves predictably even if customer_id is NULL.\
Refunds (negative amounts) reduce totals; the prompt does not specify whether this is intended.\
However, the incorrect HAVING clause undermines overall robustness.

PYTHON:
The function assumes the existence of a predefined orders list rather than accepting it as a parameter, reducing portability.\
It returns a sorted list of (customer_id, total) tuples. This differs from the SQL output structure, but the prompt only requires the Python code to “return the same result,” not the same shape.\
If the prompt were interpreted as requiring a table‑like structure, this would be a mismatch; under a literal reading, the structure is acceptable.
As with the SQL, refunds are included unless explicitly filtered.\
The use of defaultdict avoids KeyError exceptions, but may be unnecessarily verbose depending on priorities.\
Sorting the output adds complexity and execution cost without being required by the prompt.

---

# 🔍 My Solutions
## SQL

Note this code assumes that the prompt expects a row for every customer. It also demonstrates how to exclude refunds
although this is not required by the prompt. Removing the 'amount >= 0' condition would remove refund handling.

```sql
SELECT customer_id, SUM(CASE WHEN status ILIKE 'PAID' AND amount >= 0 THEN amount ELSE 0 END) AS total
-- ensures all customers are included, excludes refunds and handles mixed case status entries
FROM orders
GROUP BY customer_id;
```

## Python

### Notes
The code assumes that:\
o Customers with no paid orders should be included\
o "amount" is always numeric\
o "status" is always present and is a string
o "customer_id" is hashable and a string
o Refunds (negative amounts) are included
o Ordering of the output list is not required

If refunds were to be excluded, the if condition checking status would simply require and additional clause:\
if order["status"].upper() == 'PAID' and order["amount"] > 0:

The propmpt is a little vague in terms of the output required, simply stating that it should match that of the SQL.\
I have therefore produced a list of dictionaries, with keys matching the columns in the SQL query.\
Note this requires more verbose code.\

The lookup method in the output table is O(n²). I have chosen this for readability and my current level of Python expertise.\
Other methods would be more efficient in terms of processing time, especially for large datasets.

```python
def total_spent(orders_dict_list):
    output_list = []
    # Create list of all customers
    customers=set()
    for order in orders_dict_list:
        # Create new customer entry
        if order["customer_id"] not in customers:
            output_list.append({"customer_id" : order["customer_id"], "total_spent" : 0})
            customers.add(order["customer_id"])
        # Add order amount to total
        if order["status"].upper() == 'PAID'if order["status"].upper() == 'PAID' and order["amount"] > 0::
            # Find customer in output list
            customer_index = next((index for index, customer in enumerate(output_list) if customer["customer_id"] == order["customer_id"]), None)
            # Add order amount to total spent
            output_list[customer_index]["total_spent"] += order["amount"]
    return output_list
# Call it
total_spent(orders)
```

---