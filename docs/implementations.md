# Known Implementations

These implementations are following MCP Cases format at the moment:

- [foxytest - Golang package implmenting integration tests for MCP servers, which is a part of foxy-contexts library](https://github.com/strowk/foxy-contexts/tree/main/pkg/foxytest)

If you make anything based on this specification, please let me know so it can be listed here!

| Name | Description | License |
|------|-------------|---------|
| [foxytest](https://github.com/strowk/foxy-contexts/tree/main/pkg/foxytest) | Golang package for MCP server integration tests | MIT |

## foxytest

> foxytest is a Golang package for MCP server integration tests.

You can use this package without foxy-contexts library and even mostly without knowing Golang, as you would only ever need one 20 lines long `*_test.go` file, that looks like this:

```go
package main

import (
	"testing"

	"github.com/strowk/foxy-contexts/pkg/foxytest"
)

func TestServer(t *testing.T) {
	ts, err := foxytest.Read("testdata")
	if err != nil {
		t.Fatal(err)
	}
	ts.WithExecutable("go", []string{"run", "main.go"})
	ts.Run(t)
	ts.AssertNoErrors(t)
}
```

This script reads test cases defined in format described above from "testdata" folder as a files with `_test.yaml` suffix, and runs them against `my_installed_mcp_server` executable, that would be started with no arguments.

To run this, you would still need to install Golang and use `go test` command running it in the same folder as the `*_test.go` file (f.e name it `main_test.go`).
