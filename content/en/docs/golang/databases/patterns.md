---
title: "Patterns"
weight: 10
description: >
  Design patterns for Go and database models.
---

### Data access layer

You want to encapsulate the code that works with MySQL in its own package. Create a new file in `root/internal/models/project.go` for your SQL data model.

The data access layer consists of the following:
- An interface that holds method signatures for the data object. This facilitates testing with mocks:
  ```go
  type SnippetModelInterface interface {
	  Insert(title string, content string, expires int)   (int, error)
	  Get(id int) (*Example, error)
	  Latest() ([]*Example, error)
  }
  ```
- A struct that holds data for each individual object that you want to commit to the database. Generally, the struct fields should correspond to the table columns:
  ```go
  type Example struct {
    ID      int
    Title   string
    Content string
    Created time.Time
    Expires time.Time
  }
  ```
- An `ExampleModel` struct that wraps an `sql.DB` connection pool. This is the type that you use as the receiver on the interface methods:
  ```go
  type ExampleModel struct {
     DB *sql.DB
  }
  ```
- Interface method implementations (`Insert`, `Get`, etc.)

After you create the data access layer, you have to import it into the `main` function and inject it as a dependency into your main application struct:

```go
type application struct {
	...
    dataAccessObj   models.ExampleModelInterface
}

func main() {
	...
    db, err := openDB(*dsn)
    if err != nil {
        errorLog.Fatal(err)
    }
    defer db.Close()


    app := &application{
        errorLog:      errorLog,
        infoLog:       infoLog,
        dataAccessObj: &models.ExampleModel{DB: db},
    }
	...
}
```

## Repository pattern

The repository pattern creates a data access layer--an interface between your Go code and the database. It consists of the following structs and functions:
- Custom type that wraps a private database handle. It might include a `sync.RWMutex` to secure read and write operations.
- A constructor function that initiates and returns a database connection.
- Public methods that access the database (CRUD).

### Create the repository

