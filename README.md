SEABOTICS UI — TECHNICAL DOCUMENTATION
=====================================

Purpose
-------
This document provides a concise technical guide to the SeaboticsUI codebase for new engineers. It covers project structure, key components, theming and color system, common usage patterns, build/run instructions, troubleshooting tips and contribution guidelines. Use this as the single-source quick reference when onboarding.

Quick project summary
---------------------
- React + Next (app) style project using MUI v5 and Ant Design (small use).
- Core idea: central theme & color customization (ThemeCustomizer) that updates UI and components globally (charts, datepickers, autocomplete, etc).
- Centralized shared components (TextField, Autocomplete, DateTimePicker) that must respect theme and colors.

Tech stack
----------
- React (client components)
- Next.js (app router)
- Material UI (v5)
- @mui/styles used sparingly; prefer styled/useTheme/sx
- colord / date-fns / react-datepicker
- formik for forms
- lodash
- antd Tooltip (used in some components)
- Typescript (typing conventions used throughout)

Repository layout (relevant)
---------------------------
d:\SeaboticsUI\
- src/
  - app/               -> pages / layout / route handlers (entry points)
  - core/              -> core utilities & higher level components (charts, theme bits)
  - Components/
    - common/
      - TextField/     -> shared MUITextField wrapper (single source of truth for input styling)
      - Autocomplete/  -> shared Autocomplete component (customized MUI Autocomplete)
      - DateTimePicker/
        - index.tsx    -> main entry for date/time pickers
        - CustomDateInput.tsx -> input wrapper used by datepickers (uses shared MUITextField)
        - DatePickerWrapper.tsx -> styled wrapper for react-datepicker (theme-aware)
      - colors.ts      -> centralized color palette / helpers
  - providers/
    - ThemeProvider.jsx -> wraps app with MUI theme (CssVarsProvider/CssBaseline, etc)
  - core/
    - ThemeCustomizer.jsx -> component that lets users change primary color / mode / skin

Key concepts & patterns
-----------------------
1. Theme-first styling
   - Components should use MUI theme where possible (useTheme(), styled(), sx).
   - Avoid hardcoded colors. Use theme.palette.* or a color utility that maps keys to theme values.
   - Use CSS variables (where available) or theme.palette values to enable dynamic updates.

2. Shared inputs
   - MUITextField (src/Components/common/TextField/index.tsx) is the canonical TextField wrapper.
   - All specialized inputs (Date pickers, Autocomplete) should use MUITextField so input focus/border/adornment styling remains consistent.

3. Date & Autocomplete components
   - DateTimePicker index.tsx wires react-datepicker with a CustomDateInput for display and a DatePickerWrapper styled container to customize calendar visuals.
   - Autocomplete uses MUI Autocomplete and custom renderers for tags/options. It receives color prop but should resolve this to theme palette values.

Important files & how they interact
----------------------------------
- src/Components/common/colors.ts
  - Exports named colors (lightColors) and ColorPalette type.
  - Do not import it as a default export (use named imports).
  - Prefer using theme.palette in client components. colors.ts is useful for static fallbacks.

- src/providers/ThemeProvider.jsx
  - Wraps the app with the MUI provider. Must exist at root; otherwise useTheme() and styled callbacks will fail.
  - If you see "styles argument provided is invalid" or "theme undefined", verify ThemeProvider is mounted above the component.

- src/Components/common/DateTimePicker/CustomDateInput.tsx
  - Uses MUITextField. Important behaviors:
    - Resolves incoming color prop to actual color value (theme.palette.primary.main when color is 'primary').
    - Passes a StartIcon (DateRangeIcon) and inputStartAdornmentStyles so the icon shows with a colored background.
    - Uses sx to set focused border color to the resolved primary color:
      '& .MuiOutlinedInput-root.Mui-focused fieldset': { borderColor: primaryColor }
    - Ensure size handling: for 'medium' the adornment should stretch to full height (use 100%/minHeight/alignSelf: 'stretch').

