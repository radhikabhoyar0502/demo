# SEABOTICS UI — TECHNICAL DOCUMENTATION

## Quick Start

**Purpose:** Central reference for project architecture, component usage, theming, and onboarding.

**Tech Stack:** React + Next.js (app router) + MUI v5 + TypeScript + Formik + react-datepicker

---

## Project Structure

```
d:\SeaboticsUI\
├── src/
│   ├── app/                    # Routes & entry points
│   ├── core/                   # Core utilities & high-level components
│   ├── components/common/      # Shared components (single source of truth)
│   │   ├── TextField/          # MUITextField wrapper
│   │   ├── Autocomplete/       # Customized MUI Autocomplete
│   │   ├── DateTimePicker/     # Date picker (DatePickerWrapper + CustomDateInput)
│   │   └── colors.ts           # Color palette & helpers
│   └── providers/
│       └── ThemeProvider.jsx   # MUI theme provider (required at root)
└── core/
    └── ThemeCustomizer.jsx     # Global theme color/mode customizer
```

---

## Core Concepts

### 1. Theme-First Styling
- **Always** use `useTheme()` and `theme.palette.*` in components.
- **Never** hardcode colors; use theme values to enable dynamic updates via ThemeCustomizer.
- Prefer `styled()`, `sx` prop, or `useTheme()` over `@mui/styles makeStyles`.

### 2. Shared Components = Consistency
- **MUITextField** (`src/Components/common/TextField/index.tsx`) is the canonical input.
- All specialized inputs (DatePicker, Autocomplete) delegate to MUITextField.
- This ensures focus/border/adornment styling is uniform across the app.

### 3. Color Resolution at Render Time
- Resolve colors **inside components** during render, not at module init.
- ThemeCustomizer updates `theme.palette.primary.main` globally; components re-render and pick up the new color.
- Example: `const primaryColor = color === 'primary' ? theme.palette.primary.main : color`

---

## Key Components

### MUITextField
**Location:** `src/Components/common/TextField/index.tsx`

**Purpose:** Unified text input wrapper with theme-aware styling.

**Key Props:**
- `variant`: 'outlined' | 'standard' | 'filled'
- `size`: 'small' | 'medium'
- `color`: string (theme color key or hex)
- `StartIcon`, `EndIcon`: optional icon components
- `inputStartAdornmentStyles`: custom styles for icon background
- `field`, `form`: Formik integration
- `isError`, `errorMsg`: error state

**Usage:**
```tsx
<MUITextField
  label="Name"
  variant="outlined"
  size="small"
  color="primary"
  field={formikField}
  form={formikForm}
/>
```

---

### CustomDateInput
**Location:** `src/Components/common/DateTimePicker/CustomDateInput.tsx`

**Purpose:** Input wrapper for DatePicker; thin adapter around MUITextField with date-specific behavior.

**Key Behaviors:**
- Shows DateRangeIcon with colored background (adornment).
- Resolves `color` prop to theme color at render time.
- Focused border color changes to resolved primary color.
- Adornment height adapts to size (`small` vs `medium`).
- Displays formatted date or count of selected dates.

**Key Props:**
- `color`: theme color key or hex (e.g., 'primary', '#5774F1')
- `size`: 'small' | 'medium'
- `variant`: 'outlined' | 'standard' | 'filled'
- `date`: Date | null (displays formatted)
- `dates`: Date[] (displays count)
- `field`, `form`: Formik integration
- `disabled`, `required`, `isError`, `errorMsg`

**Usage:**
```tsx
<CustomDateInput
  label="Select date"
  size="medium"
  color="primary"
  date={selectedDate}
  field={field}
  form={form}
  onClick={() => setOpen(true)}
/>
```

---

### DatePickerWrapper
**Location:** `src/Components/common/DateTimePicker/DatePickerWrapper.tsx`

**Purpose:** Styled Box that customizes react-datepicker calendar visuals (header, selected day, ranges).

**Key Features:**
- Uses `styled(Box)` to inject theme-aware CSS into react-datepicker classes.
- Header background: `theme.palette.primary.main`
- Selected day: primary color + contrast-aware text.
- Hover/range states: primary color with opacity.
- Scrollbars: primary color (custom webkit).

