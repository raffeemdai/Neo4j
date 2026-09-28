# APOC Complete Notes — Theory, Syntax, Practice & Interview Questions (Updated Edition)

APOC = **A**wesome **P**rocedures **O**n **C**ypher. It's the most widely used Neo4j plugin: a library of hundreds of procedures and functions that fill in the gaps native Cypher doesn't cover — JSON/XML handling, bulk import/export, dynamic/parameterized Cypher, graph refactoring, batching, triggers, and more.

All practice queries below reuse the **same dataset** as `Day1_Cypher_Practice_Queries.md` (Alice/Bob/Carol/Dave people, OpenAI/Neo4j companies, and the Customer→PURCHASED→Product mini graph), so you can run them in the same sandbox. A dedicated section near the end, **"APOC + Python"**, covers calling every one of these procedures from real application code using the official Neo4j Python driver — with production-style (real-time) examples and Python-specific practice questions.

> ## ⚠️ Updated edition — reviewed against current Neo4j APOC docs (Sept 2026)
>
> Items marked **⚠️** below are deprecated, removed, or changed since older APOC versions. The consolidated list is in **Appendix A**.
>
> **Version awareness:** APOC 5.26 is the LTS line for Neo4j 5 (Cypher 5). APOC 2025.06+ pairs with Neo4j 2025.06+, which lets you choose **Cypher 5 or Cypher 25** — some procedures (e.g. the old `apoc.trigger.add`) still work under Cypher 5 but are **not available in Cypher 25**. Always check `RETURN apoc.version();` and read the docs for *your* version.
>
> **Biggest fixes vs. the old notes:** (1) triggers → `apoc.trigger.install/drop/...`, (2) `apoc.*` config settings now belong in `apoc.conf`, not `neo4j.conf`, (3) `apoc.text.join` and `apoc.date.format` are deprecated in favor of native Cypher, (4) native `CALL {} IN TRANSACTIONS` is now the first choice for batching.

---

# 0. Theory — What APOC Is and When to Use It

## What APOC Is

APOC is **not** part of core Neo4j — it's a separate plugin (a `.jar` file) that adds hundreds of extra procedures (`CALL apoc.xxx`) and functions (`apoc.xxx()`) on top of Cypher.

- **Procedures** (`CALL apoc.something(...)`) — can do multi-row, side-effecting work (writes, batching, I/O).
- **Functions** (`RETURN apoc.something(...)`) — return a single value, usable inline inside `RETURN`, `WHERE`, `WITH`.

## Why It Exists

Native Cypher is intentionally minimal. APOC fills real gaps:

| Native Cypher gap | APOC fills it with |
|---|---|
| No dynamic/parameterized labels or relationship types | `apoc.merge.node`, `apoc.create.relationship` |
| No built-in JSON parsing/serialization | `apoc.convert.*` |
| No CSV/JSON/JDBC bulk import | `apoc.load.*` |
| Large-transaction batching (older gap — native `CALL {} IN TRANSACTIONS` now exists ⚠️) | `apoc.periodic.iterate` (still useful for parallel/retry) |
| No built-in database triggers | `apoc.trigger.*` (⚠️ API changed — see §13) |
| No easy schema/graph introspection | `apoc.meta.*` |
| No node/relationship "refactoring" helpers | `apoc.refactor.*` |

## Installing / Enabling APOC

- **Neo4j Desktop / Aura:** APOC Core is usually pre-installed or a one-click plugin install.
- **Self-managed (Docker, server):** drop the `apoc-<version>-core.jar` into the `plugins/` folder and restart.
- Some procedures (file import/export, JSON APIs) require explicit config. ⚠️ **Since Neo4j 5, `apoc.*` settings are no longer supported in `neo4j.conf`** — they go in a separate **`apoc.conf`** file (same folder as `neo4j.conf`, e.g. `neo4j/conf/`). `apoc.conf` is **not created automatically** — you create it yourself.

