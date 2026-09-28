# Neo4j Movies Graph — Cypher Exercises: Questions & Corrected Answers

Source: the exercises in `github.com/lstarke/neo4j` (README), which follow the older GraphAcademy "Querying with Cypher" Movies exercises.
Each item below is a **Question** followed by a **corrected Answer** for current Neo4j (Neo4j 5.x and 2025/2026 releases, Cypher 5 and Cypher 25).

> ⚠️ **Testing note:** I checked these against the Cypher manual's deprecation/removal notes and by reading each query, but I could not run them against a live Neo4j instance. Run each on your own database before relying on it. Results depend on the GraphAcademy Movies dataset (which includes `FOLLOWS` and `REVIEWED` relationships).

## Legend

| Mark | Meaning |
|---|---|
| ✅ | Original query is still correct — unchanged |
| 🔧 | **Deprecated/removed syntax** — corrected |
| 🐞 | Bug or omission in the original (not a deprecation) — corrected |
| ✨ | Still works, but a cleaner/safer form is shown |

## What was changed (summary)

| Exercise | Type | Problem in original | Fix |
|---|---|---|---|
| 1.2, 3.1, 8.10, 9.8 | ✨ | `call db.schema.visualization` without parentheses | `CALL db.schema.visualization()` |
| 2.3, 9.7 | ✨ | `call db.propertyKeys` without parentheses | `CALL db.propertyKeys()` |
| 3.2 | 🐞 | Returned `m.name`; Movie has `title` | `m.title` |
| 4.6 | 🔧 | `NOT exists(a.born)` | `a.born IS NULL` |
| 4.7 | 🔧 | `exists(rel.roles)` | `rel.roles IS NOT NULL` |
| 4.8 | ✨ | `'James'RETURN` missing space | Add space |
| 4.11 | 🔧 | `exists((p2)-[:DIRECTED]->(m))` | `EXISTS { (p2)-[:DIRECTED]->(m) }` |
| 5.12 | 🔧 | `size((:Person)-[:DIRECTED]->(m))` | `COUNT { (:Person)-[:DIRECTED]->(m) }` |
| 8.15 | ✨ | `SET ... = null` to remove | `REMOVE m.lengthInMinutes` |
| 9.5 | 🐞 | Fragment with no `MATCH` | Full query |
| 10.1 | ✨ | Undirected match for delete | Directed match |
| 11.9, 11.10 | 🔧 | `NOT EXISTS (p.born)` | `p.born IS NULL` |
| 13.1 | 🐞 | "View query plan" but no `EXPLAIN` | Add `EXPLAIN` |


# Neo4j Movies Sample Graph – Cypher Practice Data

This script creates a sample **Movies graph** for practicing Neo4j Cypher queries.

## Graph Model

### Node Labels
- `Person`
- `Movie`

### Relationship Types
- `ACTED_IN`
- `DIRECTED`
- `WROTE`
- `PRODUCED`
- `REVIEWED`
- `FOLLOWS`

---

## Optional – Clear Existing Database

> Uncomment this query if you want to delete all existing nodes and relationships.

```cypher
MATCH (n)
DETACH DELETE n;
```

---

# 1. Create Person Nodes

```cypher
UNWIND [
  {name: 'Keanu Reeves', born: 1964},
  {name: 'Laurence Fishburne', born: 1961},
  {name: 'Carrie-Anne Moss', born: 1967},
  {name: 'Hugo Weaving', born: 1960},
  {name: 'Lilly Wachowski', born: 1967},
  {name: 'Lana Wachowski', born: 1965},
  {name: 'Joel Silver', born: 1952},
  {name: 'Dina Meyer', born: 1968},
  {name: 'Robert Longo', born: 1953},
  {name: 'Gene Hackman', born: 1930},
  {name: 'Brooke Langton', born: 1970},
  {name: 'Orlando Jones', born: 1968},
  {name: 'Howard Deutch', born: 1950},
  {name: 'Al Pacino', born: 1940},
  {name: 'Charlize Theron', born: 1975},
  {name: 'Taylor Hackford', born: 1944},
  {name: 'Tom Cruise', born: 1962},
  {name: 'Jack Nicholson', born: 1937},
  {name: 'Demi Moore', born: 1962},
  {name: 'Kevin Bacon', born: 1958},
  {name: 'James Marshall', born: 1967},
  {name: 'Rob Reiner', born: 1947},
  {name: 'Aaron Sorkin', born: 1961},
  {name: 'Tom Skerritt', born: 1933},
  {name: 'Val Kilmer', born: 1959},
  {name: 'Kelly McGillis', born: 1957},
  {name: 'Tony Scott', born: 1944},
  {name: 'Cuba Gooding Jr.', born: 1968},
  {name: 'Renee Zellweger', born: 1969},
  {name: 'Cameron Crowe', born: 1957},
  {name: 'Tom Hanks', born: 1956},
  {name: 'Gary Sinise', born: 1955},
  {name: 'Bill Paxton', born: 1955},
  {name: 'Ron Howard', born: 1954},
  {name: 'Brian Grazer', born: 1951},
  {name: 'Liv Tyler', born: 1977},
  {name: 'Tom Everett Scott', born: 1970},
  {name: 'Helen Hunt', born: 1963},
  {name: 'Robert Zemeckis', born: 1952},
  {name: 'Audrey Tautou', born: 1976},
  {name: 'Ian McKellen', born: 1939},
  {name: 'Emile Hirsch', born: 1985},
  {name: 'Susan Sarandon', born: 1946},
  {name: 'Matthew Fox', born: 1966},
  {name: 'Christina Ricci', born: 1980},
  {name: 'Robin Williams', born: 1951},
  {name: 'Monica Potter', born: 1971},
  {name: 'Tom Shadyac', born: 1958},
  {name: 'Jessica Thompson'},
  {name: 'James Thompson'},
  {name: 'Angela Scope'},
  {name: 'Paul Blythe'}
] AS row

MERGE (p:Person {name: row.name})
SET p += row;
```

---

# 2. Create Movie Nodes

