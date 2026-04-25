---
agent: agent
description: "Generate custom quiz themes with matching questions and CSS. Input: theme name (e.g., 'cyberpunk', 'nature'). Output: CSS variables, updated Questions.cs, visual before/after."
---

# Quiz Master Theme Generator

Your goal is to create a fully-themed quiz game variant with custom questions and visual design.

## Workflow

### Step 1: Discover Theme Requirements
Ask the user:
- **Theme name** (e.g., "cyberpunk", "fantasy", "tropical")
- **Color palette** preferences (dominant color, accent, text)
- **Typography** style (futuristic, classic, playful, etc.)
- **Question category** (tech culture, nature, hobbies, etc.)
- **Animation mood** (energetic, serene, dramatic)

### Step 2: Generate Design System

Create a cohesive CSS design with:

**Colors:**
```css
:root {
    --color-primary: /* dominant color */
    --color-accent: /* highlight color */
    --color-bg: /* background */
    --color-text: /* primary text */
    --color-text-secondary: /* secondary text */
    --color-marked: /* when square is marked */
    --color-border: /* borders */
}
```

**Typography:**
- Import distinctive font from Google Fonts (not generic Inter/Roboto)
- Set font-family, line-height, letter-spacing for cohesion

**Animations:**
- Page load: staggered reveal (animation-delay)
- Square interaction: click feedback + color transition
- Win state: celebratory animation (scale, pulse, glow)

**Backgrounds:**
- Use gradients, patterns, or overlays to create atmosphere
- Theme-specific textures (circuit patterns for cyberpunk, leaf patterns for nature, etc.)

### Step 3: Generate Thematic Questions

Create 25+ questions matching the theme:

**Requirements:**
- Mix difficulty levels
- Avoid offensive/exclusive content
- Keep questions short (< 10 words)
- Ensure theme authenticity

**Examples:**
- Cyberpunk: "has hacked a system", "loves retro tech", "speaks 3+ programming languages"
- Nature: "has planted a tree", "seen an aurora", "kayaked a river"
- Fantasy: "has read all HP books", "attended a convention", "plays D&D"

### Step 4: Update Files

Generate modifications to:

**`wwwroot/css/app.css`** or new `theme.css`:
- Add CSS variables
- Update component styles
- Add animations

**`Data/Questions.cs`**:
- Replace or extend `QuestionsList` with theme questions
- Add theme metadata in comments

### Step 5: Create Before/After Comparison

Provide visual comparison showing:
- Original game state (neutral design)
- Themed game board with questions
- Example "Win" state with animation
- Color palette reference card

## Implementation Notes

**Reference Implementations:**
- See `.solutions/step-04-redesign-cyberpunk/` for cyberpunk theme example
- See `.solutions/step-05-quiz-techlife/` for tech culture theme
- See `.solutions/step-06-scavenger-hunt/` for game mode variation

**Tool Usage:**
- Use `semantic_search` to find existing Questions.cs and app.css
- Use `read_file` to understand current color scheme
- Use `mcp_github2_create_or_update_file` to commit changes
- Use `mcp_microsoft_pla_browser_*` to screenshot before/after

**Testing:**
- Verify `dotnet build` passes after updates
- Open browser to http://localhost:5166 and test theme
- Check that all 25 squares have questions (no duplicates)
- Test animations on slow network (DevTools throttling)

## Example Deliverables

### Cyberpunk Theme
```
🎮 Cyberpunk Bingo

Colors:
- Primary: #0F0F1E (deep purple)
- Accent: #00FF9F (neon green)
- Background: #1A1A2E (dark)

Questions: ("has coded at 3am", "speaks binary fluently", "owns retro tech", ...)

Animations: 
- Page load: digital scan effect (top-to-bottom)
- Win state: screen glitch + neon pulse
- Hover: glow effect + color shift
```

### Nature Theme
```
🌿 Nature Bingo

Colors:
- Primary: #2D5016 (forest green)
- Accent: #F77F00 (autumn orange)
- Background: #F1FAEE (cream)

Questions: ("has planted seeds", "seen a waterfall", "camped in a tent", ...)

Animations:
- Page load: leaves falling + fade-in
- Win state: flower bloom effect
- Hover: gentle lift + shadow depth
```

## Workflow: Multi-Agent Orchestration

This prompt works well combined with:
1. **`/setup`** — Verify dev environment before testing theme
2. **`/cloud-explore`** — Generate 3 variation themes in parallel
3. **File edit + browser preview** — Iterate on design in real-time

End with: *"Create a PR with the new theme"* to hand off to GitHub Creation workflow.

---

**Related Lab Section**: [Part 03: Quiz Master](../workshop/03-quiz-master.md)
