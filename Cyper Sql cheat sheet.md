# Cypher ↔ SQL Cheat Sheet

A side-by-side reference mapping common SQL operations to their Neo4j Cypher equivalents.

---

## 1. Basic Retrieval

| SQL | Cypher |
|---|---|
| `SELECT * FROM Person;` | `MATCH (p:Person) RETURN p;` |
| `SELECT name, age FROM Person;` | `MATCH (p:Person) RETURN p.name, p.age;` |
| `SELECT DISTINCT city FROM Person;` | `MATCH (p:Person) RETURN DISTINCT p.city;` |
| `SELECT TOP 5 * FROM Person;` (or `LIMIT 5`) | `MATCH (p:Person) RETURN p LIMIT 5;` |

---

## 2. Filtering (`WHERE`)

| SQL | Cypher |
|---|---|
| `WHERE age > 30` | `WHERE p.age > 30` |
| `WHERE name = 'Alice'` | `WHERE p.name = 'Alice'` |
| `WHERE age BETWEEN 20 AND 40` | `WHERE p.age >= 20 AND p.age <= 40` |
| `WHERE name IN ('Alice','Bob')` | `WHERE p.name IN ['Alice','Bob']` |
| `WHERE name LIKE 'A%'` | `WHERE p.name STARTS WITH 'A'` |
| `WHERE name LIKE '%ce'` | `WHERE p.name ENDS WITH 'ce'` |
| `WHERE name LIKE '%li%'` | `WHERE p.name CONTAINS 'li'` |
| `WHERE age IS NULL` | `WHERE p.age IS NULL` |
| `WHERE age IS NOT NULL` | `WHERE p.age IS NOT NULL` |
| `WHERE NOT (age > 30)` | `WHERE NOT p.age > 30` |
| `WHERE a AND b OR c` | `WHERE a AND b OR c` (same operators) |

---

## 3. Sorting, Paging

| SQL | Cypher |
|---|---|
| `ORDER BY age DESC` | `ORDER BY p.age DESC` |
| `ORDER BY age ASC, name DESC` | `ORDER BY p.age ASC, p.name DESC` |
| `LIMIT 5 OFFSET 10` | `SKIP 10 LIMIT 5` |

---

## 4. Joins ↔ Pattern Matching

| SQL | Cypher |
|---|---|
| `SELECT p.name, c.name FROM Person p JOIN Company c ON p.company_id = c.id` | `MATCH (p:Person)-[:WORKS_FOR]->(c:Company) RETURN p.name, c.name` |
| `LEFT JOIN` (unmatched rows kept as NULL) | `OPTIONAL MATCH` |
| `INNER JOIN` (only matched rows) | plain `MATCH` |
| Self-join (e.g. friends-of-friends) | `MATCH (a:Person)-[:KNOWS]->(b:Person)-[:KNOWS]->(c:Person)` |
| Multi-table join (3+ tables) | multi-hop pattern: `(a)-[:REL1]->(b)-[:REL2]->(c)` |

### `OPTIONAL MATCH` example
```cypher
MATCH (c:Customer)
OPTIONAL MATCH (c)-[:PURCHASED]->(p:Product {category:'Electronics'})
RETURN c.name, p.name
```
*(Returns customer even if they bought nothing in Electronics — `p.name` will be `null`.)*

---

## 5. Aggregation

| SQL | Cypher |
|---|---|
| `COUNT(*)` | `count(*)` |
| `COUNT(DISTINCT x)` | `count(DISTINCT x)` |
| `SUM(x)` | `sum(x)` |
| `AVG(x)` | `avg(x)` |
| `MIN(x)` / `MAX(x)` | `min(x)` / `max(x)` |
| `GROUP BY city` | implicit — grouping happens automatically by non-aggregated fields in `RETURN`/`WITH` |
| `HAVING COUNT(*) > 1` | `WITH ... , count(*) AS cnt WHERE cnt > 1` |
| `STRING_AGG` / `GROUP_CONCAT` | `collect(x)` |

### Example
```sql
SELECT c.name, COUNT(*) AS purchases
FROM Customer c
JOIN Purchased pu ON c.id = pu.customer_id
GROUP BY c.name
HAVING COUNT(*) > 1;
```
```cypher
MATCH (c:Customer)-[:PURCHASED]->(p:Product)
WITH c, count(*) AS purchases
WHERE purchases > 1
RETURN c.name, purchases;
```

---

## 6. Insert / Create

| SQL | Cypher |
|---|---|
| `INSERT INTO Person (name, age) VALUES ('Alice', 30);` | `CREATE (:Person {name:'Alice', age:30});` |
| `INSERT` with relationship (needs FK) | `CREATE (a)-[:WORKS_FOR]->(c)` |
| Insert only if not exists (`UPSERT`/`MERGE`) | `MERGE (p:Person {personId:'EMP001'})` |

