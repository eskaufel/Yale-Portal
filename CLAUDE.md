# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Yale-Portal is a Blazor Server application that provides a web interface for the [Yale expression evaluation library](https://github.com/Verent/Yale). It allows users to evaluate mathematical and logical expressions at runtime by parsing and compiling inputs to CIL on the server. Live at: https://yale.verent.com/

## Build & Run Commands

```bash
# Build
dotnet build src/Yale.Portal/Yale.Portal.csproj

# Run (development)
dotnet run --project src/Yale.Portal/Yale.Portal.csproj

# Build for release
dotnet build src/Yale.Portal/Yale.Portal.csproj --configuration Release
```

There are no tests or linting steps in this project. The CI pipeline (`.github/workflows/build.yml`) only runs a Release build on pull requests.

## Architecture

**Stack:** .NET 5.0, Blazor Server (SignalR/WebSockets), Razor components, Bootstrap.

No Node.js, no REST API, no database. All state is held in scoped server-side services; client-server communication uses SignalR (Blazor's default for server-side hosting).

### Core Service: `YaleService`

`src/Yale.Portal/Services/YaleService.cs` — the single service that wraps the Yale library. It extends `ComputeInstance` and is registered as a **scoped** service (one instance per SignalR connection/session).

Key constraints enforced by the service:
- `MaxExpressions = 5`
- `MaxVariables = 5`
- `Math` type imported, so expressions like `Sin(x)` work out of the box

**AddValue** — parses `"key=value"` strings with type inference: tries int → double → bool → falls back to string.

**AddExpression** — parses `"key:expression"` strings (colon-separated). Silently swallows exceptions on invalid input; returns a bool indicating success.

**TryEvaluate** — safely evaluates a named expression and returns the result.

### Pages & Routing

Routes are declared via `@page` directives on Razor components:

| Route | Component | Purpose |
|-------|-----------|---------|
| `/` | `Pages/Index.razor` | Landing page |
| `/terminal` | `Pages/Terminal.razor` | Interactive REPL |
| `/about` | `Pages/About.razor` | Credits |
| `/updates` | `Pages/Updates.razor` | Release notes |

`Pages/_Host.cshtml` is the HTML shell (non-Blazor entry point). `App.razor` is the root Blazor router. `Shared/MainLayout.razor` wraps all pages with the sidebar layout.

### Terminal.razor — the main interactive component

The most complex component. It provides a REPL-style interface driven entirely by Blazor event bindings (`@onkeyup`, `@bind`). Input is parsed client-side with three regex patterns before being routed to `YaleService`:

- **Variable assignment** (`x=4`, `x=3.14`, `x=hello`) — matches `^[a-zA-Z]+[=][\w\s.]+$`
- **Expression definition** (`e:2*x`) — matches `[a-zA-Z]+[:].+$`
- **Evaluation** (`e`) — matches `^[a-zA-Z]+$`

The component maintains a command history and supports arrow-key navigation through it.

## Key Conventions

- **Service injection** in Razor components: `@inject YaleService service`
- **Culture** is hardcoded to `en-US` in `Startup.cs` — relevant for decimal parsing
- **Error handling** in `YaleService` is silent (swallows exceptions and returns bool). When extending the service, follow the same pattern of returning success indicators rather than propagating exceptions to the UI.
- **Static assets** live in `wwwroot/` — Bootstrap and Open Iconic are vendored there (no npm).
