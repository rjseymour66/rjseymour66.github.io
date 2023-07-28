---
title: "Security"
weight: 170
description: >
  Security and Go.
---

## Generate a TLS certificate 

A self-signed certificate has the same functionality as a normal TLS cert, but it is not cryptographically signed by a trusted certificate authority. Your browser will receive a warning, but it encrypts traffic.

Go's [crypto/tls](https://pkg.go.dev/crypto/tls) package provides the `generate_cert.go` tool so you can quickly create a self-signed cert.

1. Create a `tls` directory in your project and change into it:
   ```shell
   $ mkdir tls
   $ cd tls
   ```
2. Run the `generate_cert.go` tool, using the `/tls` directory in the Go source code:
   ```shell
   $ go run /usr/local/go/src/crypto/tls/generate_cert.go --rsa-bits=2048 --host=localhost
   2023/05/26 23:01:57 wrote cert.pem
   2023/05/26 23:01:57 wrote key.pem
   ```
  - `cert.pem` is a self-signed TLS certificate for the host `localhost`.
  - `key.pem` is the server private key.

3. Make sure that both the `cert.pem` and `key.pem` have read permissions or you receive an error when you try to server HTTPS with them:
   ```bash
   $ ls -l
   total 8
   -rw-rw-r-- 1 user group 1090 May 26 23:01 cert.pem
   -rw------- 1 user group 1704 May 26 23:01 key.pem
   ```
1. (Optional) Add the keys to your `.gitignore` file:
   ```shell
   $ echo 'tls/' >> .gitignore 
   ```
  
## TLS server

To run an HTTPS server, se the `ListenAndServeTLS` function with a certificate and the private server key:

```go
func main() {

  // If you use the alexedwards scs session package
  sessionManager.Cookie.Secure = true

  infoLog.Printf("Starting server on %s", *addr)
  // Use the ListenAndServeTLS() method to start the HTTPS server. We
  // pass in the paths to the TLS certificate and corresponding private key as
  // the two parameters.
  err = srv.ListenAndServeTLS("./tls/cert.pem", "./tls/key.pem")
  errorLog.Fatal(err)

}
```

## TLS custom configuration

To include custom TLS configurations, create a `tls.Config{}` object and pass it to the server:

```go
func main() {

  ...

	tlsConfig := &tls.Config{
		CurvePreferences: []tls.CurveID{tls.X25519, tls.CurveP256},
	}

	srv := &http.Server{
    ...
		TLSConfig: tlsConfig,
	}

	infoLog.Printf("Starting server on %s", srv.Addr)
	err = srv.ListenAndServeTLS("./tls/cert.pem", "./tls/key.pem")
	errorLog.Fatal(err)
}
```

### Restricting TLS versions

You can add a minimum and maximum required TLS version to the `tls.Config` object:

```go
tlsConfig := &tls.Config{
    MinVersion: tls.VersionTLS12,
    MaxVersion: tls.VersionTLS12,
}
```

## Email validation regex

[Follow this link!](https://html.spec.whatwg.org/multipage/input.html#valid-e-mail-address)

## bcrypt

You must encrypt sensitive data, and Go's [crypto](https://pkg.go.dev/golang.org/x/crypto) package provides encryption tools.

`bcrypt` is helpful for passwords because it has helper functions for hashing and checking passwords. You have to download the latest version to your project:

```shell
$ go get golang.org/x/crypto/bcrypt@latest
```

## CSRF

Cross-Site Request Forgery is when you are logged into an application, then navigate to another site that sends a malicious, state-changing request with your session cookie. [CRSF Demystified](https://www.gnucitizen.org/blog/csrf-demystified/) provides a detailed explanation.

You can set the SameSite cookie attribute, but that is not supported by 100% of browsers. The answer is to pass a _CSRF token_.

A CSRF token is generated and sent in the CSRF cookie. You have to hide a field in each form that contains the token, and then add middleware that verifies that the token and the value in the cookie match.

First, add a 3rd-party package:
```shell
$ go get github.com/justinas/nosurf@v1
```

Next, create the middleware. This function sets the cookie in the request:

```go
func noSurf(next http.Handler) http.Handler {
    csrfHandler := nosurf.New(next)
    csrfHandler.SetBaseCookie(http.Cookie{
        HttpOnly: true,
        Path:     "/",
        Secure:   true,
    })

    return csrfHandler
}
```

In your routes file, add the middleware to every route except the static file server:

```go
func (app *application) routes() http.Handler {
    router := httprouter.New()
    // ...
    // Use the nosurf middleware on all our 'dynamic' routes.
    dynamic := alice.New(app.sessionManager.LoadAndSave, noSurf)

    // routes

    // Because the 'protected' middleware chain appends to the 'dynamic' chain
    // the noSurf middleware will also be used on the three routes below too.
    protected := dynamic.Append(app.requireAuthentication)

    // routes cont't

    return standard.Then(router)
}
```

Next, make the CSRF token available to each template in the `templateData` struct:

```go
type templateData struct {
    CurrentYear     int
    ...
    CSRFToken       string // Add a CSRFToken field.
}
```
Inject the data into each template with the `newTemplateData` helper:

```go
func (app *application) newTemplateData(r *http.Request) *templateData {
    return &templateData{
        CurrentYear:     time.Now().Year(),
        Flash:           app.sessionManager.PopString(r.Context(), "flash"),
        IsAuthenticated: app.isAuthenticated(r),
        CSRFToken:       nosurf.Token(r), // Add the CSRF token.
    }
}
```

Finally, add the hidden `input` field to each form:

```html
<form>
  <input type='hidden' name='csrf_token' value='{{.CSRFToken}}'>
  <!-- form fields -->
</form>
```