- src/Components/common/DateTimePicker/DatePickerWrapper.tsx
  - Styled Box that targets react-datepicker classes to apply theme-based colors (header background, selected day, ranges).
  - Use theme.palette.primary.main + colord for opacity manipulation.

- src/Components/common/Autocomplete/index.tsx
  - Should use useTheme() and theme.palette to derive colors for selected options, chips, chips background, scrollbar, etc.

CustomDateInput — Component Documentation
========================================

Location
--------
src/Components/common/DateTimePicker/CustomDateInput.tsx

Purpose
-------
CustomDateInput is the input wrapper used by the project's DatePicker components. It is a thin, theme-aware adapter around the shared MUITextField that:
- shows a date icon with a colored background (adornment),
- resolves a passed color key or hex to a concrete color using the current MUI theme,
- adapts appearance for size/variant (small/medium, outlined/filled/standard),
- integrates with Formik field/form props,
- sets the focused outline border to the resolved global color.

Behavior summary
----------------
- StartIcon: DateRangeIcon is shown on the left inside a colored box (unless variant === 'standard', where the box is transparent).
- Adornment background color and focused border color are resolved at render-time from the theme so ThemeCustomizer changes are reflected immediately.
- Size/variant affect the height of the adornment; the adornment will stretch to fill the input height for `medium`.
- Displays a formatted date string if `date` prop provided; displays count if `dates` array is provided.
- Integrates with Formik to show touched/errors and set error text.

Props
-----
(All props are optional unless noted otherwise)

- size: 'small' | 'medium' — visual size of the input (affects adornment height).
  - Default: 'small'

- variant: 'outlined' | 'standard' | 'filled' — MUI TextField variant.
  - Default: 'outlined'

- color: string — color specifier for adornment and focused border. Accepted values:
  - 'primary' | 'secondary' | other theme color keys or a literal CSS color (e.g. '#5774F1' or 'rgb(…)').
  - If omitted, defaults to theme.palette.primary.main.

- value: string | number — controlled display value for the input. If provided, displayed directly.

- label: string — TextField label.

- placeholder: string — input placeholder.

- disabled: boolean — disables the input and changes adornment color to theme.palette.action.disabled.

- fullWidth: boolean — passes through to MUITextField.

- field: { name: string, value: any } — Formik field object (optional). When provided, CustomDateInput uses this to bind to form state.

- form: { touched: Record<string, boolean>, errors: Record<string,string>, setFieldValue: Function } — Formik form bag (optional). Used to display validation state and set field value.

- required: boolean — mark field required.

- isError: boolean — manual error flag.

- errorMsg: string — manual helper/error text.

- inputRef: React.Ref<HTMLInputElement> — forwarded ref to the input element.

- onClick: () => void — click handler (forwarded via MUITextField props).

- date: Date | null — if provided, displayed formatted as 'dd MMM yyyy' (example: '21 Sep 2025').

- dates: Date[] — if provided, input displays "{n} dates selected".

Notes about color resolution
---------------------------
- The component resolves the `color` prop at render-time using the MUI theme (useTheme()).
  - If color === 'primary' → uses theme.palette.primary.main
  - If color === 'secondary' → uses theme.palette.secondary.main
  - If color starts with '#' or 'rgb' → used as-is
  - Otherwise, fallback uses the provided string (useful for custom color values)

- The focused input border is set using sx:
  '& .MuiOutlinedInput-root.Mui-focused fieldset': { borderColor: primaryColor }

Adornment sizing details
------------------------
- small (default): adornment height ≈ 44px
- medium: adornment height ≈ 55.5px (stretches to container height)
- filled/standard: smaller heights (approx 48px); for standard, adornment background becomes transparent

Formik integration
-----------------
- If `field` & `form` are provided:
  - `hasError` is computed using getIn(form.touched, field.name) && getIn(form.errors, field.name) or isError prop.
  - Error message displayed from getIn(form.errors, field.name) or errorMsg
  - Important: the DatePicker parent should call form.setFieldValue when date selection changes.

