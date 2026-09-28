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

&nbsp;

## Tool Schema Design

Naming rules：

- snake_case
- Verb-noun order. get_weather, not weather_get.
- No tense markers. get_weather, not got_weather or get_weather_later.
- Stable.  不要随意改名
- Namespace prefixes for large registries. notes_list, notes_search, notes_create beats three tools named generically.
- No arguments in the name. get_weather_for_city(city), not get_weather_in_tokyo().
  
  &nbsp;

Description pattern：

- Use when {condition}. Do not use for {close-but-wrong-cases}.

&nbsp;

Atomic：拆分成原子动作
例如：

- notes_list()
- notes_create(title, body)
- notes_delete(note_id)
- notes_search(query)

&nbsp;

Parameter design：

- Enum every closed set. units: "celsius" | "fahrenheit" not units: string.
- Required vs optional. Mark the minimum needed. Everything else optional. 
- Typed IDs. note_id: string is fine but add a pattern (^note-[0-9]{8}$) to catch hallucinated ids.
- No overly flexible types. Avoid type: any. 
- Describe the field. {"type": "string", "description": "ISO 8601 date in UTC, e.g. 2026-04-22"}. 

&nbsp;

Error messages as teaching signals：
The good error teaches the model what to do next.

```
BAD  : TypeError: object of type 'NoneType' has no attribute 'lower'
GOOD : Invalid input: 'city' is required. Example: {"city": "Bengaluru"}.
```

&nbsp;

Versioning：

- Never rename a stable tool. Add get_weather_v2 and deprecate get_weather.
- Never change argument types.
- Add optional parameters freely. Safe.
- Remove tools only with a deprecation window. 

&nbsp;

Tool poisoning prevention：

- 防止常用的提示词注入：rejects descriptions containing common indirect-injection keywords: <SYSTEM>, ignore previous, URL-shortening patterns, unescaped markdown that includes hidden instructions.

&nbsp;

Benchmarks：

- StableToolBench：https://zhichengg.github.io/stb.github.io/    https://github.com/THUNLP-MT/StableToolBench
- MCPToolBench++：https://github.com/mcp-tool-bench/MCPToolBenchPP
- SafeToolBench：https://arxiv.org/abs/2509.07315

&nbsp;

https://composio.dev/blog/how-to-build-tools-for-ai-agents-a-field-guide
