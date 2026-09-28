# Neo4j, GraphRAG, MCP, RAG & Cypher — 50 Practice Questions

This study guide contains 50 multiple-choice questions from the discussion above, with the correct answer and a short explanation for each.

---

## 1. Filtering Entities During Resolution

**Question:** Complete the code to exclude entities with a `:Resolved` label from the entity resolution process.

```python
from neo4j_graphrag.experimental.components.resolver import (
    SinglePropertyExactMatchResolver,
)

# select one
filter_query = ?

resolver = SinglePropertyExactMatchResolver(
    driver,
    filter_query=filter_query
)

res = await resolver.run()
```

**Choices:**
- A. `filter_query = "WHERE NOT entity:Resolved"`
- B. `exclude_labels = ["Resolved"]`
- C. `filter_resolved = True`
- D. `skip_query = "entity:Resolved"`

**Answer:** A. `filter_query = "WHERE NOT entity:Resolved"`

**Explanation:** The resolver accepts a Cypher filter condition. `WHERE NOT entity:Resolved` excludes nodes that already have the `Resolved` label.

---

## 2. Providing Extraction Context

**Question:** What additional information can you provide to an LLM to improve entity and relationship extraction?

**Choices:**
- A. Historical query logs
- B. The server IP address
- C. Context or constraints about entity types or relationships of interest
- D. The database password

**Answer:** C. Context or constraints about entity types or relationships of interest

**Explanation:** Giving the LLM domain context and extraction constraints helps it focus on the entities and relationships that matter for the use case.

---

## 3. GraphRAG vs Vector RAG

**Question:** What is one key advantage of GraphRAG over traditional vector-only RAG approaches?

**Choices:**
- A. GraphRAG requires less storage space than vector embeddings
- B. GraphRAG processes queries faster than vector similarity search
- C. GraphRAG captures relationships between entities, enabling multi-hop reasoning
- D. GraphRAG eliminates the need for embedding models entirely

**Answer:** C. GraphRAG captures relationships between entities, enabling multi-hop reasoning

**Explanation:** GraphRAG can follow explicit relationships between entities, making it useful for questions that require reasoning across multiple connected facts.

---

## 4. Performance Characteristic

**Question:** What remains constant in Neo4j regardless of the overall size of the data when traversing relationships?

**Choices:**
- A. Disk space required for relationships
- B. Number of nodes that can be stored
- C. Query time for relationship traversals
- D. Memory usage during queries

**Answer:** C. Query time for relationship traversals

**Explanation:** Neo4j stores relationships as direct graph connections, so traversing from one connected node to another does not require scanning the entire database.

---

## 5. Knowledge Graph Structure

**Question:** What do knowledge graphs provide to enable comprehensive understanding of information?

**Choices:**
- A. Only unstructured text documents
- B. Structured representation of entities, attributes, and their relationships
- C. Pre-computed query results for all possible questions
- D. Only hierarchical parent-child relationships

**Answer:** B. Structured representation of entities, attributes, and their relationships

**Explanation:** Knowledge graphs organize information as entities, properties/attributes, and explicit relationships between entities.

---

## 6. Schema Discovery Benefits

**Question:** What is the primary benefit of using the `get-neo4j-schema` tool before generating Cypher queries?

**Choices:**
- A. It backs up the database schema
- B. It validates user credentials
- C. It prevents the LLM from hallucinating incorrect schema elements
- D. It reduces query execution time
- E. It optimizes database indexes

**Answer:** C. It prevents the LLM from hallucinating incorrect schema elements

**Explanation:** By seeing the real labels, relationship types, and properties first, the LLM is less likely to invent schema elements that do not exist.

---

## 7. Creating a Vector Index

**Question:** Complete the Cypher statement to create a vector index on the `embedding` property of `Question` nodes.

```cypher
-- select --
FOR (q:Question)
ON q.embedding
OPTIONS {indexConfig: {
  `vector.dimensions`: 1536,
  `vector.similarity_function`: 'cosine'
}}
```

