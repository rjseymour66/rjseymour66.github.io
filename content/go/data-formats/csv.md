+++
title = 'CSV'
date = '2025-10-11T17:07:58-04:00'
weight = 10
draft = false
+++

CSV (comma-separated values) is a file format for easily reading tabular data such as numbers and text. It is defined in the [RFC 4180 specification](https://datatracker.ietf.org/doc/html/rfc4180), but not all CSV files follow the spec. Some might delimit data with pipes, tabs, or semicolons.

{{< admonition "CSV values are text" note >}}
CSV is stored in a text file, so all values are assumed to be text. If you want to use the value as another type, you have to cast it as the desired type.
{{< /admonition >}}

Go provides tools to work with CSV in its [encoding/csv](https://pkg.go.dev/encoding/csv) package.

## Read entire file as 2D slice

`ReadAll` reads all records from the given Reader and stores its values in a 2D array of strings, where each array is a row:

1. Open the CSV file. `Open` returns a file handle, which is a Reader.
2. Create a new Reader that reads from the given Reader.
3. `ReadAll` reads all remaining records from the Reader caller.
4. Do some work with the records. Here, we print each row in the 2D array on its own line.

```go
func main() {
	file, err := os.Open("email.csv")                   // 1
	if err != nil {
		log.Println("Cannot open CSV file: ", err)
	}
	defer file.Close()

	csvReader := csv.NewReader(file)                    // 2
	rows, err := csvReader.ReadAll()                    // 3
	if err != nil {
		log.Println("Cannot read CSV file: ", err)
	}

	for _, row := range rows {                          // 4
		fmt.Printf("%q\n", row)
	}
}
```

## Read one row at a time

For large CSV files, it might be easier to read one row at a time rather than all at once. The `Read` method reads one record (a slice of fields) from the reader. You have to call it in an infinite for loop with a break condition to read all records:

1. Open the CSV file. `Open` returns a file handle, which is a Reader.
2. Create a new Reader that reads from the given Reader.
3. Start an infinite loop to read each record individually.
4. `Read` reads the record.
5. Check for an `EOF`. If you reach the end of the file, break out of the infinite loop and continue executing `main`.
6. Print each value in the record with a `for...range` loop.
7. Optional pattern to print each record with its index. 


```go
func main() {
	file, err := os.Open("email.csv")                           // 1
	if err != nil {
		log.Println("Cannot open CSV file: ", err)
	}
	defer file.Close()

	csvReader := csv.NewReader(file)                            // 2
	for {                                                       // 3
		record, err := csvReader.Read()                         // 4
		if err == io.EOF {                                      // 5
			break
		}
		if err != nil {
			log.Println("Cannot read CSV file: ", err)
		}
		for _, val := range record {                            // 6
			fmt.Println(val)
		}
        // for i, _ := range record {                           // 7
		// 	fmt.Printf("%s\n", record[i])
		// }
	}
}
```

## Marshal into structs

Instead of dealing with the 2D slice of strings that `ReadAll` returns, you can marshal the CSV fields into structs. This example uses the following CSV file:

```csv
email,id,first_name,last_name
rachel@yourcompany.com,9012,Rachel,Booker
laura@yourcompany.com,2070,Laura,Grey
craig@yourcompany.com,4081,Craig,Johnson
mary@yourcompany.com,9346,Mary,Jenkins
jamie@yourcompany.com,5079,Jamie,Smith
```

First, create a struct that models the CSV file:

```go
type User struct {
	Email     string
	Id        int
	FirstName string
	LastName  string
}
```

`marshalCSVToUsers` takes a Reader and returns a slice of `User` structs: 
1. Create a new CSV Reader that reads data from the given Reader.
2. `ReadAll` reads all remaining records from the Reader.
3. Create a slice of `User` structs.
4. Iterate through the 2D rows returned from the reader with a `for...range` loop.
5. In the CSV file, the `Id` is the second field and its of type `int`. Convert the value to an `int` with `Atoi`. You have to do this separately because `Atoi` returns both an `int` and an `error`.
6. The remaining fields are of type `string`, so assign each struct field to its corresponding row field with the row index. Assign the converted `id` variable to the `Id` field.
7. Append the new `user` struct to the slice.
8. Return the slice of `User` structs.

```go
func marshalCSVToUsers(r io.Reader) ([]User, error) {
	csvReader := csv.NewReader(r)                       // 1
	rows, err := csvReader.ReadAll()                    // 2
	if err != nil {
		log.Println("Cannot read CSV file: ", err)
	}

	var users []User                                    // 3
	for _, row := range rows {                          // 4
		id, _ := strconv.Atoi(row[1])                   // 5
		user := User{                                   // 6
			Email:     row[0],
			Id:        id,
			FirstName: row[2],
			LastName:  row[3],
		}
		users = append(users, user)
	}
	return users, nil
}
```

Finally, call the function in `main`:
1. Open the CSV file. `Open` returns a file handle, which is a Reader.
2. Pass the `file` Reader to `marshalCSVToUsers` to get a slice of `User` structs that contain the data from `file`.
3. Do some work with the `users` slice.

```go
func main() {
	file, err := os.Open("email.csv")
	if err != nil {
		log.Println("Cannot open CSV file: ", err)
	}
	defer file.Close()

	users, _ := marshalCSVToUsers(file)

	for _, user := range users {
		fmt.Println(user)
	}
}
```

## Remove the column headers

CSV files usually start with a header row that contains the column labels. You can remove this by calling `Read` to discard the first line, then reading the remainder of the file with `ReadAll`:
1. Open the CSV file. `Open` returns a file handle, which is a Reader.
2. Create a new Reader that reads from the given Reader.
3. Call `Read` to read the first row that contains the column headers. You don't assign the return value to anything, so it is discarded.
4. `ReadAll` reads all remaining records from the `csvReader`.
5. Do some work with the records. Here, we print each row in the 2D array on its own line.

```go
func main() {
	file, err := os.Open("email.csv")                   // 1
	if err != nil {
		log.Println("Cannot open CSV file: ", err)
	}
	defer file.Close()

	csvReader := csv.NewReader(file)                    // 2
	csvReader.Read()                                    // 3
	rows, err := csvReader.ReadAll()                    // 4
	if err != nil {
		log.Println("Cannot read CSV file: ", err)
	}

	for _, row := range rows {                          // 5
		fmt.Printf("%q\n", row)
	}
}
```

## Custom delimiters

When you create a [CSV Reader](https://pkg.go.dev/encoding/csv#Reader), it uses comma delimiters by default. To change the delimiter, set the `Comma` variable in the Reader instance:
1. Create the CSV Reader.
2. Set the Reader's `Comma` variable. Because `Comma` is a `rune`, you need to set it with a single quote.

```go
func main() {
	file, err := os.Open("semicolon.csv")
	if err != nil {
		log.Println("Cannot open CSV file: ", err)
	}
	defer file.Close()

	csvReader := csv.NewReader(file)                // 1
	csvReader.Comma = ';'                           // 2

	rows, err := csvReader.ReadAll()
	if err != nil {
		log.Println("Cannot read CSV file: ", err)
	}

    // do something with rows
}
```

## Ignore rows

In many text formats, there is a special character that indicates the line is a comment and should be ignored. For example, if you want to ignore a line of text in a bash script, you place a `#` symbol at the start of the line. CSV files do not have a standardized "comment" character, but the CSV Reader type has a `Comment` variable you can set to ignore lines that begin with the given character.

The process is similar to the [custom delimiter](#custom-delimiters) example, where you set the variable with a rune:
1. Create the CSV Reader.
2. Set the Reader's `Comment` variable. Because `Comment` is a `rune`, you need to set it with a single quote.

```go
func main() {
	file, err := os.Open("semicolon.csv")
	if err != nil {
		log.Println("Cannot open CSV file: ", err)
	}
	defer file.Close()

	csvReader := csv.NewReader(file)                    // 1
	csvReader.Comma = ';'
	csvReader.Comment = '#'                             // 2

	rows, err := csvReader.ReadAll()
	if err != nil {
		log.Println("Cannot read CSV file: ", err)
	}

	// do something with rows
}
```

## Writing CSV data

### WriteAll CSV data

Write a complete CSV file with the `WriteAll` function. The data that you write must be a 2D array:
1. Create a file that you want to write data to. `Create` returns a file handle, which is a Writer.
2. Create a 2D array that models the CSV data that you want to write to the file.
3. Create a new CSV Writer, and pass it the `file` Writer. The CSV Writer is a wrapper around the `file` Writer. The `file` writes directly to the output stream, and the CSV Writer handles CSV-specific tasks, such as escaping fields, quoting, etc.
4. `WriteAll` writes all data in the 2D array to the underlying Writer. It returns only an `error`.

```go
func main() {
	file, err := os.Create("users.csv")                         // 1
	if err != nil {
		log.Println("Cannot create CSV file: ", err)
	}
	defer file.Close()

	data := [][]string{                                         // 2
		{"email", "id", "first_name", "last_name"},
		{"rachel@yourcompany.com", "9012", "Rachel,Booker"},
		{"laura@yourcompany.com", "2070", "Laura,Grey"},
		{"craig@yourcompany.com", "4081", "Craig,Johnson"},
		{"mary@yourcompany.com", "9346", "Mary,Jenkins"},
		{"jamie@yourcompany.com", "5079", "Jamie,Smith"},
	}

	csvWriter := csv.NewWriter(file)                            // 3
	err = csvWriter.WriteAll(data)                              // 4
	if err != nil {
		log.Println("Cannot write to CSV file", err)
	}
}
```

### Write one row at a time

If you want more control over the writing operation, you can write one row at a time to the file with the `Write` function. This requires that you use a `for...range` loop to iterate over the data in the 2D array and call `Write` during each iteration:
1. Create a file that you want to write data to. `Create` returns a file handle, which is a Writer.
2. Create a 2D array that models the CSV data that you want to write to the file.
3. Create a new CSV Writer, and pass it the `file` Writer. The CSV Writer is a wrapper around the `file` Writer. The `file` writes directly to the output stream, and the CSV Writer handles CSV-specific tasks, such as escaping fields, quoting, etc.
4. Create a `for...range` loop that ranges over the 2D `data` slice.
5. Each iteration visits a `row` in the 2D slice. During each iteration, call `Write` to write the `row` to the underlying `file` Writer.
6. After the `for...range` loop exits, call `Flush` to flush the underlying `file` writer and write any remaining data to the output stream.

```go
func main() {
	file, err := os.Create("users2.csv")                        // 1
	if err != nil {
		log.Println("Cannot create CSV file: ", err)
	}
	defer file.Close()

	data := [][]string{                                         // 2
		{"email", "id", "first_name", "last_name"},
		{"rachel@yourcompany.com", "9012", "Rachel,Booker"},
		{"laura@yourcompany.com", "2070", "Laura,Grey"},
		{"craig@yourcompany.com", "4081", "Craig,Johnson"},
		{"mary@yourcompany.com", "9346", "Mary,Jenkins"},
		{"jamie@yourcompany.com", "5079", "Jamie,Smith"},
	}

	csvWriter := csv.NewWriter(file)                            // 3
	for _, row := range data {                                  // 4
		err = csvWriter.Write(row)                              // 5
		if err != nil {
			log.Println("Cannot write to CSV file: ", err)
		}
	}
	csvWriter.Flush()                                           // 6
}
```