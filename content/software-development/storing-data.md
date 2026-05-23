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

## Data persistence and management

How you access and manage data shapes your application's testability, maintainability, and performance. Three abstraction levels exist, each with different tradeoffs: direct database access, the repository pattern, and object-relational mapping.

### Direct database access

At this level, you write raw SQL or a database-specific query language. Direct access gives you full control over queries, but you take on responsibility for connection management, result scanning, and query correctness.

In Go, the `database/sql` package provides a standard interface for relational databases. The example uses `pgx` (`github.com/jackc/pgx/v5`), the recommended PostgreSQL driver:

```go
import (
    "context"
    "database/sql"
    _ "github.com/jackc/pgx/v5/stdlib" // registers the PostgreSQL driver
)

func getActiveProducts(ctx context.Context, db *sql.DB) ([]Product, error) {
    rows, err := db.QueryContext(ctx, `
        SELECT id, name, price
        FROM products
        WHERE active = true
        ORDER BY name
    `)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var products []Product
    for rows.Next() {
        var p Product
        if err := rows.Scan(&p.ID, &p.Name, &p.Price); err != nil {
            return nil, err
        }
        products = append(products, p)
    }
    return products, rows.Err()
}
```

You control exactly what hits the database. Use direct access when:

- query performance is critical
- you need database-specific features like `RETURNING` clauses or CTEs
- you want to avoid the overhead of an abstraction layer

### Repository pattern

The repository pattern places an abstraction layer between your business logic and data access code. It exposes a collection-like interface for working with domain objects, hiding how the underlying storage layer fetches and stores data.

The pattern offers three advantages:

- Separation of concerns: business logic never references SQL or connection details
- Testability: you can swap the real implementation for a fake during tests
- Code organization: all data access for a given type lives in one place

Define a Go interface that describes the operations your application needs:

```go
var ErrNotFound = errors.New("not found")

type UserRepository interface {
    GetByID(ctx context.Context, id int64) (*User, error)
    Save(ctx context.Context, user *User) error
    Delete(ctx context.Context, id int64) error
}
```

Implement the interface against a real database:

```go
type postgresUserRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) UserRepository {
    return &postgresUserRepository{db: db}
}

func (r *postgresUserRepository) GetByID(ctx context.Context, id int64) (*User, error) {
    row := r.db.QueryRowContext(ctx, "SELECT id, name, email FROM users WHERE id = $1", id)
    var u User
    if err := row.Scan(&u.ID, &u.Name, &u.Email); err != nil {
        return nil, err
    }
    return &u, nil
}

func (r *postgresUserRepository) Save(ctx context.Context, user *User) error {
    return r.db.QueryRowContext(ctx,
        "INSERT INTO users (name, email) VALUES ($1, $2) ON CONFLICT (email) DO UPDATE SET name = EXCLUDED.name RETURNING id",
        user.Name, user.Email,
    ).Scan(&user.ID)
}

func (r *postgresUserRepository) Delete(ctx context.Context, id int64) error {
    _, err := r.db.ExecContext(ctx, "DELETE FROM users WHERE id = $1", id)
    return err
}
```

For tests, implement the same interface in memory instead of connecting to a database:

```go
type inMemoryUserRepository struct {
    users map[int64]*User
    next  int64
}

func newInMemoryUserRepository() *inMemoryUserRepository {
    return &inMemoryUserRepository{users: make(map[int64]*User)}
}

func (r *inMemoryUserRepository) GetByID(_ context.Context, id int64) (*User, error) {
    u, ok := r.users[id]
    if !ok {
        return nil, ErrNotFound
    }
    return u, nil
}

func (r *inMemoryUserRepository) Save(_ context.Context, user *User) error {
    r.next++
    user.ID = r.next
    r.users[user.ID] = user
    return nil
}

func (r *inMemoryUserRepository) Delete(_ context.Context, id int64) error {
    delete(r.users, id)
    return nil
}
```

Your tests use the in-memory version without needing a running database:

```go
func TestSaveAndGet(t *testing.T) {
    repo := newInMemoryUserRepository()

    if err := repo.Save(t.Context(), &User{Name: "Alice", Email: "alice@example.com"}); err != nil {
        t.Fatal(err)
    }

    got, err := repo.GetByID(t.Context(), 1)
    if err != nil {
        t.Fatal(err)
    }
    if got.Name != "Alice" {
        t.Errorf("want Alice, got %s", got.Name)
    }
}
```