**Choices:**
- A. `CREATE VECTOR INDEX question_embeddings`
- B. `CREATE INDEX question_embeddings`
- C. `CREATE VECTOR question_embeddings`
- D. `VECTOR INDEX question_embeddings`

**Answer:** A. `CREATE VECTOR INDEX question_embeddings`

**Explanation:** Neo4j uses the `CREATE VECTOR INDEX` syntax for indexes designed to support vector similarity search.

---

## 8. Checking Vector Index Status

**Question:** How can you check the status of vector indexes in Neo4j?

**Choices:**
- A. `GET INDEX STATUS`
- B. `LIST INDEXES VECTOR`
- C. `SHOW INDEXES WHERE type = "VECTOR"`
- D. `CHECK VECTOR INDEX`

**Answer:** C. `SHOW INDEXES WHERE type = "VECTOR"`

**Explanation:** `SHOW INDEXES` lists database indexes, and filtering on `type = "VECTOR"` isolates vector indexes and their states.

---

## 9. The MERGE Clause

**Question:** Which of the following best describes the behavior of `MERGE` in Neo4j?

**Choices:**
- A. It always creates new nodes and relationships
- B. It deletes existing nodes and relationships before creating new ones
- C. It only updates existing nodes and relationships
- D. It matches existing patterns or creates new ones if they do not exist

**Answer:** D. It matches existing patterns or creates new ones if they do not exist

**Explanation:** `MERGE` behaves like a match-or-create operation. If the specified pattern exists, Neo4j matches it; otherwise, Neo4j creates it.

---

## 10. Why Use a Graph for RAG?

**Question:** Why is a knowledge graph often preferred over a vector-only store for production RAG applications?

**Choices:**
- A. Graphs provide faster exact keyword matching than full-text indexes built on relational tables
- B. Graph structure enables multi-hop reasoning and reduces hallucinations by grounding the LLM in explicit relationships
- C. Vector databases fundamentally cannot store or query embedding vectors efficiently
- D. Graphs require significantly less storage space than vector databases for the same amount of data

**Answer:** B. Graph structure enables multi-hop reasoning and reduces hallucinations by grounding the LLM in explicit relationships

**Explanation:** Graphs preserve explicit entity relationships, which lets retrieval traverse connected facts and provide richer structured grounding to the LLM.

---

## 11. Default Entity Label

**Question:** What is the default label applied to all extracted entities in the `SimpleKGPipeline` before a schema is defined?

```cypher
MATCH (e:__Entity__)
RETURN e
```

**Choices:**
- A. `DefaultEntity`
- B. `Node`
- C. `Entity`
- D. `__Entity__`

**Answer:** D. `__Entity__`

**Explanation:** Before a custom schema is supplied, extracted entities are given the generic `__Entity__` label.

---

## 12. Embedding Model Versioning

**Question:** You upgrade the embedding model used by your RAG system. What is a critical step to avoid broken or inconsistent retrieval?

**Choices:**
- A. Delete all existing vector indexes and never re-embed documents
- B. Use the new model only for new documents and leave old embeddings unchanged
- C. Re-embed existing content with the new model and rebuild or recreate vector indexes so new queries and stored embeddings use the same embedding space
- D. Keep old and new embeddings in the same index and query with the old model only

**Answer:** C. Re-embed existing content with the new model and rebuild or recreate vector indexes so new queries and stored embeddings use the same embedding space

**Explanation:** Embeddings from different models may not be comparable. Documents and queries should use the same embedding model and vector space.

---

## 13. Cypher Query Language

**Question:** Cypher is Neo4j's implementation of which ISO standard?

**Choices:**
- A. JSON
- B. SQL
- C. GraphQL
- D. GQL

**Answer:** D. GQL

**Explanation:** Cypher is a graph query language aligned with the ISO GQL, or Graph Query Language, standard.

---

## 14. Effective Node Labels

**Question:** According to Neo4j best practices, which of the following would be the MOST effective node labels? Select all that apply.

**Choices:**
- A. `People`
- B. `has_multiple_addresses`
- C. `Product`
- D. `Location`
- E. `WORKS_AT`
- F. `Person`