```cypher
UNWIND [
  {
    title: 'The Matrix',
    released: 1999,
    tagline: 'Reality is not what it seems'
  },
  {
    title: 'The Matrix Reloaded',
    released: 2003,
    tagline: 'The fight for the real world continues'
  },
  {
    title: 'The Matrix Revolutions',
    released: 2003,
    tagline: 'The final battle for both worlds'
  },
  {
    title: 'Johnny Mnemonic',
    released: 1995,
    tagline: 'A courier with a head full of secrets'
  },
  {
    title: 'The Replacements',
    released: 2000,
    tagline: 'Second-string players, first-class heart'
  },
  {
    title: 'The Devil\'s Advocate',
    released: 1997,
    tagline: 'Ambition has a price'
  },
  {
    title: 'A Few Good Men',
    released: 1992,
    tagline: 'The truth is worth fighting for'
  },
  {
    title: 'Top Gun',
    released: 1986,
    tagline: 'Fast jets, faster egos'
  },
  {
    title: 'Jerry Maguire',
    released: 2000,
    tagline: 'A sports agent finds love and a conscience'
  },
  {
    title: 'Apollo 13',
    released: 1995,
    tagline: 'A mission that became a fight to get home'
  },
  {
    title: 'That Thing You Do!',
    released: 1996,
    tagline: 'One hit wonder, one unforgettable summer'
  },
  {
    title: 'Cast Away',
    released: 2000,
    tagline: 'Stranded, and learning to survive'
  },
  {
    title: 'The Polar Express',
    released: 2004,
    tagline: 'All aboard for a magical night'
  },
  {
    title: 'The Da Vinci Code',
    released: 2006,
    tagline: 'Ancient secrets, modern danger'
  },
  {
    title: 'Speed Racer',
    released: 2008,
    tagline: 'Racing for family and glory'
  },
  {
    title: 'Patch Adams',
    released: 1998,
    tagline: 'Medicine needs a little more love and laughter'
  }
] AS row

MERGE (m:Movie {title: row.title})
SET m += row;
```

---

# 3. Create ACTED_IN Relationships

```cypher
UNWIND [
  {p: 'Keanu Reeves', m: 'The Matrix', roles: ['Neo']},
  {p: 'Laurence Fishburne', m: 'The Matrix', roles: ['Morpheus']},
  {p: 'Carrie-Anne Moss', m: 'The Matrix', roles: ['Trinity']},
  {p: 'Hugo Weaving', m: 'The Matrix', roles: ['Agent Smith']},

  {p: 'Keanu Reeves', m: 'The Matrix Reloaded', roles: ['Neo']},
  {p: 'Laurence Fishburne', m: 'The Matrix Reloaded', roles: ['Morpheus']},
  {p: 'Carrie-Anne Moss', m: 'The Matrix Reloaded', roles: ['Trinity']},
  {p: 'Hugo Weaving', m: 'The Matrix Reloaded', roles: ['Agent Smith']},

  {p: 'Keanu Reeves', m: 'The Matrix Revolutions', roles: ['Neo']},
  {p: 'Laurence Fishburne', m: 'The Matrix Revolutions', roles: ['Morpheus']},
  {p: 'Carrie-Anne Moss', m: 'The Matrix Revolutions', roles: ['Trinity']},
  {p: 'Hugo Weaving', m: 'The Matrix Revolutions', roles: ['Agent Smith']},

  {p: 'Keanu Reeves', m: 'Johnny Mnemonic', roles: ['Johnny Mnemonic']},
  {p: 'Dina Meyer', m: 'Johnny Mnemonic', roles: ['Jane']},

  {p: 'Keanu Reeves', m: 'The Replacements', roles: ['Shane Falco']},
  {p: 'Gene Hackman', m: 'The Replacements', roles: ['Jimmy McGinty']},
  {p: 'Brooke Langton', m: 'The Replacements', roles: ['Annabelle Farrell']},
  {p: 'Orlando Jones', m: 'The Replacements', roles: ['Clifford Franklin']},

  {p: 'Keanu Reeves', m: 'The Devil\'s Advocate', roles: ['Kevin Lomax']},
  {p: 'Al Pacino', m: 'The Devil\'s Advocate', roles: ['John Milton']},
  {p: 'Charlize Theron', m: 'The Devil\'s Advocate', roles: ['Mary Ann Lomax']},

  {p: 'Tom Cruise', m: 'A Few Good Men', roles: ['Lt. Daniel Kaffee']},
  {p: 'Jack Nicholson', m: 'A Few Good Men', roles: ['Col. Nathan R. Jessup']},
  {p: 'Demi Moore', m: 'A Few Good Men', roles: ['Lt. Cdr. JoAnne Galloway']},
  {p: 'Kevin Bacon', m: 'A Few Good Men', roles: ['Capt. Jack Ross']},
  {p: 'James Marshall', m: 'A Few Good Men', roles: ['Pfc. Louden Downey']},

  {p: 'Tom Cruise', m: 'Top Gun', roles: ['Maverick']},
  {p: 'Tom Skerritt', m: 'Top Gun', roles: ['Viper']},
  {p: 'Val Kilmer', m: 'Top Gun', roles: ['Iceman']},
  {p: 'Kelly McGillis', m: 'Top Gun', roles: ['Charlie']},

  {p: 'Tom Cruise', m: 'Jerry Maguire', roles: ['Jerry Maguire']},
  {p: 'Cuba Gooding Jr.', m: 'Jerry Maguire', roles: ['Rod Tidwell']},
  {p: 'Renee Zellweger', m: 'Jerry Maguire', roles: ['Dorothy Boyd']},

  {p: 'Tom Hanks', m: 'Apollo 13', roles: ['Jim Lovell']},
  {p: 'Gary Sinise', m: 'Apollo 13', roles: ['Ken Mattingly']},
  {p: 'Kevin Bacon', m: 'Apollo 13', roles: ['Jack Swigert']},
  {p: 'Bill Paxton', m: 'Apollo 13', roles: ['Fred Haise']},

  {p: 'Tom Hanks', m: 'That Thing You Do!', roles: ['Mr. White']},
  {p: 'Liv Tyler', m: 'That Thing You Do!', roles: ['Faye Dolan']},
  {p: 'Tom Everett Scott', m: 'That Thing You Do!', roles: ['Guy Patterson']},

  {p: 'Tom Hanks', m: 'Cast Away', roles: ['Chuck Noland']},
  {p: 'Helen Hunt', m: 'Cast Away', roles: ['Kelly Frears']},

  {
    p: 'Tom Hanks',
    m: 'The Polar Express',
    roles: ['Hero Boy\'s Father', 'Conductor']
  },

  {p: 'Tom Hanks', m: 'The Da Vinci Code', roles: ['Robert Langdon']},
  {p: 'Audrey Tautou', m: 'The Da Vinci Code', roles: ['Sophie Neveu']},
  {p: 'Ian McKellen', m: 'The Da Vinci Code', roles: ['Sir Leigh Teabing']},

  {p: 'Emile Hirsch', m: 'Speed Racer', roles: ['Speed Racer']},
  {p: 'Susan Sarandon', m: 'Speed Racer', roles: ['Mom Racer']},
  {p: 'Matthew Fox', m: 'Speed Racer', roles: ['Racer X']},
  {p: 'Christina Ricci', m: 'Speed Racer', roles: ['Trixie']},

  {p: 'Robin Williams', m: 'Patch Adams', roles: ['Patch Adams']},
  {p: 'Monica Potter', m: 'Patch Adams', roles: ['Carin Fisher']}
] AS row

MATCH
  (p:Person {name: row.p}),
  (m:Movie {title: row.m})

MERGE (p)-[a:ACTED_IN]->(m)

SET a.roles = row.roles;
```

