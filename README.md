# Design Tokens Toolkit

Complete toolkit for design tokens: sync between codebase and Figma + validate token structure.

## 🎯 Purpose

Complete design tokens workflow:

**Sync (Code ↔ Figma):**
- **Push colors** from CSS/Tailwind to Figma variables
- **Push typography** from component styles to Figma text styles
- **Verify documentation** matches actual variable values

**Validate (Quality Check):**
- **Validate structure** - Primitive → Semantic → Responsive hierarchy
- **Check naming** - Meaningful names, no appearance-based
- **Verify modes** - Light/Dark separation, Desktop/Mobile
- **Accessibility** - 16px baseline enforcement

## 📦 Installation

```bash
npx skills add hansdesmedt/design-tokens-figma-toolkit
```

## 🚀 Quick Start

```bash
# 1. Push tokens from CODE → FIGMA
/sync-colors       # CSS/Tailwind → Figma color variables
/sync-typography   # Component styles → Figma text styles

# 2. Update Figma docs (after variables changed)
/sync-docs         # Figma variables → Figma documentation tables

# 3. Quality check (export tokens.json first)
/validate-tokens tokens.json  # Check structure & naming
```

### Commands at a Glance

| Command | Direction | What It Does |
|---------|-----------|--------------|
| `/sync-colors` | CODE → FIGMA | Push color definitions to Figma variables |
| `/sync-typography` | CODE → FIGMA | Push text styles to Figma |
| `/sync-docs` | FIGMA ↔ FIGMA | Sync Figma docs with Figma variables (after changes) |
| `/validate-tokens` | EXPORT → CHECK | Validate token structure & naming (quality control) |

## 📋 Commands

### `/validate-tokens`
Validate token structure, naming, and best practices (quality check on exported tokens).

**Direction:** EXPORT → VALIDATE (external quality check)
**Input:** Token JSON file exported from Figma (or Style Dictionary, etc.)
**Output:** Validation errors and warnings
**Use after:** Exporting tokens from Figma to ensure proper structure

```bash
# 1. Export tokens.json from Figma
# 2. Then validate:
/validate-tokens tokens.json
```

**Checks:**
- ✅ Primitive → Semantic → Responsive hierarchy
- ✅ Light/Dark mode separation (semantic-light + semantic-dark)
- ✅ Desktop/Mobile separation (responsive-desktop + responsive-mobile)
- ✅ No raw values in semantic tokens
- ✅ Meaningful naming (not appearance-based)
- ✅ 16px baseline for paragraph.md

### `/sync-colors`
Extract colors from your codebase and **push to Figma** as color variables (Light/Dark modes).

**Direction:** CODE → FIGMA
**Input:** CSS variables, Tailwind config
**Output:** Figma color variables organized by category

```css
/* Your code */
--color-primary-500: #3B82F6;

/* Creates in Figma */
Variables → Primary → primary-500
  Light: #3B82F6
  Dark: #60A5FA (auto-adjusted)
```

### `/sync-typography`
Extract text styles from components and **push to Figma** as text styles.

**Direction:** CODE → FIGMA
**Input:** Component style files, CSS
**Output:** Figma text styles with proper naming

```tsx
// Your code
'body-md': 'text-base font-normal'

// Creates in Figma
Text Styles → Body/font-body-md
  Inter Regular, 16px, 24px line height
```

### `/sync-docs`
Verify Figma documentation tables match actual **Figma variable values** and fix mismatches.

**Direction:** FIGMA VARIABLES → FIGMA DOCS (internal sync within Figma)
**Input:** Figma variables + Figma documentation frames
**Output:** Updated Figma documentation with corrected values
**Use after:** Running `/sync-colors` or `/sync-typography` to update docs

```
Figma Variable:     color-text-title (Dark) = "neutral-white"
Figma Docs Table:   color-text-title (Dark) = "neutral-black" ❌
After /sync-docs:   color-text-title (Dark) = "neutral-white" ✅
```

## 🔄 Complete Workflow

### The Flow Explained

```
YOUR CODE (CSS/Tailwind)
    ↓
[/sync-colors, /sync-typography] ← Push to Figma
    ↓
FIGMA VARIABLES & TEXT STYLES
    ↓
[/sync-docs] ← Sync Figma docs with Figma variables (after changes)
    ↓
FIGMA DOCUMENTATION (updated)
    ↓
Export tokens.json
    ↓
[/validate-tokens] ← Quality check structure
    ↓
✅ Validated token hierarchy
```

### Initial Setup

```bash
# 1. Define tokens in your code
# (CSS variables, Tailwind config, component styles)

# 2. Push tokens to Figma
/sync-colors       # CODE → FIGMA (creates color variables)
/sync-typography   # CODE → FIGMA (creates text styles)

# 3. Create documentation frames in Figma manually
# (Token tables with columns: Token | Light | Dark)

# 4. Sync Figma docs with Figma variables
/sync-docs         # FIGMA VARIABLES → FIGMA DOCS (fixes mismatches)

# 5. Export tokens.json from Figma and validate structure
/validate-tokens tokens.json  # Quality check: hierarchy, naming, 16px baseline
```

### Daily Development

```bash
# Updated colors in code?
/sync-colors       # Push changes to Figma
/sync-docs         # Update Figma documentation tables

# Updated typography in code?
/sync-typography   # Push changes to Figma
/sync-docs         # Update Figma documentation tables

# Want to validate token quality?
# Export tokens.json from Figma, then:
/validate-tokens tokens.json
```

## 📁 Project Structure

This skill expects tokens in these locations:

```
your-project/
├── src/
│   ├── app/
│   │   └── globals.css           # ← Color definitions
│   ├── components/
│   │   └── ui/
│   │       ├── text/
│   │       │   └── styles.tsx    # ← Text variants
│   │       └── heading/
│   │           └── styles.tsx    # ← Heading variants
│   └── styles/
│       └── tokens.css
└── tailwind.config.ts            # ← Tailwind colors
```

## 🎨 Token Examples

### Colors (CSS Variables)

```css
/* globals.css */
:root {
  /* Primitives */
  --color-neutral-100: #F5F5F5;
  --color-neutral-900: #171717;
  --color-primary-500: #3B82F6;

  /* Semantic (references) */
  --color-text-primary: var(--color-neutral-900);
  --color-background: var(--color-neutral-100);
}
```

### Typography (Component Styles)

```tsx
// text/styles.tsx
export const textVariants = cva('', {
  variants: {
    variant: {
      'body-sm': 'text-sm font-normal leading-5',
      'body-md': 'text-base font-normal leading-6',
      'body-lg': 'text-lg font-normal leading-7',
      'display-lg': 'text-5xl font-bold leading-tight',
    }
  }
});
```

## ✅ Best Practices

### Do's ✓

```css
/* Good: Semantic naming */
--color-text-primary: var(--color-neutral-900);
--color-surface-default: var(--color-neutral-100);
--color-interactive-primary: var(--color-primary-500);

/* Good: Proper categories */
--spacing-4: 16px;
--border-radius-md: 8px;
```

### Don'ts ✗

```css
/* Bad: Component-specific */
--color-button-blue: #3B82F6;

/* Bad: Appearance-based */
--color-light-gray: #F5F5F5;

/* Bad: Mode in name */
--color-text-dark-mode: #FFFFFF;
```

**Note:** Use the validator skill to catch these issues!

```bash
npx skills add hansdesmedt/design-tokens-validator
/review-tokens tokens.json
```

## 🔧 Requirements

### For All Commands
- figma-friend plugin installed
- Access to Figma file
- Figma file with Variables/Styles panels

### For `/sync-colors`
- CSS files with color definitions OR
- Tailwind config with color values

### For `/sync-typography`
- Component style files OR
- CSS with font definitions
- Fonts available in Figma

### For `/sync-docs`
- Figma documentation frames with token tables
- Table structure: Frame → Content → Tokens → Table → Rows

## 🤝 Works Great With

**design-tokens-validator** - Validate token structure after syncing
```bash
npx skills add hansdesmedt/design-tokens-validator

# After syncing to Figma, export tokens.json and validate:
/review-tokens tokens.json
```

This checks:
- ✅ Proper Primitive → Semantic → Responsive hierarchy
- ✅ Light/Dark mode separation
- ✅ Naming conventions
- ✅ 16px baseline for typography
- ✅ No raw values in semantic tokens

## 📊 Example Output

### `/sync-colors`
```
✅ Synced 42 color variables to Figma

Created (18):
  • neutral-50, neutral-100, neutral-200, ... neutral-950
  • primary-500, primary-600, primary-700
  • success-500, warning-500, error-500

Updated (3):
  • primary-500: #3B82F6 → #2563EB (Light)
  • neutral-900: #171717 → #0A0A0A (Dark)
  • success-500: Updated Dark mode value

Configured Light/Dark modes ✅
```

### `/sync-typography`
```
✅ Synced 12 text styles to Figma

Created (8):
  • Display/font-display-lg (Inter Bold, 48px, LH: 56px)
  • Body/font-body-md (Inter Regular, 16px, LH: 24px)
  • Label/font-label-sm (Inter Medium, 14px, LH: 20px)

Updated (4):
  • Body/font-body-lg: 18px → 20px
  • Headline/font-headline-md: Medium → Semi Bold

Categories: Display, Headline, Body, Label
```

### `/sync-docs`
```
✅ Scanned 4 variable collections
✅ Found 5 documentation frames

Fixed (12 mismatches):
✅ stroke/color-stroke-brand Dark: "primary-300" → "primary-100"
✅ background/color-background Dark: "neutral-black" → "neutral-900"
✅ text/color-text-title Dark: "neutral-black" → "neutral-white"

Summary: 12 fixes across 4 frames
All documentation now matches variables ✅
```

## 🐛 Troubleshooting

### "Font not found in Figma"
**Solution:** Install the font in Figma or update your code to use available fonts

### "Cannot find color definitions"
**Solution:** Ensure CSS files contain recognizable color variable patterns

### "Documentation frame not discovered"
**Solution:** Check your table structure matches the expected pattern (see docs)

### "Variable already exists with different type"
**Solution:** Delete the conflicting variable in Figma or rename in code

## 📖 Documentation

- [SKILL.md](./SKILL.md) - Complete skill specification
- [sync-colors.md](./commands/sync-colors.md) - Color sync details
- [sync-typography.md](./commands/sync-typography.md) - Typography sync details
- [sync-docs.md](./commands/sync-docs.md) - Documentation sync details

## 🎯 Use Cases

### New Project Setup
1. Define tokens in code
2. Sync to Figma with this skill
3. Validate structure with validator skill
4. Start designing!

### Design System Maintenance
1. Update tokens in code
2. Sync changes to Figma
3. Verify documentation matches
4. Validate structure

### Handoff to Developers
1. Design in Figma using variables
2. Export tokens
3. Validate with validator skill
4. Implement in code

## 🤝 Contributing

Issues and pull requests welcome!

## 📄 License

MIT

## 👤 Author

Hans Desmedt

---

## 🔗 Related

- [design-tokens-validator](https://github.com/hansdesmedt/design-tokens-validator) - Validate token structure and naming