### `MERGE` with `ON CREATE` / `ON MATCH`
```cypher
MERGE (p:Person {personId:'EMP001'})
ON CREATE SET p.name = 'Alice', p.createdAt = timestamp()
ON MATCH SET p.lastSeen = timestamp();
```

---

## 7. Update

| SQL | Cypher |
|---|---|
| `UPDATE Person SET age = 31 WHERE name = 'Alice';` | `MATCH (p:Person {name:'Alice'}) SET p.age = 31;` |
| `UPDATE ... SET a = x, b = y` | `SET p.a = x, p.b = y` |
| Add new column dynamically | `SET p.newProp = value` (schema-free) |
| Replace all properties | `SET p = {name:'Alice', age:31}` |
| Merge properties (keep others) | `SET p += {age:31}` |
| Remove a column value | `REMOVE p.age` or `SET p.age = null` |

---

## 8. Delete

| SQL | Cypher |
|---|---|
| `DELETE FROM Person WHERE name = 'Alice';` | `MATCH (p:Person {name:'Alice'}) DELETE p;` |
| Delete row with FK relationships (cascade) | `MATCH (p:Person {name:'Alice'}) DETACH DELETE p;` |
| `DROP TABLE Person;` | `MATCH (p:Person) DETACH DELETE p;` (drops all nodes of that label) |
| Delete a relationship only | `MATCH (a)-[r:KNOWS]->(b) DELETE r;` |

---

## 9. Subqueries ↔ `WITH`

| SQL | Cypher |
|---|---|
| Subquery / CTE (`WITH x AS (...)`) | `WITH` (pipes results to next clause) |
| `WHERE x IN (SELECT ...)` | `WHERE x IN [subquery via WITH/COLLECT]` |
| Nested subqueries | chained `WITH` blocks |

### Example (CTE-style)
```sql
WITH BigSpenders AS (
  SELECT customer_id, SUM(amount) AS total
  FROM Purchased
  GROUP BY customer_id
  HAVING SUM(amount) > 1000
)
SELECT c.name, b.total
FROM Customer c
JOIN BigSpenders b ON c.id = b.customer_id;
```
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(:Product)
WITH c, sum(buy.amount) AS total
WHERE total > 1000
RETURN c.name, total;
```

---

## 10. Indexes & Constraints

| SQL | Cypher |
|---|---|
| `CREATE INDEX idx ON Person(name);` | `CREATE INDEX FOR (p:Person) ON (p.name);` |
| `ALTER TABLE Person ADD CONSTRAINT UNIQUE(email);` | `CREATE CONSTRAINT FOR (p:Person) REQUIRE p.email IS UNIQUE;` |
| `NOT NULL` constraint | `CREATE CONSTRAINT FOR (p:Person) REQUIRE p.email IS NOT NULL;` |
| Primary key | `CREATE CONSTRAINT FOR (p:Person) REQUIRE p.personId IS UNIQUE;` |

---

## 11. Set Operations

| SQL | Cypher |
|---|---|
| `UNION` | `UNION` (removes duplicates) |
| `UNION ALL` | `UNION ALL` (keeps duplicates) |
| `INTERSECT` | no direct keyword — use list functions or `WHERE x IN` |
| `EXCEPT` / `MINUS` | `WHERE NOT x IN [...]` |

---

## 12. Conditional Logic

| SQL | Cypher |
|---|---|
| `CASE WHEN x THEN a ELSE b END` | `CASE WHEN x THEN a ELSE b END` (same syntax) |
| `COALESCE(a, b, c)` | `coalesce(a, b, c)` |
| `IFNULL(a, b)` | `coalesce(a, b)` |

```cypher
RETURN CASE WHEN p.age > 30 THEN 'Senior' ELSE 'Junior' END AS level
```

---

## 13. Structural / Conceptual Mapping

| SQL Concept | Cypher / Neo4j Equivalent |
|---|---|
| Table | Node label (`:Person`) |
| Row | Node |
| Column | Property |
| Primary Key | Unique constraint property (e.g. `personId`) |
| Foreign Key | Relationship (`-[:WORKS_FOR]->`) |
| Join Table (many-to-many) | Relationship itself (can hold properties, e.g. `[:PURCHASED {amount:100}]`) |
| Schema (rigid) | Schema-optional / flexible properties per node |
| `VIEW` | none direct — use parameterized queries or APOC procedures |
| Transaction (`BEGIN...COMMIT`) | implicit per query, or explicit via driver transactions |

---

## 14. Functions Quick Reference

| Purpose | SQL | Cypher |
|---|---|---|
| String length | `LEN(x)` / `LENGTH(x)` | `size(x)` |
| Uppercase/Lowercase | `UPPER(x)` / `LOWER(x)` | `toUpper(x)` / `toLower(x)` |
| Substring | `SUBSTRING(x,1,3)` | `substring(x,0,3)` |
| Trim | `TRIM(x)` | `trim(x)` |
| Round | `ROUND(x,2)` | `round(x,2)` |
| Current date/time | `GETDATE()` / `NOW()` | `datetime()` / `date()` |
| Cast/convert | `CAST(x AS INT)` | `toInteger(x)` |
| Null check | `ISNULL(x, default)` | `coalesce(x, default)` |

---

## 15. Full Side-by-Side Example

**SQL:**
```sql
SELECT c.name, p.name, pu.amount
FROM Customer c
JOIN Purchased pu ON c.id = pu.customer_id
JOIN Product p ON pu.product_id = p.id
WHERE p.category = 'Electronics'
ORDER BY pu.amount DESC
LIMIT 5;
```

**Cypher:**
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(p:Product)
WHERE p.category = 'Electronics'
RETURN c.name, p.name, buy.amount
ORDER BY buy.amount DESC
LIMIT 5;
```