---

# 4. Create DIRECTED Relationships

```cypher
UNWIND [
  ['Lilly Wachowski', 'The Matrix'],
  ['Lana Wachowski', 'The Matrix'],
  ['Lilly Wachowski', 'The Matrix Reloaded'],
  ['Lana Wachowski', 'The Matrix Reloaded'],
  ['Lilly Wachowski', 'The Matrix Revolutions'],
  ['Lana Wachowski', 'The Matrix Revolutions'],
  ['Robert Longo', 'Johnny Mnemonic'],
  ['Howard Deutch', 'The Replacements'],
  ['Taylor Hackford', 'The Devil\'s Advocate'],
  ['Rob Reiner', 'A Few Good Men'],
  ['Tony Scott', 'Top Gun'],
  ['Cameron Crowe', 'Jerry Maguire'],
  ['Ron Howard', 'Apollo 13'],
  ['Tom Hanks', 'That Thing You Do!'],
  ['Robert Zemeckis', 'Cast Away'],
  ['Robert Zemeckis', 'The Polar Express'],
  ['Ron Howard', 'The Da Vinci Code'],
  ['Lilly Wachowski', 'Speed Racer'],
  ['Lana Wachowski', 'Speed Racer'],
  ['Tom Shadyac', 'Patch Adams']
] AS pair

MATCH
  (p:Person {name: pair[0]}),
  (m:Movie {title: pair[1]})

MERGE (p)-[:DIRECTED]->(m);
```

---

# 5. Create WROTE Relationships

```cypher
UNWIND [
  ['Lilly Wachowski', 'The Matrix'],
  ['Lana Wachowski', 'The Matrix'],
  ['Lilly Wachowski', 'The Matrix Reloaded'],
  ['Lana Wachowski', 'The Matrix Reloaded'],
  ['Lilly Wachowski', 'The Matrix Revolutions'],
  ['Lana Wachowski', 'The Matrix Revolutions'],
  ['Aaron Sorkin', 'A Few Good Men'],
  ['Cameron Crowe', 'Jerry Maguire'],
  ['Tom Hanks', 'That Thing You Do!'],
  ['Lilly Wachowski', 'Speed Racer'],
  ['Lana Wachowski', 'Speed Racer']
] AS pair

MATCH
  (p:Person {name: pair[0]}),
  (m:Movie {title: pair[1]})

MERGE (p)-[:WROTE]->(m);
```

---

# 6. Create PRODUCED Relationships

```cypher
UNWIND [
  ['Joel Silver', 'The Matrix'],
  ['Joel Silver', 'The Matrix Reloaded'],
  ['Joel Silver', 'The Matrix Revolutions'],
  ['Joel Silver', 'Speed Racer'],
  ['Brian Grazer', 'Apollo 13'],
  ['Brian Grazer', 'The Da Vinci Code']
] AS pair

MATCH
  (p:Person {name: pair[0]}),
  (m:Movie {title: pair[1]})

MERGE (p)-[:PRODUCED]->(m);
```

---

# 7. Create REVIEWED Relationships

The `REVIEWED` relationship contains two properties:

- `rating`
- `summary`

```cypher
UNWIND [
  {
    p: 'Jessica Thompson',
    m: 'The Matrix',
    rating: 92,
    summary: 'Groundbreaking, and a lot of Fun'
  },
  {
    p: 'Angela Scope',
    m: 'The Matrix',
    rating: 84,
    summary: 'Great action, slightly confusing plot'
  },
  {
    p: 'Paul Blythe',
    m: 'The Replacements',
    rating: 68,
    summary: 'Fun sports comedy, light and easy'
  },
  {
    p: 'Jessica Thompson',
    m: 'The Replacements',
    rating: 62,
    summary: 'Predictable but harmless'
  },
  {
    p: 'Jessica Thompson',
    m: 'A Few Good Men',
    rating: 88,
    summary: 'Sharp courtroom drama'
  },
  {
    p: 'James Thompson',
    m: 'Cast Away',
    rating: 90,
    summary: 'Powerful and moving'
  },
  {
    p: 'Angela Scope',
    m: 'Cast Away',
    rating: 64,
    summary: 'Slow in places'
  },
  {
    p: 'Paul Blythe',
    m: 'The Da Vinci Code',
    rating: 55,
    summary: 'Not as Fun as the book'
  },
  {
    p: 'Jessica Thompson',
    m: 'The Da Vinci Code',
    rating: 72,
    summary: 'Fun puzzle-box thriller'
  },
  {
    p: 'James Thompson',
    m: 'The Polar Express',
    rating: 70,
    summary: 'Fun for the whole family'
  },
  {
    p: 'Paul Blythe',
    m: 'Speed Racer',
    rating: 60,
    summary: 'Colorful and fun'
  },
  {
    p: 'Jessica Thompson',
    m: 'Jerry Maguire',
    rating: 86,
    summary: 'Warm and funny'
  }
] AS row

MATCH
  (p:Person {name: row.p}),
  (m:Movie {title: row.m})

MERGE (p)-[r:REVIEWED]->(m)

SET
  r.rating = row.rating,
  r.summary = row.summary;
```

---

# 8. Create FOLLOWS Relationships

```cypher
UNWIND [
  ['James Thompson', 'Jessica Thompson'],
  ['Angela Scope', 'Jessica Thompson'],
  ['Paul Blythe', 'Angela Scope']
] AS pair

MATCH
  (a:Person {name: pair[0]}),
  (b:Person {name: pair[1]})

MERGE (a)-[:FOLLOWS]->(b);
```

---

# 9. Verify Node Counts

Expected:

- `Movie` = **16**
- `Person` = **52**

```cypher
MATCH (n)

RETURN
  labels(n)[0] AS label,
  count(*) AS total

ORDER BY label;
```

---

# 10. Verify Relationship Counts

Expected:

