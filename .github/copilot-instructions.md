# Copilot Workspace Instructions

## Mandatory Development Checklist
- [ ] `dotnet format` (or equivalent lint/format check)
- [ ] `dotnet build SocOps/SocOps.csproj`
- [ ] `dotnet test`

## Overview
Soc Ops is a Blazor WebAssembly social bingo game built with .NET 10 and hosted in a Copilot Agent Lab workspace. Workshop guidance lives under `workshop/`.

## Key Commands
```bash
cd SocOps
dotnet format
dotnet build SocOps.csproj
dotnet run --project SocOps.csproj
dotnet test
```

## Structure
- `SocOps/Components/` — Razor UI components
- `SocOps/Services/` — game state and rules
- `SocOps/Models/` — shared data models
- `SocOps/Data/Questions.cs` — bingo prompts
- `wwwroot/css/app.css` — utility CSS system
- `workshop/` — learning modules
- `.github/instructions/` — component/style guidance

## Conventions
- Public members use `PascalCase`
- Private fields use `_camelCase`
- Async methods end with `Async`
- Prefer utility CSS classes and component-scoped CSS
- Use `EventCallback<T>` for child-to-parent interactions

## Architecture Notes
- `BingoGameService` holds the app state
- `BingoLogicService` generates boards and checks wins
- Components subscribe to `OnStateChanged` for UI updates
- Game state persists via `IJSRuntime` localStorage

## Design Guidance
- Use the CSS utility system in `.github/instructions/css-utilities.instructions.md`
- Avoid generic component styling; keep themes distinctive and polished

## Troubleshooting
- Confirm .NET 10 with `dotnet --version`
- Restart `dotnet run` if hot reload stops working
- Increment `STORAGE_VERSION` in `BingoGameService` to reset saved state