---

### 🎯 Core Mental Model
- **`MATCH`** ≈ `FROM` + `JOIN` (pattern describes the join path directly)
- **`WHERE`** ≈ `WHERE` (same role)
- **`WITH`** ≈ CTE / subquery pipe — reshape & pass data forward
- **`RETURN`** ≈ `SELECT`
- **Relationships replace foreign keys** — no join tables needed for simple many-to-many; the relationship *is* the join, and can carry its own properties.

- People:
  Alice {personId:"EMP001", name:"Alice", age:30, city:"Atlanta"}
  Bob   {personId:"EMP002", name:"Bob",   age:35, city:"Chicago"}
  Carol {personId:"EMP003", name:"Carol", age:28, city:"Atlanta"}
  Dave  {personId:"EMP004", name:"Dave",  age:41, city:"Boston"}

Companies:
  OpenAI {companyId:"COMP001", name:"OpenAI"}
  Neo4j  {companyId:"COMP002", name:"Neo4j"}

Relationships:
  Alice -[:WORKS_FOR]-> OpenAI
  Bob   -[:WORKS_FOR]-> Neo4j
  Carol -[:WORKS_FOR]-> OpenAI
  Alice -[:KNOWS]-> Bob -[:KNOWS]-> Carol -[:KNOWS]-> Dave

Customers:
  C1 Alice, C2 Bob, C3 Carol

Products:
  P1 Laptop        (Electronics, $1200)
  P2 Mouse          (Electronics, $25)
  P3 Tennis Racket  (Sports, $90)

Purchases:
  C1 -[:PURCHASED {amount:1200}]-> P1
  C1 -[:PURCHASED {amount:50}]->   P2
  C2 -[:PURCHASED {amount:1100}]-> P1
  C2 -[:PURCHASED {amount:180}]->  P3
  C3 -[:PURCHASED {amount:45}]->   P2
  
  
 # Cypher Practice Queries — Full Coverage

Using the sample graph (People, Companies, Customers, Products, Purchases). Each query includes a short explanation and expected result.

---

## SECTION 1: Basic MATCH & RETURN

### 1.1 Return all nodes of a label
```cypher
MATCH (p:Person)
RETURN p;
```
**Explanation:** Fetch every `Person` node with all its properties.

### 1.2 Return specific properties
```cypher
MATCH (p:Person)
RETURN p.name, p.age, p.city;
```
**Result:** Alice/30/Atlanta, Bob/35/Chicago, Carol/28/Atlanta, Dave/41/Boston

### 1.3 Alias properties with `AS`
```cypher
MATCH (p:Person)
RETURN p.name AS personName, p.age AS personAge;
```

### 1.4 Count all nodes of a label
```cypher
MATCH (p:Person)
RETURN count(p) AS totalPeople;
```
**Result:** 4

---

## SECTION 2: WHERE — Filtering

### 2.1 Equality filter
```cypher
MATCH (p:Person)
WHERE p.city = 'Atlanta'
RETURN p.name;
```
**Result:** Alice, Carol

### 2.2 Numeric comparison
```cypher
MATCH (p:Person)
WHERE p.age > 30
RETURN p.name, p.age;
```
**Result:** Bob(35), Dave(41)

### 2.3 Combine conditions (AND / OR)
```cypher
MATCH (p:Person)
WHERE p.city = 'Atlanta' AND p.age < 30
RETURN p.name;
```
**Result:** Carol

### 2.4 IN list filter
```cypher
MATCH (p:Person)
WHERE p.name IN ['Alice', 'Dave']
RETURN p.name, p.city;
```

### 2.5 String filters (STARTS WITH / CONTAINS / ENDS WITH)
```cypher
MATCH (p:Person)
WHERE p.name STARTS WITH 'A'
RETURN p.name;
```
```cypher
MATCH (p:Person)
WHERE p.name CONTAINS 'ar'
RETURN p.name;
```

### 2.6 NOT condition
```cypher
MATCH (p:Person)
WHERE NOT p.city = 'Atlanta'
RETURN p.name, p.city;
```
**Result:** Bob(Chicago), Dave(Boston)

### 2.7 Filter inline in MATCH pattern (alternative to WHERE)
```cypher
MATCH (p:Person {city:'Atlanta'})
RETURN p.name;
```
*(Same result as 2.1 — property filters can go directly in `{}` for simple equality.)*

