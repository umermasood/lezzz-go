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

---