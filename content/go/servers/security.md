+++
title = 'Security'
date = '2025-09-16T21:58:33-04:00'
weight = 30
draft = false
+++


## Basic Auth

Basic authentication uses a username and password in the request's `Authentication` header. The following example is a commenting app that lets you create and retrieve comments for a user. It stores the credentials in memory and a `login` function to validate the username and password. All credentials are stored in memory:

1. In-memory map that holds the `username` and `password`.
2. `login` function looks up the `username` in the `validUsers` map with Go's two-value assignment syntax. If the `password` stored as the key for `username` is the provided password, it returns `true`. Otherwise, `false`.
3. `postComments` uses `r.BasicAuth()` to get the contents of the `Authentication` header. It returns the `username`, `password`, and a Boolean that indicates whenther the header is present, correctly formatted, and decodable. Here, we store that in the `auth` variable.
4. If `auth` is `false` or the login failed, then return a 403 error on failure.


```go
type comment struct {
	username   string
	text       string
	dateString string
}

var comments []comment

var validUsers = map[string]string{                             // 1
	"bill": "abc123",
}

func login(username, password string) bool {                    // 2
	if validPassword, ok := validUsers[username]; ok {
		return validPassword == password
	}
	return false
}

func postComments(w http.ResponseWriter, r *http.Request) {
	username, password, auth := r.BasicAuth()                   // 3

	if !auth || !login(username, password) {                    // 4
		w.WriteHeader(http.StatusUnauthorized)
		return
	}

	commentText, err := io.ReadAll(r.Body)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		return
	}

	comments = append(comments, comment{
		username:   username,
		text:       string(commentText),
		dateString: time.Now().Format(time.RFC3339),
	})
	w.WriteHeader(http.StatusOK)
}

func getComments(w http.ResponseWriter, r *http.Request) {
	commentBody := ""
	for i := range comments {
		commentBody += fmt.Sprintf("%s (%s) - @%s\n",
			comments[i].text, comments[i].dateString, comments[i].username)
	}
	fmt.Fprintln(w, fmt.Sprintf("Comments: \n%s", commentBody))
}

func main() {
	http.HandleFunc("GET /comments", getComments)
	http.HandleFunc("POST /comments", postComments)

	if err := http.ListenAndServe(":8000", nil); err != nil {
		panic(err)
	}
}
```

### cURL examples

To test the preceding app, use cURL. This example adds a comment:

```bash
curl -X POST -u bill:abc123 http://localhost:8000/comments \
     -d "This is my first comment"
```

This request fails because there are no user credentials:

```bash
curl -X POST http://localhost:8000/comments -d "No credentials"
```

This request retrieve all comments:

```bash
curl http://localhost:8000/comments
```


## JSON Web Tokens

A JSON Web Token (JWT) is a data format that carries confidential information that identifies a user and their metadata. They are more secure than cookies and incur less network traffic and resource allocation for databases and session stores.

