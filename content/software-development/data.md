+++
title = 'Data'
date = '2026-05-14T23:01:18-04:00'
weight = 70
draft = false
+++

Software applications are, at their core, data machines. You model applications around data: collecting it, storing it, transforming it, and returning it to users in a meaningful form. Getting that right requires more than writing queries. You need to understand different data types, select appropriate storage solutions, design efficient data models, optimize queries, ensure data integrity, and manage data as it evolves over time.

## Data types and formats

When choosing how to represent data, consider these factors:

<dl>
<dt>Audience</dt>
<dd>Who reads or consumes the data. A public API serving mobile clients has different requirements than an internal analytics pipeline. Format choices should match what consumers can parse and what they expect.</dd>

<dt>Performance requirements</dt>
<dd>How fast data must be read, written, or transferred. Binary formats like Protocol Buffers are compact and fast to parse. JSON is human-readable but larger and slower. High-throughput systems often trade readability for performance.</dd>

<dt>Compatibility</dt>
<dd>Whether the format works across the systems, languages, and versions that interact with it. JSON and CSV are nearly universal. Proprietary binary formats may require specific libraries or versions to decode.</dd>

<dt>Complexity</dt>
<dd>How much structure the data has and how deeply nested it is. Simple flat records fit CSV. Hierarchical or relational data fits JSON or XML. Highly complex relationships may require a schema language like Protocol Buffers or Avro.</dd>

<dt>Validation needs</dt>
<dd>How strictly data must conform to a defined shape. Formats like Avro and Protocol Buffers enforce schemas at the serialization layer. JSON Schema validates at the application layer. CSV has no built-in validation.</dd>
</dl>

### Structured vs. unstructured data

These are the two fundamental categories of data.

#### Structured data

*Structured data* follows a predefined model or schema with consistent field types and relationships. That structure is typically represented in code through classes or structs that define the shape of the data.

Key characteristics:

- **Consistent format.** Every record conforms to the same schema. A customer record always has an ID, a name, and an email address. That predictability makes data easy to validate, compare, and process in bulk.
- **Well-defined relationships.** Records reference each other through foreign keys or embedded references. An order belongs to a customer; a line item belongs to an order. Those relationships are declared in the schema and enforced by the database.
- **Easy to query.** Because the shape of every record is known in advance, you can filter, sort, join, and aggregate with SQL. The database uses the schema to plan efficient queries — you describe what you want and it does the work.
- **Efficient storage.** Fixed schemas let databases store data in compact, optimized layouts. Column-oriented databases compress repeated values aggressively; row-oriented databases calculate record offsets directly from the schema.

The trade-off is rigidity. Adding or changing a field typically requires modifying both the database schema and the application code that reads and writes it. In a system with multiple services or clients, a schema change must be coordinated across all of them.

Examples: customer records in a PostgreSQL table, a CSV export of financial transactions, a JSON response from a REST API.

#### Unstructured data

*Unstructured data* has no predefined data model. It includes freeform text such as emails, social media posts, and chat transcriptions; images, audio, and video; logs with inconsistent formats; and documents that have no clearly defined structure.

Key characteristics:

- **Variable format.** No two records are guaranteed to look the same. An email has a subject, a body, and headers, but none of those fields have enforced types or lengths. A log line from one service may look nothing like a log line from another. That variability makes bulk processing harder because your code must account for every possible shape.
- **Rich content.** Unstructured data often carries more meaning per record than structured data does. A customer support transcript contains sentiment, intent, and product feedback that a structured satisfaction score cannot capture. Extracting that meaning requires additional tools: natural language processing, computer vision, or audio transcription.
- **Difficult to query traditionally.** SQL relies on known schemas to filter and join records. Unstructured data has no schema to query against. Finding all emails that mention a specific product requires full-text search or machine learning, not a WHERE clause. Specialized tools like Elasticsearch or vector databases are built for this kind of retrieval.
- **Storage challenges.** Unstructured data is often large. A single video file dwarfs thousands of customer records. Relational databases are poorly suited to storing large binary objects efficiently. Most systems store unstructured data in object storage (such as S3) and keep only a reference to it in the database.

Examples: email bodies, product reviews, scanned PDFs, photographs, log files.

Most systems handle both. A customer record (structured) may include a free-text notes field (unstructured). A media platform stores metadata in a relational database (structured) and the media files in object storage (unstructured).

## Common data formats

### JSON

*JSON* (JavaScript Object Notation) is the de facto standard for data exchange in modern applications. Its readability and simplicity make it a good choice for web APIs, configuration files, and document databases. Limitations include no support for comments, no standard date format (dates are typically serialized as ISO 8601 strings), and no native binary data support without encoding.

