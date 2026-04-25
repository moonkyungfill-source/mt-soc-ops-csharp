# Copilot Workspace Instructions

## Overview

**Soc Ops** is a GitHub Copilot Agent Lab project—a Blazor WebAssembly social bingo game built with .NET 10. This workspace demonstrates agent-driven development patterns through structured workshop parts (00-04).

- 🎮 **Game**: Social Bingo for team building/icebreakers
- 🎓 **Lab**: Learn GitHub Copilot agents across 5 workshop modules
- 🛠️ **Stack**: Blazor WebAssembly, .NET 10, C#, custom CSS utilities

## Development Checklist

Before committing any changes:

- [ ] `dotnet build SocOps/SocOps.csproj` passes with no errors
- [ ] `dotnet test` passes (when tests exist)
- [ ] Code follows C# naming: `PascalCase` for public members, `_camelCase` for private fields
- [ ] No unused variables, imports, or `using` statements
- [ ] Components tagged with `@using` only when necessary

## Project Structure

```
SocOps/                      # Main Blazor WebAssembly app
├── Components/             # Reusable Razor components
│   ├── BingoBoard.razor
│   ├── BingoSquare.razor
│   ├── BingoModal.razor
│   ├── GameScreen.razor
│   └── StartScreen.razor
├── Models/                 # Data models (GameState, BingoSquareData, BingoLine)
├── Services/               # Business logic
│   ├── BingoGameService.cs        # State management & event dispatcher
│   └── BingoLogicService.cs       # Game rules & win detection
├── Data/                   # Static data (Questions.cs)
├── Pages/                  # Routable pages (Home.razor as main)
├── Layout/                 # App shell (MainLayout.razor, NavMenu.razor)
├── _Imports.razor          # Global usings
├── App.razor               # Root component
└── wwwroot/                # Static assets
    ├── css/app.css         # Custom utility classes
    └── index.html          # Bootstrap HTML

workshop/                   # Educational lab guide (5 parts)
├── GUIDE.md                # Overview & task summary
├── 00-overview.md          # Lab objectives & architecture
├── 01-setup.md             # Context engineering & dependency setup
├── 02-design.md            # Design-first frontend with Plan Mode
├── 03-quiz-master.md       # Custom quiz theme generation
└── 04-multi-agent.md       # Multi-agent orchestration

.github/
├── instructions/           # Domain-specific guidance
│   ├── css-utilities.instructions.md     # Utility class system
│   └── frontend-design.instructions.md   # Design & aesthetics
└── prompts/                # Reusable task workflows
    ├── setup.prompt.md                   # Dev environment setup
    └── cloud-explore.prompt.md           # Multi-variation exploration
```

## Key Commands

```bash
# Build
dotnet build SocOps/SocOps.csproj

# Run dev server (http://localhost:5166, with hot reload)
dotnet run --project SocOps/SocOps.csproj

# Run tests
dotnet test

# From SocOps directory
cd SocOps && dotnet run
```

## Code Conventions

### C# Style
- **Namespaces**: `SocOps.{Feature}` (e.g., `SocOps.Services`, `SocOps.Models`)
- **Public members**: `PascalCase` (properties, methods, classes)
- **Private fields**: `_camelCase` with underscore prefix
- **Constants**: `UPPER_SNAKE_CASE`
- **Async patterns**: Use `async/await`; suffix with `Async` (e.g., `LoadGameStateAsync`)

### Blazor Components
- **File naming**: Match component name (e.g., `BingoBoard.razor` → `BingoBoard` component)
- **Parameter validation**: Use `[Parameter]` + null coalescing when needed
- **Event handling**: Prefer `@onclick`, `@onchange` over raw JavaScript
- **State updates**: Call `StateHasChanged()` only when necessary (manual updates to non-component state)

### HTML & CSS
- **Structure**: Keep markup clean; complex logic belongs in `@code` blocks
- **CSS classes**: Use custom utility classes from `app.css` (documented in `.github/instructions/css-utilities.instructions.md`)
- **Inline styles**: Avoid; use utility classes instead
- **Responsive**: Mobile-first mindset; test on small screens

## State Management

- **Single source of truth**: `BingoGameService` holds state
- **Event-driven**: Components subscribe to `OnStateChanged` event
- **Persistence**: Game state saved to browser localStorage via `IJSRuntime`
- **Initialization**: Components call `InitializeAsync()` on mount
- **Version tracking**: Update `STORAGE_VERSION` in `BingoGameService` when schema changes

**Example pattern:**
```csharp
private BingoGameService GameService { get; set; }

protected override async Task OnInitializedAsync()
{
    await GameService.InitializeAsync();
    GameService.OnStateChanged += HandleStateChanged;
}

private void HandleStateChanged() => StateHasChanged();
```

