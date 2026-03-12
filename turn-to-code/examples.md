# Turn to Code — Examples

## Example: Figma Button → React Component

**Input:** User shares a Figma button URL and says "turn to code".

**From `get_design_context`:** Height 32px, fontSize 14px, fontWeight 400 (Regular), padding 6px 12px (extract exact values from Figma).

**Output:** Reusable React button using Figma dimensions + design tokens for colors.

```tsx
// src/components/Button.tsx
// IMPORTANT: Use Figma-extracted dimensions (height, fontSize) — not design-system defaults

const sizeStyles: Record<ButtonSize, { height: string; padding: string; fontSize: string; fontWeight: number }> = {
  medium: {
    height: '32px',      // from Figma
    padding: '6px 12px', // from Figma
    fontSize: '14px',    // from Figma
    fontWeight: 400,     // from Figma (Regular); do not assume 600
  },
  // ...
};

export function Button({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  onClick,
  type = 'button',
}: ButtonProps) {
  const sizeStyle = sizeStyles[size];

  return (
    <button
      type={type}
      disabled={disabled}
      onClick={onClick}
      style={{
        fontFamily: "'SF Pro', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif",
        fontWeight: sizeStyle.fontWeight,
        lineHeight: 1.5,
        padding: sizeStyle.padding,
        fontSize: sizeStyle.fontSize,
        minHeight: sizeStyle.height,  // use Figma height, not 40px default
        color: variant === 'secondary' ? 'var(--slds-g-color-neutral-base-10)' : 'white',
        backgroundColor: variantStyles[variant],
        border: variant === 'secondary' ? '1px solid var(--slds-g-color-neutral-base-80)' : 'none',
        borderRadius: 'var(--slds-g-radius-2)',
        cursor: disabled ? 'not-allowed' : 'pointer',
      }}
      onMouseEnter={(e) => {
        if (!disabled && variant === 'primary') {
          e.currentTarget.style.backgroundColor = 'var(--slds-g-color-brand-base-30)'; // extract from Figma Hover state
        }
      }}
      onMouseLeave={(e) => {
        if (!disabled && variant === 'primary') {
          e.currentTarget.style.backgroundColor = 'var(--slds-g-color-brand-base-60)';
        }
      }}
    >
      {children}
    </button>
  );
}
```

**Usage:**
```tsx
<Button variant="primary">Save</Button>
<Button variant="secondary" size="small">Cancel</Button>
<Button variant="destructive" disabled>Delete</Button>
```

---

**State extraction:** For each variant, call `get_design_context` on the Hover and Active frame node IDs to get exact fill/stroke values. Map those to tokens. Do not assume design-system convention—Figma State=Active is often #014486 (brand-80), not brand-90.

**Pressed state:** On mouseDown, show Figma State=Active fill. Extract the exact hex from get_design_context.

**Focus vs Active:** Use Figma **Active** for pressed (mouseDown) and selected (prop). Use Figma **Focus** only for the keyboard focus ring. Implement focus ring with `:focus-visible` CSS, not `:focus`—otherwise mouse clicks show the focus border instead of the Active (pressed) state.

**Note:** When CSS variables are available in the project, prefer using a CSS module or styled approach and referencing the same tokens. The inline `style` above is for illustration; adapt to your project’s styling setup.
