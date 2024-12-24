# Listing 7.8: Integrating the HIT client

## [hit](https://github.com/inancgumus/gobyexample/blob/5d61f13859e70d49966194e76b17ed89cedc8917/hit) / [cmd](https://github.com/inancgumus/gobyexample/blob/5d61f13859e70d49966194e76b17ed89cedc8917/hit/cmd) / [hit](https://github.com/inancgumus/gobyexample/blob/5d61f13859e70d49966194e76b17ed89cedc8917/hit/cmd/hit) / [hit.go](https://github.com/inancgumus/gobyexample/blob/5d61f13859e70d49966194e76b17ed89cedc8917/hit/cmd/hit/hit.go)

> [!TIP]
> Each listing corresponds to a commit.
>
> Click the links above to see the file and its directory in their original locations and state as they were at the time of the commit.

```go
package main

import (
	"fmt"
	"io"
	"math"
	"net/http"
	"os"
	"time"

	"github.com/inancgumus/gobyexample/hit"
)

const logo = `
 __  __     __     ______
/\ \_\ \   /\ \   /\__  _\
\ \  __ \  \ \ \  \/_/\ \/
 \ \_\ \_\  \ \_\    \ \_\
  \/_/\/_/   \/_/     \/_/`

func main() {
	e := &env{
		stdout: os.Stdout,
		stderr: os.Stderr,
		args:   os.Args,
	}
	if err := run(e); err != nil {
		os.Exit(1)
	}
}

func run(e *env) error {
	c := config{
		n: 100, // default request count
		c: 1,   // default concurrency level
	}
	if err := parseArgs(&c, e.args[1:], e.stderr); err != nil {
		return err
	}
	fmt.Fprintf(
		e.stdout,
		"%s\n\nSending %d requests to %q (concurrency: %d)\n",
		logo, c.n, c.url, c.c,
	)
	if e.dryRun {
		return nil
	}

	if err := runHit(e.stdout, &c); err != nil {
		fmt.Fprintf(e.stderr, "\nerror occurred: %v\n", err)
		return err
	}

	return nil
}

func runHit(stdout io.Writer, c *config) error {
	req, err := http.NewRequest(http.MethodGet, c.url, http.NoBody)
	if err != nil {
		return fmt.Errorf("new request: %w", err)
	}
	results, err := hit.SendN(c.n, req, &hit.Options{
		Concurrency: c.c,
		RPS:         c.rps,
	})
	if err != nil {
		return fmt.Errorf("send n requests: %w", err)
	}
	printSummary(stdout, results.Summarize())

	return nil
}

func printSummary(stdout io.Writer, s hit.Summary) {
	var success float64
	if s.Requests > 0 {
		success = (float64(s.Requests-s.Errors) / float64(s.Requests)) * 100
	}

	fmt.Fprintf(stdout, `
Summary:
    Success:  %.0f%%
    RPS:      %.1f
    Requests: %d
    Errors:   %d
    Bytes:    %d
    Duration: %s
    Fastest:  %s
    Slowest:  %s
`,
		success,
		math.Round(s.RPS),
		s.Requests,
		s.Errors,
		s.Bytes,
		s.Duration.Round(time.Millisecond),
		s.Fastest.Round(time.Millisecond),
		s.Slowest.Round(time.Millisecond),
	)
}
```

## What's changed?

> [!TIP]
> The following diff shows the changes made to the file since the last commit.
> The lines starting with `+` show the new lines added, and the lines starting with `-` show the lines removed.
> The lines starting with `@@` show the line and column numbers of the changes.

```diff
@@ -3,9 +3,10 @@ package main
 import (
 	"fmt"
 	"io"
 	"math"
+	"net/http"
 	"os"
 	"time"
 
 	"github.com/inancgumus/gobyexample/hit"
 )
@@ -54,7 +55,19 @@ func run(e *env) error {
 }
 
 func runHit(stdout io.Writer, c *config) error {
-	/* TODO: integrate the hit package */
+	req, err := http.NewRequest(http.MethodGet, c.url, http.NoBody)
+	if err != nil {
+		return fmt.Errorf("new request: %w", err)
+	}
+	results, err := hit.SendN(c.n, req, &hit.Options{
+		Concurrency: c.c,
+		RPS:         c.rps,
+	})
+	if err != nil {
+		return fmt.Errorf("send n requests: %w", err)
+	}
+	printSummary(stdout, results.Summarize())
+
 	return nil
 }
 
```