Implement the [respository pattern](#repository-pattern):

1. Create the custom type:
   ```go
   type mysqlRepo struct {
	   db *sql.DB
	   sync.RWMutex
   }
   ```
2. To simplify configuration, create a configuration object with the `mysql` `Config` type. You can add properties to the `Config` type, and then use its `.FormatDSN()` function to return its data source name (DSN):
   ```go
   cfg := mysql.Config{
	   User:   os.Getenv("DBUSER"),
	   Passwd: os.Getenv("DBPASS"),
	   Net:    "tcp",
	   Addr:   "127.0.0.1:3306",
	   DBName: "database-name",
   }
   ```
3. Create a constructor that accepts the `mysql.Config` as an argument:
   ```go
   // NewMySQLRepo returns a MySQL database handle.
   func NewMySQLRepo(cfg mysql.Config) (*mysqlRepo, error) {

	   // Get the db handle
	   var err error
	   db, err := sql.Open("mysql", cfg.FormatDSN())
	   if err != nil {
		   log.Fatal(err)
		   return nil, err
	   }

	   db.SetConnMaxLifetime(time.Minute * 3)
	   db.SetMaxOpenConns(10)
	   db.SetMaxIdleConns(10)

	   // call Ping to confirm the connection
	   pingErr := db.Ping()
	   if pingErr != nil {
		   log.Fatal(pingErr)
		   return nil, pingErr
	   }

	   fmt.Println("Connected!")

	   return &mysqlRepo{
		   db: db,
	   }, nil
   }
   ```
### Create data

Create (add) data to a database with the `Exec()` function. It returns a `Result` type whose interface defines the following functions:
- `LastInsertId() (int64, error)`: verify that you added a row.
- `RowsAffected() (int64, error)`: verify which rows were updated. 

Generally, a `Create*` function should return the id of the newly created row, and an `Update*` function returns an error if no rows were affected.

The following function creates a new `Album` type and adds it to a database called "albums":

```go
func (r *mysqlRepo) CreateAlbum(alb Album) (int64, error) {
	r.Lock()
	defer r.Unlock()

	result, err := r.db.Exec("INSERT INTO album (title, artist, price) VALUES (?, ?, ?)", alb.Title, alb.Artist, alb.Price)
	if err != nil {
		return 0, fmt.Errorf("addAlbum: %v", err)
	}
	id, err := result.LastInsertId()
	if err != nil {
		return 0, fmt.Errorf("addAlbum: %v", err)
	}
	return id, nil
}
```
In the previous example, the `?` characters are _placeholders_. Placeholders prevent SQL injections.


### Retrieve data

Query the database with the database handle in the repository struct. In its most basic form, a query that retrieves data must perform the following:
- Lock the repository with a mutex.
- Execute a `Query("statement")` that returns one or more `Rows` (defer `rows.Close()`. In the statement, use a `?` character to represent values, and pass the value after the statement parameter. This protects agains SQL injection attacks. 
- Loop through the returned rows with the `.Scan(columns...)` function. You must pass as parameters pointers to each column returned in rows. 
- Check if `Query(statement)` returned an error.

#### Query multiple rows

The following function queries a database called "albums" and returns all albums by the specified artist:

```go
type Album struct {
	ID     int64
	Title  string
	Artist string
	Price  float32
}

func (r *mysqlRepo) RetrieveAlbumsByArtist(name string) ([]Album, error) {
	r.Lock()
	defer r.Unlock()

	// albums slice to hold data from returned rows
	var albums []Album

	rows, err := r.db.Query("SELECT * FROM album WHERE artist = ?", name)
	if err != nil {
		return nil, fmt.Errorf("RetrieveAlbumsByArtist %q: %v", name, err)
	}
	defer rows.Close()

	// Loop through rows, using Scan to assign column data to struct fields.
	for rows.Next() {
		var alb Album
		// pass a pointer to each colum in Album
		if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
			return nil, fmt.Errorf("RetrieveAlbumsByArtist %q: %v", name, err)
		}
		albums = append(albums, alb)
	}

	// Check if Query returned any errors
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
	}
	return albums, nil
}
```
#### Query a single row 

A query for a single row checks errors to determine whether or not the row returned a value or another type of error:

```go
func (r *mysqlRepo) RetrieveAlbumByID(id int64) (Album, error) {
	r.Lock()
	defer r.Unlock()

	// An album to hold data from the returned row.
	var alb Album


	row := r.db.QueryRow("SELECT * FROM album WHERE id = ?", id)
	if err := row.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
		// return if there were no matching rows
		if err == sql.ErrNoRows {
			return alb, fmt.Errorf("albumsById %d: no such album", id)
		}
		// return error if there was another error
		return alb, fmt.Errorf("albumsById %d: %v", id, err)
	}
	return alb, nil
}
```

## Updates (full and partial)

Full updates to a database record generally follow these steps:
1. Get the query param ID from the request URL to identify the record.
2. Fetch the record from the database with the ID.
3. Unmarshal the JSON request body into memory (a Go input struct).
4. Copy the data fromt the Go input struct into a DB record struct, either with assignment (`record.x = input.x`) or type validation (`if x == nil {...}`).
5. Validate the new DB record.
6. Call the `Update()` method to update the record in the DB.
7. Write the JSON response to the client.

### Partial updates

Partial updates use the `PATCH` HTTP method to update just a portion of the database record. Partial updates require that you distinguish between a client request that sends a zero value or no value for a field. JSON is usually unmarshalled into fields of type `string`. The Go string type zero value is an empty string (`""`), which makes this difficult to determine the client's intent.

You can change the `string` fields to pointers, which makes the zero value `nil`. When you copy the data from the Go input struct to the database record struct (step 4, above), you can check whether the input is `nil` (not provided) or blank (`""`):

```go
// 1. get param
// 2. fetch record with param
// 3. create input struct and marshal req into it
// 4. ...
if input.Field != nil {
	dbStruct.Field = input.Field
}
// ... for additional fields
// 5. validation
// 6. Update()
// 7. JSON resp to client
```

## Concurrent updates

A Go server handles each request in its own goroutine. This means that there are situations where two clients request to update the same record at the same time. This is a race condition known as a _data race_.

Manage data races with [optimistic or pessimistic locking](https://stackoverflow.com/questions/129329/optimistic-vs-pessimistic-locking/129397#129397). Optimistic locking means that the `Update()` method verifies the database record version number before the update:

```sql
UPDATE dbrecord
SET row1 = $1, row2 = $2, ..., version = version + 1
WHERE id = $N AND version = $N+1
RETURNING version
```
If this query returns no rows, then the record was deleted or edited before the change was committed to the database, and no update is created.

## Query Timeouts 