Accessibility
-------------
- Label is forwarded to MUITextField.
- StartIcon is decorative — if you need it to be announced, wrap with aria-label or provide accessible name via props.

Examples
--------

1) Simple usage (uncontrolled display)
````tsx
// usage example (not a file change)
import CustomDateInput from 'src/Components/common/DateTimePicker/CustomDateInput'

<CustomDateInput
  label="Start date"
  placeholder="Select date"
  date={selectedDate}
  onClick={() => setOpen(true)}
/>

````

2) Using in Formik controlled form
   ````tsx
   import { Formik, Form } from 'formik'
import CustomDateInput from 'src/Components/common/DateTimePicker/CustomDateInput'
import DatePicker from 'src/Components/common/DateTimePicker'

<Formik initialValues={{ startDate: null }} onSubmit={...}>
  {({ values, setFieldValue, errors, touched }) => (
    <Form>
      <DatePicker
        field={{ name: 'startDate', value: values.startDate }}
        form={{ touched, errors, setFieldValue }}
        variant="outlined"
        size="medium"
      />
    </Form>
  )}
</Formik>
````

Running the project
-------------------
- Install dependencies:
  npm install
- Start dev:
  npm run dev
- Build:
  npm run build
- Clear caches (if encountering stale build errors):
  rm -rf .next
  npm run build

Linting and formatting
----------------------
- ESLint & Prettier configs are in the repo. Run:
  npm run lint
  npm run format
- Common linter autofix:
  npx eslint --fix src --ext .ts,.tsx,.js,.jsx

Common issues & fixes
---------------------
1. "Export default doesn't exist in target module" for colors import
   - Cause: importing default from colors.ts which only has named exports.
   - Fix: change import getColor from '../colors' to named import:
     import { getColor } from '../colors' or use theme.palette.

2. "styles argument provided is invalid" / theme undefined
   - Cause: makeStyles / styled function expects theme in context but ThemeProvider not mounted above.
   - Fix: Ensure ThemeProvider (MUI provider) wraps the component tree. Prefer using styled/useTheme instead of @mui/styles makeStyles.

3. Adornment icon background not visible / wrong size
   - Cause: height/align not matching input root when size changes.
   - Fix: set adornment styles to height: '100%', minHeight per size, alignSelf: 'stretch', and ensure MUITextField propagates StartIcon styling.

4. Colors not updating on ThemeCustomizer change
   - Cause: component uses hardcoded colors or reads color at module-init instead of render.
   - Fix: resolve colors inside component using useTheme() so re-renders pick up CSS variables or palette updates.

Best practices & contribution guidelines
----------------------------------------
- All new inputs should use MUITextField for consistent look & global color behavior.
- Prefer theme.palette values and use useTheme() inside components.
- Avoid @mui/styles makeStyles for new components; use styled(), sx prop or Emotion/CSS-in-JS that consume theme properly.
- When adding new visual behavior that depends on primary color, resolve color at render time and use theme.palette or CSS variables to enable runtime updates.
- Add unit tests for complex logic (e.g., getSelectedItem in Autocomplete) and integration tests for forms if possible.

Onboarding checklist for new devs
---------------------------------
1. Clone repo, npm install, npm run dev.
2. Locate ThemeProvider at src/providers/ThemeProvider.jsx — ensure understanding of how theme is supplied.
3. Read src/Components/common/TextField/index.tsx to understand shared input behavior.
4. Run the app and toggle ThemeCustomizer to observe global color changes.
5. Make a small tweak in DatePickerWrapper (e.g., header background) to confirm styling pipeline works.

Contact and further reading
---------------------------
- Project owner / senior dev (check repo README or team wiki for contacts)
- MUI theme docs: https://mui.com/material-ui/customization/theming/
- colord: https://github.com/omgovich/colord
- react-datepicker docs for class names used in DatePickerWrapper

END OF DOCUMENT
