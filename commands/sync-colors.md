# Sync Colors to Figma

Extract color variables from the app codebase and sync them to the Figma design file.

## Usage

When you run `/sync-colors`, the assistant will:

1. Analyze color usage across the app codebase
2. Identify color categories and their variants
3. Create or update color variables in Figma with Light/Dark modes

Example: `/sync-colors`

## Color Categories

- **Neutral** - Grayscale values
- **Primary** - Brand primary colors
- **Success** - Success states
- **Warning** - Warning states
- **Error** - Error states

## Naming Pattern

Follow this pattern: `{category}-{variant}`

Examples: `neutral-100`, `primary-500`, `success-200`, `warning-400`, `error-600`

## File Sources

Search these files for color definitions:
- `src/app/globals.css` - CSS custom properties
- `tailwind.config.ts` - Tailwind color configuration
- `src/styles/*.css` - Additional stylesheets
- Component style files with inline color definitions

## Color Value Formats

Supports these formats:
- Hex: `#3B82F6`, `#fff`
- RGB: `rgb(59, 130, 246)`
- HSL: `hsl(217, 91%, 60%)`
- CSS Variables: `var(--color-primary-500)`

## Dark Mode Handling

For each color, determine Dark mode value:
- **Neutral colors:** Invert the scale (100 ↔ 900, 200 ↔ 800, etc.)
- **Brand colors:** Adjust for dark backgrounds (usually lighter variants)
- **Semantic colors:** Map to appropriate dark-friendly values

Example:
```css
/* Light Mode */
--color-neutral-100: #F5F5F5;  /* Very light gray */
--color-neutral-900: #171717;  /* Very dark gray */

/* Dark Mode (inverted) */
--color-neutral-100: #262626;  /* Dark gray */
--color-neutral-900: #FAFAFA;  /* Light gray */
```

## Instructions

1. **Scan Codebase**
   - Read CSS files for color variable definitions
   - Extract Tailwind config colors
   - Identify color patterns and naming conventions

2. **Organize Colors**
   ```javascript
   const colors = {
     neutral: { 100: '#F5F5F5', 200: '#E5E5E5', ... },
     primary: { 500: '#3B82F6', 600: '#2563EB', ... },
     success: { 500: '#10B981', ... },
     warning: { 500: '#F59E0B', ... },
     error: { 500: '#EF4444', ... }
   };
   ```

3. **Map Dark Mode Values**
   - Apply inversion rules for neutrals
   - Adjust brand colors for dark backgrounds
   - Ensure adequate contrast

4. **Connect to Figma**
   - Use figma-friend plugin
   - Verify `figma.variables` API is available

5. **Create/Update Variables**
   ```javascript
   // For each color:
   const collection = figma.variables.getLocalVariableCollections()
     .find(c => c.name === 'Colors');

   const variable = figma.variables.createVariable(
     `${category}-${variant}`,
     collection,
     'COLOR'
   );

   // Set Light mode value
   variable.setValueForMode(lightModeId, { r, g, b });

   // Set Dark mode value
   variable.setValueForMode(darkModeId, { r, g, b });
   ```

6. **Report Results**
   - List created variables
   - List updated variables
   - Show any skipped or conflicting colors

## Output Format

```
✅ Synced 42 color variables to Figma

Created:
  • neutral-50, neutral-100, neutral-200, ...
  • primary-500, primary-600, primary-700
  • success-500, warning-500, error-500

Updated:
  • primary-500: #3B82F6 → #2563EB (Light)
  • neutral-900: #171717 → #0A0A0A (Dark)

Configured Light/Dark modes ✅
```

## Example

**Input (globals.css):**
```css
:root {
  --color-neutral-100: #F5F5F5;
  --color-neutral-900: #171717;
  --color-primary-500: #3B82F6;
  --color-success-500: #10B981;
}
```

**Output (Figma Variables):**
```
Variables → Colors
├── Neutral
│   ├── neutral-100 (Light: #F5F5F5, Dark: #262626)
│   └── neutral-900 (Light: #171717, Dark: #FAFAFA)
├── Primary
│   └── primary-500 (Light: #3B82F6, Dark: #60A5FA)
└── Success
    └── success-500 (Light: #10B981, Dark: #34D399)
```

## Edge Cases

- **Duplicate definitions:** Use the last defined value
- **Invalid color values:** Skip and report warning
- **Missing Dark mode:** Generate based on Light mode with inversion rules
- **Existing variables:** Update values, preserve any references