**No direct props needed** — wraps react-datepicker element:
```tsx
<DatePickerWrapper>
  <ReactDatePicker ... />
</DatePickerWrapper>
```

---

### MUIAutocomplete
**Location:** `src/Components/common/Autocomplete/index.tsx`

**Purpose:** Customized MUI Autocomplete with multiselect, search, checkboxes, and theme-aware styling.

**Key Props:**
- `type`: 'normal' | 'multiselect' | 'search'
- `color`: theme color key or hex
- `data`: AutocompleteOption[]
- `dataStructure`: { labelApiParam: string, valueApiParam: string | number }
- `getDataQuery`, `searchDataQuery`: custom query hooks
- `StartIcon`, `EndIcon`: optional icons
- `LabelComponent`: custom option renderer
- `field`, `form`: Formik integration
- `multiselectArray`: selected items (multiselect only)
- `generateEndContent`: custom right-side content renderer

**Usage:**
```tsx
<MUIAutocomplete
  type="normal"
  color="primary"
  data={options}
  dataStructure={{ labelApiParam: 'name', valueApiParam: 'id' }}
  label="Choose item"
  field={field}
  form={form}
  onChange={(value) => form.setFieldValue('fieldName', value?.value)}
/>
```

---

### DatePickerIndex (MUIDatePicker)
**Location:** `src/Components/common/DateTimePicker/index.tsx`

**Purpose:** Main entry point; orchestrates react-datepicker + CustomDateInput + DatePickerWrapper.

**Key Props:**
- `type`: 'date' | 'range' | 'leave' | 'month' | 'year' | 'quarter'
- `color`: theme color key or hex
- `variant`: 'outlined' | 'standard' | 'filled'
- `size`: 'small' | 'medium'
- `events`: CustomEvent[] (mark holidays, applied leave, etc.)
- `field`, `form`: Formik integration

**Usage:**
```tsx
<MUIDatePicker
  type="date"
  color="primary"
  size="medium"
  label="Start date"
  field={field}
  form={form}
/>
```

---

## Creating a New Page

### Step 1: Create Route
```
src/app/my-page/page.jsx
```

### Step 2: Add Formik Setup
```tsx
'use client'
import { Formik, Form } from 'formik'
import * as z from 'zod'
import { toFormikValidationSchema } from 'zod-formik-adapter'

export default function MyPage() {
  const validationSchema = z.object({
    name: z.string().min(2, 'Required'),
    startDate: z.date().nullable(),
    selectedItems: z.array(z.object({ label: z.string(), value: z.string() })),
  })

  return (
    <Formik
      initialValues={{ name: '', startDate: null, selectedItems: [] }}
      validationSchema={toFormikValidationSchema(validationSchema)}
      onSubmit={(data) => console.log(data)}
    >
      {({ values, errors, touched, setFieldValue }) => (
        <Form>
          {/* Use shared components here */}
        </Form>
      )}
    </Formik>
  )
}
```

### Step 3: Add Shared Components
```tsx
import MUITextField from '@/Components/common/TextField'
import MUIDatePicker from '@/Components/common/DateTimePicker'
import MUIAutocomplete from '@/Components/common/Autocomplete'

// Inside your form:
<MUITextField
  label="Full Name"
  field={{ name: 'name', value: values.name }}
  form={{ touched, errors, setFieldValue }}
/>

<MUIDatePicker
  type="date"
  label="Start Date"
  field={{ name: 'startDate', value: values.startDate }}
  form={{ touched, errors, setFieldValue }}
/>

<MUIAutocomplete
  type="normal"
  data={items}
  dataStructure={{ labelApiParam: 'label', valueApiParam: 'value' }}
  field={{ name: 'selectedItems', value: values.selectedItems }}
  form={{ touched, errors, setFieldValue }}
  onChange={(value) => setFieldValue('selectedItems', value)}
/>
```

### Step 4: Theme & Test
- Colors automatically adapt to ThemeCustomizer changes.
- No hardcoding needed.
- Test by toggling theme color/mode in ThemeCustomizer.

---

## Color System

