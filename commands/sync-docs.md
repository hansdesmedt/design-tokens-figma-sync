# Sync Figma Token Documentation

Verify and update all token documentation frames in Figma to match the actual variable values.

## Usage

When you run `/sync-docs`, the assistant will:

1. Connect to the Figma file using figma-friend
2. Read all variable collections (discover dynamically)
3. Extract the actual Light and Dark mode values from each variable
4. Auto-discover all documentation frames containing token tables
5. Update any mismatches to ensure documentation is accurate

Example: `/sync-docs`

## How It Works

### 1. Discovery Phase
- Scan all variable collections in the Figma file
- Build a lookup table of all variables with their Light/Dark mode values
- Auto-discover documentation frames by looking for:
  - Frames containing a "Table" or "Tokens" child
  - Frames with rows containing token names matching variable patterns
  - Any frame structure that follows the token table pattern

### 2. Token Table Pattern Recognition

The tool recognizes token tables with this structure:
- **Frame** → **Content** → **Tokens** → **Table** → **Row[]**
- Each Row contains cells: `[Token, Separator?, Light, Separator?, Dark]`
- Token names start with patterns like `color-`, `spacing-`, `border-radius-`, etc.

### 3. Value Extraction

For each variable:
- Extract the token name (last part after `/`)
- For color variables: Show the primitive reference name (e.g., `neutral-200`)
- For numeric variables: Show the value with unit (e.g., `8px`, `12`)
- Handle both single-mode and multi-mode collections

### 4. Update Process

For each documentation frame found:
1. Identify all rows in the table
2. Extract token name from first cell
3. Look up expected values from variable collections
4. Compare displayed values with actual variable values
5. Update mismatches (Light and Dark columns separately)
6. Load necessary fonts before text updates

## Instructions

1. **Connect to Figma**
   - Use figma-friend plugin to access the Figma file
   - Verify `figma` global object is available

2. **Get All Variables**
   ```javascript
   const collections = figma.variables.getLocalVariableCollections();

   // For each collection and each mode:
   for (const collection of collections) {
     const variables = collection.variableIds.map(id =>
       figma.variables.getVariableById(id)
     );

     for (const variable of variables) {
       // Extract values for each mode (Light/Dark)
       const lightValue = variable.valuesByMode[lightModeId];
       const darkValue = variable.valuesByMode[darkModeId];

       // Convert to readable format
       if (typeof lightValue === 'object' && 'id' in lightValue) {
         // It's an alias - get the referenced variable name
         const refVariable = figma.variables.getVariableById(lightValue.id);
         lightValueString = refVariable.name;
       } else {
         // It's a literal value
         lightValueString = formatValue(lightValue);
       }
     }
   }
   ```

3. **Auto-Discover Documentation Frames**
   ```javascript
   const documentationFrames = [];

   for (const node of figma.currentPage.children) {
     if (node.type === 'FRAME') {
       // Look for nested table structure
       const hasTable = findChild(node, child =>
         child.name.toLowerCase().includes('table') ||
         child.name.toLowerCase().includes('tokens')
       );

       if (hasTable) {
         documentationFrames.push(node);
       }
     }
   }
   ```

4. **Update Each Table**
   ```javascript
   for (const frame of documentationFrames) {
     const table = findTableNode(frame);
     const rows = table.children.filter(child =>
       child.name.toLowerCase().includes('row')
     );

     // Skip header row
     const dataRows = rows.slice(1);

     for (const row of dataRows) {
       const cells = row.children;
       const tokenName = getTextContent(cells[0]);
       const displayedLight = getTextContent(cells[2]);
       const displayedDark = getTextContent(cells[4]);

       // Look up actual values
       const variable = lookupVariable(tokenName);
       const actualLight = getVariableValue(variable, lightModeId);
       const actualDark = getVariableValue(variable, darkModeId);

       // Update if mismatch
       if (displayedLight !== actualLight) {
         updateTextContent(cells[2], actualLight);
         console.log(`✅ ${frame.name}/${tokenName} Light: "${displayedLight}" → "${actualLight}"`);
       }

       if (displayedDark !== actualDark) {
         updateTextContent(cells[4], actualDark);
         console.log(`✅ ${frame.name}/${tokenName} Dark: "${displayedDark}" → "${actualDark}"`);
       }
     }
   }
   ```

