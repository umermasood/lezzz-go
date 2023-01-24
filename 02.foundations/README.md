# 2. Foundations

## 2.2. Web Application Basics

There are three essentials of a web app:
1. Handlers
2. Router (ServeMux)
3. Web Server

## 2.3. Routing Requests

### 2.3.1 Fixed Path and Subtree Patterns
- Go's `servemux` supports two different types of URL patterns.
  - Fixed Paths
  - Subtree Paths
- Fixed Paths are only matched when the request URL path is an exact match
- Subtree Paths like the Root `/` URL pattern, match `/`, followed by anything

### 2.3.2 Restricting the Root URL Pattern
- If we don't want the Root `/` URL path to act like a catch-all(matches all paths)
- We can achieve that with a simple check
- check the request URL path, if it is not `/` then throw a 404 not found

### 2.3.3 The DefaultServeMux
- The `net/http` package has a global variable called `DefaultServeMux`
- This is just like a regular `servemux` that we create and use
- But it is provided by Go as default
- We have functions in the `net/http` package like `Handle()` and `HandleFunc()`
- These functions use the `DefaultServeMux` under the hood to register routes
- Using the `DefaultServeMux` cleans up the code a bit
- But it is not recommended for production because of security purposes

#### 2.3.3.1 `Servemux` features and quirks
- Longer URLs take precedence
  - You can register patterns in any order
- Requests URL paths are automatically sanitized
- Subtree path quirk
  - If you have registered a subtree path like `/foo/`
  - Then any request to `/foo` will be redirected to `/foo/`
#### 2.3.3.2 Host name matching
- It is possible to include hostnames in URL patterns
- This is useful when we want to redirect all HTTP requests to a URL
- Host specific patterns like below are checked first
- Non-host specific patterns are checked after the host specific patterns
```
mux := http.NewServeMux()
mux.HandleFunc("foo.example.org/", fooHandler)
mux.HandleFunc("bar.example.org/", barHandler)
mux.HandleFunc("/baz", bazHandler)
```

#### 2.3.3.3 `RESTful` Routing
There are some advantages and disadvantages of using Go's `servemux`

- Go's `servemux` is pretty lightweight
- It doesn't support routing based on request method
- Doesn't support automatic sanitization (clean URLs with variables in them)
- Doesn't support RegEx based patterns
---

## 2.4. Customizing HTTP Headers

- We can restrict our routes to act on specific HTTP request methods only
- We can use the `request.Method` type to find out the type of the request
- Then we can throw a status code with `w.WriteHeader(statusCode)`
  - We can only call this method once per a response
  - If not explicitly called, it is called upon first `w.Write()` call with the `200 OK`

- Changing response header map using `w.Header().Set("Allow", "POST")`
- Changing the response header map after the call to w.WriteHeader() will have no effect on the headers received by user
- So this should be called before `WriteHeader()`

### 2.4.1 Additional Information
- Go automatically sets 3 system generated headers for you
- Content type is usually automatically detected by content sniffing the response body
- Gotcha: Cannot distinguish JSON from plain text
- We have methods like `Set(), Add(), Del(), Get(), Values()` for manipulating response headers in the response header map
- When we use the methods mentioned above, the header's name will always be canonicalized (case-insensitive)
- We can also avoid this canonicalization behavior by manipulating the header map directly
- We cannot `Del()` the system-generated headers, but they can be suppressed by modifying the underlying map
  - Just set the `w.Header()["Date"] = nil`

## 2.5. URL Query Strings
- We can access query string params from the URL
- Use `r.URL.Query().Get("id")` where `id` is the query param

## 2.6. Project Structure and Organization
```
snippetbox/
├── cmd
│   └── web
│       ├── handlers.go
│       └── main.go
├── go.mod
├── internal
├── README.md
└── ui
    ├── html
    └── static

6 directories, 4 files
```

Peter Bourgon's popular project structure https://peter.bourgon.org/go-best-practices-2016/#repository-structure

## 2.7. HTML Templating & Inheritance
- Handler functions can parse and render the HTML templates
- We can use the `html/template` package which provides functions related to templating
- We can use the `template.ParseFiles`

### 2.7.1 Composing Template
- There is some shared boilerplate that we may want to include on every page
- To avoid redundancy / duplication, we use templates

### 2.7.2 Embedding Partials
- Partials are certain bits of HTML which you take out and reuse in different pages and layouts
- An example is a navbar, which is reused across all pages of the web app

### 2.7.3 Additional Information
>**The Block Action**
> 
> `{{block}}...{{end}}`
> The `block` action allows you to specify some default content if the template being invoked doesn't exist in the
> template set. This default content can be overridden by individual pages as per need
> If we don't include any content in a block action, then the template acts like it's an optional template

### 2.7.4 Embedding Files
- The `embed` package allows you to embed files in your Go programs rather than reading them from the disk


## 2.8. Serving Static Files
- Go comes with a built-in `http.FileServer()` which allows you to serve your static files
- We can use `http.StripPrefix()` to strip a certain prefix from the URL path before sending the request to the handler

### 2.8.1. Additional Information
- Go's file server sanitizes request paths before searching for a static file
- This removes any `.` and `..` elements from the path which prevents directory traversal attacks
- Range requests are fully supported (resume-able downloads supported)
- File modification headers are transparently supported and throw a status code like 304 if a file isn't modified
after last request
- Prevents latency and processing overhead for both client and servers
- To serve a single file, we can use `http.ServeFile()`, but there's a caveat, it doesn't sanitize request URL path
- So clean it using `filepath.Clean()`
- We can also disable directory listings by adding a blank `index.html` in the directory that was being listed
- But adding this blank `index.html` is not a good solution
- A better solution is to create a custom implementation of the `http.FileSystem` and have it return `os.ErrNotExist`
for any directories
- Methods for disabling directory listings:
  - i. index.html method
  - ii. middleware method (checks for trailing `/` in request URL & throws 404)
  - iii. custom fileSystem

## 2.9. The http.Handler interface
- Any object which defines a method with the following signature `ServeHTTP(http.ResponseWriter, *http.Request)`
implements the `http.Handler` interface and is considered an http handler

### 2.9.1. Chaining Handlers
- Chaining handlers together is a very common idiom in Go
- `Servemux` is a special kind of handler which instead of handling the response itself passes the request to the second
handler
- This is what happens when the server receives a new request:
  - i. Server receives a new HTTP request
  - ii. It calls `servemux`'s `ServeHTTP` method
  - iii. This looks up the relevant handler based on the request URL's path and calls that handler's ServeHTTP method
  
Think of Go web application as a chain of ServeHTTP() methods chained together being called one after another

### 2.9.2. Requests are handled concurrently
- All incoming HTTP requests are handled in their own goroutine