**Answer:** C. `Product`, D. `Location`, F. `Person`

**Explanation:** Node labels should usually represent entity types and are commonly written as singular nouns. `WORKS_AT` is better modeled as a relationship type.

---

## 15. MCP Architecture Components

**Question:** Which of the following are core components of the MCP architecture? Select all that apply.

**Choices:**
- A. Servers that provide capabilities through tools
- B. Hosts that maintain session state and context
- C. Databases that store MCP configurations
- D. Models that process natural language
- E. Clients that manage connections to servers
- F. Tools with unique identifiers and parameters

**Answer:** A, B, E, F

**Explanation:** MCP architecture centers on hosts, clients, and servers. Servers expose capabilities such as tools, and clients connect hosts to those servers.

---

## 16. Understanding Agents

**Question:** What framework is commonly used for building agents that continuously loop through planning, reasoning, and acting?

**Choices:**
- A. LangChain
- B. ReAct (Reason/Act)
- C. RAG (Retrieval-Augmented Generation)
- D. MCP (Model Context Protocol)

**Answer:** B. ReAct (Reason/Act)

**Explanation:** ReAct is an agent pattern in which the model repeatedly reasons, acts, observes the result, and reasons again.

---

## 17. Entity Linking in Knowledge Graphs

**Question:** What is entity linking in the context of knowledge graph creation?

**Choices:**
- A. Connecting the knowledge graph to external databases
- B. Accurately identifying and associating entities with their entries in the knowledge graph
- C. Generating hyperlinks in source documents
- D. Creating relationships between entity nodes

**Answer:** B. Accurately identifying and associating entities with their entries in the knowledge graph

**Explanation:** Entity linking maps a mention in text to the correct real-world entity represented in the graph.

---

## 18. Required Properties in Schema Definition

**Question:** Complete the code to mark the `name` property as mandatory during schema validation.

```python
PropertyType(
    name="name",
    type="STRING",
    -- select --
)
```

**Choices:**
- A. `required=True`
- B. `mandatory=True`
- C. `nullable=False`
- D. `optional=False`

**Answer:** A. `required=True`

**Explanation:** `required=True` marks the property as mandatory in the schema definition.

---

## 19. Creating Vector Indexes

**Question:** What Cypher statement is used to create a vector index in Neo4j?

**Choices:**
- A. `ADD VECTOR INDEX`
- B. `CREATE VECTOR INDEX`
- C. `CREATE INDEX FOR VECTOR`
- D. `DEFINE VECTOR INDEX`

**Answer:** B. `CREATE VECTOR INDEX`

**Explanation:** Neo4j uses `CREATE VECTOR INDEX` to define an index over vector-valued properties.

---

## 20. Text Splitter Chunking Behavior

**Question:** What happens when the `approximate` flag is set to `True` in `FixedSizeSplitter`?

**Choices:**
- A. The text is split at approximate semantic boundaries using LLM analysis
- B. Chunk sizes are estimated rather than precisely measured to improve performance
- C. The splitter uses approximate token counting instead of exact character counts
- D. The splitter ensures clean chunk boundaries by avoiding words cut in the middle whenever possible

**Answer:** D. The splitter ensures clean chunk boundaries by avoiding words cut in the middle whenever possible

**Explanation:** With `approximate=True`, chunk lengths may vary slightly so text can be split at cleaner boundaries instead of cutting a word in half.

---

## 21. Vector Definition

**Question:** What is a vector in the context of semantic search?

**Choices:**
- A. A type of neural network architecture
- B. A compression algorithm
- C. A database query language
- D. A list of numbers that can represent data like text, images, or audio

**Answer:** D. A list of numbers that can represent data like text, images, or audio

**Explanation:** Embedding models turn content into numerical vectors whose positions represent semantic characteristics.

---

## 22. Lexical Graph Components

**Question:** Which components are part of the lexical graph created during knowledge graph construction?

**Choices:**
- A. Entity nodes and relationship types only
- B. Document nodes, Chunk nodes, and `NEXT_CHUNK` relationships
- C. Schema definitions and property constraints
- D. LLM-extracted nodes and their properties