5. **Handle Edge Cases**
   - Skip header rows (usually first row with "Token", "Light", "Dark")
   - Skip separator elements (lines between rows)
   - Handle frames with or without column separators
   - Support both primitive and semantic token tables
   - Handle variables with single mode vs multiple modes

## Output Format

Report summary with:
- Total collections scanned
- Total documentation frames found
- Number of mismatches fixed per frame
- List of all updates made: `✅ {frame}/{token} {mode}: "{old}" → "{new}"`
- Any variables without documentation
- Any documentation without corresponding variables

## Example Output

```
✅ Scanned 4 variable collections: Primitives, Colors, Border Radius, Spacing
✅ Found 5 documentation frames: background, stroke, icon, text, button

Fixed mismatches:
✅ stroke/color-stroke-brand Dark: "primary-300" → "primary-100"
✅ background/color-background Dark: "neutral-black" → "neutral-900"
✅ background/color-background-subtle Light: "neutral-50" → "neutral-100"
✅ text/color-text-title Dark: "neutral-black" → "neutral-white"
✅ text/color-text-body Light: "neutral-800" → "neutral-900"
✅ button/color-button-primary-bg Light: "primary-500" → "primary-600"

Summary: 21 fixes across 4 frames
```

## Table Structure Detection

### Expected Pattern
```
Frame: "Color Documentation"
└── Frame: "Content"
    └── Frame: "Tokens"
        └── Frame: "Table"
            ├── Frame: "Row" (Header)
            │   ├── Text: "Token"
            │   ├── Line: "Separator"
            │   ├── Text: "Light"
            │   ├── Line: "Separator"
            │   └── Text: "Dark"
            ├── Frame: "Row 1"
            │   ├── Text: "color-text-primary"
            │   ├── Line: "Separator"
            │   ├── Text: "neutral-900"
            │   ├── Line: "Separator"
            │   └── Text: "neutral-100"
            └── Frame: "Row 2"
                └── ...
```

### Flexible Detection
The tool should also handle:
- Different nesting depths
- Alternative naming (e.g., "Documentation", "Token Table")
- Missing separators
- Different cell arrangements

## Value Formatting

### Color Variables (Aliases)
```javascript
// If variable references another variable (alias)
if (value.type === 'VARIABLE_ALIAS') {
  const referenced = figma.variables.getVariableById(value.id);
  return referenced.name; // e.g., "neutral-900"
}
```

### Color Variables (Literals)
```javascript
// If variable has direct RGB value
if (value.r !== undefined) {
  const hex = rgbToHex(value.r, value.g, value.b);
  return hex; // e.g., "#171717"
}
```

### Numeric Variables
```javascript
// Numbers with units
if (typeof value === 'number') {
  // Check if it's a spacing token
  if (variableName.includes('spacing') || variableName.includes('size')) {
    return `${value}px`; // e.g., "16px"
  }
  return String(value); // e.g., "12"
}
```

## Font Loading

Before updating text content:
```javascript
// Load fonts used in documentation tables
await figma.loadFontAsync({ family: 'Inter', style: 'Regular' });
await figma.loadFontAsync({ family: 'Inter', style: 'Medium' });
```

## Error Handling

```javascript
try {
  // Update text
  textNode.characters = newValue;
} catch (error) {
  if (error.message.includes('font')) {
    console.warn(`⚠️  Font not loaded for ${textNode.name}`);
    await figma.loadFontAsync(textNode.fontName);
    textNode.characters = newValue;
  } else {
    console.error(`❌ Failed to update ${textNode.name}: ${error.message}`);
  }
}
```

## Edge Cases

- **Variable not found in collections:** Report as documentation without variable
- **Documentation row without variable:** Report as orphaned documentation
- **Multiple modes beyond Light/Dark:** Only sync Light and Dark columns
- **Nested variable aliases:** Resolve to final value or primitive name
- **Text node locked:** Skip and report warning
- **Mixed format tables:** Attempt to detect pattern, warn if unsuccessful

## Performance Optimization

- Cache variable lookups in a Map for faster access
- Batch font loading at the start
- Process tables in parallel where possible
- Only update text nodes that have actual changes

## Validation

After sync, verify:
- All updated values are valid references or literals
- No text nodes left with placeholder values
- All variable collections have corresponding documentation
- Documentation structure is preserved (no broken layouts)
