# Tools

## Tool

Tool interface: a contract that lets the model request an action and the host execute it.
工具类型：

- function calling
- MCP
- A2A

&nbsp;

The four-step loop： describe → decide → execute → observe

1. describe：The host declares each tool with three fields：Name、Description、Input schema，The model receives the list. 

2. 2.decide：Given the user's message and the available tools, the model chooses one of three behaviors. 
   a. Answer directly in text. No tool call.
   b. Call one or more tools. Emit structured call objects.
   c. Refuse. Strict-mode structured outputs can produce a typed refusal block instead of a call.

3. execute：The host receives the call, validates arguments against the declared schema, and runs the executor.

4. observe：The host appends the tool result to the conversation and re-invokes the model. The model now has the tool output in context and can produce a final answer or request more calls. This continues until the model stops emitting calls or the host hits a safety limit on iteration count.

&nbsp;

|                     |                              |                                                                                             |
| ------------------- | ---------------------------- | ------------------------------------------------------------------------------------------- |
| Term                | What people say              | What it actually means                                                                      |
| Tool                | "A thing the model can call" | A triple of name + JSON-Schema-typed input + executor function                              |
| Function calling    | "Native tool use"            | Provider-level API support for emitting structured tool calls instead of prose              |
| Tool call           | "The model's request to act" | A JSON payload with id, name, arguments<br><br>emitted by the model                         |
| Tool result         | "What the tool returned"     | The executor's output, wrapped in a tool role message with matching id                      |
| Parallel tool calls | "Many calls at once"         | Multiple call objects in one model turn, independent and orderable by id                    |
| Strict mode         | "Guaranteed JSON"            | Constrained decoding that forces the model's output to validate against the declared schema |
| Pure tool           | "Read-only tool"             | No side effects; safe to re-run                                                             |
| Consequential tool  | "Action tool"                | Mutates external state; requires gate, audit, or user confirmation                          |
| Four-step loop      | "The tool-call cycle"        | describe → decide → execute → observe                                                       |
| Host                | "Agent runtime"              | The program that holds the tool registry, calls the model, and runs the executor            |
