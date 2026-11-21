# Debugging a 400 on a valid request

## Symptom

The HTTP action calling the monday.com API returned:

```
400 Bad Request
{"errors":[{"message":"query must be provided"}]}
```

The request body contained a valid query that worked elsewhere.

## Approach: change one variable at a time

| Step | Test | Result | Conclusion |
|---|---|---|---|
| 1 | Replace the real query with `{ me { name } }` | Still 400 | The query is not the problem |
| 2 | Run the same query and token in the API playground | 200 OK | The token and permissions are fine |
| 3 | Inspect the raw request the flow sends | Header value ends in a newline | The request itself is malformed |

## Root cause

The API token had been stored with a trailing newline. When it was placed in the `Authorization` header, the newline ended the header early, which broke the HTTP framing. The server parsed the request without the body and reported a missing query.

## Fix and prevention

- Trim the token before building the header.
- Add a minimal `{ me { name } }` health-check action at the start of the flow, which fails fast with a clear message if authentication or framing breaks again.
- Documented the symptom and fix in the runbook.

## Takeaway

An error message describes what the server saw, not necessarily what you sent. Shrinking the problem until only one variable is left found the cause faster than editing the query.
