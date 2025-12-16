---
title: "Design Tokens Overview"
description: "Token architecture and color specifications for PenguinMails design system"
last_modified_date: "2025-11-19"
level: "2"
persona: "Designers, Frontend Developers"
---

# Design Tokens Overview

Design tokens are the fundamental building blocks of PenguinMails design system. They provide a single source of truth for design decisions, enabling consistent visual language across all platforms and ensuring scalable, maintainable design implementation.

## Implementation Reference

| Token Category | Implementation Location | Format |
|---|---|---|
| **Global Tokens** | `apps/web/app/globals.css` | CSS Variables (`:root`) |
| **Tailwind Config** | `apps/web/tailwind.config.ts` | Tailwind Theme Extension |
| **Type Definitions** | `packages/types/src/design-system.d.ts` | TypeScript Interfaces |

Changes to tokens in this document must be reflected in `globals.css` and `tailwind.config.ts` to take effect.

## Token Architecture

### Token Categories

- **Color**: Brand colors, semantic colors, and neutral palettes
- **Typography**: Font families, sizes, weights, and line heights
- **Spacing**: Consistent spacing scale for margins, padding, and layout
- **Sizing**: Component dimensions, icon sizes, and border radius
- **Shadow**: Depth and elevation system
- **Border**: Border widths, styles, and radius values
- **Animation**: Motion design tokens for interactions

### Token Naming Convention

```css
/* Pattern: --[category]-[property]-[variant]-[modifier] */
--color-primary-500          /* Primary color, base shade */
--color-semantic-success-600  /* Semantic color, success variant, darker shade */
--text-size-lg                /* Typography size, large */
--space-4                     /* Spacing scale, 4th step */
--shadow-lg                   /* Shadow, large depth */
--radius-md                   /* Border radius, medium */
```

## Color Tokens

### Brand Colors

```css
/* Primary Blue - Trust, Technology, Professionalism */
--color-brand-primary-50: #f0f9ff;
--color-brand-primary-100: #e0f2fe;
--color-brand-primary-200: #bae6fd;
--color-brand-primary-300: #7dd3fc;
--color-brand-primary-400: #38bdf8;
--color-brand-primary-500: #0ea5e9;  /* Primary brand color */
--color-brand-primary-600: #0284c7;
--color-brand-primary-700: #0369a1;
--color-brand-primary-800: #075985;
--color-brand-primary-900: #0c4a6e;

/* Secondary Colors */
--color-brand-secondary-50: #fdf4ff;
--color-brand-secondary-100: #fae8ff;
--color-brand-secondary-200: #f5d0fe;
--color-brand-secondary-300: #f0abfc;
--color-brand-secondary-400: #e879f9;
--color-brand-secondary-500: #d946ef;  /* Secondary accent */
--color-brand-secondary-600: #c026d3;
--color-brand-secondary-700: #a21caf;
--color-brand-secondary-800: #86198f;
--color-brand-secondary-900: #701a75;
```

### Semantic Colors

```css
/* Success States */
--color-semantic-success-50: #f0fdf4;
--color-semantic-success-500: #22c55e;  /* Success */
--color-semantic-success-600: #16a34a;
--color-semantic-success-700: #15803d;

/* Warning States */
--color-semantic-warning-50: #fffbeb;
--color-semantic-warning-500: #f59e0b;  /* Warning */
--color-semantic-warning-600: #d97706;
--color-semantic-warning-700: #b45309;

/* Error States */
--color-semantic-error-50: #fef2f2;
--color-semantic-error-500: #ef4444;  /* Error */
--color-semantic-error-600: #dc2626;
--color-semantic-error-700: #b91c1c;

/* Info States */
--color-semantic-info-50: #eff6ff;
--color-semantic-info-500: #3b82f6;  /* Info */
--color-semantic-info-600: #2563eb;
--color-semantic-info-700: #1d4ed8;
```

### Neutral Colors

```css
/* Gray Scale */
--color-neutral-gray-50: #f9fafb;
--color-neutral-gray-100: #f3f4f6;
--color-neutral-gray-200: #e5e7eb;
--color-neutral-gray-300: #d1d5db;
--color-neutral-gray-400: #9ca3af;
--color-neutral-gray-500: #6b7280;  /* Base text */
--color-neutral-gray-600: #4b5563;
--color-neutral-gray-700: #374151;  /* Secondary text */
--color-neutral-gray-800: #1f2937;  /* Primary text */
--color-neutral-gray-900: #111827;  /* Headlines */
```

### Special Colors

```css
/* Background Colors */
--color-background-primary: #ffffff;
--color-background-secondary: #f9fafb;
--color-background-tertiary: #f3f4f6;
--color-background-overlay: rgba(0, 0, 0, 0.5);

/* Text Colors */
--color-text-primary: #111827;      /* Highest contrast */
--color-text-secondary: #374151;    /* Medium contrast */
--color-text-tertiary: #6b7280;     /* Low contrast */
--color-text-inverse: #ffffff;      /* On dark backgrounds */

/* Border Colors */
--color-border-subtle: #e5e7eb;
--color-border-default: #d1d5db;
--color-border-strong: #9ca3af;
--color-border-focus: #0ea5e9;
```

---

**Related Documentation:**

- [Typography and Spacing](./typography-spacing) - Font families, sizes, and spacing scale
- [Effects and Breakpoints](./effects-breakpoints) - Shadows, animations, and responsive breakpoints

---

**Document Classification:** Level 2 - Design Specifications
**Business Value:** Single source of truth for consistent visual language
