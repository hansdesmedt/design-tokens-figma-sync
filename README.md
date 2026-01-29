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
# Sync to Figma
/sync-colors       # Push colors to Figma
/sync-typography   # Push text styles to Figma
/sync-docs         # Verify documentation

# Validate structure
/validate-tokens tokens.json
```

## 📋 Commands

### `/validate-tokens`
Validate token structure, naming, and best practices.

**Input:** Token JSON file (from Figma, Style Dictionary, etc.)
**Output:** Validation errors and warnings

```bash
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
Extract colors from your codebase and create/update Figma color variables with Light/Dark modes.

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
Extract text styles from components and create/update Figma text styles.

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
Verify Figma documentation tables match actual variable values and fix mismatches.

**Input:** Figma variables + documentation frames
**Output:** Updated documentation with corrected values

```
Variable:       color-text-title (Dark) = "neutral-white"
Documentation:  color-text-title (Dark) = "neutral-black" ❌
After sync:     color-text-title (Dark) = "neutral-white" ✅
```

## 🔄 Complete Workflow

### Initial Setup

```bash
# 1. Define tokens in your code
# (CSS, Tailwind, component styles)

# 2. Sync to Figma
/sync-colors
/sync-typography

# 3. Create documentation in Figma
# (Token tables showing Light/Dark values)

# 4. Verify everything matches
/sync-docs

# 5. Validate token structure (optional)
npx skills add hansdesmedt/design-tokens-validator
/review-tokens tokens.json
```

### Daily Development

```bash
# Changed colors?
/sync-colors

# Changed typography?
/sync-typography

# Want to verify docs?
/sync-docs
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