| Relationship | Count |
|---|---:|
| ACTED_IN | 52 |
| DIRECTED | 20 |
| WROTE | 11 |
| PRODUCED | 6 |
| REVIEWED | 12 |
| FOLLOWS | 3 |
| **Total** | **104** |

```cypher
MATCH ()-[r]->()

RETURN
  type(r) AS relationship,
  count(*) AS total

ORDER BY relationship;
```

---

# Quick Graph Structure

```text
(Person)-[:ACTED_IN]->(Movie)

(Person)-[:DIRECTED]->(Movie)

(Person)-[:WROTE]->(Movie)

(Person)-[:PRODUCED]->(Movie)

(Person)-[:REVIEWED {
    rating,
    summary
}]->(Movie)

(Person)-[:FOLLOWS]->(Person)
```

---

# Memory Trick

Think about the Movies graph as:

```text
PERSON
  |
  |-- ACTED_IN ---> MOVIE
  |
  |-- DIRECTED ---> MOVIE
  |
  |-- WROTE ------> MOVIE
  |
  |-- PRODUCED ---> MOVIE
  |
  |-- REVIEWED ---> MOVIE
  |
  |-- FOLLOWS ----> PERSON
```

### Easy Memory

```text
Person → Movie

Actor
Director
Writer
Producer
Reviewer

Person → Person

Follower
```
---

# Part 1 — Retrieve nodes

### Exercise 1.1: Retrieve all nodes from the database ✅
```cypher
MATCH (n) RETURN n
```
*Tip: on a large database add `LIMIT 100`.*

### Exercise 1.2: Examine the schema of your database ✨
```cypher
CALL db.schema.visualization()
```

### Exercise 1.3: Retrieve all Person nodes ✅
```cypher
MATCH (p:Person) RETURN p
```

### Exercise 1.4: Retrieve all Movie nodes ✅
```cypher
MATCH (m:Movie) RETURN m
```

---

# Part 2 — Property filters and projections

### Exercise 2.1: Retrieve all movies released in a specific year ✅
```cypher
MATCH (m:Movie {released: 2003}) RETURN m
```

### Exercise 2.2: View the retrieved results as a table ✅
```cypher
MATCH (m:Movie {released: 2003}) RETURN m.title, m.released, m.tagline
```

### Exercise 2.3: Query the database for all property keys ✨
```cypher
CALL db.propertyKeys() ;  // this will give all properties from db



MATCH (m:Movie)
RETURN  properties(m);  // this will give properties related to movie

```

### Exercise 2.4a: Movies released in a year — return titles ✅
```cypher
MATCH (m:Movie {released: 2006}) RETURN m.title
```

### Exercise 2.4b: Movies released in a year — title, tagline, released ✅
```cypher
MATCH (m:Movie {released: 2004}) RETURN m.title, m.tagline, m.released
```

### Exercise 2.5: Display title, released, and tagline for every Movie ✅
```cypher
MATCH (m:Movie) RETURN m.title, m.tagline, m.released
```

### Exercise 2.6: Display more user-friendly headers ✅
```cypher
MATCH (m:Movie)
RETURN m.title AS title, m.tagline AS tagline, m.released AS releaseYear
```
*(The original used Portuguese aliases `titulo`, `tag`, `datalancamento` — any alias works.)*

---

# Part 3 — Relationships

### Exercise 3.1: Display the schema of the database ✨
```cypher
CALL db.schema.visualization()
```

### Exercise 3.2: Retrieve all people who wrote the movie *Speed Racer* 🐞
**Original bug:** returned `m.name`, but `Movie` nodes have `title` (so that column was always `null`).
```cypher
MATCH (p:Person)-[rel:WROTE]->(m:Movie {title: 'Speed Racer'})
RETURN p.name, rel, m.title
```

**Taking it further**

Writers of another movie:
```cypher
MATCH (p:Person)-[rel:WROTE]->(m:Movie {title: 'A Few Good Men'})
RETURN p, rel, m
```
Actors in a particular movie:
```cypher
MATCH (p:Person)-[rel:ACTED_IN]->(m:Movie {title: 'Speed Racer'})
RETURN p, rel, m
```
Directors of a particular movie:
```cypher
MATCH (p:Person)-[rel:DIRECTED]->(m:Movie {title: 'Speed Racer'})
RETURN p, rel, m
```

### Exercise 3.3: Retrieve all movies connected to Tom Hanks ✅
```cypher
MATCH (p:Person {name: 'Tom Hanks'})-[rel]->(m:Movie)
RETURN p, rel, m
```
Another actor:
```cypher
MATCH (p:Person {name: 'Tom Cruise'})-[rel]->(m:Movie)
RETURN p, rel, m
```
All people connected to a particular movie:
```cypher
MATCH (p:Person)-[rel]->(m:Movie {title: 'The Matrix'})
RETURN p, rel, m
```

### Exercise 3.4: Relationship types Tom Hanks has with movies ✅
```cypher
MATCH (p:Person {name: 'Tom Hanks'})-[rel]->(m:Movie)
RETURN type(rel)
```
Another actor:
```cypher
MATCH (p:Person {name: 'Tom Cruise'})-[rel]->(m:Movie)
RETURN type(rel)
```

### Exercise 3.5: Roles that Tom Hanks acted in ✅
```cypher
MATCH (p:Person {name: 'Tom Hanks'})-[r:ACTED_IN]->(m:Movie)
RETURN p.name, r.roles
```
Another actor:
```cypher
MATCH (p:Person {name: 'Tom Cruise'})-[r:ACTED_IN]->(m:Movie)
RETURN m.title, r.roles
```
🐞 *The original repeated the same query for "all roles played in a particular movie". Correct version:*
```cypher
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie {title: 'The Matrix'})
RETURN p.name, r.roles
```

---

# Part 4 — WHERE filtering

### Exercise 4.1: Movies Tom Cruise acted in ✅
```cypher
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Cruise'
RETURN m.title
```
*(The original returned `m`; the exercise text asks for titles.)*

### Exercise 4.2: People born in the 70's ✅
```cypher
MATCH (p:Person)
WHERE p.born >= 1970 AND p.born < 1980
RETURN p.name, p.born
```

### Exercise 4.3: Actors in *The Matrix* born after 1960 ✅
```cypher
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
WHERE m.title = 'The Matrix' AND p.born > 1960
RETURN p.name, p.born
```

### Exercise 4.4: Movies by testing the node label and a property ✅
```cypher
MATCH (m)
WHERE m:Movie AND m.released = 2000
RETURN m.title
```

