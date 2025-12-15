---
title: Component Library
description: Complete catalog of reusable UI components for PenguinMails
last_modified_date: 2025-01-15
level: 2
persona: ["developer", "designer"]
keywords: [component-library, ui-components, react, design-system]
---

# Component Library

The Component Library is a comprehensive catalog of all reusable UI components available in PenguinMails. This documentation serves as the single source of truth for component specifications, usage patterns, and implementation details.

## Quick Navigation

| Guide | Components |
|-------|------------|
| [Primitives](./primitives) | Button, Input, Typography |
| [Layout](./layout) | Container, Grid, GridItem |
| [Forms](./forms) | FormField, Select, Checkbox, Radio |
| [Data & Feedback](./data-feedback) | Table, Card, Alert, Toast, Modal |
| [Navigation & Utility](./navigation-utility) | Breadcrumb, Tabs, Loading, EmptyState, Badge |

## Implementation Reference

| Component Category | Codebase Location |
|---|---|
| **Primitives** | `apps/web/components/primitives/` |
| **Layout** | `apps/web/components/layout/` |
| **Forms** | `apps/web/components/forms/` |
| **Data Display** | `apps/web/components/data-display/` |
| **Feedback** | `apps/web/components/feedback/` |
| **Navigation** | `apps/web/components/navigation/` |

All components are exported from the barrel file at `apps/web/components/index.ts` for cleaner imports.

## Implementation Guidelines

### Design Tokens

Components strictly use design tokens defined in [Design Tokens](/docs/design/design-tokens) via Tailwind CSS utility classes.

- **Colors**: Use semantic names (e.g., `bg-primary`, `text-foreground`)
- **Spacing**: Use spacing scale (e.g., `p-4`, `gap-2`)
- **Typography**: Use text utilities (e.g., `text-sm`, `font-medium`)

### Mobile Support

Follow a mobile-first approach. Default styles apply to mobile viewports, with `md:` and `lg:` overrides for larger screens.

- **Touch Targets**: Interactive elements are at least 44px high on mobile
- **Widths**: Use `w-full` on mobile for block-level actions

### Theme Support

All components support both light and dark modes using Tailwind's `dark:` modifier.

- **Backgrounds**: `bg-white dark:bg-slate-950`
- **Borders**: `border-slate-200 dark:border-slate-800`
- **Text**: `text-slate-900 dark:text-slate-50`

### Accessibility

- **Interactive Elements**: Must be keyboard accessible (focusable)
- **ARIA Attributes**: Use `aria-label`, `aria-expanded`, etc. where visual context is missing
- **Focus States**: Always define visible focus states (`focus-visible:ring-2`)

## Library Structure

```text
📁 components/
├── 📁 primitives
├── 📁 layout
├── 📁 navigation
├── 📁 forms
├── 📁 data-display
├── 📁 feedback
├── 📁 overlays
└── 📁 utilities
```

### Component Maturity Levels

- **Stable**: Production-ready components with full test coverage
- **Experimental**: New components under development
- **Deprecated**: Components scheduled for removal (with migration guides)

## Related Documentation

- [UI Library](/docs/design/ui-library/overview) - Component usage guidelines
- [Design System](/docs/design/design-system) - Complete design system overview
- [Design Tokens](/docs/design/design-tokens) - Token specifications
- [Accessibility Guidelines](/docs/design/accessibility-guidelines) - Inclusive design standards
