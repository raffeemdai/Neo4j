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