## Styling System

See [CSS Utilities Instructions](css-utilities.instructions.md) for the complete utility class reference.

**Quick reference:**
- **Layout**: `.flex`, `.flex-col`, `.grid`, `.grid-cols-5`, `.items-center`, `.justify-center`
- **Spacing**: `.p-1` to `.p-6`, `.px-*`, `.py-*`, `.mb-2` to `.mb-8`, `.mx-auto`, `.gap-1`, `.space-y-2`
- **Colors**: `.bg-accent` (blue), `.bg-marked` (green), `.text-gray-*`, `.text-green-600`, `.text-amber-500`
- **Typography**: `.text-xs` to `.text-5xl`, `.font-semibold`, `.font-bold`, `.text-center`, `.leading-tight`

**Principle**: Use utility classes for 80% of styling. Only add component-scoped CSS for animations or complex layouts.

## Design & Aesthetics

See [Frontend Design Instructions](frontend-design.instructions.md) for guidance on avoiding generic "AI slop" aesthetics.

**Key principles:**
- Choose distinctive, high-quality typography (not generic Inter/Arial)
- Commit to cohesive color palette with bold accents
- Use CSS animations for high-impact moments (page load, win state)
- Create atmosphere with gradients & patterns, not flat colors
- Think contextually: make designs that feel genuinely designed for *this* project

## Testing

- Unit tests: Place in `SocOps.Tests/` parallel to `SocOps/`
- Test naming: `[UnitUnderTest]_[Scenario]_[ExpectedBehavior]`
- Use xUnit for test framework
- Mock external dependencies (localStorage, JSInterop)

## GitHub Workflow

### Branches
- `main`: Always production-ready, deployed to GitHub Pages
- `feature/*`: Feature branches with descriptive names
- Merge via pull request with at least 1 review

### Commit Messages
- Imperative mood: "Add...", "Fix...", "Refactor...", not "Added..." or "Fixed..."
- Reference issue numbers: "Fix #42: ..."
- Keep first line under 50 characters

### Pull Requests
- Link to workshop part or issue
- Include "before/after" descriptions for UI changes
- Run `/setup` prompt to verify local dev environment works

## Lab Workshop Context

This project includes structured learning modules in `workshop/`:

1. **Part 00**: Overview & checklist (5 min)
2. **Part 01**: Setup & context engineering (15 min) — *Generate workspace instructions, lint background agent*
3. **Part 02**: Design-first frontend (15 min) — *UI redesign with Plan Mode, cloud-explore variations*
4. **Part 03**: Quiz Master (10 min) — *Custom quiz theme generation*
5. **Part 04**: Multi-agent orchestration (20 min) — *Combine agents for complex workflows*

**For instructors**: Each part has a solution in `.solutions/step-{N}/` showing expected outcomes. Use these for reference or as fallback content.

## Troubleshooting

**Dev server won't start**
- Verify .NET 10 SDK: `dotnet --version`
- Clear build artifacts: `dotnet clean SocOps/SocOps.csproj`
- Check port 5166 isn't in use: `lsof -i :5166`

**Hot reload not working**
- Restart dev server: `Ctrl+C`, then `dotnet run --project SocOps`
- Verify file is saved in VS Code

**localStorage issues**
- Clear browser cache/localStorage: DevTools → Application → Storage → Clear All
- Increment `STORAGE_VERSION` in `BingoGameService.cs` to reset saved state

**Build fails on clean machine**
- Restore dependencies: `dotnet restore`
- Verify .NET 10: `dotnet --version`
- Check `SocOps.csproj` for missing platform targets

## Tools & Prompts

**Setup Prompt** (`/setup`)
- Verify dependencies, build, run dev server, open browser

**Cloud Explore Prompt** (`/cloud-explore`)
- Explore design variations with multi-agent approach

**CSS Utilities & Frontend Design Instructions**
- Loaded automatically in relevant contexts
- Use when building/styling components

## Links

- 🎮 [Live Demo](https://dotnet-presentations.github.io/vscode-github-copilot-agent-lab/)
- 📚 [Lab Guide](https://dotnet-presentations.github.io/vscode-github-copilot-agent-lab/docs/)
- 📖 [.NET 10 Docs](https://learn.microsoft.com/dotnet/fundamentals/)
- 🎨 [Blazor Docs](https://learn.microsoft.com/aspnet/core/blazor/)
- 🏗️ [Architecture Decision Log](ARCHITECTURE.md) (if created)

---

**Last Updated**: April 2026
