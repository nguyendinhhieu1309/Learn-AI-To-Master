# MCP and Agent Interoperability

## Learning objectives

Explain what the Model Context Protocol (MCP) standardizes, distinguish its three main primitives, and connect a server without confusing interoperability with authorization.

## 1. The integration problem

Without a shared protocol, each AI application may need a custom adapter for each database, file system, ticket system, or service. MCP defines a client-server protocol for exposing capabilities and context to AI applications. A host typically manages the model and user experience; an MCP client connects to an MCP server that exposes capabilities.

MCP does not make a model intelligent, define your whole agent workflow, or prove a tool is safe. It standardizes how capabilities can be discovered and invoked; your application must still decide which servers to trust, which capabilities to expose, and which users may access them.

## 2. Three common primitives

- **Tools:** executable functions the model can call, such as searching a ticket system or creating a draft.
- **Resources:** data the application/client chooses to provide, such as a document or structured record.
- **Prompts:** reusable message templates surfaced for a user or application to invoke.

This distinction matters for control. A model-requested tool call can create side effects; a resource is context supplied by the application; a prompt is an interaction template. Names, schemas, errors, and content should be reviewed before model use.

## 3. Safe server/client integration

1. Install or run only a server whose source and dependencies you trust.
2. Inspect every declared tool, argument schema, resource, and prompt.
3. Configure credentials with minimum required permissions.
4. Prefer read-only capability during early development.
5. Validate arguments and resource ownership in the server, not only in a model prompt.
6. Add timeouts, rate limits, audit records, and human approval for sensitive side effects.
7. Test unauthorized requests, malformed arguments, server failure, and changed tool schemas.

Transports and authentication details evolve with protocol versions. Check the current specification and SDK documentation before deployment. Treat remote server output as untrusted input.

## 4. MCP compared with skills and ordinary tools

An in-process function can be the simplest choice when an application owns both sides. MCP is useful when capabilities need to be discoverable and reusable across compatible hosts/servers. A skill teaches the model a procedure; an MCP tool exposes a callable capability; a resource supplies context. One agent can use all three, with separate permission rules.

## Practice

Design a read-only MCP server for looking up public product documentation. Define one search tool, one document resource pattern, returned metadata, access restrictions, timeout behavior, and how you would detect a server returning an error or prompt injection text.

## Common mistakes

- Treating every available server as trusted because the protocol is standardized.
- Granting broad database or filesystem access to a tool server.
- Confusing an MCP resource/prompt with an executable model-called tool.
- Assuming server-side validation replaces client-side identity and authorization checks.

## Further reading

- [Model Context Protocol specification](https://modelcontextprotocol.io/specification)
- [Official MCP Python SDK](https://py.sdk.modelcontextprotocol.io/)
- [Official MCP TypeScript SDK](https://ts.sdk.modelcontextprotocol.io/v2/)