### Exercise 4.5: People who wrote movies — test the relationship type ✅
```cypher
MATCH (a)-[rel]->(m)
WHERE a:Person AND type(rel) = 'WROTE' AND m:Movie
RETURN a.name, m.title
```

### Exercise 4.6: People who do not have a property 🔧
**Removed:** `WHERE NOT exists(a.born)` — the `exists(property)` function was replaced by `IS NULL` / `IS NOT NULL`.
```cypher
MATCH (a:Person)
WHERE a.born IS NULL
RETURN a.name
```

### Exercise 4.7: People related to movies where the relationship has a property 🔧
**Removed:** `WHERE exists(rel.roles)`.
```cypher
MATCH (a:Person)-[rel]->(m:Movie)
WHERE rel.roles IS NOT NULL
RETURN a.name AS Name, m.title AS Movie, rel.roles
```

### Exercise 4.8: Actors whose name begins with "James" ✨
**Fix:** added the missing space before `RETURN`.
```cypher
MATCH (a:Person)-[:ACTED_IN]->(:Movie)
WHERE a.name STARTS WITH 'James'
RETURN DISTINCT a.name
```
*(`DISTINCT` added so an actor with several movies appears once.)*

### Exercise 4.9: REVIEWED relationships whose summary contains "Fun" ✅
```cypher
MATCH (:Person)-[r:REVIEWED]->(m:Movie)
WHERE r.summary CONTAINS 'Fun'
RETURN m.title, r.summary, r.rating
```
*`CONTAINS` is case-sensitive — `'Fun'` will not match `'fun'`.*

**Taking it further**

Movies with "love" in the tagline:
```cypher
MATCH (m:Movie)
WHERE m.tagline CONTAINS 'love'
RETURN m.title
```
Using a regular expression:
```cypher
MATCH (m:Movie)
WHERE m.tagline =~ '.*love.*'
RETURN m.title
```
Case-insensitive regex:
```cypher
MATCH (m:Movie)
WHERE m.tagline =~ '(?i).*love.*'
RETURN m.title
```

### Exercise 4.10: People who produced a movie but have not directed a movie ✨
Original (`NOT ((a)-[:DIRECTED]->(:Movie))`) still works. The subquery form below is clearer and is the current recommended style:
```cypher
MATCH (a:Person)-[:PRODUCED]->(m:Movie)
WHERE NOT EXISTS { (a)-[:DIRECTED]->(:Movie) }
RETURN a.name, m.title
```

### Exercise 4.11: Movies and actors where one of the actors also directed the movie 🔧
**Removed:** `exists((p2)-[:DIRECTED]->(m))` — the pattern form of `exists()`. Use an `EXISTS { }` subquery.
```cypher
MATCH (p1:Person)-[:ACTED_IN]->(m:Movie)<-[:ACTED_IN]-(p2:Person)
WHERE EXISTS { (p2)-[:DIRECTED]->(m) }
RETURN p1.name, p2.name, m.title
```

### Exercise 4.12: Movies released in a set of years ✅
```cypher
MATCH (m:Movie)
WHERE m.released IN [2000, 2004, 2008]
RETURN m.title, m.released
```

### Exercise 4.13: Movies where an actor's role is the movie's name ✅
```cypher
MATCH (a:Person)-[r:ACTED_IN]->(m:Movie)
WHERE m.title IN r.roles
RETURN m.title, a.name
```

---

# Part 5 — Patterns, optional data, collecting lists

### Exercise 5.1: Multiple MATCH patterns ✅
```cypher
MATCH (p1:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(p2:Person),
      (p3:Person)-[:ACTED_IN]->(m)
WHERE p1.name = 'Gene Hackman'
RETURN m.title, p2.name, p3.name
```

### Exercise 5.2: Particular nodes that have a relationship ✅
```cypher
MATCH (p1:Person)-[:FOLLOWS]-(p2:Person)
WHERE p1.name = 'James Thompson'
RETURN p1, p2
```

### Exercise 5.3: Nodes exactly three hops away ✅
```cypher
MATCH (p1:Person)-[:FOLLOWS*3]-(p2:Person)
WHERE p1.name = 'James Thompson'
RETURN p1, p2
```

### Exercise 5.4: Nodes one and two hops away ✅
```cypher
MATCH (p1:Person)-[:FOLLOWS*1..2]-(p2:Person)
WHERE p1.name = 'James Thompson'
RETURN p1, p2
```

### Exercise 5.5: Connected no matter how many hops ✅
```cypher
MATCH (p1:Person)-[:FOLLOWS*]-(p2:Person)
WHERE p1.name = 'James Thompson'
RETURN p1, p2
```
*Unbounded variable-length paths can be expensive on large graphs — prefer an upper bound when you can (`*1..6`).*

### Exercise 5.6: Optional data ✅
```cypher
MATCH (p:Person)
WHERE p.name STARTS WITH 'Tom'
OPTIONAL MATCH (p)-[:DIRECTED]->(m:Movie)
RETURN p.name, m.title
```

### Exercise 5.7: Collect a list ✅
```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name AS actor, collect(m.title) AS movies
```

### Exercise 5.8: Tom Cruise's movies with co-actors as a list ✅
```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)<-[:ACTED_IN]-(p2:Person)
WHERE p.name = 'Tom Cruise'
RETURN m.title AS movie, collect(p2.name) AS coActors
```

### Exercise 5.9: Lists with associated data ✅
```cypher
MATCH (p:Person)-[:REVIEWED]->(m:Movie)
RETURN m.title AS movie, count(p) AS numReviews, collect(p.name) AS reviewers
```

### Exercise 5.10: Nodes and their relationships as a list ✅
```cypher
MATCH (d:Person)-[:DIRECTED]->(m:Movie)<-[:ACTED_IN]-(a:Person)
RETURN d.name AS director,
       count(a) AS `number actors`,
       collect(a.name) AS `actors worked with`
```

### Exercise 5.11: Actors who acted in exactly five movies ✅
```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WITH a, count(m) AS numMovies, collect(m.title) AS movies
WHERE numMovies = 5
RETURN a.name, movies
```

### Exercise 5.12: Movies with at least 2 directors, plus optional reviewers 🔧
**Removed:** `size((:Person)-[:DIRECTED]->(m))` — using `size()` on a pattern. Use a `COUNT { }` subquery.
```cypher
MATCH (m:Movie)
WITH m, COUNT { (:Person)-[:DIRECTED]->(m) } AS directors
WHERE directors >= 2
OPTIONAL MATCH (p:Person)-[:REVIEWED]->(m)
RETURN m.title, p.name
```

---

# Part 6 — Duplicates, sorting, limiting