---

## SECTION 3: Relationship Patterns (Joins)

### 3.1 One-hop relationship
```cypher
MATCH (p:Person)-[:WORKS_FOR]->(c:Company)
RETURN p.name, c.name;
```
**Result:** Alice→OpenAI, Bob→Neo4j, Carol→OpenAI

### 3.2 Filter on relationship target
```cypher
MATCH (p:Person)-[:WORKS_FOR]->(c:Company)
WHERE c.name = 'OpenAI'
RETURN p.name;
```
**Result:** Alice, Carol

### 3.3 Two-hop chained relationship
```cypher
MATCH (a:Person)-[:KNOWS]->(b:Person)-[:KNOWS]->(c:Person)
RETURN a.name, b.name, c.name;
```
**Result:** Alice→Bob→Carol, Bob→Carol→Dave

### 3.4 Variable-length path (1 to 3 hops)
```cypher
MATCH (p:Person {name:'Alice'})-[:KNOWS*1..3]->(friend:Person)
RETURN friend.name;
```
**Result:** Bob, Carol, Dave *(everyone reachable within 3 hops)*

### 3.5 Undirected relationship match
```cypher
MATCH (a:Person)-[:KNOWS]-(b:Person)
WHERE a.name = 'Bob'
RETURN b.name;
```
**Result:** Alice (incoming), Carol (outgoing) — ignores direction

### 3.6 Relationship with properties
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(p:Product)
RETURN c.name, p.name, buy.amount;
```

### 3.7 Filter on relationship property
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(p:Product)
WHERE buy.amount > 100
RETURN c.name, p.name, buy.amount;
```
**Result:** Alice→Laptop(1200), Bob→Laptop(1100), Bob→Tennis Racket(180)

### 3.8 Multiple relationship types (OR pattern)
```cypher
MATCH (p:Person)-[:WORKS_FOR|KNOWS]->(x)
RETURN p.name, labels(x), x.name;
```
**Explanation:** Matches either relationship type.

### 3.9 Reverse direction lookup
```cypher
MATCH (c:Company)<-[:WORKS_FOR]-(p:Person)
WHERE c.name = 'Neo4j'
RETURN p.name;
```
**Result:** Bob

### 3.10 Common connections (shared company)
```cypher
MATCH (a:Person)-[:WORKS_FOR]->(c:Company)<-[:WORKS_FOR]-(b:Person)
WHERE a.name <> b.name
RETURN a.name, b.name, c.name;
```
**Result:** Alice↔Carol via OpenAI (both directions shown)

---

## SECTION 4: OPTIONAL MATCH

### 4.1 Keep all customers even without matching purchase
```cypher
MATCH (c:Customer)
OPTIONAL MATCH (c)-[:PURCHASED]->(p:Product {category:'Sports'})
RETURN c.name, p.name;
```
**Result:** Alice→null, Bob→Tennis Racket, Carol→null
**Explanation:** Unlike `MATCH`, `OPTIONAL MATCH` keeps the customer row even if no Sports purchase exists (returns `null`).

### 4.2 People without any KNOWS relationship (if any existed)
```cypher
MATCH (p:Person)
OPTIONAL MATCH (p)-[:KNOWS]->(friend)
RETURN p.name, friend.name;
```
**Result:** Dave→null (Dave has no outgoing KNOWS)

---

## SECTION 5: Aggregation

### 5.1 Count purchases per customer
```cypher
MATCH (c:Customer)-[:PURCHASED]->(p:Product)
RETURN c.name, count(p) AS numPurchases;
```
**Result:** Alice=2, Bob=2, Carol=1

### 5.2 Sum of purchase amounts per customer
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(:Product)
RETURN c.name, sum(buy.amount) AS totalSpent;
```
**Result:** Alice=1250, Bob=1280, Carol=45

### 5.3 Average product price per category
```cypher
MATCH (p:Product)
RETURN p.category, avg(p.price) AS avgPrice;
```
**Result:** Electronics=612.5, Sports=90

### 5.4 Min / Max price
```cypher
MATCH (p:Product)
RETURN min(p.price) AS cheapest, max(p.price) AS costliest;
```
**Result:** cheapest=25, costliest=1200

### 5.5 Count employees per company
```cypher
MATCH (p:Person)-[:WORKS_FOR]->(c:Company)
RETURN c.name, count(p) AS employeeCount;
```
**Result:** OpenAI=2, Neo4j=1

### 5.6 collect() — gather into a list
```cypher
MATCH (p:Person)-[:WORKS_FOR]->(c:Company)
RETURN c.name, collect(p.name) AS employees;
```
**Result:** OpenAI=[Alice,Carol], Neo4j=[Bob]

### 5.7 HAVING equivalent — filter after aggregation
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(:Product)
WITH c, sum(buy.amount) AS totalSpent
WHERE totalSpent > 200
RETURN c.name, totalSpent;
```
**Result:** Alice(1250), Bob(1280) — Carol excluded (only 45)

