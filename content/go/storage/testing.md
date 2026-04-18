+++
title = 'Testing'
date = '2026-03-22T22:33:25-04:00'
weight = 70
draft = false
+++


## Create a test database

This tests that you are correctly connecting to the database. It creates a unique database for each test:
1. Designate the test function as a helper.
2. `mode=memory` creates a unique, in-memory database named after the database (`tb.Name()`) that is shared (`cache=shared`) by all database connections in the test.
3. `Cleanup` runs after the test finishes to clean up the test db resources. This is a special method for testing. Don't use `defer`---it would close the connection when the function completes, which is too soon for database operations.

```go
func DialTestDB(tb testing.TB) *sql.DB {
	tb.Helper()

	dsn := fmt.Sprintf("file:%s?mode=memory&cache=shared", tb.Name())
	db, err := Dial(tb.Context(), dsn)
	if err != nil {
		tb.Fatalf("DialTestDB: %v", err)
	}
	tb.Cleanup(func() {
		if err := db.Close(); err != nil {
			tb.Logf("DialTestDB: closing: %v", err)
		}
	})

	return db
}
```

## Integration testing

This tests whether the `Shotener` function properly calls the `link` package to shorten and persist a link:
1. Create a new shortener service with the [test database](#create-a-test-database).
2. Use the testing context. This is canceled when the test finishes.

```go
func TestShortenerShorten(t *testing.T) {
	t.Parallel()

	lnk := link.Link{
		Key: "foo",
		URL: "https://new.link",
	}

	shortener := NewShortener(DialTestDB(t))

	// Shortens a link
	key, err := shortener.Shorten(t.Context(), lnk)
	if err != nil {
		t.Fatalf("got err = %v, want nil", err)
	}
	if key != "foo" {
		t.Errorf(`got key %q, want "foo"`, key)
	}

	// Disallows shortening a link with a duplicate key
	_, err = shortener.Shorten(t.Context(), lnk)
	if !errors.Is(err, link.ErrConflict) {
		t.Fatalf("\ngot err = %v\nwant ErrConflict for duplicate key", err)
	}
}
```
