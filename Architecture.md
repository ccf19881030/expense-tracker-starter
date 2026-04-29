# Architecture Documentation

## Project Overview

- **Project Name**: Finance Tracker
- **Type**: React Web Application
- **Build Tool**: Vite 7.x
- **React Version**: 19.x

---

## Entry Point Chain

```
index.html
    └── src/main.jsx (entry point)
            ├── createRoot() renders App
            └── src/App.jsx (root component)
                    ├── Summary
                    ├── TransactionForm
                    └── TransactionList
```

### Detailed Chain

1. **index.html** - Loads the app via `<script src="/src/main.jsx"></script>`

2. **src/main.jsx** - React entry point
   - Uses `React.StrictMode` for development checks
   - Renders `App` component into DOM element `#root`
   - Imports global styles from `./index.css`

3. **src/App.jsx** - Root component
   - Manages `transactions` state (single source of truth)
   - Passes transactions to child components via props
   - Handles transaction additions via `handleAddTransaction` callback

---

## Component Structure

```
src/
├── main.jsx          - Entry point (renders App)
├── App.jsx           - Root component (state management)
├── App.css          - Root component styles
├── Summary.jsx     - Displays income/expenses/balance
├── TransactionForm.jsx - Form to add new transactions
└── TransactionList.jsx  - Displays filtered transaction list
```

### Component Responsibilities

| Component | Responsibilities |
|----------|------------------|
| `App.jsx` | Holds transactions state, handles adding new transactions |
| `Summary.jsx` | Calculates totals from transactions prop, displays summary cards |
| `TransactionForm.jsx` | Manages form state, validates input, creates new transaction on submit |
| `TransactionList.jsx` | Manages filter state, filters transactions, displays table |

### Data Flow

```
App (state: transactions[])
    │
    ├─── transactions ──► Summary (calculates totals)
    │               │
    ├─── transactions ──► TransactionList (filters & displays)
    │
    └─── onAdd ◄───── TransactionForm (submits new transaction)
```

---

## Transaction Model

### Shape

```javascript
{
  id: number,           // Unique identifier (Date.now())
  description: string, // User-provided description
  amount: number,     // Transaction amount (always numeric)
  type: "income" | "expense",
  category: string,    // "food" | "housing" | "utilities" | "transport" | "entertainment" | "salary" | "other"
  date: string        // ISO date string (YYYY-MM-DD)
}
```

### Initial Data

```javascript
[
  { id: 1, description: "Salary", amount: 5000, type: "income", category: "salary", date: "2025-01-01" },
  { id: 2, description: "Rent", amount: 1200, type: "expense", category: "housing", date: "2025-01-02" },
  { id: 3, description: "Groceries", amount: 150, type: "expense", category: "food", date: "2025-01-03" },
  { id: 4, description: "Freelance Work", amount: 800, type: "expense", category: "salary", date: "2025-01-05" },
  { id: 5, description: "Electric Bill", amount: 95, type: "expense", category: "utilities", date: "2025-01-06" },
  { id: 6, description: "Dinner Out", amount: 65, type: "expense", category: "food", date: "2025-01-07" },
  { id: 7, description: "Gas", amount: 45, type: "expense", category: "transport", date: "2025-01-08" },
  { id: 8, description: "Netflix", amount: 15, type: "expense", category: "entertainment", date: "2025-01-10" },
]
```

### Categories

```javascript
const categories = ["food", "housing", "utilities", "transport", "entertainment", "salary", "other"];
```

### Calculations

```javascript
// Total Income: sum of all transactions where type === "income"
const totalIncome = transactions
  .filter(t => t.type === "income")
  .reduce((sum, t) => sum + t.amount, 0);

// Total Expenses: sum of all transactions where type === "expense"
const totalExpenses = transactions
  .filter(t => t.type === "expense")
  .reduce((sum, t) => sum + t.amount, 0);

// Balance
const balance = totalIncome - totalExpenses;
```

---

## ESLint Configuration

**File**: `eslint.config.js`

### Plugins

| Plugin | Purpose |
|--------|---------|
| `@eslint/js` | Core ESLint rules |
| `eslint-plugin-react-hooks` | React hooks rules (deps check) |
| `eslint-plugin-react-refresh` | Hot reload compatibility |

### Rules

```javascript
{
  rules: {
    // Error on unused variables except those starting with uppercase (React components)
    'no-unused-vars': ['error', { varsIgnorePattern: '^[A-Z_]' }],
  }
}
```

### Running ESLint

```bash
npm run lint
```

---

## Available Scripts

```bash
npm run dev      # Start development server
npm run build   # Build for production
npm run lint    # Run ESLint
npm run preview # Preview production build
```

---

## Build Configuration

**File**: `vite.config.js`

- Uses `@vitejs/plugin-react` for React Fast Refresh
- ESM module format (`"type": "module"` in package.json)

---

## Development Notes

1. **Amount Handling** - All amounts are stored as numbers, not strings. The `TransactionForm` uses `parseFloat()` to ensure numeric input.

2. **State Management** - Single source of truth in `App.jsx`. Child components receive data via props and communicate back via callbacks.

3. **Filtering** - `TransactionList` manages its own filter state locally (not synced with parent).

4. **Component Isolation** - Each component manages its own internal state; no context or global state management used.