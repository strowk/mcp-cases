# Known Implementations

These implementations are following MCP Cases format at the moment:

- [foxytest - Golang package implmenting integration tests for MCP servers, which is a part of foxy-contexts library](https://github.com/strowk/foxy-contexts/tree/main/pkg/foxytest)

If you make anything based on this specification, please let me know so it can be listed here!

| Name | Description | License |
|------|-------------|---------|
| [mcp-autotest](https://github.com/strowk/mcp-autotest) | CLI utility for language agnostic MCP server integration tests | MIT |
| [foxytest](https://github.com/strowk/foxy-contexts/tree/main/pkg/foxytest) | Golang package for MCP server integration tests | MIT |
| [mcptee](https://github.com/strowk/mcptee) | Logging MCP server communication for troubleshooting | MIT |

## mcp-autotest

> mcp-autotest is a CLI utility for language agnostic MCP server integration tests.

Once [installed](https://github.com/strowk/mcp-autotest#installation) and with test cases with names ending on `_test.yaml` put in f.e `testdata` folder, you can run it like this:

```bash
mcp-autotest run testdata <path/to/mcp_server_executable> <args...>
```

For example: `mcp-autotest run testdata go run main.go` for Golang or `mcp-autotest run testdata uv run main.py` for Python and so on.

You can quickly try it via npx using `npx mcp-autotest`.

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

## mcptee

> A simple tool that proxies input from MCP client to MCP server and returns the output back to the client, while also logging both to the file line

Prepend the command to run MCP server with `mcptee <path/to/file>` and you will see what happens when client sends requests to the server and what server responds back in that file.

See more about examples and usage in [mcptee repository](https://github.com/strowk/mcptee?tab=readme-ov-file#usage).