### 5.8 Count distinct
```cypher
MATCH (c:Customer)-[:PURCHASED]->(p:Product)
RETURN count(DISTINCT p.category) AS distinctCategories;
```
**Result:** 2

---

## SECTION 6: ORDER BY / SKIP / LIMIT

### 6.1 Order by age descending
```cypher
MATCH (p:Person)
RETURN p.name, p.age
ORDER BY p.age DESC;
```
**Result:** Dave(41), Bob(35), Alice(30), Carol(28)

### 6.2 Order by multiple fields
```cypher
MATCH (p:Person)
RETURN p.city, p.name
ORDER BY p.city ASC, p.name DESC;
```

### 6.3 Top-N pattern (most expensive product)
```cypher
MATCH (p:Product)
RETURN p.name, p.price
ORDER BY p.price DESC
LIMIT 1;
```
**Result:** Laptop, 1200

### 6.4 Pagination (skip + limit)
```cypher
MATCH (p:Person)
RETURN p.name
ORDER BY p.name
SKIP 1
LIMIT 2;
```
**Result:** Bob, Carol *(skips Alice, then takes next 2)*

### 6.5 Top spender per customer using WITH before RETURN
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(:Product)
WITH c, sum(buy.amount) AS totalSpent
ORDER BY totalSpent DESC
LIMIT 1
RETURN c.name, totalSpent;
```
**Result:** Bob, 1280

---

## SECTION 7: WITH — Multi-Stage Queries

### 7.1 Filter → aggregate → filter again
```cypher
MATCH (p:Person)
WHERE p.age > 25
WITH p
MATCH (p)-[:WORKS_FOR]->(c:Company)
RETURN p.name, c.name;
```

### 7.2 Aggregate then re-expand (Top spender's purchases)
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(:Product)
WITH c, sum(buy.amount) AS totalSpent
ORDER BY totalSpent DESC
LIMIT 1
MATCH (c)-[:PURCHASED]->(p:Product)
RETURN c.name, totalSpent, collect(p.name) AS products;
```
**Result:** Bob, 1280, [Laptop, Tennis Racket]

### 7.3 Distinct companies, then list their employees
```cypher
MATCH (p:Person)-[:WORKS_FOR]->(c:Company)
WITH DISTINCT c
MATCH (c)<-[:WORKS_FOR]-(emp:Person)
RETURN c.name, collect(emp.name) AS employees;
```

### 7.4 Friend chain with 2 filtering stages
```cypher
MATCH (p:Person)-[:KNOWS]->(f1:Person)
WHERE p.age < 40
WITH p, f1
MATCH (f1)-[:KNOWS]->(f2:Person)
WHERE f2.city = 'Boston'
RETURN p.name AS person, f1.name AS friend, f2.name AS friendOfFriend;
```
**Result:** Bob → Carol → Dave

---

## SECTION 8: CASE / Conditional Logic

### 8.1 Categorize by age
```cypher
MATCH (p:Person)
RETURN p.name,
  CASE WHEN p.age >= 35 THEN 'Senior'
       WHEN p.age >= 30 THEN 'Mid'
       ELSE 'Junior' END AS ageGroup;
```
**Result:** Alice=Mid, Bob=Senior, Carol=Junior, Dave=Senior

### 8.2 Flag high-value purchases
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(p:Product)
RETURN c.name, p.name, buy.amount,
  CASE WHEN buy.amount > 500 THEN 'High' ELSE 'Low' END AS tier;
