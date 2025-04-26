# Go Best Practices, Six Years In

## Introduction
In 2014, I gave a talk at the inaugural GopherCon titled *Best Practices in Production Environments*. Since then, Go has evolved significantly. In March 2016, I had the opportunity to revisit these best practices at QCon London, reviewing what still holds and what’s changed. Below is the essence of that talk, with key takeaways highlighted as **Top Tips**. These are practical, actionable guidelines to improve your Go programming.

✪ **Top Tip** — Use Top Tips to level up your Go game.

---

## Development Environment
Go has conventions centered around `GOPATH`. In 2014, I strongly advocated for a single, global `GOPATH`. While I still believe this is the best default approach, I’ve softened my stance. For teams or projects producing primarily binaries, a per-project `GOPATH` can make sense depending on your workflow.

✪ **Top Tip** — Use a single, global `GOPATH` unless your project or team has specific needs that justify otherwise.

---

## Testing
My views on testing have also evolved. I no longer dismiss testing DSLs or frameworks outright. If a team finds value in a specific testing package, they should use it—but only for clear, well-defined reasons. For designing testable code, I recommend Mitchell Hashimoto’s talk on the subject ([SpeakerDeck](https://speakerdeck.com/mitchellh/advanced-testing-with-go), [YouTube](https://www.youtube.com/watch?v=8hQG7Wa9A7E)). Writing Go in a functional style, with explicit dependencies and small, tightly-scoped interfaces, naturally makes code easier to test.

✪ **Top Tip** — Use many small interfaces to model dependencies for easier testing.

---

## Building
A key lesson, courtesy of Dave Cheney: prefer `go install` over `go build`. The `install` command caches build artifacts in `$GOPATH/pkg`, speeding up builds, and places binaries in `$GOPATH/bin`, making them easier to locate. For projects producing binaries, tools like `gb` can simplify the build process. Since Go 1.5, cross-compilation is built-in—just set `GOOS` and `GOARCH` environment variables, eliminating the need for extra tools.

✪ **Top Tip** — Use `go install` for faster builds and better binary management.

For deployment, Go binaries are relatively easy to deploy compared to languages like Ruby, Python, or Java. If using containers, follow Kelsey Hightower’s advice and build Docker images `FROM scratch` for minimal, secure deployments.

---

## Project Structure
Organize library code under a `pkg/` subdirectory and binaries under a `cmd/` subdirectory. Always use fully-qualified import paths (e.g., `github.com/peterbourgon/foo/pkg/fs`) rather than relative imports. This structure ensures compatibility with the broader Go ecosystem and keeps your code consumable.

✪ **Top Tip** — Put library code under a `pkg/` subdirectory. Put binaries under a `cmd/` subdirectory.

✪ **Top Tip** — Always use fully-qualified import paths. Never use relative imports.

Follow the [Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments) as the minimum standard for code reviews. For naming disputes, refer to Andrew Gerrand’s [idiomatic naming conventions](https://talks.golang.org/2014/names.slide).

---

## Top Tip #9: Make Dependencies Explicit
Implicit dependencies make code fragile, hard to understand, and difficult to test. For example, consider middleware that pulls dependencies from a global state:

```go
func MyMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        db := globalDB // Implicit dependency
        // Use db
        next.ServeHTTP(w, r)
    })
}
```

This is problematic because users of `MyMiddleware` must read its implementation to understand its dependencies. Instead, make dependencies explicit:

```go
type MyMiddleware struct {
    DB *sql.DB
}

func (m *MyMiddleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Use m.DB
}
```

Explicit dependencies are type-safe, compiler-checkable, and easier to test. This principle applies broadly—avoid global state and magic behavior.

✪ **Top Tip** — Make dependencies explicit to improve code clarity and testability.

---

## Context and Dependencies
When using the `context` package, prefer storing values (e.g., strings, structs) rather than references (e.g., pointers, handles). For example, avoid storing a logger in the context, though a request-scoped logger might be an exception in some cases. Explicit dependencies align with this principle, as discussed in [my blog post on context](https://peter.bourgon.org/blog/2016/07/11/context.html).

---

## Conclusion
These best practices reflect lessons learned from years of Go programming in production environments. They emphasize simplicity, explicitness, and alignment with Go’s idioms. While some recommendations have evolved, the core principles—clarity, testability, and ecosystem compatibility—remain timeless.

For more details, watch the full talk on [YouTube](https://www.youtube.com/watch?v=8hQG7Wa9A7E) or explore my other talks and workshops at [peter.bourgon.org](https://peter.bourgon.org).
