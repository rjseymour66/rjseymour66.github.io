+++
title = 'Storing Data'
date = '2026-05-17T08:16:24-04:00'
weight = 80
draft = false
+++

After you determine your data structures, select a database type based on these factors:

- Data structure complexity: how structured or variable your schema is
- Read versus write patterns: whether reads or writes dominate your workload
- Query complexity: whether you need joins, aggregations, or simple key lookups
- Scalability needs: horizontal versus vertical scaling requirements
- Consistency requirements: whether every read must reflect the latest write

### Database types

Different database types serve different needs:

- *Relational databases* offer structure and consistency for well-defined relationships, enforcing schemas that keep data predictable and reliable.
- *Document databases* provide flexibility for evolving or variable schemas, storing each record as an independent document.
- *Key-value stores* deliver speed for simple data access patterns, making them ideal for caching and session management.
- *Graph databases* excel at managing complex, highly connected relationships where traversing links between entities is the primary operation.
- *Vector databases* power many AI applications by storing high-dimensional embeddings and enabling similarity-based retrieval.

## Relational databases

Relational databases organize data into tables with rows and columns, and establish relationships between tables using keys. They ensure data integrity through *ACID* properties. Popular relational databases include PostgreSQL, MySQL, SQLite, Microsoft SQL Server, and Oracle Database.

### ACID properties

- *Atomicity*: transactions are all-or-nothing operations. If any part fails, the entire transaction rolls back.
- *Consistency*: the database remains in a valid state before and after each transaction.
- *Isolation*: concurrent transactions don't interfere with one another. Each transaction appears to execute in isolation, preventing issues like dirty reads or lost updates.
- *Durability*: after a transaction commits, changes are permanently stored and survive system failures.

### Use cases

Use a relational database when:

- entities have clear relationships
- your queries require joins or complex operations
- transactions and data consistency are critical
- your data is structured and follows a uniform schema

## Document databases

Document databases store data in flexible, JSON-like documents. Each document can have a different structure, making them well-suited for variable or evolving schemas. Popular document databases include MongoDB, CouchDB, Google Firestore, and Amazon DynamoDB.

### Use cases

Use a document database when:

- data structures vary among records
- you need horizontal scaling for large datasets
- your application requirements change frequently
- you're building a content management system or catalog

## Vector databases

Vector databases store data as high-dimensional vectors and retrieve results through *similarity search*, finding the items most similar to a query. They power many AI applications. Popular vector databases include Pinecone, Weaviate, Qdrant, Milvus, and Chroma.

### Use cases

- *Semantic search*: search by meaning rather than exact keywords, returning results relevant to the user's intent
- *Recommendation systems*: find items similar to what a user has previously interacted with or rated
- *Retrieval-Augmented Generation (RAG)*: retrieve contextually relevant documents to provide an LLM with accurate, up-to-date information at query time
- *Image and video search*: find visually similar images or video frames based on content rather than metadata
- *Anomaly detection*: identify data points that deviate significantly from established patterns in your dataset
- *Deduplication*: detect and merge near-duplicate records by comparing their vector representations
