---
name: turn-to-code
description: Converts selected Figma designs into reusable React components following design-system-acb. Use when user says "turn to code", "convert to code", or wants to transform Figma elements into React. Requires Figma URL. Output uses design tokens only, never hard-coded values. Suitable for prototyping and reuse across projects.
metadata:
  mcp-server: figma
---

# Turn to Code

Transforms Figma selections into production-ready, reusable React components. All output follows the `YOUR_TEAM_RULES_HERE` rules and uses design tokens—never hard-coded colors, spacing, or typography.

## When Unclear—Ask the User

**If anything is ambiguous** (variant structure, state names, which node to use, color intent)—ask the user before implementing. Do not guess.

## When User Says "Turn to Code"

If the user has not yet provided a Figma link, respond with:

**Hint:**
> To turn your Figma design into code, please attach a Figma link (e.g. `https://figma.com/design/:fileKey/:fileName?node-id=X-Y`).

Then ask these clarification questions as needed:

1. **Component name** – What should the created component be named? (e.g. `Button`, `Card`, `Input`)
2. **Variations** – Does this component have variants (e.g. primary/secondary, sizes, states)? Which should be implemented?
3. **Default state** – What is the default appearance (variant, size, disabled)?
4. **Interaction** – Any required behavior (e.g. loading, icon-only, with label)?
5. **Output path** – Where should the component be saved? (e.g. `src/components/`, `components/` for prototyping)

Only proceed with implementation once you have:
- Figma URL with node ID
- Component name (from user)
- Answers to the above (or reasonable defaults)
- Target output path

## Workflow

### Step 1: Fetch Design Context

Parse the Figma URL and call `get_design_context`:

- `fileKey`: segment after `/design/` in the URL
- `nodeId`: value of `node-id` query param (use `:` not `-` in API calls)

### Step 2: Extract Figma Dimensions (Source of Truth)

**Figma dimensions override design-system-acb defaults.** Extract from `get_design_context`:

- **Height** – Use the exact pixel height from Figma (e.g. 32px, 40px)
- **Font size** – Use the exact fontSize from Figma (e.g. 14px, 12px)
- **Font weight** – Use the exact fontWeight from Figma (e.g. 400 Regular, 600 Semibold). Do not assume Semibold (600)—buttons often use Regular (400).
- **Padding** – Use Figma padding; map to tokens only when values match token definitions
- **Line height** – Use Figma lineHeightPx when available

Design-system-acb values (40px height, 12px font, 600 weight) are **fallbacks only** when Figma does not specify.

### Step 3: Extract Variant and State Styling from Figma

**Critical:** Use the **named states** in Figma. There is a state called "Hover", "Active", "Focus", "Disabled"—use those exact nodes. Call `get_design_context` on each state's node and take its fill, stroke, and text color. Do not derive or assume.

**Do not confuse Focus with Active (selected):**
- **Focus** (Figma) = keyboard focus indicator. Use **only** for `:focus-visible` styling (outline, ring). Do NOT use `:focus`—when the user clicks with a mouse, the element receives focus and would show the focus ring, incorrectly hiding the Active (pressed) state. Use `:focus-visible` so the focus ring appears only for keyboard (Tab), not for mouse clicks.
- **Active** (Figma) = two usages: (1) **Pressed** (mouse down)—action buttons show State=Active during click; (2) **Selected** (prop-driven)—tabs/toolbar icons show State=Active when `active`/`selected` is true. Use Figma State=Active for both; never use State=Focus.

