# SEABOTICS UI — TECHNICAL DOCUMENTATION

## Table of Contents
1. [Quick Start](#quick-start)
2. [Project Architecture](#project-architecture)
3. [Folder Structure & Responsibilities](#folder-structure--responsibilities)
4. [Core Concepts](#core-concepts)
5. [Key Components](#key-components)
6. [Creating a New Page](#creating-a-new-page)
7. [Color System](#color-system)
8. [State Management](#state-management)
9. [API & Data Flow](#api--data-flow)
10. [Common Issues & Fixes](#common-issues--fixes)
11. [Best Practices](#best-practices)
12. [Running & Building](#running--building)
13. [Onboarding Checklist](#onboarding-checklist)

---

## Quick Start

**Purpose:** Central reference for project architecture, component usage, theming, and onboarding.

**Tech Stack:** React + Next.js (app router) + MUI v5 + TypeScript + Formik + Redux Toolkit (optional) + react-datepicker

---

## Project Architecture

### High-Level Flow
```
User Interaction (Page Component)
    ↓
Form (Formik) + Shared Components (TextField, DatePicker, Autocomplete)
    ↓
State Management (Redux Store / Local State)
    ↓
API Call (Utils / Hooks)
    ↓
Response → Theme + Display (MUI Theme Provider)
```

### Key Design Principles
1. **Centralized Theming** — ThemeCustomizer controls global colors/mode; all components respond dynamically.
2. **Shared Components** — MUITextField, MUIDatePicker, MUIAutocomplete are single sources of truth for consistent UI.
3. **Separation of Concerns** — Containers (pages) → Presentation (components) → Business Logic (utils, hooks, redux).
4. **Type Safety** — TypeScript enforced throughout; interfaces for all major entities.

---

## Folder Structure & Responsibilities

### `src/app/` — Routes & Pages
**Purpose:** Next.js app router pages and layouts.

**Contents:**
- `page.jsx` — homepage / entry point
- `layout.jsx` — root layout (wraps all pages with providers, theme, etc.)
- `[feature]/page.jsx` — feature-specific pages (e.g., `/dashboard`, `/forms`, `/settings`)

**Responsibility:**
- Define route segments and nested layouts
- Wrap pages with providers (ThemeProvider, Redux store, etc.)
- Handle page-level data fetching or client initialization
- Import and compose shared components

**Example Structure:**
```
src/app/
├── layout.jsx              # Root layout with ThemeProvider, Redux
├── page.jsx                # Homepage
├── dashboard/
│   └── page.jsx            # Dashboard page
├── forms/
│   └── page.jsx            # Forms showcase
└── settings/
    └── page.jsx            # Settings page
```

---

### `src/components/` — Shared & Local Components
**Purpose:** Reusable UI components, organized by responsibility.

**Structure:**
```
src/components/
├── common/                 # Shared across entire app (single source of truth)
│   ├── TextField/          # MUI TextField wrapper (theme-aware, Formik-ready)
│   ├── Autocomplete/       # MUI Autocomplete wrapper (multiselect, search, etc.)
│   ├── DateTimePicker/     # Date picker suite
│   │   ├── index.tsx       # Main DatePicker component
│   │   ├── CustomDateInput.tsx  # Input adapter (uses MUITextField)
│   │   └── DatePickerWrapper.tsx # Calendar styling (react-datepicker wrapper)
│   ├── Typography/         # Custom Typography wrapper
│   ├── Button/             # Custom Button wrapper (if exists)
│   └── colors.ts           # Color palette constants & utilities
├── features/               # Feature-specific components (not shared)
│   ├── Dashboard/
│   │   ├── DashboardCard.tsx
│   │   └── StatWidget.tsx
│   └── Forms/
│       ├── LoginForm.tsx
│       └── UserForm.tsx
└── layouts/                # Layout components
    ├── Sidebar.tsx
    ├── Header.tsx
    └── Footer.tsx
```

**Responsibilities:**

- **`common/`** — Components that must be consistent across the app:
  - MUITextField, MUIDatePicker, MUIAutocomplete (use shared styling & theme logic)
  - Generic wrappers (Button, Card, Typography) if they enforce app-wide style standards
  - colors.ts exports ColorPalette type and color constants (reference, not primary)

- **`features/`** — Components specific to one feature; can be tightly coupled:
  - LoginForm uses MUITextField + custom validation
  - DashboardCard displays data in a feature-specific layout

- **`layouts/`** — Page structure components:
  - Sidebar, Header, Footer
  - Shared across multiple pages but not globally generic

---

### `src/core/` — Core Utilities & Higher-Level Components
**Purpose:** Business logic, API integrations, and non-visual utilities.

**Contents:**
```
src/core/
├── ThemeCustomizer.jsx     # Global theme color/mode picker
├── components/
│   └── apex-chart/ApexCharts.jsx # Example: chart component using theme colors
├── hooks/                  # Custom React hooks
│   ├── useTheme.ts        # Wrapper around MUI useTheme (optional)
│   ├── useFetch.ts        # Data fetching hook
│   └── useFormHandler.ts  # Formik helper hooks
├── redux/                  # Redux store, slices, actions (if using Redux)
│   ├── store.ts           # Redux store configuration
│   ├── slices/
│   │   ├── authSlice.ts   # Auth state (user, token, etc.)
│   │   ├── uiSlice.ts     # UI state (theme, modal visibility, etc.)
│   │   └── dataSlice.ts   # App data (users, items, etc.)
│   └── selectors/
│       ├── authSelectors.ts
│       └── dataSelectors.ts
├── services/               # API client & data fetching
│   ├── api.ts             # Axios or fetch client setup
│   ├── authService.ts     # Auth API calls
│   └── dataService.ts     # Generic data API calls
├── utils/                  # Utility functions (non-React)
│   ├── formatters.ts      # Date, number, string formatting
│   ├── validators.ts      # Validation helpers (e.g., email, zip)
│   └── helpers.ts         # Generic helpers (debounce, deepClone, etc.)
└── types/                  # Shared TypeScript interfaces/types
    ├── common.ts          # Common types (User, Item, etc.)
    └── api.ts             # API response/request types
```

**Responsibilities:**

- **`ThemeCustomizer.jsx`** — UI component that dispatches Redux actions to change theme palette/mode globally.

- **`hooks/`** — Custom React hooks encapsulating logic:
  - `useFetch()` — handle API calls with loading/error/data states
  - `useFormHandler()` — Formik setup helper
  - Example: `const { data, loading } = useFetch('/api/users')`

- **`redux/`** — Centralized state management (if app is complex):
  - `store.ts` — creates Redux store with slices
  - `slices/` — Redux Toolkit slices (reducers + actions)
  - `selectors/` — memoized state selectors
  - Example: `dispatch(setThemeColor('primary'))` updates global theme

- **`services/`** — API & external data communication:
  - `api.ts` — Axios instance with base URL, interceptors, error handling
  - `authService.ts` — login(), logout(), refreshToken(), etc.
  - `dataService.ts` — getUsers(), createUser(), updateUser(), etc.

- **`utils/`** — Pure utility functions (no React dependencies):
  - Date/time formatting: `formatDate(date, 'dd MMM yyyy')`
  - Validation: `isValidEmail(email)`
  - Helpers: `debounce()`, `throttle()`, `deepClone()`

- **`types/`** — Shared TypeScript definitions:
  - User, Product, Order interfaces
  - API response shapes
  - Example:
    ```tsx
    export interface User {
      id: string
      name: string
      email: string
    }
    ```

---

### `src/providers/` — App Context & Setup
**Purpose:** Providers that wrap the app for global functionality.

**Contents:**
```
src/providers/
├── ThemeProvider.jsx       # MUI CssVarsProvider + CssBaseline
└── ReduxProvider.jsx       # Redux store provider (if using Redux)
```

**Responsibilities:**

- **`ThemeProvider.jsx`** — Supplies MUI theme to entire app:
  - Wraps app with `CssVarsProvider` (handles CSS variables for dynamic theming)
  - Includes `CssBaseline` (resets default styles)
  - Called from `src/app/layout.jsx`
  - When ThemeCustomizer changes theme, this provider triggers re-renders globally

- **`ReduxProvider.jsx`** — Supplies Redux store:
  - Wraps app with `<Provider store={store}>` from react-redux
  - Optional (only if Redux is used)

**Usage in layout.jsx:**
```tsx
import ThemeProvider from '@/providers/ThemeProvider'
import ReduxProvider from '@/providers/ReduxProvider' // if using Redux

export default function RootLayout({ children }) {
  return (
    <ThemeProvider>
      <ReduxProvider>
        {children}
      </ReduxProvider>
    </ThemeProvider>
  )
}
```

---

### `src/configs/` — Configuration Files
**Purpose:** App-wide constants, environment variables, and settings.

**Contents:**
```
src/configs/
├── constants.ts            # App constants (API base URL, timeout, etc.)
├── theme.ts                # MUI theme configuration (if not in ThemeProvider)
└── environment.ts          # Environment-specific config
```

**Example — `constants.ts`:**
```tsx
export const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3000/api'
export const API_TIMEOUT = 30000 // ms
export const PAGINATION_SIZE = 20
export const DATE_FORMAT = 'dd MMM yyyy'

export const COLORS = {
  PRIMARY: '#5774F1',
  ERROR: '#E43628',
  SUCCESS: '#34B752',
}
```

**Example — `environment.ts`:**
```tsx
export const isDev = process.env.NODE_ENV === 'development'
export const isProd = process.env.NODE_ENV === 'production'
```

---

### `src/utils/` — Utility Functions
**Purpose:** Pure, reusable helper functions (no React dependencies).

**Contents:**
```
src/utils/
├── formatters.ts           # Date, number, currency formatting
├── validators.ts           # Form validators (email, URL, phone, etc.)
├── helpers.ts              # Generic helpers (debounce, clone, etc.)
└── storage.ts              # LocalStorage helpers
```

**Example — `formatters.ts`:**
```tsx
import { format } from 'date-fns'

export const formatDate = (date: Date, pattern = 'dd MMM yyyy') => {
  return format(date, pattern)
}

export const formatCurrency = (value: number, currency = 'USD') => {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(value)
}
```

**Example — `validators.ts`:**
```tsx
export const isValidEmail = (email: string): boolean => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}

export const isValidPhoneNumber = (phone: string): boolean => {
  return /^\+?[\d\s\-()]{7,}$/.test(phone)
}
```

---

### `src/hooks/` — Custom React Hooks
**Purpose:** Reusable React logic.

**Contents:**
```
src/hooks/
├── useFetch.ts             # Data fetching hook
├── useDebounce.ts          # Debounce hook
├── useLocalStorage.ts      # LocalStorage hook
└── useFormHandler.ts       # Formik setup helper
```

**Example — `useFetch.ts`:**
```tsx
import { useState, useEffect } from 'react'

export const useFetch = (url: string) => {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState(null)

  useEffect(() => {
    const fetchData = async () => {
      setLoading(true)
      try {
        const res = await fetch(url)
        setData(await res.json())
      } catch (err) {
        setError(err)
      } finally {
        setLoading(false)
      }
    }

    fetchData()
  }, [url])

  return { data, loading, error }
}
```

**Usage in component:**
```tsx
const { data, loading } = useFetch('/api/users')
```

---

### `src/services/` — API & Data Services
**Purpose:** Encapsulate all external data communication.

**Contents:**
```
src/services/
├── api.ts                  # Axios instance + interceptors
├── authService.ts          # Auth endpoints
├── userService.ts          # User CRUD operations
└── dataService.ts          # Generic data operations
```

**Example — `api.ts`:**
```tsx
import axios from 'axios'
import { API_BASE_URL, API_TIMEOUT } from '@/configs/constants'

const api = axios.create({
  baseURL: API_BASE_URL,
  timeout: API_TIMEOUT,
})

// Request interceptor (add auth token)
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('authToken')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// Response interceptor (handle errors globally)
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // redirect to login or refresh token
    }
    return Promise.reject(error)
  }
)

export default api
```

**Example — `userService.ts`:**
```tsx
import api from './api'

export const userService = {
  getUsers: () => api.get('/users'),
  getUserById: (id: string) => api.get(`/users/${id}`),
  createUser: (data) => api.post('/users', data),
  updateUser: (id: string, data) => api.put(`/users/${id}`, data),
  deleteUser: (id: string) => api.delete(`/users/${id}`),
}
```

---

### `src/types/` — TypeScript Definitions
**Purpose:** Centralized type definitions for the app.

**Contents:**
```
src/types/
├── common.ts               # User, Product, Order, etc.
├── api.ts                  # API request/response types
└── forms.ts                # Form data shapes
```

**Example — `common.ts`:**
```tsx
export interface User {
  id: string
  name: string
  email: string
  createdAt: Date
}

export interface Product {
  id: string
  title: string
  price: number
  stock: number
}
```

---

### `src/core/redux/` — Redux State Management (Optional but Recommended for Complex Apps)
**Purpose:** Centralized state management for complex apps.

**When to use:**
- App has shared state across many pages (theme, auth, notifications)
- Complex state logic or multiple dependent state updates
- Need for time-travel debugging or persistence

**When NOT to use:**
- Simple pages with Formik + local state (overkill)
- One-off features with no state sharing

**Structure:**
```
src/core/redux/
├── store.ts                # Configure Redux store
├── slices/
│   ├── authSlice.ts        # Auth state: user, token, isLoading
│   ├── uiSlice.ts          # UI state: theme, modal visibility
│   └── dataSlice.ts        # App data: users, products, etc.
└── selectors/
    ├── authSelectors.ts    # useSelector(selectUser), etc.
    └── dataSelectors.ts
```

**Example — `store.ts`:**
```tsx
import { configureStore } from '@reduxjs/toolkit'
import authSlice from './slices/authSlice'
import uiSlice from './slices/uiSlice'

export const store = configureStore({
  reducer: {
    auth: authSlice,
    ui: uiSlice,
  },
})

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

**Example — `authSlice.ts`:**
```tsx
import { createSlice } from '@reduxjs/toolkit'

interface AuthState {
  user: { id: string; name: string; email: string } | null
  isLoading: boolean
  error: string | null
}

const initialState: AuthState = { user: null, isLoading: false, error: null }

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    setUser: (state, action) => {
      state.user = action.payload
    },
    logout: (state) => {
      state.user = null
    },
  },
})

export const { setUser, logout } = authSlice.actions
export default authSlice.reducer
```

**Usage in component:**
```tsx
import { useSelector, useDispatch } from 'react-redux'
import { selectUser } from '@/core/redux/selectors/authSelectors'
import { logout } from '@/core/redux/slices/authSlice'

export function Profile() {
  const user = useSelector(selectUser)
  const dispatch = useDispatch()

  return (
    <div>
      {user?.name}
      <button onClick={() => dispatch(logout())}>Logout</button>
    </div>
  )
}
```

---

## Core Concepts

### 1. Theme-First Styling
- **Always** use `useTheme()` and `theme.palette.*` in components.
- **Never** hardcode colors; use theme values to enable dynamic updates via ThemeCustomizer.
- Prefer `styled()`, `sx` prop, or `useTheme()` over `@mui/styles makeStyles`.

**Example — Correct:**
```tsx
const MyComponent = () => {
  const theme = useTheme()
  return <Box sx={{ color: theme.palette.primary.main }}>Text</Box>
}
```

**Example — Wrong:**
```tsx
// ❌ Hardcoded color
const MyComponent = () => <Box sx={{ color: '#5774F1' }}>Text</Box>
```

---

### 2. Shared Components = Consistency
- **MUITextField** (`src/components/common/TextField/index.tsx`) is the canonical input.
- All specialized inputs (DatePicker, Autocomplete) delegate to MUITextField.
- This ensures focus/border/adornment styling is uniform across the app.

---

### 3. Color Resolution at Render Time
- Resolve colors **inside components** during render, not at module init.
- ThemeCustomizer updates `theme.palette.primary.main` globally; components re-render and pick up the new color.

```tsx
const resolveColor = (colorProp: string, theme: Theme) => {
  if (colorProp === 'primary') return theme.palette.primary.main
  if (colorProp === 'secondary') return theme.palette.secondary.main
  if (colorProp?.startsWith('#') || colorProp?.startsWith('rgb')) return colorProp
  return theme.palette.primary.main // default
}
```

---

### 4. Formik Integration Pattern
- Always pass `field` and `form` props to shared inputs.
- Use `getIn()` from Formik to safely access nested touched/errors.
- Call `form.setFieldValue()` when handling custom input changes.

```tsx
<MUITextField
  field={{ name: 'email', value: values.email }}
  form={{ touched, errors, setFieldValue }}
/>
```

---

### 5. API & Data Flow
1. **Service layer** (services/api.ts) handles HTTP requests.
2. **Redux slices** (optional) or **local state** holds fetched data.
3. **Components** consume data via `useSelector()` (Redux) or `useState()` (local).
4. **Error handling** is centralized in API interceptors.

---

## Key Components

### MUITextField
**Location:** `src/components/common/TextField/index.tsx`

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
**Location:** `src/components/common/DateTimePicker/CustomDateInput.tsx`

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
**Location:** `src/components/common/DateTimePicker/DatePickerWrapper.tsx`

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
**Location:** `src/components/common/Autocomplete/index.tsx`

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
**Location:** `src/components/common/DateTimePicker/index.tsx`

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
src/app/my-feature/page.jsx
```

### Step 2: Add Formik Setup
```tsx
'use client'
import { Formik, Form } from 'formik'
import * as z from 'zod'
import { toFormikValidationSchema } from 'zod-formik-adapter'

export default function MyFeaturePage() {
  const validationSchema = z.object({
    name: z.string().min(2, 'Required'),
    startDate: z.date().nullable(),
    selectedItems: z.array(z.object({ label: z.string(), value: z.string() })),
  })

  const onSubmit = async (data) => {
    try {
      const response = await userService.createUser(data) // call service
      console.log('Success:', response.data)
    } catch (error) {
      console.error('Error:', error)
    }
  }

  return (
    <Formik
      initialValues={{ name: '', startDate: null, selectedItems: [] }}
      validationSchema={toFormikValidationSchema(validationSchema)}
      onSubmit={onSubmit}
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
import MUITextField from '@/components/common/TextField'
import MUIDatePicker from '@/components/common/DateTimePicker'
import MUIAutocomplete from '@/components/common/Autocomplete'

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

### Step 4: (Optional) Connect to Redux
```tsx
import { useSelector, useDispatch } from 'react-redux'
import { selectData } from '@/core/redux/selectors/dataSelectors'
import { fetchData } from '@/core/redux/slices/dataSlice'

export default function MyFeaturePage() {
  const dispatch = useDispatch()
  const data = useSelector(selectData)

  useEffect(() => {
    dispatch(fetchData()) // async thunk
  }, [dispatch])

  // Render data...
}
```

### Step 5: Theme & Test
- Colors automatically adapt to ThemeCustomizer changes.
- No hardcoding needed.
- Test by toggling theme color/mode in ThemeCustomizer.

---

## Color System

### How It Works
1. **ThemeCustomizer** (src/core/ThemeCustomizer.jsx) updates `theme.palette.primary.main` (or other palette values).
2. MUI's `CssVarsProvider` propagates the change via CSS variables.
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
**Location:** `src/components/common/colors.ts`

Exports `ColorPalette` type and light color constants. Use **only for static fallbacks**; prefer `theme.palette` in components.

```tsx
// Do NOT import as default
import { getColor } from '../colors' // ✓ Correct (named import)

// Prefer this instead (in modern components)
const theme = useTheme()
const color = theme.palette.primary.main // ✓ Best practice
```

---

## State Management

### Local State (Simple Forms)
**When to use:** Single-page forms, one-off features, no state sharing.

```tsx
const [data, setData] = useState(null)
const [loading, setLoading] = useState(false)

useEffect(() => {
  setLoading(true)
  fetch('/api/data').then((res) => setData(res.json())).finally(() => setLoading(false))
}, [])
```

### Redux (Complex Apps)
**When to use:** Shared state across multiple pages, complex logic, time-travel debugging.

**Setup:**
1. Define slice in `src/core/redux/slices/mySlice.ts`
2. Add to store in `src/core/redux/store.ts`
3. Use `useSelector()` and `useDispatch()` in components

```tsx
const data = useSelector(selectMyData)
dispatch(fetchMyData()) // async thunk
```

---

## API & Data Flow

### Service Layer Example
```tsx
// src/core/services/userService.ts
import api from './api'

export const userService = {
  getUsers: () => api.get('/users'),
  createUser: (data) => api.post('/users', data),
  updateUser: (id: string, data) => api.put(`/users/${id}`, data),
}
```

### Using Services in Components
```tsx
import { userService } from '@/core/services/userService'

const onSubmit = async (formData) => {
  try {
    const response = await userService.createUser(formData)
    console.log('Created:', response.data)
  } catch (error) {
    console.error('Error:', error)
  }
}
```

### Using with Redux (Async Thunks)
```tsx
// src/core/redux/slices/userSlice.ts
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit'
import { userService } from '@/core/services/userService'

export const fetchUsers = createAsyncThunk('users/fetchUsers', async () => {
  const response = await userService.getUsers()
  return response.data
})

const userSlice = createSlice({
  name: 'users',
  initialState: { items: [], status: 'idle' },
  extraReducers: (builder) => {
    builder.addCase(fetchUsers.fulfilled, (state, action) => {
      state.items = action.payload
      state.status = 'success'
    })
  },
})
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
| Redux state not updating | Reducer not defined for action | Check `store.ts` includes the slice; check reducer name |
| Formik validation not showing | `touched` not set before form blur | Ensure `<Form>` from Formik wraps inputs; fields auto-set `touched` on blur |

---

## Best Practices

1. **Always use shared components** (MUITextField, MUIDatePicker, MUIAutocomplete) for consistency.
2. **Resolve colors at render time** with `useTheme()` — never hardcode hex values.
3. **Use `sx` prop or `styled()`** for styling; avoid `@mui/styles makeStyles` for new components.
4. **Integrate Formik correctly**: pass `field` and `form` props so validation/touched states display.
5. **Centralize API calls** in `src/core/services/`; don't fetch directly in components.
6. **Use Redux for shared state**; keep local state for isolated forms.
7. **Test theme changes**: toggle ThemeCustomizer to verify colors update dynamically.
8. **Keep components thin**: delegate visual styling to MUITextField or DatePickerWrapper; keep custom logic minimal.
9. **Use type-safe APIs**: define request/response types in `src/types/api.ts`.
10. **Document complex logic**: add JSDoc comments for non-obvious business logic.

---

## Running & Building

```bash
# Install dependencies
npm install

# Start dev server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Start production server
npm start

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
- [ ] Read `src/components/common/TextField/index.tsx` — understand shared input pattern
- [ ] Review `src/core/` folder structure — understand services, hooks, utils
- [ ] Check `src/core/redux/` if Redux is used — understand state management
- [ ] Open DevTools → toggle ThemeCustomizer → verify colors update globally
- [ ] Create a test page with MUITextField + MUIDatePicker + MUIAutocomplete + Formik
- [ ] Call a service endpoint to fetch data and display in a component
- [ ] Change theme primary color and confirm all components update
- [ ] Read this documentation end-to-end for reference

---

## Quick Reference Links

- [MUI Theme Customization](https://mui.com/material-ui/customization/theming/)
- [MUI useTheme Hook](https://mui.com/material-ui/api/use-theme/)
- [Formik Docs](https://formik.org/)
- [Redux Toolkit Docs](https://redux-toolkit.js.org/)
- [react-datepicker](https://reactdatepicker.com/)
- [colord (Color Manipulation)](https://github.com/omgovich/colord)
- [Axios Docs](https://axios-http.com/)

---

## Support & Questions

Contact project owner or senior dev for:
- Architecture decisions and rationale
- Theme/color system deep dives
- Component extension patterns
- Form handling best practices
- Redux store design
- API integration guidelines

---

**Last Updated:** November 2025  
**Version:** 1.0  
**Status:** Active & Maintained