```

---

## SECTION 9: String / Utility Functions

### 9.1 Uppercase name
```cypher
MATCH (p:Person)
RETURN toUpper(p.name) AS upperName;
```

### 9.2 String length
```cypher
MATCH (p:Person)
RETURN p.name, size(p.name) AS nameLength;
```

### 9.3 Round a computed value
```cypher
MATCH (c:Customer)-[buy:PURCHASED]->(:Product)
WITH c, avg(buy.amount) AS avgSpend
RETURN c.name, round(avgSpend, 2) AS avgSpendRounded;
```

---

## SECTION 10: MERGE / CREATE / SET / DELETE

### 10.1 Create a new node
```cypher
CREATE (:Person {personId:'EMP005', name:'Eve', age:26, city:'Denver'});
```

### 10.2 MERGE — create only if not exists
```cypher
MERGE (p:Person {personId:'EMP001'})
ON CREATE SET p.name = 'Alice', p.age = 30
ON MATCH SET p.lastChecked = timestamp();
```

### 10.3 Update a property
```cypher
MATCH (p:Person {name:'Alice'})
SET p.age = 31;
```

### 10.4 Add a new relationship
```cypher
MATCH (a:Person {name:'Dave'}), (b:Person {name:'Alice'})
CREATE (a)-[:KNOWS]->(b);
```

### 10.5 Delete a relationship only
```cypher
MATCH (a:Person {name:'Alice'})-[r:KNOWS]->(b:Person {name:'Bob'})
DELETE r;
```

### 10.6 Delete node with all its relationships
```cypher
MATCH (p:Person {name:'Eve'})
DETACH DELETE p;
```

---

## SECTION 11: UNWIND

### 11.1 Turn a list into rows
```cypher
UNWIND ['Electronics', 'Sports'] AS cat
MATCH (p:Product {category: cat})
RETURN cat, p.name;
```

### 11.2 UNWIND a collected list back into rows
```cypher
MATCH (c:Customer)-[:PURCHASED]->(p:Product)
WITH c, collect(p.name) AS items
UNWIND items AS item
RETURN c.name, item;
```

---

## SECTION 12: Combined Real-World Style Queries

### 12.1 People in Atlanta who work for a company and know someone outside Atlanta
```cypher
MATCH (p:Person)-[:WORKS_FOR]->(c:Company)
WHERE p.city = 'Atlanta'
WITH p, c
MATCH (p)-[:KNOWS]->(friend:Person)
WHERE friend.city <> 'Atlanta'
RETURN p.name, c.name, friend.name, friend.city;
```
**Result:** Alice → OpenAI → Bob (Chicago); Carol → OpenAI → Dave (Boston)

### 12.2 Customers who bought Electronics but NOT Sports
```cypher
MATCH (c:Customer)-[:PURCHASED]->(elec:Product {category:'Electronics'})
WHERE NOT EXISTS {
  MATCH (c)-[:PURCHASED]->(:Product {category:'Sports'})
}
RETURN DISTINCT c.name;
```
**Result:** Alice, Carol *(Bob is excluded — he bought Sports too)*

### 12.3 Total revenue per product, ranked
```cypher
MATCH (:Customer)-[buy:PURCHASED]->(p:Product)
WITH p, sum(buy.amount) AS revenue
ORDER BY revenue DESC
RETURN p.name, revenue;
```
**Result:** Laptop=2300, Tennis Racket=180, Mouse=95

### 12.4 Employees of a company who are also customers, with what they bought
```cypher
MATCH (person:Person)-[:WORKS_FOR]->(c:Company)
MATCH (cust:Customer {name: person.name})-[:PURCHASED]->(prod:Product)
RETURN person.name, c.name AS company, collect(prod.name) AS purchases;
```

---

## 🎯 Coverage Summary

| Concept | Covered in Section |
|---|---|
| Basic MATCH/RETURN | 1 |
| WHERE filters (=, >, IN, strings, NOT) | 2 |
| Relationship patterns, multi-hop, variable-length, undirected | 3 |
| OPTIONAL MATCH | 4 |
| Aggregation (count, sum, avg, min/max, collect, HAVING) | 5 |
| ORDER BY / SKIP / LIMIT / Top-N | 6 |
| Multi-stage WITH pipelines | 7 |
| CASE conditional logic | 8 |
| String/utility functions | 9 |
| CREATE / MERGE / SET / DELETE | 10 |
| UNWIND | 11 |
| Combined real-world queries | 12 |

Practice these in order — each section builds on the previous one's concepts.

# Cypher Practice — Gap-Fill Set (Missing Day 1 Topics)

Continues from the earlier practice sheet, using the same dataset (Alice/Bob/Carol/Dave, OpenAI/Neo4j, Customers C1–C3, Products P1–P3). Covers exactly the topics your Day 1 file has that weren't in the first sheet: `REMOVE`, `DELETE` vs `DETACH DELETE`, `UNWIND` bulk upsert, `shortestPath()`, Two Degrees of Separation, `EXISTS` subquery, `CALL` subquery, Constraints, Indexes, `EXPLAIN`/`PROFILE`, Database Objects.

---

## 1. REMOVE — deleting properties and labels

### 1.1 Remove a property
**Explanation:** Deletes a property entirely — the node keeps existing, just without that key. Different from `SET p.age = null`, which sets it to null but the key can still show up in some contexts; `REMOVE` drops it outright.

```cypher
MATCH (p:Person {name:'Alice'})
REMOVE p.age
RETURN p;
```
**Before:** `Alice {personId:"EMP001", name:"Alice", age:30, city:"Atlanta"}`
**After:** `Alice {personId:"EMP001", name:"Alice", city:"Atlanta"}` — `age` is gone, not `null`.

### 1.2 Remove a label
**Explanation:** First add an extra label, then strip it — shows `REMOVE` works on labels the same way it works on properties.

```cypher
// Add a label first
MATCH (p:Person {name:'Alice'})
SET p:Employee;

