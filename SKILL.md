# Design Tokens Figma Sync

Bidirectional synchronization between your codebase and Figma design tokens.

## How It Works

This skill provides commands to sync design tokens between your app codebase and Figma:
- **Push colors** from code to Figma variables (Light/Dark modes)
- **Push typography** from code to Figma text styles
- **Verify Figma docs** match actual variable values
- **Export tokens** from Figma to JSON for validation

## Requirements

- **figma-friend plugin** installed and connected
- Access to your Figma file
- Figma file with variables/styles setup

## Commands

### `/sync-colors`

Extract color variables from codebase and sync to Figma.

**What it does:**
1. Scans codebase for color definitions (CSS variables, Tailwind config, etc.)
2. Organizes into categories: Neutral, Primary, Success, Warning, Error
3. Creates/updates Figma color variables with Light/Dark modes

**Usage:**
```
/sync-colors
```

**Input sources:**
- `src/app/globals.css` - CSS custom properties
- `tailwind.config.ts` - Tailwind color configuration
- Any CSS/SCSS files with color definitions

**Example:**
```css
/* globals.css */
--color-neutral-100: #F5F5F5;
--color-primary-500: #3B82F6;
```

**Creates in Figma:**
```
Variables → Colors
├── Neutral
│   └── neutral-100 (Light: #F5F5F5, Dark: #262626)
└── Primary
    └── primary-500 (Light: #3B82F6, Dark: #60A5FA)
```

**Output:**
```
✅ Created 12 color variables
✅ Updated 5 existing variables
✅ Configured Light/Dark modes
```

### `/sync-typography`

Extract text styles from codebase and sync to Figma text styles.

**What it does:**
1. Analyzes typography usage in components
2. Identifies semantic categories and size variants
3. Creates/updates Figma text styles following pattern: `{Category}/font-{category}-{size}`

**Usage:**
```
/sync-typography
```

**Input sources:**
- `src/components/ui/text/styles.tsx` - Text component variants
- `src/components/ui/heading/styles.tsx` - Heading variants
- `src/app/globals.css` - Font definitions
- `tailwind.config.ts` - Typography config

**Example:**
```tsx
// text/styles.tsx
export const textVariants = {
  body: {
    md: { fontSize: '16px', fontWeight: 400, lineHeight: '24px' }
  },
  display: {
    lg: { fontSize: '48px', fontWeight: 700, lineHeight: '56px' }
  }
}
```

**Creates in Figma:**
```
Text Styles
├── Body/font-body-md
│   ├── Family: Inter
│   ├── Weight: Regular (400)
│   ├── Size: 16px
│   └── Line Height: 24px
└── Display/font-display-lg
    ├── Family: Inter
    ├── Weight: Bold (700)
    ├── Size: 48px
    └── Line Height: 56px
```

**Output:**
```
✅ Created 8 text styles
✅ Updated 3 existing styles
✅ Categories: Display, Body, Label, Caption
```

### `/sync-docs`

Verify and update Figma documentation frames to match actual variable values.

**What it does:**
1. Reads all Figma variable collections
2. Auto-discovers documentation frames containing token tables
3. Compares displayed values with actual variable values
4. Updates any mismatches in Light/Dark columns

**Usage:**
```
/sync-docs
```

**Auto-discovers frames with:**
- Frames containing "Table" or "Tokens" children
- Rows with token names matching variable patterns
- Tables with Light/Dark mode columns

**Example mismatch:**
```
Figma Variable:  color-text-title (Dark) = "neutral-white"
Documentation:   color-text-title (Dark) = "neutral-black" ❌
```

**After sync:**
```
✅ color-text-title Dark: "neutral-black" → "neutral-white"
```

**Output:**
```
✅ Scanned 4 variable collections
✅ Found 5 documentation frames
✅ Fixed 12 mismatches:
   - stroke/color-stroke-brand Dark: "primary-300" → "primary-100"
   - background/color-background Dark: "neutral-black" → "neutral-900"
   - text/color-text-title Dark: "neutral-black" → "neutral-white"
```

## Complete Workflow

### Initial Setup (New Project)