```json
{
  "id": "ord-8821",
  "customer": {
    "id": "cust-441",
    "email": "alice@example.com"
  },
  "items": [
    { "sku": "WIDGET-01", "quantity": 2, "price": 14.99 },
    { "sku": "GADGET-07", "quantity": 1, "price": 49.99 }
  ],
  "total": 79.97,
  "createdAt": "2026-05-14T18:30:00Z"
}
```

### XML

*XML* (eXtensible Markup Language) is more verbose than JSON but offers robust validation through XML Schema Definition (XSD) and Document Type Definition (DTD). XPath enables precise data selection, and XSLT enables structured transformations. XML supports namespaces, which prevents naming conflicts when combining data from multiple sources. It remains prevalent in enterprise systems, document formats like DOCX and SVG, and configuration files for complex applications. Use XML when schema validation, namespaces, or compatibility with document-oriented systems is important.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<order id="ord-8821">
  <customer id="cust-441">
    <email>alice@example.com</email>
  </customer>
  <items>
    <item sku="WIDGET-01" quantity="2" price="14.99"/>
    <item sku="GADGET-07" quantity="1" price="49.99"/>
  </items>
  <total>79.97</total>
  <createdAt>2026-05-14T18:30:00Z</createdAt>
</order>
```

### CSV

*CSV* (comma-separated values) stores tabular data as plain text, using commas to separate values and newlines to separate records. The first row typically contains column headers. CSV works well for data exports, simple data exchange, and compatibility with spreadsheet applications.

Always account for edge cases: empty fields, fields that contain commas or quotes, and newlines within field values. Not all CSV parsers implement the full RFC 4180 specification consistently, so test against the parsers your consumers actually use.

```
id,customer_id,sku,quantity,price
ord-8821,cust-441,WIDGET-01,2,14.99
ord-8821,cust-441,GADGET-07,1,49.99
```

### YAML

*YAML* (YAML Ain't Markup Language) is more human-friendly than JSON or XML. It supports comments, references between nodes, and multiline text. These features make it well suited to configuration files, especially in DevOps tooling like Docker Compose, Kubernetes manifests, and CI/CD pipelines.

YAML requires consistent indentation. A single misplaced space can change the meaning of a document. Validate YAML files before deployment and use a YAML-aware editor to catch formatting issues before they reach production.

```yaml
# docker-compose.yml
services:
  api:
    image: myapp:latest
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgres://db:5432/myapp
      LOG_LEVEL: info
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
```

### Binary data

*Binary data* cannot be read or manipulated directly without proper encoding and decoding. Follow these practices when working with it:

- Use dedicated libraries designed for specific file formats rather than parsing bytes manually.
- Use Base64 encoding when binary data must be included in a text-based format like JSON or XML.
- Use hexadecimal representation to examine individual bytes during debugging and analysis.
- Manage memory carefully with large binary files. Loading an entire file into memory is often not viable.
- Always implement error handling to catch corrupt or incomplete data.

```go
data, err := os.ReadFile("image.png")
if err != nil {
    return err
}
encoded := base64.StdEncoding.EncodeToString(data)

payload := map[string]string{
    "filename": "image.png",
    "content":  encoded,
}
```

### Date and time

Date and time handling is a frequent source of bugs. Follow these practices:

- Store dates internally in UTC. Convert to local time zones only for display.
- Use ISO 8601 format (`2026-05-14T18:30:00Z`) when exchanging dates between systems.
- Use modern date and time libraries rather than building custom parsing logic.
- Be explicit about time zones in user interfaces to avoid ambiguity.

```
Stored internally:  2026-05-14T18:30:00Z
Displayed to user:  May 14, 2026 at 2:30 PM EDT
```

### Large datasets

Datasets too large to process in memory require a different approach:

- Use streaming to process data incrementally rather than loading it all at once.
- Implement pagination for API responses and user interfaces so consumers receive manageable chunks.
- Apply database optimizations — indexing, query tuning, and execution plan analysis — before reaching for more infrastructure.
- Evaluate specialized technology like data warehouses or distributed processing frameworks for extremely large datasets.
- Implement proper concurrency controls using locks, atomic operations, or immutable data structures to prevent data races.

Test with realistic data volumes early in development. A query that performs well against a hundred rows can fail or time out against a million. Problems found early are cheap to fix.

```go
rows, err := db.QueryContext(ctx, "SELECT id, payload FROM events ORDER BY id")
if err != nil {
    return err
}
defer rows.Close()

for rows.Next() {
    var id int
    var payload []byte
    if err := rows.Scan(&id, &payload); err != nil {
        return err
    }
    process(id, payload)
}
return rows.Err()
```