// Now remove it
MATCH (p:Person {name:'Alice'})
REMOVE p:Employee
RETURN labels(p);
```
**Before:** `labels(Alice) = ['Person', 'Employee']`
**After:** `labels(Alice) = ['Person']`

---

## 2. DELETE vs DETACH DELETE

### 2.1 Plain DELETE — fails on a connected node
**Explanation:** `DELETE` refuses to remove a node that still has relationships attached. Alice has `WORKS_FOR` and `KNOWS` edges, so this throws an error.

```cypher
MATCH (p:Person {name:'Alice'})
DELETE p;
```
**Result:** **Error** — `Cannot delete node<0>, because it still has relationships.` No change made to the graph.

### 2.2 DETACH DELETE — succeeds regardless of connections
**Explanation:** Deletes the node **and** every relationship attached to it, in one atomic step.

```cypher
// Create a throwaway node to demo safely
CREATE (temp:Person {personId:'EMP999', name:'TempUser', age:99, city:'Nowhere'})-[:WORKS_FOR]->(:Company {companyId:'COMPX', name:'TempCo'});

MATCH (p:Person {name:'TempUser'})
DETACH DELETE p;
```
**Result:** `TempUser` node and its `WORKS_FOR` relationship are both removed. (`TempCo` company node remains, now orphaned — relationships are gone, node isn't.)

---

## 3. UNWIND — bulk upsert pattern

### 3.1 Explode a literal list (read-only, recap)
```cypher
UNWIND ['Neo4j', 'Python', 'AWS'] AS skill
RETURN skill;
```
**Result:** 3 rows — `Neo4j`, `Python`, `AWS`.

### 3.2 Bulk upsert — add multiple customers in one query
**Explanation:** Combines `UNWIND` with `MERGE` to load many records at once without creating duplicates — the standard pattern for bulk ingestion (CSV, API payload, etc.).

```cypher
UNWIND [
  {customerId: 'C4', name: 'Diana'},
  {customerId: 'C5', name: 'Ethan'}
] AS row
MERGE (c:Customer {customerId: row.customerId})
SET c.name = row.name
RETURN c;
```
**Before:** 3 customers (C1 Alice, C2 Bob, C3 Carol)
**After:** 5 customers — Diana and Ethan added, both with **zero** `PURCHASED` relationships. Keep them around — Sections 6 and 7 below use them to show the difference between `EXISTS` and `CALL`.

---

## 4. Paths — shortestPath()

**Data used:** the `KNOWS` chain — `Alice → Bob → Carol → Dave`.

### 4.1 1 to 3 hops (recap)
```cypher
MATCH path = (a:Person {name:'Alice'})-[:KNOWS*1..3]->(b:Person)
RETURN path;
```
**Result:** 3 paths — `Alice→Bob`, `Alice→Bob→Carol`, `Alice→Bob→Carol→Dave`.

### 4.2 shortestPath() between two named people
**Explanation:** Finds the minimum-hop connection between two nodes. The `-` (no arrow) means it searches in either direction.

```cypher
MATCH p = shortestPath(
    (a:Person {name:'Alice'})-[:KNOWS*]-(d:Person {name:'Dave'})
)
RETURN p, length(p) AS hops;
```
**Result:** `Alice → Bob → Carol → Dave`, `hops = 3` — the only path here, so also the shortest.

---

## 5. Two Degrees of Separation

**Data used:** same `KNOWS` chain.

**Explanation:** Walk exactly 2 `KNOWS` hops from a starting person, then exclude the starting person and anyone directly (1-hop) connected — so what's left is *exactly* 2 degrees away, not 1 or 3.

```cypher
MATCH (alice:Person {name:'Alice'})
      -[:KNOWS]->(:Person)
      -[:KNOWS]->(candidate:Person)
WHERE candidate <> alice
  AND NOT (alice)-[:KNOWS]-(candidate)
