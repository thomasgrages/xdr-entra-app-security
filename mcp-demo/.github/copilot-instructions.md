---
applyTo: '**'
---
## Target Tenant

The default Azure / Entra ID tenant for all queries is **b22dee98-83da-4207-b9ab-5ba931866f44**.

## Querying Microsoft Documentation

You have access to MCP tools called `microsoft_docs_search`, `microsoft_docs_fetch`, and `microsoft_code_sample_search` - these tools allow you to search through and fetch Microsoft's latest official documentation and code samples, and that information might be more detailed or newer than what's in your training data set.

When handling questions around how to work with native Microsoft technologies, such as C#, F#, ASP.NET Core, Microsoft.Extensions, NuGet, Entity Framework, the `dotnet` runtime - please use these tools for research purposes when dealing with specific / narrowly defined questions that may occur.

## Querying Microsoft Graph (Entra ID / Conditional Access)

You have access to MCP tools prefixed with `microsoft_graph_` — use these to query the Microsoft Graph API for Entra ID objects including Conditional Access policies, users, groups, service principals, and directory roles.

Key queries for Conditional Access policy reporting:
- **List all CA policies:** `GET /beta/identity/conditionalAccess/policies?$count=true&$select=id,displayName,createdDateTime,modifiedDateTime,state&$orderBy=modifiedDateTime desc`
- **Enabled policies only:** add `$filter=state eq 'enabled'`
- **Policy details:** `GET /v1.0/identity/conditionalAccess/policies/{id}` for full grant controls, session controls, and conditions
- Use `microsoft_graph_suggest_queries` to discover the right Graph endpoint for any Entra ID question.
- Use `microsoft_graph_get` to execute Graph API calls.
- Use `microsoft_graph_list_properties` to explore response schemas.

## Querying Azure Resources

You have access to Azure MCP tools — use these for querying Azure subscriptions, resource groups, and resources via Azure Resource Graph. These tools can help correlate Azure resource configurations with Conditional Access policy assignments.