**Answer:** B. Document nodes, Chunk nodes, and `NEXT_CHUNK` relationships

**Explanation:** The lexical graph captures the source-document structure, including documents, chunks, and the sequence of chunks.

---

## 23. Relationships in Graph Databases

**Question:** How are relationships treated in graph databases compared to other database technologies?

**Choices:**
- A. Relationships are treated with the same importance as the nodes they connect
- B. Relationships are computed at query time using indexes
- C. Relationships are optional and rarely used
- D. Relationships are stored as foreign keys in separate tables

**Answer:** A. Relationships are treated with the same importance as the nodes they connect

**Explanation:** In Neo4j, nodes and relationships are both first-class graph elements, and relationships can have their own types and properties.

---

## 24. Cypher Relationship Pattern

**Question:** Complete the Cypher pattern to match `Person` nodes connected by `KNOWS` relationships.

```cypher
MATCH (p1:Person)-- select --(p2:Person)
RETURN p1.name, p2.name
```

**Choices:**
- A. `-[:KNOWS]->`
- B. `[:KNOWS]`
- C. `[KNOWS]->`
- D. `->[:KNOWS]-`

**Answer:** A. `-[:KNOWS]->`

**Explanation:** A directed Cypher relationship pattern is written as `(node)-[:TYPE]->(node)`.

---

## 25. Schema Configuration in SimpleKGPipeline

**Question:** What happens when you set `schema="EXTRACTED"` or omit the schema parameter in `SimpleKGPipeline`?

**Choices:**
- A. The schema is automatically extracted from the input text once using an LLM
- B. Entity and relation extraction proceeds without any schema guidance
- C. A predefined schema is loaded from the Neo4j database
- D. The schema must be manually defined before processing

**Answer:** B. Entity and relation extraction proceeds without any schema guidance

**Explanation:** Without a predefined schema, extraction is not constrained to specific entity or relationship types.

---

## 26. Purpose of Chunking Data

**Question:** What is the primary purpose of chunking data when constructing a knowledge graph with an LLM?

**Choices:**
- A. To create more relationships in the graph
- B. To improve query performance
- C. To reduce storage costs in Neo4j
- D. To break down data into right-sized parts for LLM processing

**Answer:** D. To break down data into right-sized parts for LLM processing

**Explanation:** Chunking keeps input small enough for reliable LLM processing and entity/relationship extraction.

---

## 27. Text2CypherRetriever Purpose

**Question:** What does the `Text2CypherRetriever` component do?

**Choices:**
- A. Translates text chunks into Cypher `CREATE` statements
- B. Embeds Cypher queries as vectors for similarity search
- C. Converts Cypher query results into natural language text
- D. Uses an LLM to convert natural language questions into Cypher queries

**Answer:** D. Uses an LLM to convert natural language questions into Cypher queries

**Explanation:** `Text2CypherRetriever` translates a user's natural-language question into a Cypher query that can retrieve data from Neo4j.

---

## 28. Relationship Direction

**Question:** In Neo4j, do all relationships have a direction?

**Choices:**
- A. Direction is optional and depends on the query pattern
- B. No, relationships can be created without direction
- C. Only relationships with certain types require direction
- D. Yes, all relationships have a direction that is stored when created

**Answer:** D. Yes, all relationships have a direction that is stored when created

**Explanation:** Every stored Neo4j relationship has a direction, although a Cypher query can match it without specifying direction.

---

## 29. VectorCypherRetriever `retrieval_query` Benefits

**Question:** What can you achieve by adding a `retrieval_query` to a `VectorCypherRetriever`? Select all that apply.

```cypher
MATCH (c:Chunk)-[:FROM_DOCUMENT]-(d:Document)-[:PDF_OF]-(l:Lesson)
MATCH (c)<-[:FROM_CHUNK]-(e:__Entity__)
RETURN c.text, l.name, l.url, collect(e.name) AS entities
```