### Object-relational mapping

*Object-relational mapping* (ORM) is the highest level of abstraction for database interaction. An ORM bridges the gap between object-oriented programming and relational databases by mapping your domain types directly to database tables. You work with structs and method calls rather than SQL strings.

*GORM* is the most widely used ORM in Go. Define a struct and GORM handles table creation, queries, and updates:

```go
import "gorm.io/gorm"

type Product struct {
    gorm.Model
    Name  string
    Price float64
    Stock int
}

// Create a record
db.Create(&Product{Name: "Widget", Price: 9.99, Stock: 100})

// Find by primary key
var p Product
db.First(&p, 1)

// Update a field
db.Model(&p).Update("Price", 12.99)

// Delete
db.Delete(&p)
```

ORMs let you move quickly because you write less boilerplate and avoid raw SQL for common operations. The tradeoff is opacity: when queries misbehave, you need to understand both the ORM's behavior and the SQL it generates. Use `db.Debug()` to log generated queries during development.

Each GORM method returns `*gorm.DB`. Check its `Error` field in production code to handle failures. The example above omits these checks for brevity.

`gorm.Model` embeds a `DeletedAt` field that enables soft deletes. `db.Delete(&p)` sets `DeletedAt` to the current time rather than removing the row. Subsequent queries exclude soft-deleted records by default.

Use an ORM when:

- you want to move quickly on a new project
- your query patterns are simple
- schema migrations and model changes happen often

Prefer direct access or the repository pattern when:

- query performance is a primary concern
- you need complex joins, CTEs, or database-specific features
- debugging generated SQL is slowing you down

## Database connections and transactions

### Database connections

Most applications use a *connection pool* to reuse database connections rather than opening a new one for each operation. Opening a connection is expensive: it requires a network round trip, authentication, and resource allocation on the server. A pool keeps a set of connections open and lends them to goroutines on demand.

In Go, `*sql.DB` is itself a connection pool. `sql.Open` registers the driver and validates the connection string but does not open a connection. Call `db.PingContext` after opening to verify the database is reachable:

```go
import (
    "context"
    "database/sql"
    "time"
    _ "github.com/jackc/pgx/v5/stdlib"
)

func openDB(dsn string) (*sql.DB, error) {
    db, err := sql.Open("pgx", dsn)
    if err != nil {
        return nil, err
    }

    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(25)
    db.SetConnMaxLifetime(5 * time.Minute)

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    if err := db.PingContext(ctx); err != nil {
        return nil, err
    }

    return db, nil
}
```

Three settings control pool behavior:

- `SetMaxOpenConns`: the maximum number of open connections to the database. Setting this prevents your application from overwhelming the database server.
- `SetMaxIdleConns`: the maximum number of idle connections the pool retains. Idle connections are ready for reuse without the cost of opening a new one.
- `SetConnMaxLifetime`: the maximum time a connection stays in the pool before the pool closes and replaces it. Rotating connections prevents issues with stale or dropped connections.

### Transactions

A *transaction* groups multiple database operations into a single unit of work that either succeeds completely or fails completely. If any operation within the transaction fails, the database rolls back all changes made since the transaction began, leaving the data in its original state.

Use a transaction any time two or more operations must succeed together. A funds transfer is a good example: you must debit one account and credit another. If the credit fails after the debit succeeds, the data is corrupted.

```go
func transferFunds(ctx context.Context, db *sql.DB, fromID, toID int64, amount float64) error {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()

    _, err = tx.ExecContext(ctx,
        "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
        amount, fromID,
    )
    if err != nil {
        return err
    }

    _, err = tx.ExecContext(ctx,
        "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
        amount, toID,
    )
    if err != nil {
        return err
    }

    return tx.Commit()
}
```

`defer tx.Rollback()` is the idiomatic Go pattern for transaction cleanup. If `tx.Commit()` succeeds, the deferred `Rollback` is a no-op. If any operation fails before `Commit`, the function returns early and the deferred `Rollback` cleans up the transaction.

`db.BeginTx` accepts a `*sql.TxOptions` argument for setting the isolation level. Passing `nil` uses the database default, which is sufficient for most use cases.