1. **Identify variants** – Use `get_metadata` on the root node to see the structure. Find frames like "Button - Primary", "Button - Secondary".
2. **Find state nodes** – For each variant, locate nodes named `State=Hover`, `State=Active`, `State=Focus`, `State=Disabled`, `State=Default`. (Figma uses this naming.)
3. **Call get_design_context on each state** – For Hover: use Hover node. For **selected** appearance: use **Active** node (not Focus). For **focus ring**: use Focus node (outline/border only), and apply via `:focus-visible` CSS, not `:focus`.
4. **Use the Hover state for hover** – Do not guess hover colors. Use the fill/stroke from the frame explicitly named "State=Hover" (or "Hover").
5. **Use the Active state for pressed and selected** – (a) For action buttons: on mouseDown, show the fill from Figma **State=Active**. (b) For tabs/toolbar: when `active` prop is true, show State=Active. Never use State=Focus for either.
6. **Extract exact Active color** – Call `get_design_context` on State=Active node. Use its fill hex (e.g. `#014486`). Do NOT assume design-system mapping (e.g. "active = brand-90")—Figma may use brand-80 (#014486) or another shade. Map the extracted hex to the matching token.
7. **Map to tokens** – After extracting: map each hex to the closest design-system token. If no token matches, add a CSS variable to tokens.css.

**When unclear** (e.g. multiple Hover nodes, ambiguous naming)—ask the user.

### Step 4: Map to Design Tokens (Colors, Semantics)

Map colors and semantics to design-system-acb tokens. Use tokens for:

| Figma Value | Map To |
|-------------|--------|
| Color | `var(--slds-g-color-...)` (neutral, brand, success, error, warning, info) |
`DEFINE YOUR SPACING, RADIUS, SHADOW, AND ICON TOKEN MAPPINGS HERE BASED ON YOUR DESIGN SYSTEM HERE.`

**Dimensions and typography (height, font size, font weight, line height):** Use Figma values directly. Add CSS variables for them if reusable (e.g. `--button-height: 32px`, `--button-font-size: 14px`, `--button-font-weight: 400`).

### Step 5: Build React Component

- Name the component using the name the user provided (e.g. `Button`, `Card`).
- Use functional components with TypeScript
- Accept `variant`, `size`, `disabled`, `children`, etc. as props
- Implement hover/focus/active states for interactive elements
- **Use Figma-extracted dimensions and typography** in the component (height, fontSize, fontWeight, padding)
- **Use Figma-extracted state colors** for each variant—Default, Hover, Active, Focus, Disabled. Extract exact hex from State=Active; do not assume brand-90 (Figma may use brand-80 or other).
- **Pressed state (mouseDown)**: Show Figma State=Active fill for action buttons.
- **Selected state**: Show Figma State=Active when `active`/`selected` prop is true.
- **Focus** = keyboard focus ring only. Use `:focus-visible` in CSS so the focus ring appears only when the user tabs to the element, not when they click (which would incorrectly show focus instead of Active pressed state). Add a class like `btn-focus-visible` with `:focus { outline: none }` and `:focus-visible { outline: 2px solid ... }`. Never use `:focus` or React state for focus styling on buttons.

### Step 6: Save Output

Save to the path the user specified, or a sensible default such as:

- `src/components/` for app-integrated projects
- `components/` for stand-alone prototyping

Components saved here are meant to be reused for further prototyping and shared via git.

## Token Reference (design-system-acb)

**Colors:**  
`--slds-g-color-neutral-base-100` (white) … `-10` (dark)  
Brand: `-60` default, `-80` active/pressed (#014486), `-90` darker, `-30` light, `--brand-base-hover` (#0b5cab)  
Note: Figma State=Active often uses #014486 (brand-80), not brand-90. Extract and verify.

**Spacing:** `spacing-1` (2px) … `spacing-8` (24px)  
**Radius:** `radius-1` (2px) … `radius-6` (16px), `radius-circle`

**Typography:** SF Pro, weights 400/600, scale from Header Large (24/30) to Label X-small (9/12)

For full token details, see `YOUR_TEAM_RULES_HERE`.

## Example: Button Component

See [examples.md](examples.md) for a full button example.

## Anti-Patterns

- Do not use hard-coded hex colors (`#0176d3`) or raw px for spacing/radius; use tokens
- Do not override Figma dimensions/typography with design-system-acb defaults—Figma is source of truth for height, font size, font weight, padding
- Do not assume hover/active state colors from design-system—extract each variant's state styling (fills, strokes, text color) from Figma. Figma State=Active is often #014486 (brand-80), not brand-90; always call get_design_context on State=Active to get the exact fill
- Do not use Figma Focus state for selected/active appearance—Focus = keyboard focus ring only; Active = selected (prop-driven). Use `:focus-visible` for focus ring, not `:focus`, so mouse clicks show Active (pressed) state instead of focus ring.
- Do not introduce new icon libraries; follow icon-system-acb priority
- Do not skip clarification when multiple variants or states exist
- Do not assume output path; ask or use a clear default

## Integration with Figma MCP

This skill works with the Figma MCP server. When the Figma plugin is connected, use `get_design_context` and `get_screenshot` to fetch design data. If the user provides "implement design" or similar, the implement-design skill may also apply; turn-to-code focuses on the "turn to code" trigger, strict token usage, and the clarification workflow above.
