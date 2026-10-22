## Tooling Notes

GitHub itself maintains an MCP server for its own API. Rather than guessing from memory, this was located through GitHub code search.

### What code search found

- Repository: **`github/github-mcp-server`** (owner `github`, name `github-mcp-server`), described as "GitHub's official MCP Server". https://github.com/github/github-mcp-server

### Files actually opened

- `pkg/github/context_tools.go` — where the `get_me` tool is registered
- `pkg/github/issues_granular.go` — where the `create_issue` tool is registered

### How it wires up its tools

Each tool is a Go function that returns an `inventory.ServerTool`, built by a generic helper (`NewTool[...]` / `NewToolWithSchemaOptions[...]`) that takes the toolset metadata, an `mcp.Tool` literal carrying the name, description, annotations, and input schema, a scope requirement, and the handler function. Below are the two registration lines, quoted verbatim.

**The tool that returns the authenticated user's own profile** (`pkg/github/context_tools.go`):

```go
// GetMe creates a tool to get details of the authenticated user.
func GetMe(t translations.TranslationHelperFunc) inventory.ServerTool {
	inputSchema := contextToolInputSchema[GetMeInput](nil)
	return NewToolWithSchemaOptions[GetMeInput, MinimalUser](
		ToolsetMetadataContext,
		mcp.Tool{
			Name:        "get_me",
			Description: t("TOOL_GET_ME_DESCRIPTION", "Get details of the authenticated GitHub user. Use this when a request is about the user's own profile for GitHub. Or when information is missing to build other tool calls."),
```

**The tool that files a new issue** (`pkg/github/issues_granular.go`):

```go
// GranularCreateIssue creates a tool to create a new issue.
func GranularCreateIssue(t translations.TranslationHelperFunc) inventory.ServerTool {
	st := NewTool[GranularCreateIssueInput, *MinimalResponse](
		ToolsetMetadataIssues,
		mcp.Tool{
			Name:        "create_issue",
			Description: t("TOOL_CREATE_ISSUE_DESCRIPTION", "Create a new issue in a GitHub repository with a title and optional body."),
```

### Notes for the release team

- Both tools are registered as MCP tools with explicit names (`get_me`, `create_issue`) inside the `mcp.Tool` struct literal.
- `get_me` is annotated `ReadOnlyHint: true` and requires no scopes.
- `create_issue` is annotated as a write tool (not read-only, not destructive) and requires `scopes.RequireAll(scopes.Repo)`.
- Tool descriptions are internationalized through a `translations.TranslationHelperFunc`.
