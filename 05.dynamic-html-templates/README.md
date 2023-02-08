# 5. Dynamic HTML Templates

## Displaying Dynamic Data
- We can pass in dynamic data to our template set in the `ts.ExecuteTemplate` call

### Additional Information
- `html/template` is context-aware while escaping
- It knows whether data is rendered in HTML, CSS, JS or URI part of the page

#### Nested Templates
- While invoking one template from another, explicitly pass the .
- We can also call methods for types which yield data between {{}}
- We can also pass arguments to those methods

```
<span>{{.Snippet.Created.Weekday}}</span>

<span>{{.Snippet.Created.AddDate 0 6 0}}</span>
```

This syntax of passing arguments to AddDate method is different than Go's syntax

#### HTML comments
- `html/template` strips out HTML comments from templates, including conditional templates
- helps avoid XSS

---

## Template actions and functions
