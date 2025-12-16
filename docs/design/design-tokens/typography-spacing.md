---
title: "Typography and Spacing Tokens"
description: "Font families, sizes, weights, spacing scale, and sizing tokens"
last_modified_date: "2025-11-19"
level: "2"
persona: "Designers, Frontend Developers"
---

# Typography and Spacing Tokens

Comprehensive typography and spacing specifications for consistent text rendering and layout across all platforms.

## Typography Tokens

### Font Families

```css
--font-family-primary: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
--font-family-mono: 'JetBrains Mono', 'SF Mono', Monaco, 'Cascadia Code', 'Roboto Mono', monospace;
--font-family-display: 'Cal Sans', 'Inter', sans-serif;
```

### Font Sizes

```css
/* Display sizes */
--font-size-display-2xl: 4.5rem;    /* 72px - Hero headlines */
--font-size-display-xl: 3.75rem;    /* 60px - Section headers */
--font-size-display-lg: 3rem;       /* 48px - Page titles */
--font-size-display-md: 2.25rem;    /* 36px - Major headings */
--font-size-display-sm: 1.875rem;   /* 30px - Sub headings */

/* Text sizes */
--font-size-text-2xl: 1.5rem;       /* 24px - Large body text */
--font-size-text-xl: 1.25rem;       /* 20px - Small headings */
--font-size-text-lg: 1.125rem;      /* 18px - Body text */
--font-size-text-base: 1rem;        /* 16px - Base body text */
--font-size-text-sm: 0.875rem;      /* 14px - Small body text */
--font-size-text-xs: 0.75rem;       /* 12px - Captions */
--font-size-text-2xs: 0.625rem;     /* 10px - Micro text */
```

### Font Weights

```css
--font-weight-thin: 100;
--font-weight-light: 300;
--font-weight-regular: 400;
--font-weight-medium: 500;
--font-weight-semibold: 600;
--font-weight-bold: 700;
--font-weight-extrabold: 800;
--font-weight-black: 900;
```

### Line Heights

```css
--line-height-tight: 1.25;     /* Headlines */
--line-height-normal: 1.5;     /* Body text */
--line-height-relaxed: 1.625;  /* Spacious reading */
--line-height-loose: 2;        /* Form inputs */
```

### Letter Spacing

```css
--letter-spacing-tight: -0.025em;   /* Headlines */
--letter-spacing-normal: 0;         /* Body text */
--letter-spacing-wide: 0.025em;     /* Small caps, buttons */
--letter-spacing-wider: 0.05em;     /* All caps */
--letter-spacing-widest: 0.1em;     /* Special emphasis */
```

## Spacing Tokens

### Space Scale

```css
--space-0: 0;                    /* 0px */
--space-px: 1px;                 /* 1px */
--space-0-5: 0.125rem;           /* 2px */
--space-1: 0.25rem;              /* 4px */
--space-1-5: 0.375rem;           /* 6px */
--space-2: 0.5rem;               /* 8px */
--space-2-5: 0.625rem;           /* 10px */
--space-3: 0.75rem;              /* 12px */
--space-3-5: 0.875rem;           /* 14px */
--space-4: 1rem;                 /* 16px */
--space-5: 1.25rem;              /* 20px */
--space-6: 1.5rem;               /* 24px */
--space-7: 1.75rem;              /* 28px */
--space-8: 2rem;                 /* 32px */
--space-9: 2.25rem;              /* 36px */
--space-10: 2.5rem;              /* 40px */
--space-11: 2.75rem;             /* 44px */
--space-12: 3rem;                /* 48px */
--space-14: 3.5rem;              /* 56px */
--space-16: 4rem;                /* 64px */
--space-20: 5rem;                /* 80px */
--space-24: 6rem;                /* 96px */
--space-32: 8rem;                /* 128px */
--space-40: 10rem;               /* 160px */
--space-48: 12rem;               /* 192px */
--space-64: 16rem;               /* 256px */
--space-80: 20rem;               /* 320px */
--space-96: 24rem;               /* 384px */
```

### Spacing Usage Guidelines

- **space-1 to space-3**: Small gaps, borders, icon spacing
- **space-4 to space-6**: Component padding, form spacing
- **space-8 to space-12**: Section spacing, card padding
- **space-16+**: Large layouts, page sections

## Sizing Tokens

### Component Heights

```css
--size-height-xs: 1.5rem;        /* 24px - Small buttons */
--size-height-sm: 2rem;          /* 32px - Small inputs */
--size-height-md: 2.5rem;        /* 40px - Standard inputs/buttons */
--size-height-lg: 3rem;          /* 48px - Large buttons */
--size-height-xl: 3.5rem;        /* 56px - Extra large buttons */
```

### Icon Sizes

```css
--size-icon-xs: 0.75rem;         /* 12px */
--size-icon-sm: 1rem;            /* 16px */
--size-icon-md: 1.25rem;         /* 20px */
--size-icon-lg: 1.5rem;          /* 24px */
--size-icon-xl: 2rem;            /* 32px */
--size-icon-2xl: 2.5rem;         /* 40px */
```

### Border Radius

```css
--radius-none: 0;
--radius-xs: 0.125rem;           /* 2px */
--radius-sm: 0.25rem;            /* 4px */
--radius-md: 0.375rem;           /* 6px */
--radius-lg: 0.5rem;             /* 8px */
--radius-xl: 0.75rem;            /* 12px */
--radius-2xl: 1rem;              /* 16px */
--radius-3xl: 1.5rem;            /* 24px */
--radius-full: 9999px;           /* Fully rounded */
```

### Border Width

```css
--border-width-none: 0;
--border-width-thin: 1px;
--border-width-medium: 2px;
--border-width-thick: 4px;
--border-width-heavy: 8px;
```

---

**Related Documentation:**

- [Design Tokens Overview](./overview) - Token architecture and color specifications
- [Effects and Breakpoints](./effects-breakpoints) - Shadows, animations, and responsive breakpoints

---

**Document Classification:** Level 2 - Design Specifications
**Business Value:** Consistent typography and spacing across all interfaces
