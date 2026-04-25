---
description: "Blazor component patterns, lifecycle, performance, and accessibility. Use when building or editing .razor files in the Components/ folder."
applyTo: "SocOps/Components/**/*.razor"
---

# Blazor Component Development

## Component Structure Template

All components should follow this structure:

```razor
@using SocOps.Models
@inherits LayoutComponentBase
@* or @implements IAsyncDisposable for custom lifecycle *@

<div class="component-root">
    @if (IsLoading)
    {
        <p>Loading...</p>
    }
    else
    {
        @ChildContent
    }
</div>

@code {
    [Parameter]
    public RenderFragment? ChildContent { get; set; }
    
    [Inject]
    public required BingoGameService GameService { get; set; }
    
    private bool IsLoading { get; set; } = true;

    protected override async Task OnInitializedAsync()
    {
        await GameService.InitializeAsync();
        IsLoading = false;
    }

    protected override void OnParametersSet()
    {
        // Validate incoming parameters
    }
}
```

## Parameters & Cascading

**Best Practices:**
- Mark parameters `[Parameter]` for parent-to-child communication
- Use `[CascadingParameter]` only for truly global state (don't cascade GameService—inject instead)
- Validate parameters in `OnParametersSet()`:
  ```csharp
  protected override void OnParametersSet()
  {
      if (SquareId < 0)
          throw new ArgumentException("SquareId must be non-negative");
  }
  ```

**Immutability:**
- Parameters are immutable from the component's perspective
- Changes flow via parent re-rendering
- For two-way binding, use `@bind-Value` with `[Parameter] public EventCallback<T> ValueChanged`

## Lifecycle Patterns

**Initialization Order** (first render):
1. `OnInitializedAsync()` — Load initial state, fetch data
2. `OnParametersSetAsync()` — React to parameter changes
3. `OnAfterRenderAsync()` — DOM-dependent logic (focus, scroll, canvas setup)

**Example: Subscribe to State Changes**
```csharp
protected override async Task OnInitializedAsync()
{
    await GameService.InitializeAsync();
    GameService.OnStateChanged += HandleStateChanged;
}

private void HandleStateChanged()
{
    StateHasChanged(); // Trigger re-render
}

async ValueTask IAsyncDisposable.DisposeAsync()
{
    GameService.OnStateChanged -= HandleStateChanged;
}
```

**Cleanup:**
- Unsubscribe from events in `OnAfterRender` → cleanup in `IAsyncDisposable.DisposeAsync()`
- Cancel long-running tasks when component unmounts
- Release JSInterop callbacks

## Rendering & Performance

**Avoid Unnecessary Renders:**
- Only call `StateHasChanged()` when *YOUR* state actually changes
- Let parent re-renders cascade naturally
- Don't manually fix Blazor's diffing—trust it

**Key Optimization:**
```csharp
// ❌ BAD: Manual state management that causes extra renders
private int Count { get; set; }
public void Increment() { Count++; StateHasChanged(); }

// ✅ GOOD: Let parameters/events drive renders
[Parameter] public int Count { get; set; }
[Parameter] public EventCallback<int> CountChanged { get; set; }
public async Task Increment() => await CountChanged.InvokeAsync(Count + 1);
```

**EventCallbacks vs Events:**
- Use `EventCallback<T>` for parent-child communication
- Use internal `event Action` only for component-internal observers
- Example from BingoGameService:
  ```csharp
  public event Action OnStateChanged; // Internal state observers
  // (parents invoke via EventCallback instead)
  ```

## Accessibility (a11y)

**Keyboard Navigation:**
- All interactive elements must be keyboard accessible
- Use `@onkeypress` for custom key handling
- Example for BingoSquare:
  ```razor
  <button @onkeypress="HandleKeyPress" 
          class="bingo-square"
          aria-pressed="@IsMarked">
      @Text
  </button>
  ```

**ARIA Labels:**
- Buttons with only icons: `aria-label="Mark square"`
- Game state announcements: `role="status"` + `aria-live="polite"`
- Modal dialogs: `role="dialog"` + `aria-labelledby="modal-title"`

**Screen Reader Hints:**
```razor
<!-- ✅ Good: Modal is properly announced -->
<div role="dialog" aria-labelledby="modal-title" aria-modal="true">
    <h2 id="modal-title">You Won!</h2>
    <p>You got 5 in a row!</p>
</div>
```

## Styling Components

**CSS Approach:**
1. Use utility classes from `app.css` for 80% of styling
2. Component-scoped CSS (`Component.razor.css`) for complex or animated state
3. Never use inline `style="..."`

**Component-Scoped CSS Example** (`BingoSquare.razor.css`):
```css
:host {
    --square-bg: var(--color-white);
    --square-border: var(--color-gray-300);
}

.bingo-square {
    background-color: var(--square-bg);
    border-color: var(--square-border);
    transition: all 0.2s ease;
}

.bingo-square.marked {
    --square-bg: var(--color-marked);
    --square-border: var(--color-marked-dark);
}

.bingo-square:hover {
    transform: scale(1.05);
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.bingo-square:focus {
    outline: 2px solid var(--color-accent);
    outline-offset: 2px;
}
```

**Why Not Inline Styles:**
- Can't apply hover/focus states
- Can't use transitions/animations
- Harder to maintain
- Accessible color contrast harder to verify

## Event Handling

**Button Clicks:**
```razor
<button @onclick="HandleClick">Click Me</button>

@code {
    private void HandleClick() { /* ... */ }
}
```

**Form Events:**
```razor
<input @onchange="HandleChange" />
<input @onkeyup="HandleKeyUp" />

@code {
    private void HandleChange(ChangeEventArgs args) 
    { 
        var value = args.Value?.ToString();
    }
}
```

**Prevent Default & Stopping Propagation:**
```razor
<form @onsubmit:preventDefault="true" 
      @onsubmit="HandleSubmit">
    <input type="submit" />
</form>
```

## Common Patterns in SocOps

### Pattern 1: Toggle Square (BingoSquare.razor)
```razor
@if (Square.IsMarked)
{
    <div class="checkmark">✓</div>
}

<button @onclick="HandleClick"
        class="@CssClass"
        aria-pressed="@Square.IsMarked"
        aria-label="@Square.Text">
    @Square.Text
</button>

@code {
    [Parameter] public required BingoSquareData Square { get; set; }
    [Parameter] public EventCallback OnClick { get; set; }

    private string CssClass => $"bingo-square {(Square.IsMarked ? "marked" : "")}";

    private async Task HandleClick()
    {
        await OnClick.InvokeAsync();
    }
}
```

### Pattern 2: Modal Overlay (BingoModal.razor)
```razor
@if (Visible)
{
    <div class="modal-overlay" @onclick="Close">
        <div class="modal-content" @onclick:stopPropagation>
            <h2>@Title</h2>
            @ChildContent
        </div>
    </div>
}

@code {
    [Parameter] public required string Title { get; set; }
    [Parameter] public RenderFragment? ChildContent { get; set; }
    [Parameter] public bool Visible { get; set; }
    [Parameter] public EventCallback OnClose { get; set; }

    private async Task Close() => await OnClose.InvokeAsync();
}
```

### Pattern 3: Subscribe to Service Events (GameScreen.razor)
```csharp
protected override async Task OnInitializedAsync()
{
    await GameService.InitializeAsync();
    GameService.OnStateChanged += StateChanged;
}

private void StateChanged() => StateHasChanged();

async ValueTask IAsyncDisposable.DisposeAsync()
{
    GameService.OnStateChanged -= StateChanged;
    await ValueTask.CompletedTask;
}
```

## Testing Components

**Unit Test Pattern:**
```csharp
[Fact]
public async Task BingoSquare_OnClick_InvokesCallback()
{
    // Arrange
    var squareData = new BingoSquareData(1, "Test", false);
    var callbackInvoked = false;
    
    var cut = new TestContext().RenderComponent<BingoSquare>(
        (nameof(BingoSquare.Square), squareData),
        (nameof(BingoSquare.OnClick), EventCallback.Factory.Create(null, () => callbackInvoked = true))
    );

    // Act
    cut.Find("button").Click();

    // Assert
    Assert.True(callbackInvoked);
}
```

**Test Utilities:**
- `bunit` for component testing
- Mock `BingoGameService` for integration tests
- Use `WaitForAssertion()` for async operations

## Common Gotchas

| Issue | Solution |
|-------|----------|
| Component doesn't re-render after state change | Call `StateHasChanged()` or trigger from parent |
| Parameter changes not detected | Ensure parameter is marked `[Parameter]` |
| Event handler not firing | Use `@onclick` (not `onclick`), use `EventCallback` |
| Memory leak (subscriptions) | Unsubscribe in `DisposeAsync()` |
| Focus lost after render | Store focus in `OnAfterRenderAsync()`, restore after render |
| Styling not applied | Check component scope (`.razor.css` vs global `app.css`) |
| Keyboard accessibility broken | Add `@onkeydown`, test with Tab key |

## Links

- [Blazor Docs: Components](https://learn.microsoft.com/aspnet/core/blazor/components/)
- [Blazor Docs: Lifecycle](https://learn.microsoft.com/aspnet/core/blazor/components/lifecycle)
- [bUnit Testing Library](https://bunit.dev/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [Accessibility Tree Inspector](https://www.w3.org/WAI/test-evaluate/preliminary/)

---

**Last Updated**: April 2026