RETURN DISTINCT candidate.name;
```
**Result:** `Carol` only.
- Path traced: `Alice → Bob → Carol` (Bob is the 1-hop friend, Carol is the 2-hop candidate).
- Dave is 3 hops away, so he's correctly excluded too.

---

## 6. EXISTS Subquery

**Data used:** all 5 customers after Section 3.2 — only Alice/Bob/Carol have purchases; Diana/Ethan have none.

**Explanation:** `EXISTS { ... }` is a boolean filter — true if the inner pattern matches at least once. It doesn't return any data from inside, just filters rows.

```cypher
MATCH (c:Customer)
WHERE EXISTS {
    MATCH (c)-[:PURCHASED]->(:Product {category:'Electronics'})
}
RETURN c.name;
```
**Result:** `Alice`, `Bob`, `Carol` — each bought at least one Electronics item. Diana and Ethan are **excluded entirely** — no purchases at all, so the inner pattern never matches.

---

## 7. CALL Subquery

**Data used:** same 5 customers.

**Explanation:** `CALL (c) { ... }` runs once **per row** from the outer `MATCH`, computing a value independently for each — including customers with zero matches, since `count()` on an empty match returns `0` instead of dropping the row.

```cypher
MATCH (c:Customer)
CALL (c) {
    MATCH (c)-[:PURCHASED]->(p:Product)
    RETURN count(p) AS purchaseCount
}
RETURN c.name, purchaseCount;
```
**Result:**

| c.name | purchaseCount |
|---|---|
| Alice | 2 |
| Bob | 2 |
| Carol | 1 |
| Diana | 0 |
| Ethan | 0 |

**Key contrast with Section 6:** `EXISTS` *filters out* Diana/Ethan; `CALL` *keeps* them with `0`. This is the classic interview question — "when would you use `EXISTS` vs a `CALL` subquery?"

---

## 8. Constraints

### 8.1 Create a uniqueness constraint
**Explanation:** Ensures no two `Customer` nodes can share a `customerId`. Creating it also auto-creates a backing index (see Section 9).

```cypher
CREATE CONSTRAINT customer_id_unique IF NOT EXISTS
FOR (c:Customer)
REQUIRE c.customerId IS UNIQUE;
```
**Before:** `SHOW CONSTRAINTS` → 0 rows.
**After:** `SHOW CONSTRAINTS` → 1 row (`customer_id_unique`, type `UNIQUENESS`, on `Customer.customerId`).

### 8.2 View constraints
```cypher
SHOW CONSTRAINTS;
```
**Try it:** Attempt `CREATE (:Customer {customerId:'C1', name:'Duplicate'})` — it should fail with `ConstraintValidationFailed` since `C1` already exists.

---

## 9. Indexes

### 9.1 Create an index
**Explanation:** Adds a range index on `Customer.name` so exact-match lookups don't scan every node.

```cypher
CREATE INDEX customer_name_index IF NOT EXISTS
FOR (c:Customer)
ON (c.name);
```
**Before:** `SHOW INDEXES` → 1 row (the auto-created backing index from the constraint above).
**After:** `SHOW INDEXES` → 2 rows — the constraint-backed index + `customer_name_index`.

### 9.2 View indexes
```cypher
SHOW INDEXES;
```

---

## 10. Query Plan — EXPLAIN vs PROFILE

### 10.1 EXPLAIN — plan only, no execution
**Explanation:** Shows the planned strategy **without running** the query — zero rows returned.

```cypher
EXPLAIN
MATCH (c:Customer {customerId:'C1'})
RETURN c;
```
**Result:** An execution plan (likely `NodeUniqueIndexSeek`, thanks to the constraint) — **0 data rows**.

### 10.2 PROFILE — executes and shows runtime stats
**Explanation:** Actually runs the query **and** attaches real stats (DB hits, rows produced, time) to the same kind of plan.

```cypher
PROFILE
MATCH (c:Customer {customerId:'C1'})
RETURN c;
```
**Result:** 1 row — `Customer {customerId:'C1', name:'Alice'}` — plus the annotated plan showing DB hits for the index seek.

---

## 11. Database Objects — Admin, Schema, Procedural

**Explanation:** `SHOW` commands inspect objects Neo4j manages *around* the graph — they don't touch nodes/relationships.

### 11.1 Schema objects
```cypher
SHOW CONSTRAINTS;
SHOW INDEXES;
```
**Result:** 1 constraint, 2 indexes (state left behind by Sections 8–9).

### 11.2 Admin / server objects
```cypher
SHOW DATABASES;
SHOW USERS;
SHOW ROLES;
```
**Result:** `SHOW DATABASES` → `neo4j` and `system` (always present). `SHOW USERS`/`SHOW ROLES` require Enterprise/Aura — Community edition doesn't support multi-user role security.

### 11.3 Procedural objects
```cypher
SHOW PROCEDURES;
SHOW FUNCTIONS;
```
**Result:** Long built-in lists — e.g. `dbms.components`, `db.schema.visualization` among procedures; `count()`, `labels()`, `collect()` among functions. Plus APOC procedures if that plugin is installed.

---

## 🎯 Coverage Checklist (combined with earlier sheet)

| Topic | Status |
|---|---|
| CREATE / MATCH / RETURN / WHERE | ✅ (earlier sheet) |
| ORDER BY / SKIP / LIMIT | ✅ (earlier sheet) |
| MERGE (ON CREATE/ON MATCH) | ✅ (earlier sheet, basic) |
| SET (update props, labels, replace) | ✅ (earlier sheet, basic) |
| **REMOVE** | ✅ this sheet |
| **DELETE vs DETACH DELETE (fail case)** | ✅ this sheet |
| OPTIONAL MATCH | ✅ (earlier sheet) |
| WITH pipelining | ✅ (earlier sheet) |
| Aggregations | ✅ (earlier sheet) |
| UNWIND (explode + **bulk upsert**) | ✅ this sheet |
| Variable-length paths | ✅ (earlier sheet) |
| **shortestPath()** | ✅ this sheet |
| **Two Degrees of Separation** | ✅ this sheet |
| **EXISTS subquery** | ✅ this sheet |
| **CALL subquery** | ✅ this sheet |
| **Constraints** | ✅ this sheet |
| **Indexes** | ✅ this sheet |
| **EXPLAIN / PROFILE** | ✅ this sheet |
| **Database Objects (SHOW)** | ✅ this sheet |
| Mini Project Recap | ✅ (earlier sheet, equivalent queries) |

**Together, the two sheets now cover 100% of your Day 1 file's topic list.**
