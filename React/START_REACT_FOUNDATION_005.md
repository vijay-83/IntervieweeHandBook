
# START_REACT_FOUNDATION_005

# React Hooks Foundation (Q31–Q40)

## Q31. What are Hooks?
Hooks are special React functions that let functional components use state, lifecycle features, context, refs, and other React capabilities without class components.

---

## Q32. What is useState?

```jsx
const [count, setCount] = useState(0);
```

- Stores local component state
- Updating state triggers a re-render
- State updates are scheduled by React

---

## Q33. How does useState work internally?

React associates each Hook with a component instance and preserves Hook order between renders. This is why Hooks must always be called at the top level.

---

## Q34. What is useEffect?

```jsx
useEffect(() => {
  document.title = "Dashboard";
}, []);
```

Used for:
- API calls
- Timers
- Event listeners
- Logging
- Subscriptions

---

## Q35. Dependency Array

| Syntax | Behavior |
|---------|----------|
| useEffect(fn) | Runs after every render |
| useEffect(fn, []) | Runs once after mount |
| useEffect(fn, [value]) | Runs when value changes |

---

## Q36. Cleanup Function

```jsx
useEffect(() => {
  const id = setInterval(() => {}, 1000);

  return () => clearInterval(id);
}, []);
```

Used to prevent memory leaks.

---

## Q37. What is useRef?

- Stores mutable values
- Does not trigger re-render
- Can reference DOM elements

---

## Q38. What is useMemo?

Caches expensive computed values.

```jsx
const total = useMemo(() => calculate(items), [items]);
```

---

## Q39. What is useCallback?

Caches function references.

```jsx
const handleClick = useCallback(() => {
  save();
}, []);
```

Useful with React.memo.

---

## Q40. Rules of Hooks

1. Call Hooks only at the top level.
2. Never call Hooks inside loops or conditions.
3. Call Hooks only from React components or custom Hooks.

## Interview Tip

Know when **not** to use `useMemo` or `useCallback`; unnecessary memoization can make code more complex without improving performance.