**Choices:**
- A. Add related entities from the knowledge graph as additional context
- B. Change the number of chunks returned by the vector search
- C. Traverse relationships to gather structured data connected to chunks
- D. Include metadata from Document or Lesson nodes in the context
- E. Improve the accuracy of the vector similarity calculation itself

**Answer:** A, C, D

**Explanation:** A `retrieval_query` enriches vector-search results by traversing the graph and collecting related entities and metadata. It does not directly change the similarity calculation.

---

## 30. AI Text Completion Return Type

**Question:** What data type does the `ai.text.completion()` function return?

**Choices:**
- A. `LIST` with multiple response variations
- B. `MAP` containing the response and metadata
- C. `VECTOR` representing semantic meaning
- D. `STRING` containing the generated text

**Answer:** D. `STRING` containing the generated text

**Explanation:** `ai.text.completion()` returns generated text as a string.

---

## 31. Read Transaction Safety

**Question:** Why does the `read-neo4j-cypher` tool execute queries within read transactions?

**Choices:**
- A. To cache query results
- B. To improve query performance
- C. To enable parallel query execution
- D. To prevent the tool from modifying or deleting data
- E. To reduce memory usage

**Answer:** D. To prevent the tool from modifying or deleting data

**Explanation:** A read transaction limits operations to data retrieval and protects the database from accidental writes or deletes.

---

## 32. Required SimpleKGPipeline Components

**Question:** What components are required to create a `SimpleKGPipeline` instance? Select all that apply.

```python
pipeline = SimpleKGPipeline(
    llm=llm,
    driver=driver,
    embedder=embedder
)
```

**Choices:**
- A. A defined schema with nodes and relationships
- B. A pre-existing vector index in Neo4j
- C. An LLM
- D. A Neo4j driver instance
- E. An embedder for creating vector embeddings

**Answer:** C, D, E

**Explanation:** The required core components shown are an LLM, a Neo4j driver, and an embedder. A custom schema and pre-existing vector index are not required just to instantiate the pipeline.

---

## 33. Organizing Data in Neo4j

**Question:** When importing data into Neo4j, how should you organize your data model?

**Choices:**
- A. Store all information as properties on a single node type
- B. Use only relationships without labels to keep the model flexible
- C. Represent entities as labeled nodes and connections as relationships
- D. Store relationships as properties instead of using relationship structures

**Answer:** C. Represent entities as labeled nodes and connections as relationships

**Explanation:** Neo4j models domain objects as nodes and the connections between them as explicit relationships.

---

## 34. Creating a Neo4j Driver Instance

**Question:** Complete the Python code to create a Neo4j driver instance.

```python
from neo4j import GraphDatabase

driver = GraphDatabase.-- select --(
    uri,
    auth=(username, password)
)
```

**Choices:**
- A. `driver`
- B. `connect`
- C. `create_driver`
- D. `new_driver`

**Answer:** A. `driver`

**Explanation:** The Python Neo4j driver is created with `GraphDatabase.driver(uri, auth=(username, password))`.

---

## 35. MCP Resource URI Template

**Question:** Complete the MCP resource URI template for retrieving a movie by ID.

```python
@mcp.resource("movie://{id}")
async def get_movie(id: str, ctx: Context) -> dict:
    ...
```

**Choices:**
- A. `"movie://id"`
- B. `"movie://{id}"`
- C. `"movie/{id}"`
- D. `"movie://<id>"`

**Answer:** B. `"movie://{id}"`

**Explanation:** Dynamic MCP resource parameters are placed inside braces in the URI template.

---

## 36. MCP Hosts

**Question:** What is the role of a Host in the MCP architecture?

**Choices:**
- A. To translate natural language into tool calls
- B. To execute Cypher queries against Neo4j databases
- C. To store MCP tool definitions in a central registry
- D. To provide authentication for MCP servers
- E. To manage one or more clients and maintain session state

**Answer:** E. To manage one or more clients and maintain session state

**Explanation:** The MCP host is the main application that coordinates MCP clients and maintains the broader session/application context.

---

## 37. RAG Data Sources

**Question:** What types of data sources can be used in RAG applications? Select all that apply.