## Consistency models

The foundation of any system lies in its ability to manage data reliably while delivering high performance.

### Consistency models

Consistency models define how your application handles data accuracy across different parts of the system. While transactions maintain data integrity on individual operations, consistency models determine how that integrity is maintained across multiple servers or when multiple users access the same information simultaneously.

*Consistency* is the guarantee that any transaction brings the database from one valid state to another, ensuring all data adheres to defined rules, constraints, and relationships without contradiction.

#### Strong consistency

Each part of your system sees the same data at the same time. The system may temporarily block operations to maintain this guarantee, prioritizing accuracy over availability.

Consider a bank account balance. When you withdraw money at an ATM, every system must immediately reflect the updated balance. If your balance is $500 and you withdraw $300, no other ATM or online banking session can show $500 after the transaction completes. The system accepts temporary blocking because accuracy is non-negotiable.

#### Eventual consistency

The system prioritizes availability by allowing temporary inconsistency that resolves over time. The system remains operational during partial outages. It works well when immediate accuracy is not critical.

Consider a social media like count. When you like a post, different users may see slightly different counts for a few seconds while servers synchronize. The brief inconsistency is acceptable because the application must remain available globally and the exact count is not critical to any individual user.

#### Causal consistency

The system maintains order between causally related operations. This provides a middle ground between strong and eventual consistency.

Consider a comment thread. If you post a reply to a comment, every user who sees your reply must also see the original comment you replied to. The system does not require all users to see all comments in the same order, but it guarantees that causally related content appears in the correct sequence.

#### Session consistency

The system ensures consistency within individual user sessions while allowing differences between sessions. This creates a reliable experience for each user without requiring global synchronization.

Consider an online checkout. During your session, every page shows the same cart contents and prices. Another user's session may reflect different inventory availability, but your session remains internally consistent throughout. The system avoids the cost of global synchronization while still delivering a predictable experience for each user.

### CAP theorem and its implications

The *CAP theorem* explains a fundamental trade-off in distributed systems: you can guarantee only two of the following three properties at the same time.

- *Consistency*: every read receives the most recent write or an error. All nodes in the system see the same data at the same time.
- *Availability*: every request receives a non-error response, even if the data is not the most recent. The system stays operational.
- *Partition tolerance*: the system continues to operate even when network failures prevent some nodes from communicating with others.

In any real distributed system, network partitions are inevitable. The practical choice is between consistency and availability when a partition occurs. This trade-off produces three system designs:

#### CP systems

CP systems prioritize consistency and partition tolerance. When a partition occurs, the system refuses requests that cannot be served consistently rather than returning stale data. Banking applications, inventory management systems, and financial services use this design because incorrect data is more costly than temporary unavailability.

#### AP systems

AP systems prioritize availability and partition tolerance, accepting eventual consistency. When a partition occurs, the system continues to serve requests with potentially stale data, and reconciles differences once the partition heals. Content delivery networks (CDNs) and social media feeds use this design because users expect the application to respond immediately even if the data is slightly out of date.

#### CA systems

CA systems prioritize consistency and availability but cannot tolerate network partitions. They typically run as single-node databases or tightly coupled clusters on reliable networks. Traditional relational database management systems (RDBMS) fall into this category. CA designs are rare in modern distributed architectures because partition tolerance is necessary at any meaningful scale.

### Choosing the right consistency model

Match the consistency model to your application's tolerance for stale data and its availability requirements.

- *Strong consistency*: financial transactions, payment processing, inventory systems where overselling has direct costs, medical records, and authentication systems.
- *Eventual consistency*: social media feeds, view and like counters, recommendation engines, search indexes, and DNS records.
- *Causal consistency*: comment threads, collaborative editors, chat applications, and version control systems.
- *Session consistency*: shopping carts, user preference settings, draft editors, and personalized dashboards.

When designing a new system, identify the cost of stale data for each operation. Use strong consistency only where incorrect data causes real harm. For everything else, choosing a weaker model improves availability and performance.

## Caching strategies

*Caching* is a technique that stores frequently used data in faster storage locations to reduce response time and database load. When your application requests data, the cache acts as a quick-access storage layer between your application and the database.

