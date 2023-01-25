# 3. Configuration and Error Handling

## 3.1. Managing Configuration Settings

- There should be separation of concerns (separation between configuration and code)

Environment variables vs Command line flags

- Environment variables have some limitations
- They don't have default values like flags
- They don't support automatic type conversion of flag values
- They don't have usage help information
- You get an empty string as a result if some environment variable doesn't exist
- The return value is always a string

We can use both of them together
- Pass the environment variables as the flag values to your application

Pre-existing variables
- We can also pass in flag values to pre-existing variables
- Particularly / Specially useful if we want to store flag values in a config struct

```
type config struct {
    addr string
    staticDir string
}

...

var cfg config

flag.StringVar(&cfg.addr, "addr", ":4000", "HTTP network address")
flag.StringVar(&cfg.staticDir, "static-dir", "./ui/static", "Path to static assets")

flag.Parse()
```


## 3.2. Levelled Logging

We can break our logs in two types / levels

1. Informational logs ("starting the server on :4000")
2. Error logs (log.Fatal(err))

We're gonna add leveled logging capability:

1. Prefix informational logs with "INFO" and output them to `stdout`
2. Prefix error logs with "ERROR" and output them to `stderr` along with file name and line number to help with debugging


Decoupled Logging

- Separation of concerns - application isn't concerned with routing or storage of logs
- makes logging easy, depending on environment (i.e. dev, test, prod)
- gives the benefit to redirect our log streams to a desired final destiantion like a logging service like splunk or maybe disk files
- `go run ./cmd/web >>/tmp/info.log 2>>/tmp/error.log`

The http.Server error log
- by default Go's http server logs using the standard logger
- we can use custom loggers by initializing a new `http.Server` containing the configuration settings for our server


Additional Information

- Rule of thumb, don't `panic` and `log.Fatal` outside `main`
- return errors from outside `main`
- only `panic` and exit directly from `main`
- Author's recommendation to log messages to standard streams and then redirect the logs to files at runtime
- But we can also open a file in Go and use it as log destination


## 3.3. Dependency Injection
How can we make any dependency available to our handlers?
Theres plenty ways to do this: https://www.alexedwards.net/blog/organising-database-access

- Easy solution is to just make the dependency global
- Good practice is to inject the dependency into handlers (makes code more explicit / less error prone / easier to unit test)

Closures for Dependency Injection / The Closure Pattern
- If handlers are spread across multiple packages, then we should use the Closure Pattern
- Create a config package which exports the Application struct
- And have your handlers close over this to form closure
- https://gist.github.com/alexedwards/5cd712192b4831058b21

```
func main() {
    app := &config.Application{
    ErrorLog: log.New(os.Stderr, "ERROR\t", log.Ldate|log.Ltime|log.Lshortfile)
}

    mux.Handle("/", examplePackage.ExampleHandler(app))
}

func ExampleHandler(app *config.Application) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ts, err := template.ParseFiles(files...)
        if err != nil {
            app.ErrorLog.Println(err.Error())
            http.Error(w, "Internal Server Error", 500)
            return
        }
...
    }
}
```

Centralized Error Handling

- Move some of the error handling code into helper methods

Isolating application routes
