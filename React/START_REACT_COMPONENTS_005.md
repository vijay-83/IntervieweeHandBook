
# START_REACT_COMPONENTS_005
## Enterprise Component Design (Q61–Q75)

### Q61. What is Atomic Design?
Atomic Design organizes UI into Atoms, Molecules, Organisms, Templates, and Pages to improve reuse and consistency.

### Q62. What is a Design System?
A collection of reusable components, design guidelines, themes, and accessibility standards used across applications.

### Q63. What is Storybook?
Storybook is a tool to develop, document, and test UI components in isolation.

### Q64. How do you build reusable components?
- Single responsibility
- Configurable props
- Composition
- TypeScript interfaces
- Documentation

### Q65. Why is Accessibility important?
Use semantic HTML, keyboard navigation, ARIA attributes, and proper color contrast.

### Q66. How do you implement theming?
Common approaches include Context API, CSS Variables, Tailwind themes, and ThemeProvider.

### Q67. Enterprise Component Library Structure
components/, hooks/, utils/, theme/, icons/, tests/, docs/.

### Q68. Performance Best Practices
Use React.memo, lazy loading, code splitting, stable keys, and profile before optimizing.

### Q69. How do you test components?
Use Jest, React Testing Library, and Cypress for unit, integration, and E2E tests.

### Q70. What are Design Tokens?
Reusable values such as colors, spacing, typography, and border radius shared across applications.

### Q71. Semantic Versioning
Version packages using Major.Minor.Patch.

### Q72. CI/CD for Component Libraries
Run linting, tests, build, Storybook validation, and automated package publishing.

### Q73. Common Enterprise Mistakes
Avoid duplicate components, poor naming, missing accessibility, and business logic inside UI components.

### Q74. Senior Interview Scenario
Design a reusable Button supporting variants, sizes, icons, loading, disabled state, accessibility, and theming.

### Q75. Architect Question
How would you build a shared component library for multiple teams? Discuss governance, versioning, documentation, testing, CI/CD, and design tokens.