A cache stores data in memory, reducing response times by 10x to 100x or more. It introduces complexity around data freshness and adds another potential point of failure. A good caching strategy defines how your application manages the relationship between cached data and the source of truth in your database.

### Common caching strategies

The three main caching strategies differ in when and how they update the cache relative to the database.

#### Cache-aside

In the *cache-aside* pattern, also called lazy loading, your application code manages both the cache and the database directly. The cache is populated on demand: data is only loaded when a request misses.

On a cache miss, the application queries the database, writes the result to the cache, and returns the data. On a cache hit, the application reads directly from the cache.

```go
func GetProduct(ctx context.Context, cache *redis.Client, db *sql.DB, id int64) (*Product, error) {
    key := fmt.Sprintf("product:%d", id)

    val, err := cache.Get(ctx, key).Result()
    if err == nil {
        var p Product
        if err := json.Unmarshal([]byte(val), &p); err != nil {
            return nil, err
        }
        return &p, nil
    }
    if !errors.Is(err, redis.Nil) {
        return nil, err
    }

    row := db.QueryRowContext(ctx, "SELECT id, name, price FROM products WHERE id = $1", id)
    var p Product
    if err := row.Scan(&p.ID, &p.Name, &p.Price); err != nil {
        return nil, err
    }

    data, err := json.Marshal(p)
    if err != nil {
        return nil, err
    }
    _ = cache.Set(ctx, key, data, 5*time.Minute).Err()

    return &p, nil
}
```

The error from `cache.Set` is intentionally discarded. A failed cache write is not fatal in this pattern because the application already fetched the data from the database and will re-fetch on the next cache miss.

Use cache-aside when:

- read-heavy workloads dominate
- data changes infrequently
- you can tolerate brief inconsistency between the cache and the database

#### Write-through

In the *write-through* pattern, every write operation updates both the cache and the database at the same time. The cache always reflects the latest state.

```go
func SaveProduct(ctx context.Context, cache *redis.Client, db *sql.DB, p *Product) error {
    _, err := db.ExecContext(ctx,
        "INSERT INTO products (name, price) VALUES ($1, $2) ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name, price = EXCLUDED.price",
        p.Name, p.Price,
    )
    if err != nil {
        return err
    }

    data, err := json.Marshal(p)
    if err != nil {
        return err
    }
    key := fmt.Sprintf("product:%d", p.ID)
    return cache.Set(ctx, key, data, 5*time.Minute).Err()
}
```

Use write-through when:

- you need strong consistency between the cache and the database
- data is read frequently after being written
- read performance matters more than write performance

#### Write-behind

In the *write-behind* pattern, also called write-back, the application writes data to the cache immediately and updates the database asynchronously. This improves write performance but introduces the risk of data loss if the cache fails before the database is updated.

```go
func SaveProductAsync(ctx context.Context, cache *redis.Client, queue chan<- *Product, p *Product) error {
    data, err := json.Marshal(p)
    if err != nil {
        return err
    }
    key := fmt.Sprintf("product:%d", p.ID)
    if err := cache.Set(ctx, key, data, 5*time.Minute).Err(); err != nil {
        return err
    }

    select {
    case queue <- p:
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}
```

The application writes to the cache and queues the database write for a background worker. If the process crashes before the worker flushes the queue, those writes are lost.

Use write-behind when:

- your application is write-heavy and needs maximum write throughput
- you can accept some risk of data loss in exchange for better performance
- eventual consistency between the cache and the database is acceptable

### When to use caching

Caching adds a moving part to your application that can fail, consume memory, and serve stale data. Before adding a cache, confirm that the complexity is justified. A cache makes sense when your database must handle a high volume of simultaneous queries and response times are no longer acceptable.

A *cache eviction policy* determines which items the cache removes when it reaches capacity. The most common policy is Least Recently Used (LRU), which removes the item that was accessed longest ago. Other policies include Least Frequently Used (LFU), which removes the item accessed least often, and Time-To-Live (TTL), which expires items after a fixed duration regardless of access patterns.

Ask these questions before adding a cache:

- How often does the data change?
- What is the cost of serving stale data?
- How expensive are your current queries?
- Are there current performance bottlenecks that a cache would address?

If data changes frequently, the cost of keeping it fresh may outweigh the performance benefit. If queries are cheap and response times are acceptable, a cache adds complexity without measurable gain.

