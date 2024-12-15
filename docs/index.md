# MCP Cases

## Format

> The Model Context Protocol (MCP) is an open protocol that enables seamless integration between LLM applications and external data sources and tools. 

MCP is based on JSON-RPC 2.0 protocol with JSON objects sent from clients to servers and back. If you need to test server or mock it, it would be useful to describe input and output JSON objects as a separate cases.

Since YAML is a superset of JSON, it is possible to define information as a stream of YAML documents.

### Example

Consider this example case of some simple MCP server:

```yaml

case: List tools

# when Client requests list of tools
in: {"jsonrpc": "2.0", "method": "tools/list", "id": 1}

# then Server gives one tool in the list
out:
  {
    "jsonrpc": "2.0",
    "result":
      {
        "tools":
          [
            {
              "description": "Lists files in the current directory",
              "inputSchema": { "type": "object" },
              "name": "list-current-dir-files",
            },
          ],
      },
    "id": 1,
  }
```

This file shows a test case as a document in YAML-formatted file. Case has one input and one output JSON objects following JSON-RPC 2.0 spec.

Here is another example that replicates lifecycle described in the [MCP specification](https://spec.modelcontextprotocol.io/specification/basic/lifecycle/), when you might need two inputs for one output:

```yaml

case: Initialization lifecycle

# Client requesting initialization
in_request_to_initialize: {
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {}, # made it emptier for brevity
    "clientInfo": {
      "name": "ExampleClient",
      "version": "1.0.0"
    }
  }
}

# Server responding to initialization
out: {
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {}, # made it emptier for brevity
    "serverInfo": {
      "name": "ExampleServer",
      "version": "1.0.0"
    }
  }
}

# Clint notifies that initialization is done
in_notification_initialized: {
  "jsonrpc": "2.0",
  "method": "notifications/initialized"
}

---
#  ... other cases ...

```

You might have noticed that both messages from client to server are starting from "in", while server again sends back "out". This is not a coincidence and in fact what this document is proposing - every key that starts with "in" indicates a request, response or notification sent from client to server, while "out" is a response or notification sent from server to client. For this protocol it is irrelevant whether the message is notification or not, as it is just a matter of the "id" field. "in/out" only indicate the expected direction of the communication.

As YAML supports multiple documents in one file, it is possible to define multiple cases in one file, separated by `---` line.

### Formal

MCP Cases is a multi-document YAML file called cases where each document describes a separate named case with lists of input and output JSON objects.

The case document MAY have one `case` key mapped to string value, and it SHOULD be a human-readable name of the case and there MUST be only one such key in the case document.

The case document MAY have one or multiple keys starting with `in`, each of them MUST map to a valid object that SHOULD be matching correct JSON-RPC 2.0 request, response or notification that is intended to be sent from **client to server**.

The case document MAY have one or multiple keys starting with `out`, each of them MUST map to a valid object that SHOULD be matching correct JSON-RPC 2.0 request, response or notification that is intended to be sent from **server to client**.

Note: When we say "matching", the meaning is that it is not necessary to represent that object as JSON, usual YAML can be used as well, but when parsed and printed as JSON, it SHOULD be valid JSON-RPC 2.0 object.

For example such way of writing is also valid:

```yaml
in:
  jsonrpc: "2.0"
  method: "tools/list"
  id: 1
  params: {}
```

, however writers of cases SHOULD be consistent and use the same style for all cases, and they SHOULD prefer to use JSON (quoted, objects and arrays in braces) style in favor of more generic YAML style for those objects, at the same time they MAY use YAML style for other parts of the document, such as root keys (`case`, `in`'s, `out`'s).

That is everything regarding formal definition!

### Extensibility

As this format is based around YAML and only reserves one keyword and two key prefixes, it is very easy to extend it with new features, such as for example including hooks for setup and teardown of the test cases, or including some metadata about the test cases that could be used for filtering or grouping them.

MCP Cases format is considered a base for further development and is open for suggestions and improvements, that could be incorported based on what actual implementations would do.

One convention, however would need to be established and that is that all root keys for extensions should be prefixed with `_` to avoid conflicts with future versions of this format.

For example, if we would like to add a setup hook to the test case, we could do it like this:

```yaml
case: List tools
_before: { command: "start-mcp-server", arguments: ["--port", "1234"] }
# ...
```

This ensures that anything that further versions of this format would add cannot be in conflict with any of implementations in the wild and such would ease the process of upgrading to newer versions of this specification.

## Motivation

### Contract Testing

Besids tests for MCP servers, this can be used it to create mock servers that you could use when checking client implementations. 
The approach like that is known as `contract testing` and is a way to ensure that both client and server are working as expected, and that they are compatible with each other without actually running them together. You write list of cases that is a specification of things that you would like to make possible, and then you can run them against both client and server to see if they are compatible up to the point of those cases. 
Such list of cases is then called a `contract`. Contracts then could be used to pass around between teams as a way of communication of what is expected and what has possibly gone wrong or got changed, etc.

#### Rapid Prototyping

When you only starting to develop your MCP server and not exactly sure how it should look like, you can start by writing MCP stories for the future implementation, while you collect requirements from experts and stakeholders. Having such active document could be a great way to communicate between people with different competences and to ensure that everyone is on the same page, while at the same time giving you a way to test your implementation later as well as seeing how LLMs would interact with it before you even start building.

Same applies for any new features you would be implementing. Not sure if it is better to be a prompt or a tool? Sketch a quick case, give it to mock server and see how it would look like from the actual client's perspective.

### Documentation

This format can be used to generate documentation for MCP servers to be published on the web.

