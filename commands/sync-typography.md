# Sync Typography to Figma

Extract text styles from the app codebase and sync them to the Figma design file.

## Usage

When you run `/sync-typography`, the assistant will:

1. Analyze typography usage across the app codebase
2. Identify text style patterns and semantic groups
3. Create or update text styles in Figma following the pattern: `{Category}/font-{category}-{size}`

Example: `/sync-typography`

## Key Files to Analyze

- `src/components/ui/text/styles.tsx` - Text component variants
- `src/components/ui/heading/styles.tsx` - Heading component variants
- `src/app/globals.css` - Font definitions and CSS custom properties
- `tailwind.config.ts` - Tailwind typography configuration

## Text Style Requirements

Each text style should specify:
- Font family
- Font weight (Regular, Medium, Semi Bold, Bold)
- Font size
- Line height
- Letter spacing (if applicable)

## Naming Pattern

Follow this pattern: `{Category}/font-{category}-{size}`

- The weight is applied to the style but NOT included in the name
- Categories: Display, Headline, Title, Body, Label, Button, Navigation, Caption
- Sizes: xs, sm, md, lg, xl, 2xl, 3xl, 4xl

**Examples:**
- `Display/font-display-lg` (48px, Bold)
- `Body/font-body-md` (16px, Regular)
- `Label/font-label-sm` (14px, Medium)

## Instructions

1. **Scan Component Styles**
   ```tsx
   // text/styles.tsx
   export const textVariants = cva('...', {
     variants: {
       size: {
         sm: 'text-sm',
         md: 'text-base',
         lg: 'text-lg'
       },
       weight: {
         normal: 'font-normal',
         medium: 'font-medium',
         semibold: 'font-semibold',
         bold: 'font-bold'
       }
     }
   });
   ```

2. **Extract Font Definitions**
   ```css
   /* globals.css */
   @font-face {
     font-family: 'Inter';
     font-weight: 400;
     src: url('/fonts/Inter-Regular.woff2');
   }
   ```

3. **Map Tailwind Classes to Values**
   ```javascript
   const sizeMap = {
     'text-sm': '14px',
     'text-base': '16px',
     'text-lg': '18px',
     'text-xl': '20px',
     // ...
   };

   const weightMap = {
     'font-normal': 400,
     'font-medium': 500,
     'font-semibold': 600,
     'font-bold': 700
   };
   ```

4. **Group by Semantic Category**
   - **Display:** Large headings (32px+)
   - **Headline:** Section headings (24px-32px)
   - **Title:** Subsection headings (18px-24px)
   - **Body:** Paragraph text (14px-18px)
   - **Label:** Form labels, captions (12px-14px)
   - **Caption:** Small text (10px-12px)
   - **Button:** Button text (14px-16px)

5. **Connect to Figma**
   ```javascript
   // Use figma-friend plugin
   const textStyle = figma.createTextStyle();
   textStyle.name = `${category}/font-${category}-${size}`;
   textStyle.fontName = { family: 'Inter', style: 'Regular' };
   textStyle.fontSize = 16;
   textStyle.lineHeight = { value: 24, unit: 'PIXELS' };
   textStyle.letterSpacing = { value: 0, unit: 'PIXELS' };
   ```

6. **Handle Font Loading**
   - Load required fonts before creating styles
   - Map weight values to Figma font styles:
     - 400 → 'Regular'
     - 500 → 'Medium'
     - 600 → 'Semi Bold'
     - 700 → 'Bold'

7. **Report Results**
   - List created text styles
   - List updated text styles
   - Show any missing fonts or errors

## Output Format

```
✅ Synced 12 text styles to Figma

Created:
  • Display/font-display-lg (Inter Bold, 48px, LH: 56px)
  • Display/font-display-md (Inter Bold, 36px, LH: 44px)
  • Body/font-body-md (Inter Regular, 16px, LH: 24px)
  • Label/font-label-sm (Inter Medium, 14px, LH: 20px)

Updated:
  • Body/font-body-lg: 18px → 20px
  • Headline/font-headline-md: Medium → Semi Bold

Categories: Display, Headline, Body, Label, Caption
```

## Example Mapping

**Input (text/styles.tsx):**
```tsx
export const textVariants = cva('', {
  variants: {
    variant: {
      'body-md': 'text-base font-normal leading-6',
      'display-lg': 'text-5xl font-bold leading-tight',
      'label-sm': 'text-sm font-medium leading-5'
    }
  }
});
```

**Output (Figma Text Styles):**
```
Text Styles
├── Body/font-body-md
│   ├── Family: Inter
│   ├── Style: Regular (400)
│   ├── Size: 16px
│   └── Line Height: 24px
├── Display/font-display-lg
│   ├── Family: Inter
│   ├── Style: Bold (700)
│   ├── Size: 48px
│   └── Line Height: 56px
└── Label/font-label-sm
    ├── Family: Inter
    ├── Style: Medium (500)
    ├── Size: 14px
    └── Line Height: 20px
```

## Tailwind Class Mappings

### Font Sizes
- `text-xs`: 12px
- `text-sm`: 14px
- `text-base`: 16px
- `text-lg`: 18px
- `text-xl`: 20px
- `text-2xl`: 24px
- `text-3xl`: 30px
- `text-4xl`: 36px
- `text-5xl`: 48px

### Font Weights
- `font-normal`: 400 (Regular)
- `font-medium`: 500 (Medium)
- `font-semibold`: 600 (Semi Bold)
- `font-bold`: 700 (Bold)

### Line Heights
- `leading-none`: 1
- `leading-tight`: 1.25
- `leading-snug`: 1.375
- `leading-normal`: 1.5
- `leading-relaxed`: 1.625
- `leading-loose`: 2

## Edge Cases

- **Font not available in Figma:** Report warning, skip style
- **Duplicate style names:** Update existing style
- **Invalid font weight mapping:** Default to Regular
- **Missing line height:** Calculate as 1.5× font size
- **Custom fonts:** Ensure fonts are installed in Figma first

## Font Loading Example

```javascript
// Before creating text styles
await Promise.all([
  figma.loadFontAsync({ family: 'Inter', style: 'Regular' }),
  figma.loadFontAsync({ family: 'Inter', style: 'Medium' }),
  figma.loadFontAsync({ family: 'Inter', style: 'Semi Bold' }),
  figma.loadFontAsync({ family: 'Inter', style: 'Bold' })
]);
```