### Caching and consistency

When you cache data, you create a temporary copy that may become stale when the original changes. Each caching strategy handles this differently.

- Cache-aside with *TTL* (time-to-live): the cache entry expires after a fixed duration, forcing a fresh database read on the next request. This limits how stale cached data can become without requiring explicit invalidation.
- Write-through gives you strong consistency between the cache and the database, but every write pays the cost of updating both.
- Write-behind accepts temporary inconsistency in favor of write performance. If the cache fails before the database is updated, those writes are lost.

## Planning for data growth

Planning for growth means regularly reviewing performance metrics and capacity needs. Watch for these key indicators before they become critical:

- Query response times exceeding acceptable thresholds
- Database CPU consistently above a certain percentage
- Memory usage climbing steadily over time
- An increase in user complaints about slow load times

These signals tell you that your current infrastructure is approaching its limits. Acting on them before they become outages gives you time to scale deliberately rather than reactively.

### Scaling strategies

*Vertical scaling* adds resources to your existing database server, such as increasing CPU cores or RAM. It is the simplest approach and requires no changes to your application code. Vertical scaling has a ceiling: at some point, adding more resources to a single server becomes too expensive or reaches the hardware limit.

*Horizontal scaling* distributes your data across multiple servers. Use it when vertical scaling reaches its limits or when you need geographic distribution for global applications.

Two common horizontal scaling techniques are read replicas and sharding.

*Read replicas* are copies of your primary database that serve read operations. They reduce load on the primary database by routing read queries to one or more replicas. The primary database handles all writes and replicates changes to the replicas asynchronously.

*Sharding* partitions your data across multiple databases based on a *shard key*. Each shard holds a subset of the data. For example, you might shard a user table by user ID range so that users 1–1,000,000 live on one database and users 1,000,001–2,000,000 live on another. Sharding scales both reads and writes but adds significant complexity to your application and queries.

#### Read replicas vs. caching

Both read replicas and caches reduce load on the primary database, but they work differently.

- Read replicas provide eventual consistency automatically. The database engine handles replication and your application code requires no changes.
- Caches require application code changes to populate, invalidate, and manage consistency. In exchange, they deliver lower latency than replicas because data is stored in memory closer to your application.

Use read replicas when you want to scale reads with minimal application changes. Use a cache when latency is the primary concern and you can manage the additional complexity.

### Maintaining performance during growth

As your data grows, queries that performed well at small scale can degrade. Build these practices into your regular operations to catch problems early.

*Data archiving* moves older, infrequently accessed records out of your primary tables into separate archive tables or cold storage. Smaller tables mean faster queries and more efficient indexes.

Database *indexes* speed up queries by allowing the database to find rows without scanning the entire table. Review your indexes regularly: add indexes for columns that appear frequently in WHERE clauses, JOIN conditions, or ORDER BY expressions, and remove unused indexes that slow down writes without benefiting reads.

Schedule regular database maintenance tasks including:

- analyzing query performance to identify slow queries
- reviewing execution plans for queries that have degraded over time
- running optimizer statistics updates so the query planner makes accurate decisions
- reclaiming space from deleted rows through vacuuming or reorganization, depending on your database engine

## Querying and managing data performance

A poorly optimized query can consume excessive server resources, create bottlenecks, and lead to a frustrating user experience. Writing efficient queries and understanding how the database processes them helps you avoid these problems before they reach production.

### Efficient query writing

Efficient queries request only the data they need. Selecting unnecessary columns wastes memory, increases network transfer, and prevents the database from using covering indexes.

### Basic query optimization

The first step in optimization is identifying what makes queries inefficient. Full-table scans, unindexed filters, and returning more data than needed are the most common causes.

Always specify the columns you need rather than using `SELECT *`:

```sql
-- Avoid: returns every column, including ones your application never uses
SELECT * FROM orders WHERE customer_id = 123;

-- Prefer: returns only what your application needs
SELECT id, total, created_at FROM orders WHERE customer_id = 123;
```

Specifying columns lets the database use *covering indexes*, where the index itself contains all the data the query needs, eliminating reads from the table.

Consider pagination for any query that may return a large result set. Returning thousands of rows at once strains both the database and the client. Fetching a fixed number of rows per request keeps response times predictable.