`conf/neo4j.conf` (Neo4j's own setting — still lives here):

```text
dbms.security.procedures.unrestricted=apoc.*
```

`conf/apoc.conf` (APOC settings — create the file if it doesn't exist):

```text
apoc.export.file.enabled=true
apoc.import.file.enabled=true
```

- Environment variables (e.g. Docker `--env`) also work and **override** `apoc.conf`.
- By default, file paths are relative to the Neo4j import directory (`server.directories.import`), because `apoc.import.file.use_neo4j_config=true`. Set it to `false` in `apoc.conf` only if you truly need to read from anywhere on disk (security trade-off).
- If you get *"Import from files not enabled, please set apoc.import.file.enabled=true in your apoc.conf"* — this is the fix.

## Interview Position (how to talk about APOC)

Don't say "APOC replaces Cypher." Say:

> I prefer native Cypher when possible, and reach for APOC when it simplifies specialized transformation, integration, collection handling, import/export, batching, or utility operations that Cypher doesn't cover natively.

### Memory Trick

```text
APOC = Cypher's utility toolbox
Native Cypher first. APOC when Cypher runs out of tools.
```

---

# APOC Object / Topic Map

APOC's procedures and functions are organized into namespaces (think of these as APOC's "packages"). These are the ones worth knowing cold for interviews:

| # | Namespace | Purpose |
|---|---|---|
| 1 | `apoc.coll.*` | List/collection utilities |
| 2 | `apoc.text.*` | String utilities (⚠️ `apoc.text.join` deprecated) |
| 3 | `apoc.convert.*` | JSON / type conversion |
| 4 | `apoc.map.*` | Map (dictionary) utilities |
| 5 | `apoc.date.*` / `apoc.temporal.*` | Date & time utilities (⚠️ some deprecated in favor of native Cypher) |
| 6 | `apoc.create.*` | Dynamic / virtual node & relationship creation |
| 7 | `apoc.merge.*` | Dynamic MERGE (runtime label/type) |
| 8 | `apoc.refactor.*` | Restructure existing graph data |
| 9 | `apoc.path.*` | Path expansion / traversal helpers |
| 10 | `apoc.load.*` | Import from JSON, CSV, JDBC, XML |
| 11 | `apoc.export.*` | Export to JSON, CSV, Cypher, GraphML |
| 12 | `apoc.periodic.*` | Batching, scheduling, background jobs (⚠️ native `CALL {} IN TRANSACTIONS` preferred for simple batching) |
| 13 | `apoc.trigger.*` | Run Cypher automatically on data changes (⚠️ `install/drop/start/stop`, not `add/remove`) |
| 14 | `apoc.meta.*` | Schema / graph introspection |
| 15 | `apoc.schema.*` | Constraint/index inspection |
| 16 | `apoc.algo.*` | Legacy graph algorithms (mostly superseded by GDS) |
| 17 | `apoc.util.*` / `apoc.log.*` | Misc utilities, validation, logging |
| 18 | `apoc.help` / `apoc.version` | Discovery — find what's installed |

### Memory Trick

```text
C-T-C-M-D  |  C-M-R-P  |  L-E-P-T  |  M-S-A-U-H
Coll, Text, Convert, Map, Date   → basic data utilities
Create, Merge, Refactor, Path    → graph-shaping utilities
Load, Export, Periodic, Trigger  → integration & automation utilities
Meta, Schema, Algo, Util, Help   → introspection & misc
```

---

# 1. `apoc.coll.*` — Collection (List) Utilities

## Theory

Cypher has some list functions built in (`size()`, `head()`, `range()`), but APOC adds set operations, sorting, partitioning, and more — the kind of things you'd reach for `set()`/`sorted()` for in Python.

## Syntax & Examples

```cypher
// Deduplicate a list (like Python's set())
RETURN apoc.coll.toSet(["Neo4j", "AWS", "Neo4j"]) AS uniqueSkills;
// -> ["Neo4j", "AWS"]
```

```cypher
// Intersection of two lists
RETURN apoc.coll.intersection(
    ["Neo4j", "Python", "AWS"],
    ["Python", "Docker", "AWS"]
) AS common;
// -> ["Python", "AWS"]
```

```cypher
// Difference (subtract)
RETURN apoc.coll.subtract(
    ["Neo4j", "Python", "AWS"],
    ["Python"]
) AS remaining;
// -> ["Neo4j", "AWS"]
```

```cypher
// Sort a list of maps by a key
MATCH (c:Customer)-[r:PURCHASED]->(p:Product)
WITH collect({name: p.name, amount: r.amount}) AS purchases
RETURN apoc.coll.sortMaps(purchases, "amount") AS sortedByAmount;
```

### Memory Trick

```text
apoc.coll = "collections" = set math on lists
toSet, intersection, subtract, sort — like Python set()/sorted()
```

---

# 2. `apoc.text.*` — String Utilities

## Theory

Fills gaps in Cypher's limited string functions — capitalization, fuzzy matching, phonetic comparison, string distance, formatting.

## Syntax & Examples

```cypher
RETURN apoc.text.capitalize("alice") AS name;
// -> "Alice"
```

```cypher
// Fuzzy/typo-tolerant matching — useful for entity resolution
RETURN apoc.text.levenshteinDistance("Neo4j", "Neo4J") AS distance;
// -> 1
```

```cypher
// ⚠️ DEPRECATED — the APOC docs mark apoc.text.join as deprecated
RETURN apoc.text.join(["Neo4j", "Python", "AWS"], ", ") AS skillsLine;
// -> "Neo4j, Python, AWS"
```

```cypher
// ✅ Native replacement (APOC docs point to Cypher's string.join() — confirm it exists on your Neo4j version)
RETURN string.join(["Neo4j", "Python", "AWS"], ", ") AS skillsLine;
```

```cypher
// ✅ Version-safe native alternative (works on any Neo4j 5 — no extra function needed)
WITH ["Neo4j", "Python", "AWS"] AS skills
RETURN reduce(line = head(skills), s IN tail(skills) | line + ", " + s) AS skillsLine;
// -> "Neo4j, Python, AWS"
```

### Memory Trick

```text
apoc.text = string toolbox Cypher forgot: capitalize, distance, join, format
```

---

# 3. `apoc.convert.*` — JSON & Type Conversion

## Theory

Neo4j's property values are limited to primitives and lists of primitives — you can't store a nested map as a property. APOC's `convert` namespace serializes/deserializes JSON so you can pass structured data in and out of Cypher.

## Syntax & Examples

```cypher
// Map -> JSON string
RETURN apoc.convert.toJson({
    name: "Alice",
    skills: ["Neo4j", "Python"]
}) AS json;
```

```cypher
// JSON string -> Map
WITH '{"name":"Alice","age":30}' AS jsonString
RETURN apoc.convert.fromJsonMap(jsonString) AS parsed;
```

```cypher
// JSON string -> List
WITH '["Neo4j","Python","AWS"]' AS jsonString
RETURN apoc.convert.fromJsonList(jsonString) AS parsed;
```

### Memory Trick

```text
apoc.convert = the JSON <-> Cypher bridge
toJson = serialize, fromJsonMap/fromJsonList = deserialize
```

---

# 4. `apoc.map.*` — Map (Dictionary) Utilities

## Theory

Helpers for merging, filtering, and reshaping map/dictionary-like data — handy when you've just pulled a JSON payload apart with `apoc.convert`.

## Syntax & Examples

```cypher
// Merge two maps (second one wins on key conflicts)
RETURN apoc.map.merge(
    {name: "Alice", age: 30},
    {age: 31, city: "Atlanta"}
) AS merged;
// -> {name:"Alice", age:31, city:"Atlanta"}
```

```cypher
// Remove keys from a map
RETURN apoc.map.removeKeys(
    {name:"Alice", age:31, city:"Atlanta"},
    ["city"]
) AS trimmed;
// -> {name:"Alice", age:31}
```

### Memory Trick

```text
apoc.map = merge/remove/filter for dictionaries, like Python dict.update()
```

---

# 5. `apoc.date.*` / `apoc.temporal.*` — Date & Time Utilities

## Theory

Neo4j has native `date()`, `datetime()`, `duration()` functions, but APOC adds conversions to/from epoch millis/seconds and custom formatting — common when ingesting timestamps from external systems.

## Syntax & Examples

```cypher
// ⚠️ DEPRECATED — apoc.date.format is marked deprecated in the APOC docs (they point to native Cypher formatting)
RETURN apoc.date.format(1735689600000, "ms", "yyyy-MM-dd") AS formatted;
// -> "2025-01-01"

// ✅ Native equivalent (epoch millis -> date string)
RETURN toString(date(datetime({epochMillis: 1735689600000}))) AS formatted;
// -> "2025-01-01"
```

```cypher
// Parse a formatted string into epoch millis (APOC version — check the docs page for your version's status)
RETURN apoc.date.parse("2026-09-25", "ms", "yyyy-MM-dd") AS epochMillis;

// ✅ Native equivalent (date string -> epoch millis; assumes UTC unless your default timezone differs)
RETURN datetime({date: date("2026-09-25")}).epochMillis AS epochMillis;
```

> ⚠️ Also note: `apoc.date.parseAsZonedDateTime` was **removed** in APOC 5.0 → use `apoc.temporal.toZonedTemporal(...)`, and `apoc.date.expire` / `expireIn` were removed → use `apoc.ttl.expire` / `apoc.ttl.expireIn`.

### Memory Trick

```text
apoc.date = epoch <-> human-readable date. Prefer native date()/datetime() first; APOC only when native can't do it — and check deprecation status before relying on it.
```

---

# 6. `apoc.create.*` — Dynamic & Virtual Node/Relationship Creation

## Theory

Native Cypher requires labels and relationship types to be **literal** at query-write time — you can't do `CREATE (n:$label)`. APOC lets you pass label/type names as **runtime strings**, which matters for generic ingestion code. It also supports **virtual nodes/relationships** — in-memory-only objects for visualization, never persisted to the graph.

## Syntax & Examples

```cypher
// Dynamic label at runtime
CALL apoc.create.node(["Person", "Employee"], {name: "Grace"})
YIELD node
RETURN node;
```

```cypher
// Dynamic relationship type at runtime
MATCH (a:Person {name:"Alice"}), (b:Company {name:"OpenAI"})
CALL apoc.create.relationship(a, "CONTRACTED_BY", {since: 2024}, b)
YIELD rel
RETURN rel;
```

```cypher
// Virtual node — exists only in the query result, not saved to the graph
RETURN apoc.create.vNode(
    ["Summary"],
    {label: "Total Customers: 3"}
) AS previewNode;
```

### Memory Trick

```text
apoc.create = CREATE with variables instead of literals
vNode/vRelationship = "preview only" nodes — never written to disk
```

---

# 7. `apoc.merge.*` — Dynamic MERGE

## Theory

Same idea as `apoc.create`, but for `MERGE` semantics (find-or-create) — needed when your ingestion pipeline doesn't know the label name until runtime.

## Syntax & Examples

```cypher
CALL apoc.merge.node(
    ["Customer"],
    {customerId: "C6"},
    {name: "Fiona"}
) YIELD node
RETURN node;
```

### Memory Trick

```text
apoc.merge = MERGE with a variable label, same find-or-create idea
```

---

# 8. `apoc.refactor.*` — Restructuring Existing Data

## Theory

Cypher can't easily rename a relationship type or merge two duplicate nodes into one — these are common cleanup tasks after a messy import, and `apoc.refactor` handles them directly.

## Syntax & Examples

```cypher
// Rename a relationship type (Cypher has no native way to do this)
MATCH (a:Person)-[r:KNOWS]->(b:Person)
CALL apoc.refactor.setType(r, "CONNECTED_TO")
YIELD input, output
RETURN output;
```

```cypher
// Merge two duplicate nodes into one
MATCH (a:Person {name:"Alice"}), (dup:Person {name:"Alice_duplicate"})
CALL apoc.refactor.mergeNodes([a, dup], {properties: "combine"})
YIELD node
RETURN node;
```

```cypher
// Convert a node's category into a label ("category-to-label" pattern)
MATCH (p:Product)
CALL apoc.create.addLabels(p, [p.category])
YIELD node
RETURN node;
```

### Memory Trick

```text
apoc.refactor = graph "fix-it" toolkit: rename types, dedupe nodes, promote properties to labels
```

---

# 9. `apoc.path.*` — Path Expansion

## Theory

Native variable-length patterns (`*1..3`) are great for simple cases, but real traversals often need constraints Cypher's pattern syntax can't express cleanly — filter by relationship type, direction, node label, min/max depth, all in one config map.

## Syntax & Examples

```cypher
MATCH (start:Person {name:"Alice"})
CALL apoc.path.expandConfig(start, {
    relationshipFilter: "KNOWS>",
    minLevel: 1,
    maxLevel: 2,
    uniqueness: "NODE_GLOBAL"
})
YIELD path
RETURN path;
```

### Memory Trick

```text
apoc.path.expandConfig = variable-length traversal with a full config map
(relationship filter + direction + label filter + depth, all in one call)
```

---

# 10. `apoc.load.*` — Import from External Sources

## Theory

Bulk-loading real-world data usually means pulling from a file or another system. `apoc.load` handles JSON, CSV, XML, and JDBC sources directly inside Cypher — the standard way to bring external data into a graph without writing separate ETL code.

## Syntax & Examples

```cypher
// Load JSON from a URL or file (requires apoc.import.file.enabled or a public URL)
CALL apoc.load.json("https://example.com/customers.json")
YIELD value
MERGE (c:Customer {customerId: value.customerId})
SET c.name = value.name;
```

```cypher
// Load CSV
CALL apoc.load.csv("file:///customers.csv")
YIELD map
MERGE (c:Customer {customerId: map.customerId})
SET c.name = map.name;
```

### Memory Trick

```text
apoc.load = "bring data in" — JSON/CSV/XML/JDBC → Cypher rows
```

---

# 11. `apoc.export.*` — Export to External Formats

## Theory

The reverse of `apoc.load` — dump query results or the whole graph out as JSON, CSV, Cypher statements, or GraphML (for tools like Gephi).

## Syntax & Examples

```cypher
// Export a query's results to CSV on disk
CALL apoc.export.csv.query(
    "MATCH (c:Customer)-[:PURCHASED]->(p:Product) RETURN c.name, p.name",
    "purchases.csv",
    {}
);
```

```cypher
// Export the whole graph as a Cypher script (great for migrating between environments)
CALL apoc.export.cypher.all("full_graph_backup.cypher", {});
```

### Memory Trick

```text
apoc.export = "send data out" — the mirror image of apoc.load
```

---

# 12. `apoc.periodic.*` — Batching & Background Jobs

## Theory

A single giant `MATCH ... SET` over millions of rows can blow out memory and lock the whole transaction. `apoc.periodic.iterate` runs the write in small batches (e.g. 10,000 rows at a time), each in its own transaction — the standard production pattern for large migrations.

## Syntax & Examples

```cypher
CALL apoc.periodic.iterate(
  "MATCH (c:Customer) RETURN c",
  "SET c.migrated = true",
  {batchSize: 1000, parallel: false}
)
YIELD batches, total, errorMessages
RETURN batches, total, errorMessages;
```

```cypher
// Schedule a job to run every N seconds
CALL apoc.periodic.repeat(
  "cleanup-job",
  "MATCH (n:TempNode) WHERE n.createdAt < datetime() - duration('P1D') DETACH DELETE n",
  3600
);
```

### Memory Trick

```text
apoc.periodic.iterate = "big job? break it into small batched transactions"
```

## ⚠️ Native Alternative — `CALL {} IN TRANSACTIONS`

Neo4j 5 introduced batching natively in Cypher, so `apoc.periodic.iterate` is no longer the only answer. (The old APOC `rock_n_roll` procedures were removed in APOC 5.0 with exactly this native replacement in mind.)

```cypher
// Neo4j Browser needs :auto for CALL ... IN TRANSACTIONS
:auto
MATCH (c:Customer)
CALL (c) {                       // scope clause syntax — Neo4j 5.23+
  SET c.migrated = true
} IN TRANSACTIONS OF 1000 ROWS;
```

```cypher
// Older Neo4j 5.x syntax (importing WITH)
:auto
MATCH (c:Customer)
CALL {
  WITH c
  SET c.migrated = true
} IN TRANSACTIONS OF 1000 ROWS;
```

**When to use which (good interview answer):**

| Situation | Choice |
|---|---|
| Simple batched write, want zero plugin dependency | Native `CALL {} IN TRANSACTIONS` |
| Need built-in retries, per-batch error reporting (`errorMessages`), or `parallel: true` on older versions | `apoc.periodic.iterate` |
| Managed environment where APOC isn't available | Native |

---

# 13. `apoc.trigger.*` — Database Triggers

## Theory

Neo4j core has no built-in trigger system. `apoc.trigger` lets you register Cypher that runs automatically whenever nodes/relationships are created, updated, or deleted — useful for maintaining derived properties or audit logs.

> ⚠️ **The trigger API changed.** The old `apoc.trigger.add / remove / removeAll / pause / resume` are **deprecated in Cypher 5 and removed in Cypher 25** (removed from APOC Core in APOC 2025.06; they can still be used under Cypher 5). They were not designed for clusters. The replacements take the **database name as the first argument** and are applied "eventually" (asynchronously, cluster-safe).

## Enable Triggers First

Triggers are **disabled by default**. Add to `apoc.conf` (not `neo4j.conf`) and restart:

```text
apoc.trigger.enabled=true
apoc.trigger.refresh=60000
```

## Syntax & Examples (current API)

```cypher
// ✅ Install a trigger: (databaseName, name, statement, selector, config)
CALL apoc.trigger.install(
  "neo4j",
  "set-created-timestamp",
  "UNWIND $createdNodes AS n SET n.createdAt = datetime()",
  {phase: "before"},
  {}
);
```

```cypher
// List triggers installed for the current session database
CALL apoc.trigger.list();

// Show triggers for a specific database (from the system database)
CALL apoc.trigger.show("neo4j");
```

```cypher
// Pause / resume a trigger
CALL apoc.trigger.stop("neo4j", "set-created-timestamp");
CALL apoc.trigger.start("neo4j", "set-created-timestamp");

// Remove one trigger / all triggers for a database
CALL apoc.trigger.drop("neo4j", "set-created-timestamp");
CALL apoc.trigger.dropAll("neo4j");
```

**Selector phases:** `before` (inside the transaction, before commit), `after` (after commit), `afterAsync` (after commit, asynchronous), `rollback`. Inside the statement you can use parameters such as `$createdNodes`, `$deletedNodes`, `$assignedNodeProperties`.

## Old → New Mapping

| ⚠️ Deprecated / removed in Cypher 25 | ✅ Use instead |
|---|---|
| `apoc.trigger.add(name, stmt, selector [, config])` | `apoc.trigger.install(db, name, stmt, selector, config)` |
| `apoc.trigger.remove(name)` | `apoc.trigger.drop(db, name)` |
| `apoc.trigger.removeAll()` | `apoc.trigger.dropAll(db)` |
| `apoc.trigger.pause(name)` | `apoc.trigger.stop(db, name)` |
| `apoc.trigger.resume(name)` | `apoc.trigger.start(db, name)` |
| `apoc.trigger.list()` | still valid; `apoc.trigger.show(db)` for a specific database |

## Interview Points

- Triggers in the `before` phase run **inside** the user's transaction — keep them light, or they slow every write.
- `install` is *eventual*: the trigger may take a moment (refresh interval) to become active — don't assume it fires on the very next statement.
- If asked "how would you audit changes without triggers?" — mention application-level logic or Neo4j's Change Data Capture (CDC) as alternatives.

### Memory Trick

```text
add → install   |  remove → drop  |  removeAll → dropAll  |  pause → stop  |  resume → start
New API = database name FIRST.  Enable with apoc.trigger.enabled=true in apoc.conf.
```

---

# 14. `apoc.meta.*` — Graph & Schema Introspection

## Theory

Answers "what's actually in this graph?" — label counts, relationship type counts, property keys — useful for exploring an unfamiliar database or generating documentation.

## Syntax & Examples

```cypher
// High-level graph statistics
CALL apoc.meta.stats()
YIELD labelCount, relTypeCount, nodeCount, relCount
RETURN labelCount, relTypeCount, nodeCount, relCount;
```

```cypher
// Full schema graph (labels, relationship types, and how they connect)
CALL apoc.meta.schema()
YIELD value
RETURN value;
```

### Memory Trick

```text
apoc.meta = "describe this database to me" — like SQL's INFORMATION_SCHEMA
```

---

# 15. `apoc.schema.*` — Constraint & Index Helpers

## Theory

Complements native `SHOW CONSTRAINTS` / `SHOW INDEXES` with more programmatic helpers (e.g. checking schema state before running a migration script).

## Syntax & Examples

```cypher
CALL apoc.schema.nodes()
YIELD label, properties, type
RETURN label, properties, type;
```

### Memory Trick

```text
apoc.schema = programmatic version of SHOW CONSTRAINTS / SHOW INDEXES
```

---

# 16. `apoc.algo.*` — Legacy Graph Algorithms

## Theory

Before the **Graph Data Science (GDS)** library existed, APOC shipped its own graph algorithms (PageRank, shortest path variants, etc.). GDS is now the standard, production-grade choice — `apoc.algo` mostly survives for backward compatibility.

## Syntax & Example

```cypher
MATCH (a:Person {name:"Alice"}), (d:Person {name:"Dave"})
CALL apoc.algo.dijkstra(a, d, "KNOWS", "weight")
YIELD path, weight
RETURN path, weight;
```

> ⚠️ **Notes:** (1) `apoc.algo.dijkstraWithDefaultWeight` was **removed** in APOC 5.0 — use `apoc.algo.dijkstra(start, end, 'REL', 'weightProp', defaultWeight, numberOfWantedPaths)`. (2) The sample `KNOWS` relationships in this dataset have **no `weight` property**, so pass a default weight if you run the example:
>
> ```cypher
> MATCH (a:Person {name:"Alice"}), (d:Person {name:"Dave"})
> CALL apoc.algo.dijkstra(a, d, "KNOWS", "weight", 1.0)
> YIELD path, weight
> RETURN path, weight;
> ```

### Interview Trap

Don't say "APOC and GDS are the same thing." Say:

> APOC has some legacy algorithm procedures, but GDS is the dedicated, actively developed graph algorithms library — that's what I'd reach for in production for PageRank, community detection, centrality, or embeddings.

### Memory Trick

```text
apoc.algo = the "before GDS existed" algorithms — GDS is the modern choice now
```

---

# 17. `apoc.util.*` / `apoc.log.*` — Misc Utilities

## Theory

A grab bag: validation helpers, sleep/wait for testing, and structured logging from inside a Cypher query.

## Syntax & Examples

```cypher
// Fail the query with a custom message if a condition isn't met
MATCH (c:Customer {customerId:"C1"})
CALL apoc.util.validate(c.name IS NULL, "Customer name cannot be null", [])
RETURN c;
```

```cypher
// Write to the Neo4j log from within a query
CALL apoc.log.info("Migration batch completed successfully");
```

### Memory Trick

```text
apoc.util/apoc.log = validation + logging you'd otherwise have to do in application code
```

---

# 18. `apoc.help` / `apoc.version` — Discovery

## Theory

Since APOC availability and exact procedure names vary by version/environment, these let you check what's actually installed before relying on a specific procedure.

## Syntax & Examples

```cypher
// Search for anything related to "json"
CALL apoc.help("json");
```

```cypher
// List every installed apoc.* procedure/function
SHOW PROCEDURES YIELD name
WHERE name STARTS WITH 'apoc.'
RETURN name
ORDER BY name;
```

```cypher
RETURN apoc.version();
```

### Memory Trick

```text
apoc.help = "what APOC procedures actually exist in this database right now?"
```

---

# Practice Questions

Each one names the APOC namespace it targets — try to write the query yourself before checking the solution.

### PQ1. Deduplicate a list (`apoc.coll`)

**Question:** Given the list `["Alice", "Bob", "Alice", "Carol", "Bob"]`, return the unique names.

**Solution:**
```cypher
RETURN apoc.coll.toSet(["Alice", "Bob", "Alice", "Carol", "Bob"]) AS uniqueNames;
```

### PQ2. Find shared skills between two candidates (`apoc.coll`)

**Question:** Candidate A knows `["Cypher", "Python", "AWS"]`, Candidate B knows `["Python", "Docker", "AWS"]`. Return the skills they have in common.

**Solution:**
```cypher
RETURN apoc.coll.intersection(
  ["Cypher", "Python", "AWS"],
  ["Python", "Docker", "AWS"]
) AS sharedSkills;
```

### PQ3. Serialize a customer's purchase history to JSON (`apoc.convert`)

**Question:** For Alice, return a JSON string containing her name and a list of product names she purchased.

**Solution:**
```cypher
MATCH (c:Customer {name:"Alice"})-[:PURCHASED]->(p:Product)
WITH c.name AS name, collect(p.name) AS products
RETURN apoc.convert.toJson({name: name, products: products}) AS json;
```

### PQ4. Parse a JSON payload back into properties (`apoc.convert`)

**Question:** Given the JSON string `'{"customerId":"C7","name":"Grace"}'`, create a matching `Customer` node.

**Solution:**
```cypher
WITH apoc.convert.fromJsonMap('{"customerId":"C7","name":"Grace"}') AS data
MERGE (c:Customer {customerId: data.customerId})
SET c.name = data.name
RETURN c;
```

### PQ5. Create a node with a label decided at query time (`apoc.create`)

**Question:** Write a query that creates a node whose label comes from a parameter `$label` (e.g. `"VIP"`), something plain `CREATE (:$label)` cannot do.

**Solution:**
```cypher
CALL apoc.create.node([$label], {name: "Priority Customer"})
YIELD node
RETURN node;
```

### PQ6. Rename a relationship type across the whole graph (`apoc.refactor`)

**Question:** Rename every `KNOWS` relationship to `CONNECTED_TO`.

**Solution:**
```cypher
MATCH (:Person)-[r:KNOWS]->(:Person)
CALL apoc.refactor.setType(r, "CONNECTED_TO")
YIELD output
RETURN count(output) AS relationshipsRenamed;
```

### PQ7. Promote a property to a label (`apoc.refactor` / `apoc.create`)

**Question:** Every `Product` has a `category` property. Add each product's category as an extra label on that node (e.g. a Laptop gets `:Product:Electronics`).

**Solution:**
```cypher
MATCH (p:Product)
CALL apoc.create.addLabels(p, [p.category])
YIELD node
RETURN node;
```

### PQ8. Traverse up to 2 hops of a specific relationship type (`apoc.path`)

**Question:** Starting from Alice, find everyone reachable within 2 hops of outgoing `KNOWS` relationships.

**Solution:**
```cypher
MATCH (start:Person {name:"Alice"})
CALL apoc.path.expandConfig(start, {
    relationshipFilter: "KNOWS>",
    minLevel: 1,
    maxLevel: 2
})
YIELD path
RETURN path;
```

### PQ9. Batch-update every Customer without one giant transaction (`apoc.periodic`)

**Question:** Add a `loyaltyTier: "standard"` property to every `Customer` node, processed in batches of 500.

**Solution:**
```cypher
CALL apoc.periodic.iterate(
  "MATCH (c:Customer) RETURN c",
  "SET c.loyaltyTier = 'standard'",
  {batchSize: 500}
)
YIELD batches, total
RETURN batches, total;
```

### PQ10. Get a quick statistical overview of the whole graph (`apoc.meta`)

**Question:** Without writing your own aggregation query, find out how many labels, relationship types, nodes, and relationships exist in the database.

**Solution:**
```cypher
CALL apoc.meta.stats()
YIELD labelCount, relTypeCount, nodeCount, relCount
RETURN labelCount, relTypeCount, nodeCount, relCount;
```

### PQ11. Check what JSON-related APOC procedures are available (`apoc.help`)

**Question:** You're not sure of the exact JSON procedure names in this environment — find them.

**Solution:**
```cypher
CALL apoc.help("json");
```

---

# Interview Questions

## Q1. What does APOC stand for, and what is it?

APOC stands for **Awesome Procedures On Cypher**. It's a plugin library for Neo4j that adds hundreds of extra procedures and functions covering things native Cypher doesn't handle well — JSON/XML processing, bulk import/export, dynamic labels/types, graph refactoring, batching large writes, database triggers, and schema introspection.

---

## Q2. Is APOC part of core Neo4j?

No. It's a separate plugin that must be installed (a `.jar` in the `plugins/` folder for self-managed instances, or pre-installed/one-click in Desktop and Aura). Some procedures also need explicit config flags (e.g. `apoc.import.file.enabled=true`) before they'll run.

---

## Q3. When would you use APOC instead of native Cypher?

When native Cypher can't express what you need directly — dynamic labels/relationship types decided at runtime, JSON parsing/serialization, importing from CSV/JSON/JDBC, batching a write across millions of rows safely, renaming relationship types, merging duplicate nodes, or running Cypher automatically via triggers. The right framing: prefer native Cypher first, reach for APOC when it provides a clear capability Cypher lacks.

---

## Q4. What's the difference between an APOC procedure and an APOC function?

A **procedure** (called with `CALL apoc.xxx(...)  YIELD ...`) can return multiple rows and perform side effects like writes or I/O. A **function** (called inline, e.g. `RETURN apoc.xxx(...)`) returns a single value and can be used directly inside `RETURN`, `WHERE`, or `WITH` clauses like a built-in function.

---

## Q5. How would you safely update millions of nodes without one giant transaction?

Use `apoc.periodic.iterate`, passing a `MATCH`-only query to fetch rows and a separate write statement to run per batch:

```cypher
CALL apoc.periodic.iterate(
  "MATCH (c:Customer) RETURN c",
  "SET c.migrated = true",
  {batchSize: 1000}
)
YIELD batches, total, errorMessages
RETURN batches, total, errorMessages;
```

This runs the writes in small, separate transactions instead of one massive one, avoiding memory blowups and long lock contention.

⚠️ **Modern alternative:** on Neo4j 5+ you can do the same natively, with no plugin:

```cypher
:auto
MATCH (c:Customer)
CALL (c) {
  SET c.migrated = true
} IN TRANSACTIONS OF 1000 ROWS;
```

Interview answer: *"I'd use native `CALL {} IN TRANSACTIONS` by default, and reach for `apoc.periodic.iterate` when I need its retry/error-reporting options or I'm on an older setup."*

---

## Q6. How do you rename a relationship type in Neo4j? Can Cypher do this natively?

No — Cypher has no native `ALTER` for relationship types; a relationship type is fixed at creation. You have to use `apoc.refactor.setType(rel, "NEW_TYPE")`, which internally creates a new relationship of the new type and removes the old one.

---

## Q7. How would you merge two duplicate nodes that represent the same real-world entity?

`apoc.refactor.mergeNodes([nodeA, nodeB], {properties: "combine"})` — it merges both nodes' relationships and properties into one surviving node, with configurable conflict-resolution for properties (`"combine"`, `"overwrite"`, `"discard"`).

---

## Q8. What's the difference between `apoc.algo` and GDS (Graph Data Science)?

`apoc.algo` is APOC's older, legacy set of graph algorithms (shortest path, some centrality). GDS is Neo4j's dedicated, actively maintained graph algorithms library with far more algorithms (PageRank, Louvain community detection, node embeddings like FastRP/Node2Vec, similarity, etc.) and better performance at scale. In an interview, the correct position is: GDS is the modern, production choice; `apoc.algo` mostly exists for backward compatibility.

---

## Q9. How do you inspect what's actually in a graph you didn't build?

`CALL apoc.meta.stats()` for high-level counts (labels, relationship types, nodes, relationships), or `CALL apoc.meta.schema()` for the full label/relationship-type structure — similar in spirit to querying `INFORMATION_SCHEMA` in a relational database.

---

## Q10. How does APOC help with JSON, and why does Neo4j need help with JSON at all?

Neo4j property values can only be primitives or lists of primitives — you cannot store a nested object as a single property natively. `apoc.convert.toJson()` serializes a map/object into a JSON string (which *can* be stored as a plain string property or sent to an external API), and `apoc.convert.fromJsonMap()` / `fromJsonList()` parse a JSON string back into a usable Cypher map or list.

---

## Q11. What are virtual nodes/relationships in APOC, and why use them?

`apoc.create.vNode()` / `apoc.create.vRelationship()` create nodes/relationships that exist **only in the query result** — they are never persisted to the graph. They're useful for building custom visualizations or preview/summary nodes (e.g. an aggregate "Total: $1,250" node) without polluting the real data model.

---

## Q12. How would you import a large CSV file of customers into Neo4j?

`apoc.load.csv("file:///customers.csv") YIELD map` streams each row as a map, which you then pipe into a `MERGE` (using a business key like `customerId`) to upsert without creating duplicates — usually wrapped in `apoc.periodic.iterate` if the file is large, to batch the writes.

---

## Q13. Does Neo4j support database triggers natively? How do you get similar behavior?

No native trigger system exists in core Neo4j. APOC provides one: `apoc.trigger.install(databaseName, name, statement, selector, config)` registers Cypher that runs automatically before/after node or relationship changes — for example, auto-stamping a `createdAt` timestamp on every new node.

⚠️ The older `apoc.trigger.add(name, statement, selector)` (plus `remove`, `removeAll`, `pause`, `resume`) is **deprecated in Cypher 5 and removed in Cypher 25**; use `install`, `drop`, `dropAll`, `stop`, `start` instead. Triggers must also be enabled with `apoc.trigger.enabled=true` in `apoc.conf`.

---

## Q14. What's a real risk of using `apoc.load.json` or `apoc.load.csv` against an external URL in production?

It introduces an external dependency and potential security/availability risk — the query depends on a third-party endpoint being reachable and returning well-formed data at query time. In production pipelines, it's usually safer to first land the file in trusted, controlled storage (or pass validated data in as parameters) rather than trusting `apoc.load` to hit an arbitrary external URL directly inside a live Cypher query.

---

## Q15. Give an example of a bad use of APOC — something you'd push back on in a code review.

Using `apoc.create.node()` with a dynamic/user-supplied label string with no allow-list or validation — that lets arbitrary label names flow straight from user input into the graph schema, which is both a data-quality risk and, in a web-facing system, an injection-style risk. The fix: validate/allow-list the possible label values before passing them to APOC.

---

# APOC + Python — Calling APOC from Application Code

## Theory

In real production systems, nobody runs raw Cypher by hand in Neo4j Browser — application backends, ETL jobs, and data pipelines call Neo4j through an official driver. In Python, that's the `neo4j` package. APOC procedures/functions are called **exactly like any other Cypher** from Python — you're just sending a Cypher string with `apoc.xxx(...)` in it, with parameters bound safely using `$paramName` placeholders instead of string-formatting values into the query (which would risk Cypher injection, the same way unparameterized SQL risks SQL injection).

## Setup & Syntax

```python
from neo4j import GraphDatabase

driver = GraphDatabase.driver(
    "neo4j+s://<your-host>",
    auth=("neo4j", "<password>")
)

# The simplified driver API (Neo4j Python driver 5.x+)
records, summary, keys = driver.execute_query(
    "RETURN apoc.version() AS version",
    database_="neo4j"
)

print(records[0]["version"])
driver.close()
```

**Key syntax rules:**
- Always pass values as **query parameters** (`$skills`, `$payload`), never by f-string/`.format()`-ing them into the Cypher text.
- `execute_query()` returns a 3-tuple: `(records, summary, keys)`.
- Use `database_="neo4j"` (note the trailing underscore) to target a specific database in a multi-database setup.
- For older driver versions or manual transaction control, use `with driver.session(database="neo4j") as session: session.run(...)` instead.

### Memory Trick

```text
APOC from Python = same Cypher, just sent through driver.execute_query()
Parameters ($name), never string-formatting — that's the injection-safe rule
```

---

## Real-Time / Production Examples

### RT1. Bulk customer upsert from an API payload (`apoc.periodic.iterate` + Python)

**Scenario:** Your backend receives a batch of new customers from an upstream system (e.g. a nightly CRM export) as a Python list of dicts, and needs to upsert thousands of them safely.

```python
new_customers = [
    {"customerId": "C101", "name": "Grace"},
    {"customerId": "C102", "name": "Henry"},
    # ... thousands more
]

driver.execute_query(
    """
    UNWIND $rows AS row
    CALL apoc.periodic.iterate(
      "UNWIND $rows AS row RETURN row",
      "MERGE (c:Customer {customerId: row.customerId})
       SET c.name = row.name",
      {batchSize: 500, params: {rows: $rows}}
    )
    YIELD batches, total, errorMessages
    RETURN batches, total, errorMessages
    """,
    rows=new_customers,
    database_="neo4j"
)
```

**Why this matters in production:** running a plain `UNWIND` + `MERGE` over 500,000 rows in one transaction risks running out of memory or holding locks too long. Wrapping it in `apoc.periodic.iterate` from the Python layer keeps each batch small and independently committed — the standard pattern for nightly ETL jobs.

### RT2. Deduplicating tags from a resume-parsing pipeline (`apoc.coll`)

**Scenario:** An NLP pipeline extracts skill mentions from a resume, but produces duplicates and inconsistent casing before your Python code cleans it up.

```python
extracted_skills = ["Neo4j", "python", "AWS", "Neo4j", "Python", "aws"]

records, _, keys = driver.execute_query(
    """
    RETURN apoc.coll.toSet(
        [s IN $skills | apoc.text.capitalize(toLower(s))]
    ) AS cleanSkills
    """,
    skills=extracted_skills,
    database_="neo4j"
)

print(records[0]["cleanSkills"])
# -> ['Neo4j', 'Python', 'Aws']
```

### RT3. Serializing a query result to JSON for a REST API response (`apoc.convert.toJson`)

**Scenario:** A Flask/FastAPI endpoint needs to return a customer's purchase history as a JSON string, built inside the query instead of reshaping rows in Python.

```python
records, _, _ = driver.execute_query(
    """
    MATCH (c:Customer {customerId: $customerId})-[:PURCHASED]->(p:Product)
    WITH c.name AS name, collect(p.name) AS products
    RETURN apoc.convert.toJson({name: name, products: products}) AS json
    """,
    customerId="C1",
    database_="neo4j"
)

api_response_body = records[0]["json"]
```

### RT4. Parsing an inbound webhook payload and merging it into the graph (`apoc.convert.fromJsonMap`)

**Scenario:** A third-party webhook (e.g. a payment provider) posts a raw JSON string to your backend; you pass it straight through to Cypher instead of pre-parsing it in Python.

```python
webhook_body = '{"orderId":"O5001","customerId":"C1","productId":"P1","amount":1200}'

driver.execute_query(
    """
    WITH apoc.convert.fromJsonMap($body) AS event
    MATCH (c:Customer {customerId: event.customerId})
    MATCH (p:Product {productId: event.productId})
    MERGE (c)-[r:PURCHASED {orderId: event.orderId}]->(p)
    SET r.amount = event.amount
    """,
    body=webhook_body,
    database_="neo4j"
)
```

### RT5. Dynamic labeling decided by business logic in Python (`apoc.merge.node`)

**Scenario:** A Python service classifies customers into tiers (`Standard`, `Premium`, `VIP`) using business logic that lives in the application, not in Cypher — the label to apply isn't known until runtime.

```python
def upsert_customer_with_tier(driver, customer_id, name, total_spend):
    tier = "VIP" if total_spend > 1000 else "Premium" if total_spend > 200 else "Standard"

    driver.execute_query(
        """
        CALL apoc.merge.node(
            ["Customer", $tier],
            {customerId: $customerId},
            {name: $name}
        ) YIELD node
        RETURN node
        """,
        tier=tier,
        customerId=customer_id,
        name=name,
        database_="neo4j"
    )

upsert_customer_with_tier(driver, "C1", "Alice", 1250)
```

### RT6. Scheduled subgraph export for a nightly backup job (`apoc.export.csv.query`)

**Scenario:** A cron-triggered Python script exports the day's purchase activity to CSV for a downstream analytics system.

```python
driver.execute_query(
    """
    CALL apoc.export.csv.query(
        "MATCH (c:Customer)-[r:PURCHASED]->(p:Product)
         RETURN c.customerId AS customerId, p.name AS product, r.amount AS amount",
        "/backups/daily_purchases.csv",
        {}
    )
    """,
    database_="neo4j"
)
```

### RT7. Health-check / monitoring script using `apoc.meta.stats()`

**Scenario:** An ops dashboard polls basic graph size metrics every few minutes to detect unexpected growth or drop-offs (e.g. a failed ingestion job).

```python
def get_graph_health(driver):
    records, _, _ = driver.execute_query(
        "CALL apoc.meta.stats() YIELD labelCount, relTypeCount, nodeCount, relCount "
        "RETURN labelCount, relTypeCount, nodeCount, relCount",
        database_="neo4j"
    )
    return records[0].data()

print(get_graph_health(driver))
# -> {'labelCount': 4, 'relTypeCount': 3, 'nodeCount': 13, 'relCount': 8}
```

### Memory Trick

```text
Real-time APOC+Python = ETL upsert, JSON in/out, dynamic tiers, scheduled export, health checks
Same idea every time: Python builds the parameters, APOC does the graph-native heavy lifting
```

---

## Practice Examples — APOC + Python

### PPy1. Clean and deduplicate a list of interests

**Question:** Given a Python list `["reading", "Reading", "hiking", "HIKING", "cooking"]`, return the unique, capitalized interests using APOC — don't dedupe in Python.

**Solution:**
```python
interests = ["reading", "Reading", "hiking", "HIKING", "cooking"]

records, _, _ = driver.execute_query(
    """
    RETURN apoc.coll.toSet(
        [i IN $interests | apoc.text.capitalize(toLower(i))]
    ) AS uniqueInterests
    """,
    interests=interests,
    database_="neo4j"
)
print(records[0]["uniqueInterests"])
# -> ['Reading', 'Hiking', 'Cooking']
```

### PPy2. Return a customer's full purchase history as JSON, from Python

**Question:** Write a Python function `get_customer_json(driver, customer_id)` that returns a JSON string of `{name, products: [...], totalSpend}` for a given customer.

**Solution:**
```python
def get_customer_json(driver, customer_id):
    records, _, _ = driver.execute_query(
        """
        MATCH (c:Customer {customerId: $customerId})-[r:PURCHASED]->(p:Product)
        WITH c.name AS name, collect(p.name) AS products, sum(r.amount) AS totalSpend
        RETURN apoc.convert.toJson({
            name: name, products: products, totalSpend: totalSpend
        }) AS json
        """,
        customerId=customer_id,
        database_="neo4j"
    )
    return records[0]["json"]

print(get_customer_json(driver, "C1"))
```

### PPy3. Bulk-tag every customer above a spend threshold, in batches

**Question:** From Python, add a `highValue: true` property to every customer whose total spend exceeds `$1000`, using `apoc.periodic.iterate` so it's safe at scale.

**Solution:**
```python
driver.execute_query(
    """
    CALL apoc.periodic.iterate(
      "MATCH (c:Customer)-[r:PURCHASED]->(:Product)
       WITH c, sum(r.amount) AS totalSpend
       WHERE totalSpend > $threshold
       RETURN c",
      "SET c.highValue = true",
      {batchSize: 200, params: {threshold: $threshold}}
    )
    YIELD batches, total
    RETURN batches, total
    """,
    threshold=1000,
    database_="neo4j"
)
```

### PPy4. Parse and ingest a batch of JSON events from an external queue

**Question:** You're pulling messages off a queue (e.g. Kafka/SQS) as raw JSON strings in Python. Write a function that parses and merges each one into the graph as a `PURCHASED` relationship, using APOC to do the JSON parsing inside Cypher rather than Python's own `json` module.

**Solution:**
```python
def ingest_purchase_event(driver, raw_json: str):
    driver.execute_query(
        """
        WITH apoc.convert.fromJsonMap($raw) AS event
        MATCH (c:Customer {customerId: event.customerId})
        MATCH (p:Product {productId: event.productId})
        MERGE (c)-[r:PURCHASED]->(p)
        SET r.amount = event.amount
        """,
        raw=raw_json,
        database_="neo4j"
    )

ingest_purchase_event(driver, '{"customerId":"C2","productId":"P2","amount":25}')
```

**Interview point:** why parse JSON with APOC instead of Python's `json.loads()`? Either works — but doing it with `apoc.convert.fromJsonMap` inside the Cypher keeps the parsing and the graph write in the **same query/transaction**, and works well when the JSON is arriving as a property value or from a nested `apoc.load.json` call rather than fully materialized in Python first.

---

# Appendix A — Deprecated, Removed & Changed Items ⚠️

Reviewed against the official Neo4j APOC docs ("Deprecations and additions", per-procedure pages, config and Cypher-version pages), Sept 2026.

## A.1 Triggers (deprecated in Cypher 5, removed in Cypher 25)

| Old | New |
|---|---|
| `apoc.trigger.add` | `apoc.trigger.install(db, name, stmt, selector, config)` |
| `apoc.trigger.remove` | `apoc.trigger.drop(db, name)` |
| `apoc.trigger.removeAll` | `apoc.trigger.dropAll(db)` |
| `apoc.trigger.pause` | `apoc.trigger.stop(db, name)` |
| `apoc.trigger.resume` | `apoc.trigger.start(db, name)` |

## A.2 Deprecated — use native Cypher

| Deprecated | Replacement |
|---|---|
| `apoc.text.join(list, delim)` | `string.join(list, delim)` per APOC docs (verify on your version), or a `reduce(...)` fallback |
| `apoc.date.format(...)` | Native Cypher date/time formatting, e.g. `toString(date(datetime({epochMillis: ms})))` |
| `apoc.create.uuid()` | `randomUUID()` |
| `apoc.create.uuids(n)` | `UNWIND range(1, n) AS row RETURN row, randomUUID() AS uuid` (also not available in Cypher 25) |
| `apoc.custom.declareFunction` / `declareProcedure` | `apoc.custom.installFunction` / `installProcedure` |
| `apoc.custom.removeFunction` / `removeProcedure` | `apoc.custom.dropFunction` / `dropProcedure` |

## A.3 Removed in APOC 5.0 (old name → replacement)

| Removed | Replacement |
|---|---|
| `apoc.periodic.rock_n_roll`, `rock_n_roll_while` | `CALL {} IN TRANSACTIONS OF n ROWS` |
| `apoc.export.cypherAll / cypherData / cypherGraph / cypherQuery` | `apoc.export.cypher.all / data / graph / query` |
| `apoc.meta.type / types / isType / typeName` | `apoc.meta.cypher.type / types / isType` |
| `apoc.math.round(...)` | native `round(value, precision)` |
| `apoc.coll.reverse(list)` | native `reverse(list)` |
| `apoc.date.expire / expireIn` | `apoc.ttl.expire / expireIn` |
| `apoc.date.parseAsZonedDateTime` | `apoc.temporal.toZonedTemporal` |
| `apoc.algo.dijkstraWithDefaultWeight` | `apoc.algo.dijkstra(..., defaultValue, numberOfWantedResults)` |
| `apoc.refactor.cloneNodesWithRelationships` | `apoc.refactor.cloneNodes(nodes, true, skipProperties)` |
| `apoc.create.vPattern / vPatternFull` | `apoc.create.virtualPath` |
| `apoc.xml.import` | `apoc.import.xml` |
| `apoc.mongodb.*` | `apoc.mongo.*` |
| `apoc.load.jdbcParams` | `apoc.load.jdbc(urlOrKey, '', [params])` |
| `apoc.cypher.runFirstColumn` | `apoc.cypher.runFirstColumnMany` / `runFirstColumnSingle` |

## A.4 Configuration changes

| Old habit | Current |
|---|---|
| `apoc.*` settings in `neo4j.conf` | Put all `apoc.*` settings in **`conf/apoc.conf`** (Neo4j 5+) or environment variables |
| `apoc.initializer.cypher` | Removed — use database-specific `apoc.initializer.<databaseName>` |
| Triggers "just work" | Must set `apoc.trigger.enabled=true` |

## A.5 Checked and still current

`apoc.coll.toSet`, `apoc.convert.toJson`, `apoc.convert.fromJsonMap`, `apoc.convert.fromJsonList`, and `apoc.periodic.iterate` are all still listed in the current docs (`iterate` is not deprecated, though native `CALL {} IN TRANSACTIONS` is often the better first choice).

## A.6 Not individually verified

I did **not** check each of these one by one: `apoc.refactor.setType`, `apoc.create.addLabels`, `apoc.util.validate`, `apoc.load.*`, `apoc.meta.stats` / `apoc.meta.schema`, `apoc.merge.node`, `apoc.schema.nodes`, `apoc.map.*`, `apoc.path.expandConfig`. No deprecation was found for them, but confirm on the docs page for your APOC version before relying on them in an interview or production.

**How to verify quickly:**

```cypher
RETURN apoc.version();          // which APOC build is installed
CALL apoc.help("trigger");      // list what your install actually exposes
```

Then check the procedure's own page in the APOC docs (deprecated ones carry a notice at the top).

## A.7 Interview soundbite

> "APOC has evolved with Neo4j 5 and Cypher 25 — I check the deprecation list for the version I'm on. For example, I use `apoc.trigger.install` rather than the old `apoc.trigger.add`, native `CALL {} IN TRANSACTIONS` for simple batching, and native Cypher functions where APOC ones have been deprecated."

---

# Quick Reference Cheat Sheet

| Need to... | Use |
|---|---|
| Deduplicate / intersect / diff lists | `apoc.coll.*` |
| Clean up or compare strings | `apoc.text.*` ⚠️ (`join` deprecated) |
| Convert between JSON and Cypher maps/lists | `apoc.convert.*` |
| Merge/filter maps | `apoc.map.*` |
| Convert epoch <-> formatted date | Native `date()` / `datetime()` first; `apoc.date.*` ⚠️ (some deprecated) |
| Create a node/relationship with a runtime label/type | `apoc.create.*` |
| Find-or-create with a runtime label | `apoc.merge.*` |
| Rename a relationship type / merge duplicate nodes | `apoc.refactor.*` |
| Traverse with fine-grained filters/depth | `apoc.path.expandConfig` |
| Import CSV/JSON/JDBC/XML | `apoc.load.*` |
| Export to CSV/JSON/Cypher/GraphML | `apoc.export.*` |
| Safely batch a huge write | Native `CALL {} IN TRANSACTIONS` first; `apoc.periodic.iterate` for retries / parallel |
| Run Cypher automatically on data changes | `apoc.trigger.install / drop / start / stop` ⚠️ (not `add`) |
| Inspect graph/schema shape | `apoc.meta.*`, `apoc.schema.*` |
| Legacy graph algorithms (prefer GDS instead) | `apoc.algo.*` |
| Validate / log inside a query | `apoc.util.*`, `apoc.log.*` |
| Discover what's installed | `apoc.help`, `apoc.version` |

### Final Memory Formula

```text
APOC = Awesome Procedures On Cypher
"If Cypher can't do it natively, APOC probably can."
Native Cypher first. APOC when it adds real capability.
```
