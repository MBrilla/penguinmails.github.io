---
title: "Effects and Breakpoints"
description: "Shadow tokens, animation patterns, responsive breakpoints, and implementation guidelines"
last_modified_date: "2025-11-19"
level: "2"
persona: "Designers, Frontend Developers"
---

# Effects and Breakpoints

Shadow elevation, animation patterns, responsive breakpoints, and implementation guidelines for the design system.

## Shadow Tokens

### Elevation Scale

```css
--shadow-none: none;
--shadow-xs: 0 1px 2px 0 rgb(0 0 0 / 0.05);
--shadow-sm: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
--shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
--shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
--shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
--shadow-2xl: 0 25px 50px -12px rgb(0 0 0 / 0.25);
--shadow-inner: inset 0 2px 4px 0 rgb(0 0 0 / 0.05);
```

### Colored Shadows

```css
--shadow-primary-xs: 0 1px 2px 0 rgb(14 165 233 / 0.2);
--shadow-primary-sm: 0 1px 3px 0 rgb(14 165 233 / 0.2), 0 1px 2px -1px rgb(14 165 233 / 0.2);
--shadow-primary-md: 0 4px 6px -1px rgb(14 165 233 / 0.2), 0 2px 4px -2px rgb(14 165 233 / 0.2);

--shadow-success-xs: 0 1px 2px 0 rgb(34 197 94 / 0.2);
--shadow-success-sm: 0 1px 3px 0 rgb(34 197 94 / 0.2), 0 1px 2px -1px rgb(34 197 94 / 0.2);

--shadow-warning-xs: 0 1px 2px 0 rgb(245 158 11 / 0.2);
--shadow-warning-sm: 0 1px 3px 0 rgb(245 158 11 / 0.2), 0 1px 2px -1px rgb(245 158 11 / 0.2);

--shadow-error-xs: 0 1px 2px 0 rgb(239 68 68 / 0.2);
--shadow-error-sm: 0 1px 3px 0 rgb(239 68 68 / 0.2), 0 1px 2px -1px rgb(239 68 68 / 0.2);
```

## Animation Tokens

### Duration

```css
--duration-instant: 0ms;
--duration-fast: 150ms;
--duration-normal: 300ms;
--duration-slow: 500ms;
```

### Easing Functions

```css
--ease-in: cubic-bezier(0.4, 0, 1, 1);
--ease-out: cubic-bezier(0, 0, 0.2, 1);
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
--ease-bounce: cubic-bezier(0.68, -0.55, 0.265, 1.55);
```

### Animation Patterns

```css
--transition-fast: all 150ms cubic-bezier(0.4, 0, 0.2, 1);
--transition-normal: all 300ms cubic-bezier(0.4, 0, 0.2, 1);
--transition-slow: all 500ms cubic-bezier(0.4, 0, 0.2, 1);
```

## Breakpoint Tokens

### Responsive Breakpoints

```css
--breakpoint-xs: 475px;          /* Extra small devices */
--breakpoint-sm: 640px;          /* Small devices */
--breakpoint-md: 768px;          /* Medium devices */
--breakpoint-lg: 1024px;         /* Large devices */
--breakpoint-xl: 1280px;         /* Extra large devices */
--breakpoint-2xl: 1536px;        /* 2x large devices */
--breakpoint-3xl: 1920px;        /* 3x large devices */
```

### Container Sizes

```css
--container-xs: 20rem;            /* 320px */
--container-sm: 24rem;            /* 384px */
--container-md: 28rem;            /* 448px */
--container-lg: 32rem;            /* 512px */
--container-xl: 36rem;            /* 576px */
--container-2xl: 42rem;           /* 672px */
--container-3xl: 48rem;           /* 768px */
--container-4xl: 56rem;           /* 896px */
--container-5xl: 64rem;           /* 1024px */
--container-6xl: 72rem;           /* 1152px */
--container-7xl: 80rem;           /* 1280px */
--container-full: 100%;           /* Full width */
```

## Implementation Guidelines

### Token Usage Priority

1. **Design Tokens**: Always use tokens instead of hard-coded values
2. **Semantic Naming**: Use meaningful names that convey purpose
3. **Platform Consistency**: Ensure tokens work across all platforms
4. **Accessibility**: Verify all token combinations meet accessibility standards

### Token Modification Process

1. **Impact Assessment**: Evaluate effects on existing components
2. **Platform Testing**: Test across all supported platforms
3. **Documentation Update**: Update design system documentation
4. **Migration Plan**: Plan rollout and deprecation strategy
5. **Communication**: Notify all stakeholders of changes

### Version Control

- **Semantic Versioning**: Major.Minor.Patch for token changes
- **Deprecation Notices**: Clear timeline for token removal
- **Migration Tools**: Automated migration assistance
- **Rollback Plan**: Ability to revert changes if needed

---

**Related Documentation:**

- [Design Tokens Overview](./overview) - Token architecture and color specifications
- [Typography and Spacing](./typography-spacing) - Font families, sizes, and spacing scale
- [Design System](/docs/design/design-system) - Complete design system overview
- [Component Library](/docs/design/component-library) - Reusable component catalog
- [Accessibility Guidelines](/docs/design/accessibility-guidelines) - Inclusive design standards

---

**Document Classification:** Level 2 - Design Specifications
**Business Value:** Consistent effects and responsive design across all platforms
