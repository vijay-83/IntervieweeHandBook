# 02_React_Components_JSX_README.md
## Part 004 - Advanced Component Patterns (Q46–Q60)

### Q46. What is Component Composition?
**Answer:** Component composition means building complex UIs by combining smaller reusable components instead of using inheritance.

```jsx
<Card>
  <Card.Header />
  <Card.Body />
  <Card.Footer />
</Card>
```

**Benefits**
- Reusable
- Flexible
- Easier testing
- Better maintainability

---

### Q47. Composition vs Inheritance

| Composition | Inheritance |
|---|---|
| Recommended | Rarely used |
| Flexible | Rigid |
| Reusable | Less reusable |

---

### Q48. What is React.memo?
**Answer:** Prevents unnecessary re-renders when props haven't changed.

```jsx
const UserCard = React.memo(UserCard);
```

Use for expensive components after profiling.

---

### Q49. What is a Higher Order Component (HOC)?
A function that takes a component and returns an enhanced component.

```jsx
const Protected = withAuth(Dashboard);
```

Common uses:
- Authentication
- Logging
- Permissions

---

### Q50. HOC vs Custom Hooks

| HOC | Custom Hook |
|---|---|
| Wraps components | Shares logic |
| Older pattern | Preferred in modern React |

---

### Q51. What are Render Props?
Share logic by passing a function as a prop.

### Q52. What are Compound Components?
Related components sharing state, e.g. Tabs, Accordion.

### Q53. What are Headless Components?
Logic without UI. Consumer controls rendering.

### Q54. Controlled vs Uncontrolled Components
Controlled uses React state; uncontrolled uses refs/DOM.

### Q55. Container vs Presentational Components
Container = business logic.
Presentational = UI only.

### Q56. What are React Portals?
Render UI outside the normal DOM tree.

### Q57. What is Prop Drilling?
Passing props through many intermediate components.

### Q58. Context API vs Props
Use Context for shared global data like theme or auth.

### Q59. Component Design Best Practices
- Single responsibility
- Composition
- Small components
- Reusable APIs

### Q60. Senior Interview Scenario
Design an enterprise Button supporting variants, sizes, icons, loading, accessibility, theming, TypeScript and Storybook.