### Prepared statements

A *prepared statement* is a precompiled SQL query that lets you safely insert data values at runtime. The database parses and optimizes the query once, then reuses that execution plan for subsequent calls with different parameters. This improves performance for repeated queries and prevents SQL injection by handling parameter escaping automatically.

In Go, `db.PrepareContext` compiles the statement and `stmt.QueryContext` executes it with the provided values:

```go
func GetProductsByCategory(ctx context.Context, db *sql.DB, category string, maxPrice float64) ([]Product, error) {
    stmt, err := db.PrepareContext(ctx, `
        SELECT id, name, price
        FROM products
        WHERE category = $1 AND price < $2
        ORDER BY price
    `)
    if err != nil {
        return nil, err
    }
    defer stmt.Close()

    rows, err := stmt.QueryContext(ctx, category, maxPrice)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var products []Product
    for rows.Next() {
        var p Product
        if err := rows.Scan(&p.ID, &p.Name, &p.Price); err != nil {
            return nil, err
        }
        products = append(products, p)
    }
    return products, rows.Err()
}
```

The `$1` and `$2` placeholders are PostgreSQL positional parameters. Go's `database/sql` driver passes values separately from the query string, so the database treats them as data rather than SQL. This makes it structurally impossible to inject SQL through parameter values.

For frequently called queries, prepare statements once at application startup and store them as struct fields rather than preparing on each call.

### Index management

Database indexes solve slow queries by creating a lookup structure that maps column values directly to the rows where they are stored. Instead of scanning every row in a table, the database consults the index and jumps directly to the matching rows.

```sql
-- Index on a frequently queried column
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- Composite index for queries that filter on multiple columns
CREATE INDEX idx_orders_customer_status ON orders(customer_id, status);

-- Check index usage in PostgreSQL
SELECT indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
WHERE relname = 'orders';
```

Indexes are not free. They consume storage and slow down write operations because the database must update each index whenever data changes.

Index these columns for the most performance gain:

- columns used frequently in WHERE clauses
- columns used in JOIN conditions
- columns used in ORDER BY or GROUP BY clauses
- combinations of columns that appear together in filters (use a composite index)
- avoid indexing columns with low cardinality, such as a boolean field with only two possible values

### Handling large result sets

*Pagination* lets you navigate through query results in manageable chunks. Instead of fetching every matching row at once, you fetch a fixed number per request and let the caller advance through pages.

Offset-based pagination uses `LIMIT` to cap the number of rows returned and `OFFSET` to skip rows from previous pages:

```go
func GetProducts(ctx context.Context, db *sql.DB, page, pageSize int) ([]Product, error) {
    offset := (page - 1) * pageSize
    rows, err := db.QueryContext(ctx, `
        SELECT id, name, price
        FROM products
        ORDER BY id
        LIMIT $1 OFFSET $2
    `, pageSize, offset)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var products []Product
    for rows.Next() {
        var p Product
        if err := rows.Scan(&p.ID, &p.Name, &p.Price); err != nil {
            return nil, err
        }
        products = append(products, p)
    }
    return products, rows.Err()
}
```

Page 1 passes `offset = 0`, page 2 passes `offset = pageSize`, and so on. The `ORDER BY id` clause is required: without a consistent sort order, the database may return the same row on multiple pages or skip rows entirely.

Offset-based pagination degrades at high page numbers. To reach page 1000 with a page size of 20, the database scans and discards 19,980 rows before returning the 20 you need. For deep pagination, consider *keyset pagination* instead, which filters on the last seen value rather than a row offset.

### Tools and best practices

A *query planner* acts as a compiler for SQL. When you submit a query, the planner analyzes it and determines the most efficient way to retrieve the data. It considers available indexes, table statistics, row counts, and the cost of different execution strategies before choosing a plan.

PostgreSQL's query planner uses a *cost-based optimizer*. It assigns an estimated cost to each possible execution strategy, where cost is measured in units representing disk reads, CPU time, and memory usage. The planner chooses the strategy with the lowest estimated total cost.

The planner relies on table statistics collected by the `ANALYZE` command. These statistics include column cardinality, data distribution, and row counts. Stale statistics cause the planner to make poor decisions. Run `ANALYZE` after large data changes, or configure `autovacuum` to keep statistics current automatically.