### Exercise 6.1: A query that returns duplicate records ✅
```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.released >= 1990 AND m.released < 2000
RETURN DISTINCT m.released, m.title, collect(a.name)
```

### Exercise 6.2: Modify the query to eliminate duplication ✅
```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.released >= 1990 AND m.released < 2000
RETURN m.released, collect(m.title), collect(a.name)
```

### Exercise 6.3: Eliminate more duplication ✅
```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.released >= 1990 AND m.released < 2000
RETURN m.released, collect(DISTINCT m.title), collect(a.name)
```

### Exercise 6.4: Sort results ✅
```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.released >= 1990 AND m.released < 2000
RETURN m.released, collect(DISTINCT m.title), collect(a.name)
ORDER BY m.released DESC
```

### Exercise 6.5: Top 5 ratings and their movies ✅
```cypher
MATCH (:Person)-[r:REVIEWED]->(m:Movie)
RETURN m.title AS movie, r.rating AS rating
ORDER BY r.rating DESC
LIMIT 5
```

### Exercise 6.6: Actors that have not appeared in more than 3 movies ✅
```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WITH a, count(m) AS numMovies, collect(m.title) AS movies
WHERE numMovies <= 3
RETURN a.name, movies
```

---

# Part 7 — Lists, UNWIND, dates

### Exercise 7.1: Collect and use lists ✅
*Actors and producers per movie, no duplicates, ordered by size of the actor list.*
```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie), (m)<-[:PRODUCED]-(p:Person)
WITH m, collect(DISTINCT a.name) AS cast, collect(DISTINCT p.name) AS producers
RETURN DISTINCT m.title, cast, producers
ORDER BY size(cast)
```

### Exercise 7.2: Actors in more than five movies — collect their movies ✅
```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WITH p, collect(m) AS movies
WHERE size(movies) > 5
RETURN p.name, movies
```

### Exercise 7.3: Unwind the list ✅
```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WITH p, collect(m) AS movies
WHERE size(movies) > 5
UNWIND movies AS movie
RETURN p.name, movie.title
```

### Exercise 7.4: Calculation with the date type ✅
*Title, release year, years ago, and Tom Hanks's age at release.*
```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WHERE a.name = 'Tom Hanks'
RETURN m.title,
       m.released,
       date().year - m.released AS yearsAgoReleased,
       m.released - a.born AS `age of Tom`
ORDER BY yearsAgoReleased
```

---

# Part 8 — Create, update, and remove nodes and properties

### Exercise 8.1: Create a Movie node ✅
```cypher
CREATE (:Movie {title: 'Forrest Gump'})
```

### Exercise 8.2: Retrieve the newly-created node ✅
```cypher
MATCH (m:Movie) WHERE m.title = 'Forrest Gump' RETURN m
```

### Exercise 8.3: Create a Person node ✅
```cypher
CREATE (:Person {name: 'Robin Wright'})
```

### Exercise 8.4: Retrieve the Person node ✅
```cypher
MATCH (p:Person) WHERE p.name = 'Robin Wright' RETURN p
```

### Exercise 8.5: Add a label to movies released before 2010 ✅
```cypher
MATCH (m:Movie)
WHERE m.released < 2010
SET m:OlderMovie
RETURN DISTINCT labels(m)
```

### Exercise 8.6: Retrieve nodes using the new label ✅
```cypher
MATCH (m:OlderMovie) RETURN m.title, m.released
```

### Exercise 8.7: Add the Female label to people whose name starts with Robin ✅
```cypher
MATCH (p:Person)
WHERE p.name STARTS WITH 'Robin'
SET p:Female
```

### Exercise 8.8: Retrieve all Female nodes ✅
```cypher
MATCH (p:Female) RETURN p.name
```

### Exercise 8.9: Remove the Female label ✅
```cypher
MATCH (p:Female) REMOVE p:Female
```

### Exercise 8.10: View the current schema ✨
```cypher
CALL db.schema.visualization()
```

### Exercise 8.11: Add properties to a movie ✅
```cypher
MATCH (m:Movie)
WHERE m.title = 'Forrest Gump'
SET m:OlderMovie,
    m.released = 1994,
    m.tagline = "Life is like a box of chocolates...you never know what you're gonna get.",
    m.lengthInMinutes = 142
```

### Exercise 8.12: Confirm the label and properties ✅
```cypher
MATCH (m:OlderMovie) WHERE m.title = 'Forrest Gump' RETURN m
```

### Exercise 8.13: Add properties to Robin Wright ✅
```cypher
MATCH (p:Person)
WHERE p.name = 'Robin Wright'
SET p.born = 1966, p.birthPlace = 'Dallas'
```

### Exercise 8.14: Confirm the Person update ✅
```cypher
MATCH (p:Person) WHERE p.name = 'Robin Wright' RETURN p
```

### Exercise 8.15: Remove a property from a Movie node ✨
`SET m.lengthInMinutes = null` also works, but `REMOVE` states the intent directly:
```cypher
MATCH (m:Movie)
WHERE m.title = 'Forrest Gump'
REMOVE m.lengthInMinutes
```

### Exercise 8.16: Confirm the property was removed ✅
```cypher
MATCH (m:Movie) WHERE m.title = 'Forrest Gump' RETURN m
```

### Exercise 8.17: Remove a property from a Person node ✅
```cypher
MATCH (p:Person)
WHERE p.name = 'Robin Wright'
REMOVE p.birthPlace
```

### Exercise 8.18: Confirm the removal ✅
```cypher
MATCH (p:Person) WHERE p.name = 'Robin Wright' RETURN p
```

---

# Part 9 — Relationships: create, update, remove

### Exercise 9.1: Create ACTED_IN relationships ✨
```cypher
MATCH (m:Movie {title: 'Forrest Gump'})
MATCH (p:Person)
WHERE p.name IN ['Tom Hanks', 'Robin Wright', 'Gary Sinise']
CREATE (p)-[:ACTED_IN]->(m)
```
⚠️ Running `CREATE` twice makes duplicate relationships. Use `MERGE` (see 11.14) if you may re-run.

### Exercise 9.2: Create a DIRECTED relationship ✅
```cypher
MATCH (m:Movie) WHERE m.title = 'Forrest Gump'
MATCH (p:Person) WHERE p.name = 'Robert Zemeckis'
CREATE (p)-[:DIRECTED]->(m)
```

### Exercise 9.3: Create a HELPED relationship ✅
```cypher
MATCH (p1:Person) WHERE p1.name = 'Tom Hanks'
MATCH (p2:Person) WHERE p2.name = 'Gary Sinise'
CREATE (p1)-[:HELPED]->(p2)
```