### How It Works
1. **ThemeCustomizer** (core/ThemeCustomizer.jsx) updates `theme.palette.primary.main` (or other palette values).
2. MUI's `CssVarsProvider` propagates the change.
3. Components use `useTheme()` at render time to pick up the new color.
4. On focus/hover, borders and backgrounds update to match.

### Using Colors in Components
```tsx
const theme = useTheme()

// Resolve color prop to actual color value
const resolveColor = (colorProp) => {
  if (colorProp === 'primary') return theme.palette.primary.main
  if (colorProp === 'secondary') return theme.palette.secondary.main
  if (colorProp?.startsWith('#') || colorProp?.startsWith('rgb')) return colorProp
  return theme.palette.primary.main // default fallback
}

// Use in styling
const primaryColor = resolveColor(color)
const sx = {
  '& .MuiOutlinedInput-root.Mui-focused fieldset': {
    borderColor: primaryColor,
  },
}
```

### colors.ts (Reference, Not Required)
**Location:** `src/Components/common/colors.ts`

Exports `ColorPalette` type and light color constants. Use **only for static fallbacks**; prefer `theme.palette` in components.

```tsx
// Do NOT import as default
import { getColor } from '../colors' // ✓ Correct (named import)

// Prefer this instead (in modern components)
const theme = useTheme()
const color = theme.palette.primary.main // ✓ Best practice
```

---

## Common Issues & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| "Export default doesn't exist" | `import getColor from '../colors'` | Use named import: `import { getColor }` or use `theme.palette` |
| "styles argument invalid" / theme undefined | Component rendered before ThemeProvider mounted | Ensure ThemeProvider wraps root of app (check `src/providers/ThemeProvider.jsx`) |
| Adornment icon not visible / wrong size | Height/alignment not adapting to size prop | Use `height: '100%'`, `minHeight`, `alignSelf: 'stretch'` in adornment styles |
| Colors not updating on ThemeCustomizer change | Hardcoded colors or colors read at module init | Call `useTheme()` inside component (render time), not at module top-level |
| Autocomplete options not showing custom color | Color prop not resolved to theme value | Use `theme.palette[color]` or parse hex; pass to styled component |

---

## Best Practices

1. **Always use shared components** (MUITextField, MUIDatePicker, MUIAutocomplete) for consistency.
2. **Resolve colors at render time** with `useTheme()` — never hardcode hex values.
3. **Use `sx` prop or `styled()`** for styling; avoid `@mui/styles makeStyles` for new components.
4. **Integrate Formik correctly**: pass `field` and `form` props so validation/touched states display.
5. **Test theme changes**: toggle ThemeCustomizer to verify colors update dynamically.
6. **Keep components thin**: delegate visual styling to MUITextField or DatePickerWrapper; keep custom logic minimal.

---

## Running & Building

```bash
# Install dependencies
npm install

# Start dev server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Lint & format
npm run lint
npm run format

# Fix lint issues automatically
npx eslint --fix src --ext .ts,.tsx,.js,.jsx
```

---

## Onboarding Checklist (New Devs)

- [ ] Clone repo, run `npm install && npm run dev`
- [ ] Open `src/providers/ThemeProvider.jsx` — understand MUI theme setup
- [ ] Read `src/Components/common/TextField/index.tsx` — understand shared input pattern
- [ ] Open DevTools → toggle ThemeCustomizer → verify colors update globally
- [ ] Create a test page with MUITextField + MUIDatePicker + MUIAutocomplete
- [ ] Change theme primary color and confirm all components update
- [ ] Review a form in `src/app/` to see Formik + shared component integration

---

## Quick Reference Links

- [MUI Theme Customization](https://mui.com/material-ui/customization/theming/)
- [MUI useTheme Hook](https://mui.com/material-ui/api/use-theme/)
- [Formik Docs](https://formik.org/)
- [react-datepicker](https://reactdatepicker.com/)
- [colord (Color Manipulation)](https://github.com/omgovich/colord)

---

## Support & Questions

Contact project owner or senior dev for:
- Architecture decisions
- Theme/color system deep dives
- Component extension patterns
- Form handling best practices

---

**Last Updated:** November 2025  
**Version:** 1.0  
**Status:** Active
