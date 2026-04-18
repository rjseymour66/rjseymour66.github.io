+++
title = 'Maps'
date = '2025-11-02T09:56:45-05:00'
weight = 40
draft = false
+++


A map is an unordered collection of key/value pairs. Because maps are unordered, there’s no way to predict the order in which the key/value pairs will be returned. 

Internally, a map is a pointer to the `runtime.hmap` structure. This hash table contains a collection of buckets. When you’re storing, removing, or looking up a key/value pair, everything starts with selecting a bucket. This is performed by passing the key to the map’s hash function. The hash function generates an index that evenly distributes key/value pairs across all available buckets.

The strength of a map is its ability to retrieve data quickly based on the key. They do not have a capacity or a restriction on growth.

## Keys

A map key can be a value from any built-in or struct type as long as the value can be used in an expression with the == operator. You cannot use:
- slices
- functions
- struct types that contain slices


## Creating a map

There are multiple options to create a map:
1. nil map. If you declare a nil map with the `var` keyword, then you have to use `make` to initialize it.
2. Rather than create a nil map, you can declare and initialize it in one expression with `make`.
3. The idiomatic way to create a map is to initialize it with values.

```go
func main() {
	var people map[string]int           // 1
	people = make(map[string]int)       // 1 map[]

	initMap := make(map[string]int)     // 2
	initMap["Sally"] = 22               // map[Sally:22]

	colors := map[string]string{        // 3 map[Orange:#e95a22 Red:#da1337]
		"Red":    "#da1337",
		"Orange": "#e95a22",
	}
}
```

## Accessing maps

Access values in a map with the bracket notation:
1. Bracket notation uses the format `<map-name>["<value>"]`.
2. If the value does not exist, it returns nothing. 
3. Check if a map contains a key with the "comma-ok" idiom. This expression returns a value (`color`) and a boolean (`ok`). The boolean is `true` if the key exists in the map.
4. If the boolean is `true`, do some work with the returned value

```go
func main() {
	colors := map[string]string{
		"Red":    "#da1337",
		"Orange": "#e95a22",
		"White":  "#ffffff",
		"Black":  "#000000",
	}
    fmt.Println(colors["Orange"])               // 1 #e95a22
    fmt.Println(colors["Blue"])                 // 2

	color, ok := colors["Red"]                  // 3
	if ok {                                     // 4
		fmt.Println("Red value:", color)        // Red value: #da1337
	}
}
```

Here is the compact version:

```go
if val, ok := mapname["blue"]; ok {
    // ...
}
```

Some Go code uses the word `exists` or `found` in place of `ok`:

```go
value, exists := colors["Blue"]

if exists {
    fmt.Println(value)
}
```

### for...range

Loop through a map with the `for...range` loop:
1. You can access the keys and values the same way you access the index and value in a slice.
2. Omit the values to return only the keys.
3. You can't omit the keys and return only the values. Instead, you have to create a slice, ignore the key with an underscore (`_`), the append the values to the slice.

```go
func main() {
	colors := map[string]string{
		"Red":    "#da1337",
		"Orange": "#e95a22",
		"White":  "#ffffff",
		"Black":  "#000000",
	}

	for k, v := range colors {      // 1
		fmt.Println(k, v)
	}

	for k := range colors {         // 2
		fmt.Println(k)
	}

	var vals []string               // 3
	for _, v := range colors {
		vals = append(vals, v)
	}
	fmt.Println(vals)
}
```

## Modifying elements

To modify a value, reassign the existing value with bracket notation. Here, we override the `"Red"` value with `"not ascii"`:

```go
func main() {
	colors := map[string]string{
		"Red":    "#da1337",
		"Orange": "#e95a22",
		"White":  "#ffffff",
		"Black":  "#000000",
	}

	colors["Red"] = "not ascii"
}
```


## Deleting elements

Use the `delete` method to remove a key from the map:
1. `delete` accepts a map and the key you want to remove from the map.
```go
func main() {
	colors := map[string]string{
		"Red":    "#da1337",
		"Orange": "#e95a22",
		"White":  "#ffffff",
		"Black":  "#000000",
	}

	for k, v := range colors {              // Red: #da1337, Orange: #e95a22, White: #ffffff, Black: #000000,
		fmt.Printf("%s: %s, ", k, v)
	}

	delete(colors, "White")                 // 1
	
	for k, v := range colors {              // Orange: #e95a22, Black: #000000, Red: #da1337
		fmt.Printf("%s: %s, ", k, v)
	}
}
```

## Sorting maps

To sort a map, you have to put the keys into a slice, sort the slice, then access the map using the slice:
1. Create a slice for the keys.
2. Loop over the `keys` slice, and append each key to the slice.
3. Sort the slice to put `keys` in order.
4. Range over the `keys` slice, printing the keys in order. You can access the keys in the map with bracket notation.

```go
func main() {
	people := map[string]int{
		"Ricky": 34,
		"Sally": 52,
		"Cal":   9,
		"Betty": 23,
	}

	var keys []string                       // 1
	for k := range people {                 // 2
		keys = append(keys, k)
	}

	sort.Strings(keys)                      // 3

	for _, key := range keys {              // 4
		fmt.Println(key, people[key])
	}
}
```

## Passing to functions

Functions do not make copies of the map. Any changes made to the map by the function are reflected by all references to the map:
1. `removeColor` removes keys from the specified map. Pass the map as `<name> map[type]type`
2. Create a map of colors and color hex codes.
3. Call the function to remove the specified key.
4. Loop over the map to display all the colors in the map.
   
```go
func removeColor(colors map[string]string, key string) {
    delete(colors, key)
}

func main() {
    colors := map[string]string{
       "AliceBlue":   "#f0f8ff",
       "Coral":       "#ff7F50",
       "DarkGray":    "#a9a9a9",
       "ForestGreen": "#228b22",
    }

    removeColor(colors, "Coral")

    for key, value := range colors {
        fmt.Printf("Key: %s  Value: %s\n", key, value)
    }
}


```