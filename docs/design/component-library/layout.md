---
title: Layout Components
description: Container and Grid components for page layout
last_modified_date: 2025-01-15
level: 2
persona: ["developer"]
keywords: [layout, container, grid, responsive]
---

# Layout Components

Components for structuring page layouts and responsive grids.

## Container

```typescript
import { Container } from '@/components/layout/Container';

interface ContainerProps {
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full';
  padding?: 'none' | 'sm' | 'md' | 'lg' | 'xl';
  centered?: boolean;
  children: React.ReactNode;
}

// Usage Examples
<Container size="lg" padding="md">
  <h1>Main Content</h1>
  <p>Container with large max-width and medium padding</p>
</Container>

<Container size="full" centered padding="none">
  <HeroSection />
</Container>
```

### Size Specifications

| Size | Max Width |
|------|-----------|
| sm | 640px |
| md | 768px |
| lg | 1024px |
| xl | 1280px |
| full | 100% |

## Grid & GridItem

```typescript
import { Grid, GridItem } from '@/components/layout/Grid';

interface GridProps {
  columns?: number; // 1-12
  gap?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
  responsive?: boolean;
  children: React.ReactNode;
}

interface GridItemProps {
  span?: number; // 1-12 columns to span
  offset?: number; // 0-11 columns to offset
  order?: number;
  children: React.ReactNode;
}

// Usage Example
<Grid columns={12} gap="md" responsive>
  <GridItem span={8}>
    <MainContent />
  </GridItem>
  <GridItem span={4}>
    <Sidebar />
  </GridItem>
</Grid>
```

### Responsive Breakpoints

| Breakpoint | Columns |
|------------|---------|
| Mobile | 1 column (span resets to 12) |
| Tablet | 8 columns max |
| Desktop | 12 columns max |

## Related Documentation

- [Primitive Components](./primitives) - Button, Input
- [Form Components](./forms) - FormField, Select
- [Component Library Overview](./) - Full component catalog
