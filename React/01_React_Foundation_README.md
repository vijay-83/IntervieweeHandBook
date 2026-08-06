# 01_React_Foundation_README

## React Foundation Interview Guide

### Q1. What is React?

**Answer:** React is an open-source JavaScript library for building component-based user interfaces.

### Q2. Why React?

**Answer:** Reusable components, Virtual DOM, declarative UI, strong ecosystem.

### Q3. What is SPA?

**Answer:** A Single Page Application loads one HTML page and updates content dynamically without full page reloads.

### Q4. What is Virtual DOM?

**Answer:** A lightweight in-memory representation of the Real DOM used to efficiently update only changed elements.

### Q5. Real DOM vs Virtual DOM

| Real DOM | Virtual DOM |
|---|---|
| Slow updates | Fast diffing |
| Updates whole tree | Updates changed nodes |

### Q6. What is JSX?

**Answer:** JSX is JavaScript XML syntax that is transpiled into `React.createElement()` calls.

### Q7. Functional vs Class Components

| Functional | Class |
|---|---|
| Hooks | Lifecycle methods |
| Less boilerplate | More boilerplate |

### Q8. Props vs State

| Props | State |
|---|---|
| Read-only | Mutable |
| Passed by parent | Owned by component |

### Q9. Component Lifecycle

Mount → Update → Unmount. In functional components, `useEffect` is used to handle lifecycle effects.

### Q10. What are Hooks?

Hooks let functional components use React state and lifecycle features. Common hooks: `useState`, `useEffect`, `useMemo`, `useCallback`, `useRef`, `useContext`, `useReducer`.

> This is Chapter 1 starter. The complete series will expand this file to ~50 detailed Q&A with examples and diagrams.