**Choices:**
- A. Only vector databases
- B. Documents such as articles, reports, and manuals
- C. Only structured SQL tables
- D. Knowledge graphs with entity relationships
- E. APIs providing real-time data

**Answer:** B, D, E

**Explanation:** RAG can retrieve from many sources, including documents, graphs, APIs, databases, and vector stores. The wording “only” makes A and C incorrect.

---

## 38. LLM Graph Builder Features

**Question:** What can you configure or customize in the Neo4j LLM Graph Builder? Select all that apply.

**Choices:**
- A. Post-processing options like de-duplication
- B. The Neo4j database storage engine
- C. The LLM model used for extraction
- D. Types of entities and relationships to extract

**Answer:** A, C, D

**Explanation:** The Graph Builder lets you control extraction behavior, the LLM, entity/relationship types, and post-processing. It does not let you replace Neo4j's underlying storage engine.

---

## 39. `ai.text.embedBatch()` Output

**Question:** Complete the Cypher query to return the index and vector from `ai.text.embedBatch()`.

```cypher
MATCH (m:Movie)
LIMIT 20

WITH collect(m.plot) AS batch

CALL ai.text.embedBatch(batch, 'OpenAI', $config)
-- select --

WITH index, vector
RETURN index, vector;
```

**Choices:**
- A. `RETURN index, vector`
- B. `YIELD index, vector`
- C. `WITH index, vector`
- D. `OUTPUT index, vector`

**Answer:** B. `YIELD index, vector`

**Explanation:** Procedure output fields are introduced into the query with `YIELD`.

---

## 40. Similarity Function

**Question:** What is the most commonly used similarity function for comparing vector embeddings in Neo4j?

**Choices:**
- A. cosine
- B. euclidean
- C. jaccard
- D. pearson

**Answer:** A. cosine

**Explanation:** Cosine similarity compares the direction of vectors and is widely used for semantic embeddings.

---

## 41. Comparing Vectors

**Question:** How are vectors used to find relevant information in RAG?

**Choices:**
- A. By counting common words
- B. By measuring text length
- C. By comparing vector representations to determine semantic similarity
- D. By matching exact text strings

**Answer:** C. By comparing vector representations to determine semantic similarity

**Explanation:** Queries and documents are converted to vectors and compared so the retriever can find semantically related content.

---

## 42. Unstructured Data Challenges

**Question:** Which of the following are challenges when analyzing unstructured data? Select all that apply.

**Choices:**
- A. Lack of Structure — does not follow a predefined model or format
- B. Variety — comes in many formats requiring different tools
- C. Volume — often massive amounts of data to process
- D. Standardization — all unstructured data follows the same format

**Answer:** A, B, C

**Explanation:** Unstructured data is challenging because it lacks a consistent structure, comes in many forms, and can exist at large scale.

---

## 43. Node Label Convention

**Question:** In Neo4j, what grammatical form should node labels typically take?

**Choices:**
- A. Singular noun, e.g. `Person`, `Product`, `Event`
- B. Verb, e.g. `Creates`, `Owns`, `Manages`
- C. Adjective, e.g. `Active`, `Primary`, `Valid`
- D. Plural noun, e.g. `Persons`, `Products`, `Events`

**Answer:** A. Singular noun

**Explanation:** Node labels represent entity types, so singular nouns are the clearest and most common naming convention.

---

## 44. Structured Output in Entity Extraction

**Question:** What is the primary benefit of enabling `use_structured_output=True` in `LLMEntityRelationExtractor` with OpenAI or Vertex AI?

**Choices:**
- A. It reduces API costs by using fewer tokens
- B. It enables parallel processing of multiple chunks simultaneously
- C. It generates more entities and relationships from the same text
- D. It passes the Pydantic model as `response_format`, ensuring automatic type validation

**Answer:** D. It passes the Pydantic model as `response_format`, ensuring automatic type validation

**Explanation:** Structured output constrains the model to a known schema and allows the response to be validated automatically.

---

## 45. Duplicate Rows

**Question:** How can you eliminate duplicate rows in the results of a Cypher query?

