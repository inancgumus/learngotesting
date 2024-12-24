# Listing 3.10: Fixing the Parse function

## [testing](https://github.com/inancgumus/gobyexample/blob/c27997172625235247842837497c60900f12cd18/testing) / [url](https://github.com/inancgumus/gobyexample/blob/c27997172625235247842837497c60900f12cd18/testing/url) / [url.go](https://github.com/inancgumus/gobyexample/blob/c27997172625235247842837497c60900f12cd18/testing/url/url.go)

> [!TIP]
> Each listing corresponds to a commit.
>
> Click the links above to see the file and its directory in their original locations and state as they were at the time of the commit.

```go
package url

import (
	"errors"
	"fmt"
	"strings"
)

// A URL represents a parsed URL.
type URL struct {
	Scheme string
	Host   string
	Path   string
}

// Parse parses a raw url into a URL structure.
func Parse(rawURL string) (*URL, error) {
	scheme, rest, ok := strings.Cut(rawURL, ":")
	if !ok {
		return nil, errors.New("missing scheme")
	}

	if !strings.HasPrefix(rest, "//") {
		return &URL{Scheme: scheme}, nil
	}

	host, path, _ := strings.Cut(rest[2:], "/")

	u := &URL{
		Scheme: scheme,
		Host:   host,
		Path:   path,
	}

	return u, nil
}

// String reassembles the URL into a URL string.
func (u *URL) String() string {
	return fmt.Sprintf("%s://%s/%s", u.Scheme, u.Host, u.Path)
}
```

## What's changed?

> [!TIP]
> The following diff shows the changes made to the file since the last commit.
> The lines starting with `+` show the new lines added, and the lines starting with `-` show the lines removed.
> The lines starting with `@@` show the line and column numbers of the changes.

```diff
@@ -15,20 +15,24 @@ type URL struct {
 
 // Parse parses a raw url into a URL structure.
 func Parse(rawURL string) (*URL, error) {
-	scheme, rest, ok := strings.Cut(rawURL, "://")
+	scheme, rest, ok := strings.Cut(rawURL, ":")
 	if !ok {
 		return nil, errors.New("missing scheme")
 	}
 
-	host, path, _ := strings.Cut(rest, "/")
+	if !strings.HasPrefix(rest, "//") {
+		return &URL{Scheme: scheme}, nil
+	}
+
+	host, path, _ := strings.Cut(rest[2:], "/")
 
 	u := &URL{
 		Scheme: scheme,
 		Host:   host,
 		Path:   path,
 	}
 
 	return u, nil
 }
 
 // String reassembles the URL into a URL string.
```