### Exercise 9.4: Nodes connected to Forrest Gump with their relationships ✅
```cypher
MATCH (p:Person)-[rel]-(m:Movie)
WHERE m.title = 'Forrest Gump'
RETURN p, rel, m
```

### Exercise 9.5: Add `roles` to the ACTED_IN relationships 🐞
**Original bug:** the README showed only `SET rel.roles = CASE ...` with no `MATCH`, so it cannot run. Full query:
```cypher
MATCH (p:Person)-[rel:ACTED_IN]->(m:Movie {title: 'Forrest Gump'})
SET rel.roles = CASE p.name
  WHEN 'Tom Hanks'    THEN ['Forrest Gump']
  WHEN 'Robin Wright' THEN ['Jenny Curran']
  WHEN 'Gary Sinise'  THEN ['Lieutenant Dan Taylor']
END
```

### Exercise 9.6: Add `research` to the HELPED relationship ✅
```cypher
MATCH (p1:Person)-[rel:HELPED]->(p2:Person)
WHERE p1.name = 'Tom Hanks' AND p2.name = 'Gary Sinise'
SET rel.research = 'war history'
```

### Exercise 9.7: View property keys ✨
```cypher
CALL db.propertyKeys()
```

### Exercise 9.8: View the schema ✨
```cypher
CALL db.schema.visualization()
```

### Exercise 9.9: Names and roles of actors in Forrest Gump ✅
```cypher
MATCH (p:Person)-[rel:ACTED_IN]->(m:Movie)
WHERE m.title = 'Forrest Gump'
RETURN p.name, rel.roles
```

### Exercise 9.10: Retrieve HELPED relationships ✅
```cypher
MATCH (p1:Person)-[rel:HELPED]-(p2:Person)
RETURN p1.name, rel, p2.name
```

### Exercise 9.11: Modify a relationship property ✅
```cypher
MATCH (p:Person)-[rel:ACTED_IN]->(m:Movie)
WHERE m.title = 'Forrest Gump' AND p.name = 'Gary Sinise'
SET rel.roles = ['Lt. Dan Taylor']
```

### Exercise 9.12: Remove a relationship property ✅
```cypher
MATCH (p1:Person)-[rel:HELPED]->(p2:Person)
WHERE p1.name = 'Tom Hanks' AND p2.name = 'Gary Sinise'
REMOVE rel.research
```

### Exercise 9.13: Confirm the modifications ✅
```cypher
MATCH (p:Person)-[rel:ACTED_IN]->(m:Movie)
WHERE m.title = 'Forrest Gump'
RETURN p, rel, m
```

---

# Part 10 — Deleting

### Exercise 10.1: Delete the HELPED relationship ✨
A directed match avoids visiting the same relationship twice:
```cypher
MATCH (:Person)-[rel:HELPED]->(:Person)
DELETE rel
```

### Exercise 10.2: Confirm the relationship is gone ✅
```cypher
MATCH (:Person)-[rel:HELPED]-(:Person)
RETURN rel
```

### Exercise 10.3: A movie and all of its relationships ✅
```cypher
MATCH (p:Person)-[rel]-(m:Movie)
WHERE m.title = 'Forrest Gump'
RETURN p, rel, m
```

### Exercise 10.4: Try deleting a node without detaching its relationships ✅
```cypher
MATCH (m:Movie) WHERE m.title = 'Forrest Gump' DELETE m
```
**Expected error** (the node number will differ):
```text
Cannot delete node<513>, because it still has relationships. To delete this node, you must first delete its relationships.
```

### Exercise 10.5: Delete a Movie node with its relationships ✅
```cypher
MATCH (m:Movie) WHERE m.title = 'Forrest Gump' DETACH DELETE m
```

### Exercise 10.6: Confirm the Movie node is deleted ✅
```cypher
MATCH (p:Person)-[rel]-(m:Movie)
WHERE m.title = 'Forrest Gump'
RETURN p, rel, m
```
*Expected: no rows.*

---

# Part 11 — MERGE

### Exercise 11.1: MERGE to create a Movie node ✅
```cypher
MERGE (m:Movie {title: 'Forrest Gump'})
ON CREATE SET m.released = 1994
RETURN m
```

### Exercise 11.2: MERGE to update a node (ON MATCH) ✅
```cypher
MERGE (m:Movie {title: 'Forrest Gump'})
ON CREATE SET m.released = 1994
ON MATCH SET m.tagline = "Life is like a box of chocolates...you never know what you're gonna get."
RETURN m
```

### Exercise 11.3: MERGE to create a Production node ✅
```cypher
MERGE (p:Production {title: 'Forrest Gump'})
ON CREATE SET p.year = 1994
RETURN p
```

### Exercise 11.4: Find all labels for nodes with a given title ✅
```cypher
MATCH (m)
WHERE m.title = 'Forrest Gump'
RETURN labels(m)
```

### Exercise 11.5: MERGE to update a Production node ✅
```cypher
MERGE (p:Production {title: 'Forrest Gump'})
ON MATCH SET p.company = 'Paramount Pictures'
RETURN p
```

### Exercise 11.6: MERGE to add a label ✅
```cypher
MERGE (m:Movie {title: 'Forrest Gump'})
ON MATCH SET m:OlderMovie
RETURN labels(m)
```

### Exercise 11.7: MERGE that creates two nodes and a relationship ✅
```cypher
MERGE (p:Person {name: 'Robert Zemeckis'})-[:DIRECTED]->(m {title: 'Forrest Gump'})
```
**Why this is a teaching example:** `MERGE` treats the *whole pattern* as one unit. If the full pattern doesn't exist, it creates **all** of it — including a **duplicate** `Person` node for Robert Zemeckis (with no `born`) and an unlabeled node for the movie. The next exercises clean that up.

### Exercise 11.8: Run the same MERGE again ✅
Same statement as 11.7. **Expected output:** no changes, no records — the pattern now exists, so `MERGE` matches instead of creating.

### Exercise 11.9: Find the correct Person node to delete 🔧
**Removed:** `WHERE NOT EXISTS (p.born)`.
```cypher
MATCH (p:Person {name: 'Robert Zemeckis'})-[rel]-(x)
WHERE p.born IS NULL
RETURN p, rel, x
```

### Exercise 11.10: Delete that Person node with its relationships 🔧
**Removed:** `WHERE NOT EXISTS (p.born)`. The `--()` in the original isn't needed; `DETACH DELETE` handles the relationships.
```cypher
MATCH (p:Person {name: 'Robert Zemeckis'})
WHERE p.born IS NULL
DETACH DELETE p
```