### Using query execution plans

Most databases expose the query planner's chosen strategy through the `EXPLAIN` command. Running `EXPLAIN` before a query shows the execution plan the database will use without executing the query. Adding `ANALYZE` executes the query and shows actual row counts and timing alongside the estimates.

```sql
EXPLAIN ANALYZE
SELECT o.id, o.total, c.name
FROM orders o
JOIN customers c ON c.id = o.customer_id
WHERE o.created_at > '2024-01-01'
ORDER BY o.total DESC
LIMIT 20;
```

An execution plan typically shows:

- Scan type: whether the database used a sequential scan (full table), index scan, or index-only scan for each table
- Join strategy: how tables were joined (nested loop, hash join, or merge join)
- Estimated vs actual rows: how accurately the planner predicted row counts. Large discrepancies indicate stale statistics.
- Cost estimates: startup cost and total cost for each operation, in planner units
- Actual timing: with `EXPLAIN ANALYZE`, the real execution time for each step

### Database monitoring and analysis

Real-world performance depends on how your application interacts with the database under load. Understanding that behavior requires *observability*: the ability to understand the internal state of a system based on the outputs it produces.

Three concepts form the foundation of database observability:

- *Logging* captures discrete events as they happen. Database logs record slow queries, connection errors, lock conflicts, and failed authentication attempts. Configure your database to log queries that exceed a threshold (in PostgreSQL, set `log_min_duration_statement`) so you can identify and optimize the slowest operations.

- *Metrics* are numeric measurements collected over time. Key database metrics include query latency (p50, p95, p99), connection pool utilization, cache hit rate, replication lag, and disk I/O. Metrics let you spot trends before they become incidents. A steadily rising p99 latency is a warning sign even if average latency looks healthy.

- *Tracing* tracks a request as it moves through your system, from the HTTP handler to the service layer to the database query and back. A trace shows the full call chain and the time spent at each step, making it possible to identify which database query is responsible for a slow API response.

Real-world scenarios where observability provides actionable insights:

- A spike in p99 query latency without a corresponding spike in p50 latency points to a specific slow query rather than general load. Use logs to identify the query and `EXPLAIN ANALYZE` to diagnose it.
- Connection pool exhaustion shows up as a sudden increase in request latency and errors. Metrics on pool utilization reveal whether you need more connections or whether a slow query is holding connections longer than expected.
- A distributed trace reveals that a single API endpoint makes 50 database queries per request. The fix is to batch those queries or cache the results rather than optimizing each individual query.
- Replication lag metrics alert you when a read replica falls behind the primary, preventing your application from routing reads to a replica serving data too stale for the use case.

### Balancing complexity and performance

Query optimization sometimes produces code that is harder to read and maintain. The goal is not maximum performance at any cost. It is to meet your performance requirements with the simplest code that achieves them.

Consider fetching a user and their recent orders. A readable approach uses two queries:

```go
func getUserOrders(ctx context.Context, db *sql.DB, userID int64) (*UserOrders, error) {
    var u User
    err := db.QueryRowContext(ctx,
        "SELECT id, name, email FROM users WHERE id = $1", userID,
    ).Scan(&u.ID, &u.Name, &u.Email)
    if err != nil {
        return nil, err
    }

    rows, err := db.QueryContext(ctx, `
        SELECT id, total, created_at
        FROM orders
        WHERE user_id = $1
        ORDER BY created_at DESC
        LIMIT 10
    `, userID)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var orders []Order
    for rows.Next() {
        var o Order
        if err := rows.Scan(&o.ID, &o.Total, &o.CreatedAt); err != nil {
            return nil, err
        }
        orders = append(orders, o)
    }
    return &UserOrders{User: u, Orders: orders}, rows.Err()
}
```

This makes two round trips to the database. Combining the queries into a single JOIN reduces that to one, but produces code that is harder to read, test, and modify:

```sql
SELECT u.id, u.name, u.email, o.id, o.total, o.created_at
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE u.id = $1
ORDER BY o.created_at DESC
LIMIT 10;
```

Choose the simpler two-query version first. Switch to the JOIN only if profiling shows the extra round trip is a measurable bottleneck for real users.

## Data migration and transformation

