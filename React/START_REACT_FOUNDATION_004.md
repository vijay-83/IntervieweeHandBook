# START_REACT_FOUNDATION_004

# React Components, Props, State & Lifecycle (Q21–Q30)

## Q21. What is a React component?
A reusable, independent UI building block that accepts props and returns JSX.

## Q22. Types of components
- Functional Components (recommended)
- Class Components (legacy)

## Q23. What makes a component reusable?
- Configurable via props
- Single responsibility
- No duplicated logic

## Q24. What are props?
Read-only inputs passed from parent to child components.

## Q25. Can props be modified?
No. Props are immutable.

## Q26. What is state?
Component-owned mutable data that triggers re-render when updated.

## Q27. Props vs State
| Props | State |
|---|---|
| Parent owned | Component owned |
| Immutable | Mutable |
| External input | Internal data |

## Q28. What triggers a re-render?
- State updates
- Prop changes
- Context updates
- Parent re-render

## Q29. React lifecycle in functional components
Mount → Update → Unmount, typically managed with useEffect.

## Q30. Best practices
- Keep components small
- Lift state only when necessary
- Prefer composition over inheritance
- Avoid unnecessary state