For detailed information, including a list of JWT libraries for each language, go to [jwt.io](https://www.jwt.io/introduction#what-is-json-web-token).

### Composition

A JWT consists of a header, payload, and signature:
- Header: Defines the type of token and algorithm used to sign it.
- Payload: Contains the token claims. A claim is a piece of information thats packaged inside a token, either user data or metadata. There are registered, public, and private claims. Registered claims are standardized by [RFC7519](https://datatracker.ietf.org/doc/html/rfc7519).
- Signature: A hash of header.payload created with a secret key.

An encoded JWT uses the following format: `header.payload.signature`. For example:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkFsaWNlIiwiYWRtaW4iOnRydWUsImV4cCI6MTcxMDAwMDAwMH0.
h1sRrK3T9T64aY0o9fGlfi0OvlrKXrXjZzUMgC8M6sE
```

### Claim types

There are registered, public, and private claims:

| Claim Type     | Claim Key | Description                                           | Example Value                  |
| -------------- | --------- | ----------------------------------------------------- | ------------------------------ |
| **Registered** | `iss`     | Issuer – who issued the token                         | `"https://myapp.com"`          |
|                | `sub`     | Subject – who the token is about (usually user ID)    | `"1234567890"`                 |
|                | `aud`     | Audience – who the token is intended for              | `"my-service"`                 |
|                | `exp`     | Expiration time (epoch seconds)                       | `1710000000`                   |
|                | `nbf`     | Not before – earliest time token is valid             | `1709000000`                   |
|                | `iat`     | Issued at – when the token was issued                 | `1708996400`                   |
|                | `jti`     | JWT ID – unique identifier for the token              | `"abc123-xyz789"`              |
| **Public**     | `name`    | User’s full name                                      | `"Alice Johnson"`              |
|                | `email`   | User’s email address                                  | `"alice@example.com"`          |
|                | `picture` | URL to user’s profile picture                         | `"https://example.com/me.png"` |
|                | `role`    | User’s role in the system                             | `"admin"`                      |
| **Private**    | Any key   | Custom claims shared only between issuer and consumer | `"tier": "premium"`            |
|                |           |                                                       | `"department": "finance"`      |


### PEM keys

JWTs uses PEM keys to authenticate user data:
- Private: Server uses this key to sign the JWT.
- Public: Clients or other servers use the public key to verify the JWT.

Here are the commands to generate your PEM keys and inspect the files:

1. Generate the private key:
   ```bash
   openssl genpkey -algorithm RSA -out private.pem -pkeyopt rsa_keygen_bits:2048
   ```
2. Extract the public key from the private key:
   ```bash
   openssl rsa -in private.pem -pubout -out public.pem
   ```
3. Inspect the private key:
   ```bash
   openssl rsa -in private.pem -text -noout
   ```
4. Inspect the public key:
   ```bash
   openssl rsa -in public.pem -pubin -text -noout
   ```

### Basic example

The following examples use [jwt-go](https://github.com/golang-jwt/jwt):
1. For this example, we use a plain byte slice as the signing key. The logic uses an ES256 key, so you would need to create those keys in production.
2.  `RegisteredClaims` includes standard fields like `exp`, `iat`, `nbf`, `sub`.
3.  Here, you create a claim that expires in an hour and identifies the token user with email.
4.  Create the token by signing the claims with an algorithm. Here, we use the ES256 algorithm. This sets the algorithm in the JWT header and attaches the claims to the payload.
5.  Sign the token with the private key, and return the base64-encoded signed string.
6.  Generate the JWT token. Assign the variables in an `if` clause to limit their scope. The `else` clause only runs if `err == nil`, so the `signed` variable exists only in the `else` condition branch.
    {{< admonition "Idiomatic Go" note >}}
	Idiomatic Go would write this code sequentially rather than in an `if/else` statement.
	{{< /admonition >}}
7.  Parse the JWT string back into a token object. You use the `SIGNING_KEY` for verification.
8.  Extract the claims from the token object. Here, you store the claims in a `map[string]any` and log the `sub` (Subject) claim if the token is valid.

```go
var SIGNING_KEY = []byte("secret-pem-key-value") 							// 1

type claim struct { 														// 2
	jwt.RegisteredClaims
}

func generateClaim() (string, error) { 										// 3
	claims := claim{
		jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(time.Now().Add(time.Hour)),
			IssuedAt:  jwt.NewNumericDate(time.Now()),
			NotBefore: jwt.NewNumericDate(time.Now()),
			Subject:   "nobody@example.com",
		},
	}

	token := jwt.NewWithClaims(jwt.SigningMethodES256, claims) 				// 4
	ss, err := token.SignedString(SIGNING_KEY) 								// 5
	if err != nil {
		return "", err
	}
	return ss, nil
}

func main() {
	if signed, err := generateClaim(); err != nil { 						// 6
		panic(err)
	} else {
		token, err := jwt.Parse(signed, func(t *jwt.Token) (any, error) { 	// 7
			return SIGNING_KEY, nil
		})
		if err != nil {
			panic(err)
		}
		if validatedClaims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid { 	// 8
			log.Println(validatedClaims["sub"])
		} else {
			panic("error getting claims")
		}
	}
}
```

### Set in a cookie

This example stores the JWT in a cookie:

```go
package main

import (
	"fmt"
	"net/http"
	"time"

	"github.com/golang-jwt/jwt/v5"
)

var jwtKey = []byte("my_secret_key")

// Create claims structure
type Claims struct {
	Username string `json:"username"`
	jwt.RegisteredClaims
}

func loginHandler(w http.ResponseWriter, r *http.Request) {
	username := r.FormValue("username")

	// Create JWT claims
	claims := &Claims{
		Username: username,
		RegisteredClaims: jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(time.Now().Add(15 * time.Minute)),
			IssuedAt:  jwt.NewNumericDate(time.Now()),
		},
	}

	// Create token
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	tokenString, err := token.SignedString(jwtKey)
	if err != nil {
		http.Error(w, "could not create token", http.StatusInternalServerError)
		return
	}

	// Set JWT inside a cookie
	http.SetCookie(w, &http.Cookie{
		Name:     "token",
		Value:    tokenString,
		HttpOnly: true,
		Secure:   false, // set true in production (HTTPS)
		Path:     "/",
	})
	fmt.Fprintln(w, "Logged in, JWT set in cookie")
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
	// Read cookie
	cookie, err := r.Cookie("token")
	if err != nil {
		http.Error(w, "no token", http.StatusUnauthorized)
		return
	}

	// Parse JWT
	tokenStr := cookie.Value
	claims := &Claims{}

	token, err := jwt.ParseWithClaims(tokenStr, claims, func(t *jwt.Token) (interface{}, error) {
		return jwtKey, nil
	})

	if err != nil || !token.Valid {
		http.Error(w, "invalid token", http.StatusUnauthorized)
		return
	}

	// Token is valid → access granted
	fmt.Fprintf(w, "Welcome %s!", claims.Username)
}

func main() {
	http.HandleFunc("/login", loginHandler)
	http.HandleFunc("/home", homeHandler)

	fmt.Println("Server running on http://localhost:8080")
	http.ListenAndServe(":8080", nil)
}

```