Development teams regularly upgrade database systems, connect with external services, or combine data from multiple sources. Each scenario requires moving data from one place to another. Planning how data moves, transforms, and synchronizes is as important as the migration itself.

### Understanding data movement fundamentals

A successful migration requires careful planning, execution, and validation. Skipping any of these phases risks data loss, downtime, or inconsistencies that are costly to correct after the fact.

#### Big bang vs phased migration

A *big bang migration* moves all data at one time, typically during a scheduled downtime window. The approach is simpler to execute because the source system is frozen while the migration runs. The risk is proportional to data volume: the larger the dataset, the longer the downtime window and the more damage a failure can cause.

A *phased migration* moves data in stages. You migrate a segment, validate that it arrived correctly, then move the next segment. Phased migrations keep the source system online and limit the blast radius of any individual failure. The tradeoff is increased complexity: you must run both systems in parallel until the migration is complete.

Choose a phased migration for any dataset where extended downtime is unacceptable.

#### ETL processes

*ETL* (Extract, Transform, Load) is the standard pattern for moving data between systems.

- *Extraction* reads data from the source system. Extraction can trigger rate limits on external APIs or place performance load on the source database. Batch your reads and schedule extraction during off-peak hours when possible.
- *Transformation* reshapes the extracted data to match the target system's structure and constraints. Transformation exposes data quality issues: missing required fields, inconsistent formats, duplicate records, and values that violate the target schema. Build validation into this stage rather than discovering problems at load time.
- *Loading* writes the transformed data into the target system. Loading can trigger constraint violations, foreign key failures, and uniqueness conflicts. Process records in transactions so a failed batch can roll back cleanly.

ETL requires detailed error handling and logging. Each stage should record which records succeeded, which failed, and why. Without this, identifying and reprocessing failed records becomes a manual investigation.

#### Data synchronization

When you run source and target systems in parallel during a migration, you need a strategy to keep them in sync. Three approaches handle this well.

*Message queues* capture writes to the source system as events and replay them against the target. The queue acts as a buffer, decoupling the migration from the application's write path and allowing the target to catch up asynchronously.

*Change data capture* (CDC) reads the database's transaction log to stream changes in real time. CDC requires no changes to application code and captures every insert, update, and delete as it happens. Tools like Debezium work directly with PostgreSQL's logical replication.

*Reconciliation processes* periodically compare source and target data to find and correct discrepancies. Use reconciliation as a safety net alongside queues or CDC rather than as the primary sync mechanism.

### Handling schema changes

Table structures change as applications evolve. Fields get added, removed, renamed, or have their types modified. Without a controlled process, schema changes become a source of risk for every team member touching the database.

#### Version control for data structures

Without version control for your schema, you risk losing critical migration history, overwriting a colleague's changes, and losing the ability to roll back a problematic change.

Tools like Flyway, Liquibase, and Rails Migrations solve this by tracking which migrations have run on each environment. Organize your migrations as SQL files with sequential version numbers:

```
migrations/
  V001__create_users.sql
  V002__add_email_to_users.sql
  V003__create_orders.sql
```

Important practices:

- *Idempotent migrations*: an idempotent operation produces the same result whether it runs once or many times. Use `CREATE TABLE IF NOT EXISTS`, `ADD COLUMN IF NOT EXISTS`, and conditional inserts so that re-running a migration on an already-migrated database does not produce errors.
- Ensure migration scripts are backwards compatible. A deployed application should continue to function while the migration is running. Add columns as nullable before populating them, and retire old columns across multiple releases rather than dropping them immediately.
- Include the schema change and any necessary data transformations in the same migration script so they apply atomically.

#### Managing data dependencies and transformations

Schema changes often require transforming the data they affect. Version these transformations alongside the schema change that requires them. This ensures the schema change and data transformation apply together, the migration is atomic, and you can track which environments have received the change.

For complex transformations that affect large datasets, consider these approaches:

- *Background jobs*: run the transformation asynchronously after the schema change deploys. The application handles both old and new data formats during the transition period.
- *Dual-write*: write data to both the old and new structures simultaneously during the transition. Once all existing records are backfilled, retire the old structure.
- A rollback plan: before running any migration in production, define the steps to reverse it and test them. Know what the database state should look like if the migration fails partway through.