```bash
# 1. Define tokens in your code
# (CSS variables, Tailwind config, component styles)

# 2. Push to Figma
/sync-colors       # Creates Figma color variables
/sync-typography   # Creates Figma text styles

# 3. Create documentation in Figma
# (Add token table frames manually)

# 4. Verify documentation matches
/sync-docs         # Checks and fixes any mismatches
```

### Daily Development Workflow

```bash
# Changed colors in code?
/sync-colors       # Update Figma variables

# Changed typography?
/sync-typography   # Update Figma text styles

# Want to verify everything is in sync?
/sync-docs         # Check documentation accuracy
```

### Token Validation Workflow

After syncing to Figma, validate your token structure:

```bash
# 1. Export tokens from Figma (manually or with export command)
# 2. Validate structure with companion skill:
npx skills add hansdesmedt/design-tokens-validator
/review-tokens tokens.json

# This checks:
# - Proper Primitive → Semantic → Responsive hierarchy
# - Light/Dark mode separation
# - Naming conventions
# - Accessibility constraints (16px baseline)
```

## Token Structure Best Practices

This skill syncs tokens **as-is** from your codebase. For proper token structure, follow these guidelines:

### ✅ Good Token Setup

```css
/* Primitives - Raw values */
--color-blue-500: #3B82F6;
--spacing-4: 16px;

/* Semantic - References with meaning */
--color-text-primary: var(--color-blue-500);
--color-interactive-primary: var(--color-blue-500);
```

### ❌ Bad Token Setup

```css
/* Avoid component-specific names */
--color-button-blue: #3B82F6;  /* Too specific */

/* Avoid appearance-based names */
--color-light-gray: #F5F5F5;   /* Describes appearance, not usage */

/* Avoid mode in name */
--color-text-dark-mode: #FFF;  /* Use separate mode collections instead */
```

**Note:** The validator skill will catch these issues and suggest fixes!

## File Structure Expectations

The skill looks for tokens in these locations (customizable):

```
your-project/
├── src/
│   ├── app/
│   │   └── globals.css           # Color definitions
│   ├── components/
│   │   └── ui/
│   │       ├── text/
│   │       │   └── styles.tsx    # Text variants
│   │       └── heading/
│   │           └── styles.tsx    # Heading variants
│   └── styles/
│       └── tokens.css            # Additional token definitions
└── tailwind.config.ts            # Tailwind configuration
```

## Figma Setup Requirements

### For `/sync-colors`
- Figma file with **Variables** panel accessible
- Ability to create/edit variable collections

### For `/sync-typography`
- Fonts used in codebase must be available in Figma
- Ability to create/edit text styles

### For `/sync-docs`
- Documentation frames with table structure:
  - Frame → Content → Tokens → Table → Row[]
  - Each row: [Token Name, Light Value, Dark Value]
  - Header row with "Token", "Light", "Dark" labels

## Common Issues

### "Font not found in Figma"
**Solution:** Install missing fonts in Figma or map to available fonts

### "Cannot find color definitions"
**Solution:** Ensure CSS files contain color variables with recognizable patterns

### "Documentation frame not auto-discovered"
**Solution:** Check table structure matches expected pattern (see Figma Setup Requirements)

### "Variable already exists but with different type"
**Solution:** Delete conflicting variable in Figma or rename in code

## Related Skills

- **design-tokens-validator** - Validate token structure and naming conventions
  - Install: `npx skills add hansdesmedt/design-tokens-validator`
  - Use after syncing to verify proper hierarchy and best practices

## Output Format

All commands provide clear, actionable output:

```
✅ Success messages with counts
⚠️  Warnings for potential issues
❌ Errors with fix suggestions
📊 Summary statistics
```

## Notes

- This skill performs **active synchronization** (modifies Figma)
- Always review changes in Figma after syncing
- For **validation only** (no modifications), use `design-tokens-validator`
- Supports both single-direction (code → Figma) and verification (Figma docs ↔ Figma variables)
- Does not export Figma → Code (use Figma's built-in export or plugins)

## When to Use This Skill

Use this skill when:
- User asks to "sync tokens to Figma"
- User mentions "update Figma colors/typography"
- User wants to "verify Figma documentation"
- User provides codebase with design token definitions
- User needs to keep Figma in sync with code
