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

### 2.3.3.1 `Servemux` features and quirks
- Longer URLs take precedence
  - You can register patterns in any order
- Requests URL paths are automatically sanitized
- Subtree path quirk
  - If you have registered a subtree path like `/foo/`
  - Then any request to `/foo` will be redirected to `/foo/`
### 2.3.3.2 Host name matching
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

### 2.3.3.3 `RESTful` Routing
There are some advantages and disadvantages of using Go's `servemux`

- Go's `servemux` is pretty lightweight
- It doesn't support routing based on request method
- Doesn't support clean URLs with variables in them
- Doesn't support RegEx based patterns
---