### Exercise 11.11: Find the correct Forrest Gump node to delete ✅
```cypher
MATCH (m)
WHERE m.title = 'Forrest Gump' AND labels(m) = []
RETURN m, labels(m)
```

### Exercise 11.12: Delete the unlabeled Forrest Gump node ✅
```cypher
MATCH (m)
WHERE m.title = 'Forrest Gump' AND labels(m) = []
DETACH DELETE m
```

### Exercise 11.13: MERGE the DIRECTED relationship ✅
```cypher
MATCH (p:Person), (m:Movie)
WHERE p.name = 'Robert Zemeckis' AND m.title = 'Forrest Gump'
MERGE (p)-[:DIRECTED]->(m)
```

### Exercise 11.14: MERGE the ACTED_IN relationships ✅
```cypher
MATCH (p:Person), (m:Movie)
WHERE p.name IN ['Tom Hanks', 'Gary Sinise', 'Robin Wright']
  AND m.title = 'Forrest Gump'
MERGE (p)-[:ACTED_IN]->(m)
```

### Exercise 11.15: Set the roles ✅
```cypher
MATCH (p:Person)-[rel:ACTED_IN]->(m:Movie)
WHERE m.title = 'Forrest Gump'
SET rel.roles = CASE p.name
  WHEN 'Tom Hanks'    THEN ['Forrest Gump']
  WHEN 'Robin Wright' THEN ['Jenny Curran']
  WHEN 'Gary Sinise'  THEN ['Lt. Dan Taylor']
END
```

---

# Part 12 — Parameters

> `:param` and `:params` are **Neo4j Browser commands**, not Cypher statements. From application code, pass parameters through the driver instead (example at the end of this part).

### Exercise 12.1: Reviewers of movies and the actors in them ✅
```cypher
MATCH (r:Person)-[rel:REVIEWED]->(m:Movie)<-[:ACTED_IN]-(a:Person)
RETURN DISTINCT r.name, m.title, m.released, rel.rating, collect(a.name)
```

### Exercise 12.2: Add a parameter named `year` with value 2000 ✅
```text
:param year => 2000
```

### Exercise 12.3: Use the `year` parameter in the query ✅
```cypher
MATCH (r:Person)-[rel:REVIEWED]->(m:Movie)<-[:ACTED_IN]-(a:Person)
WHERE m.released = $year
RETURN DISTINCT r.name, m.title, m.released, rel.rating, collect(a.name)
```

### Exercise 12.4: Change `year` to 2006 and retest ✅
```text
:param year => 2006
```

### Exercise 12.5: Add a second parameter `ratingValue` = 65 ✅
```text
:params {year: 2006, ratingValue: 65}
```

### Exercise 12.6: Use both parameters ✅
```cypher
MATCH (r:Person)-[rel:REVIEWED]->(m:Movie)<-[:ACTED_IN]-(a:Person)
WHERE m.released = $year AND rel.rating > $ratingValue
RETURN DISTINCT r.name, m.title, m.released, rel.rating, collect(a.name)
```

### Exercise 12.7: Change `ratingValue` to 60 and retest ✅
```text
:params {year: 2006, ratingValue: 60}
```

**Same query from Python (driver parameters):**
```python
records, summary, keys = driver.execute_query(
    """
    MATCH (r:Person)-[rel:REVIEWED]->(m:Movie)<-[:ACTED_IN]-(a:Person)
    WHERE m.released = $year AND rel.rating > $ratingValue
    RETURN DISTINCT r.name, m.title, m.released, rel.rating, collect(a.name)
    """,
    year=2006, ratingValue=60,
)
```

---

# Part 13 — Query plans and profiling

*These assume `year` and `ratingValue` are set as in Part 12.*

### Exercise 13.1: View the query plan for a statement 🐞
**Original bug:** the exercise asks for the *plan*, but the README showed only the plain query, which just runs it. Add `EXPLAIN`:
```cypher
EXPLAIN
MATCH (r:Person)-[rel:REVIEWED]->(m:Movie)<-[:ACTED_IN]-(a:Person)
WHERE m.released = $year AND rel.rating > $ratingValue
RETURN DISTINCT r.name, m.title, m.released, rel.rating, collect(a.name)
```
`EXPLAIN` shows the planned strategy **without executing** the query.

### Exercise 13.2: View metrics when the statement executes ✅
```cypher
PROFILE
MATCH (r:Person)-[rel:REVIEWED]->(m:Movie)<-[:ACTED_IN]-(a:Person)
WHERE m.released = $year AND rel.rating > $ratingValue
RETURN DISTINCT r.name, m.title, m.released, rel.rating, collect(a.name)
```
`PROFILE` **executes** the query and reports rows and db hits per operator.

### Exercise 13.3: Remove the labels and compare db hits ✅
```cypher
PROFILE
MATCH (r)-[rel]->(m)<-[:ACTED_IN]-(a)
WHERE m.released = $year AND rel.rating > $ratingValue
RETURN DISTINCT r.name, m.title, m.released, rel.rating, collect(a.name)
```
**What to notice:** without labels the planner can't start from a label scan, so db hits are typically higher. Labels help the planner narrow the search — this is why the exercise compares the two.

---

# Appendix — Old → New syntax quick reference

| Old (removed in Neo4j 5) | Current |
|---|---|
| `exists(n.prop)` | `n.prop IS NOT NULL` |
| `NOT exists(n.prop)` | `n.prop IS NULL` |
| `exists((a)-[:REL]->(b))` | `EXISTS { (a)-[:REL]->(b) }` |
| `size((a)-[:REL]->())` | `COUNT { (a)-[:REL]->() }` |
| `CREATE INDEX ON :Label(prop)` | `CREATE INDEX FOR (n:Label) ON (n.prop)` |
| `CREATE CONSTRAINT ON (n:Label) ASSERT n.prop IS UNIQUE` | `CREATE CONSTRAINT FOR (n:Label) REQUIRE n.prop IS UNIQUE` |
| `call db.schema.visualization` (no parentheses) | `CALL db.schema.visualization()` |

**Sources checked:** the Neo4j Cypher manual's "Additions, deprecations, removals, and compatibility" pages (including the Cypher 25 removals list for Neo4j 2025.06+) and the 4.3 manual entry marking `exists(prop)` as deprecated in favor of `IS NOT NULL` / `IS NULL`. The Neo4j 5.0 *removal* of `exists()` and `size(pattern)` comes from background knowledge and was not re-confirmed in the docs during this session — verify by running the old forms on your instance.