**Choices:**
- A. Using the `DISTINCT` keyword
- B. Using the `UNIQUE` keyword
- C. Using the `REMOVE DUPLICATES` function
- D. Using the `DELETE` clause

**Answer:** A. Using the `DISTINCT` keyword

**Explanation:** `DISTINCT` removes duplicate result rows, for example `RETURN DISTINCT p.name`.

---

## 46. Pattern Syntax

**Question:** In a Cypher pattern, what symbol is used to represent nodes?

**Choices:**
- A. Parentheses `( )`
- B. Square brackets `[ ]`
- C. Curly braces `{ }`
- D. Angle brackets `< >`

**Answer:** A. Parentheses `( )`

**Explanation:** Nodes are written inside parentheses. Relationships are commonly written inside square brackets.

---

## 47. Storing Organizing Principles

**Question:** How are organizing principles stored in a knowledge graph?

**Choices:**
- A. As nodes in the graph alongside the actual data
- B. In database metadata only
- C. In external documentation
- D. In separate configuration files

**Answer:** A. As nodes in the graph alongside the actual data

**Explanation:** Concepts such as topics, categories, and classifications can be modeled directly as nodes and linked to the underlying data.

---

## 48. Schema Property Filtering

**Question:** In the `GraphSchema` configuration, what happens when `additional_properties=False` is set for a node or relationship type?

**Choices:**
- A. The LLM is prevented from extracting any properties for that type
- B. Only properties defined in the schema are retained; all others are removed
- C. The node or relationship is removed from the graph if it has extra properties
- D. All properties are removed from the node or relationship during graph construction

**Answer:** B. Only properties defined in the schema are retained; all others are removed

**Explanation:** `additional_properties=False` restricts extraction to the properties explicitly declared in the schema.

---

## 49. Retriever Input and Output

**Question:** What type of input does a retriever typically take, and what does it search for?

**Choices:**
- A. SQL queries, searches for SQL tables
- B. Unstructured input such as questions/prompts, searches for structured data
- C. Binary input, searches for binary data
- D. Structured input, searches for unstructured data

**Answer:** B. Unstructured input such as questions/prompts, searches for structured data

**Explanation:** A retriever usually accepts a natural-language request and uses it to find relevant information from a structured or indexed knowledge source.

---

## 50. Contextual Search Example

**Question:** A user searches for **"best apple varieties for baking pies."** How does semantic search determine that `apple` refers to fruit rather than technology?

**Choices:**
- A. By always showing both fruit and technology results equally ranked
- B. By checking the user's device type and operating system only
- C. By analyzing the surrounding context words like `baking` and `pies`
- D. By randomly selecting between fruit and technology categories

**Answer:** C. By analyzing the surrounding context words like `baking` and `pies`

**Explanation:** Semantic search considers the meaning of the full query. Words like `baking` and `pies` strongly indicate the fruit meaning of `apple`.

---

# Quick Revision Summary

| Topic | Memory Trick |
|---|---|
| `MERGE` | Match if found, create if missing |
| Node syntax | `(Node)` |
| Relationship syntax | `[:RELATIONSHIP]` |
| Duplicate rows | `DISTINCT` |
| Vector search | Text → Embedding → Vector → Similarity |
| Common vector similarity | Cosine |
| GraphRAG | Similarity + relationships + multi-hop reasoning |
| Knowledge graph | Entities + properties + relationships |
| Node labels | Singular nouns |
| Relationship direction | Stored with direction |
| `Text2CypherRetriever` | Text → Cypher → Neo4j |
| SimpleKGPipeline | LLM + Driver + Embedder |
| Chunking | Break content into LLM-friendly pieces |
| Structured output | Schema-constrained + validated output |
| MCP | Host → Client → Server → Tools |
| ReAct | Reason → Act → Observe → Repeat |
| `__Entity__` | Default extracted entity label |
| `CREATE VECTOR INDEX` | Creates vector index |
| `SHOW INDEXES WHERE type = "VECTOR"` | Check vector indexes |
| `retrieval_query` | Vector result → graph enrichment |
