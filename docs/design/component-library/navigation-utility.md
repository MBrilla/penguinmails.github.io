---
title: Navigation & Utility Components
description: Breadcrumb, Tabs, Loading, EmptyState, and Badge components
last_modified_date: 2025-01-15
level: 2
persona: ["developer"]
keywords: [navigation, breadcrumb, tabs, loading, empty-state, badge]
---

{% raw %}

# Navigation & Utility Components

Components for navigation, loading states, and utility displays.

## Breadcrumb

```typescript
import { Breadcrumb } from '@/components/navigation/Breadcrumb';

interface BreadcrumbItem {
  label: string;
  href?: string;
  icon?: React.ComponentType;
  current?: boolean;
}

interface BreadcrumbProps {
  items: BreadcrumbItem[];
  separator?: React.ComponentType;
  maxItems?: number;
  collapsible?: boolean;
}

// Usage Example
<Breadcrumb
  items={[
    { label: 'Dashboard', href: '/dashboard', icon: HomeIcon },
    { label: 'Campaigns', href: '/campaigns' },
    { label: 'Summer Sale 2025', current: true }
  ]}
  maxItems={5}
/>
```

## Tabs

```typescript
import { Tabs } from '@/components/navigation/Tabs';

interface TabItem {
  id: string;
  label: string;
  icon?: React.ComponentType;
  badge?: string | number;
  disabled?: boolean;
  content?: React.ReactNode;
}

interface TabsProps {
  tabs: TabItem[];
  activeTab: string;
  variant?: 'default' | 'pills' | 'underline' | 'buttons';
  size?: 'sm' | 'md' | 'lg';
  fullWidth?: boolean;
  lazy?: boolean;
  onChange: (tabId: string) => void;
}

// Usage Example
<Tabs
  tabs={[
    {
      id: 'overview',
      label: 'Overview',
      icon: DashboardIcon,
      content: <CampaignOverview campaign={campaign} />
    },
    {
      id: 'performance',
      label: 'Performance',
      badge: 'New',
      content: <CampaignPerformance campaign={campaign} />
    },
    {
      id: 'recipients',
      label: 'Recipients',
      content: <CampaignRecipients campaign={campaign} />
    },
    {
      id: 'settings',
      label: 'Settings',
      icon: SettingsIcon,
      content: <CampaignSettings campaign={campaign} />
    }
  ]}
  activeTab={activeTab}
  variant="underline"
  onChange={setActiveTab}
/>
```

## Loading

```typescript
import { Loading, Skeleton } from '@/components/utilities/Loading';

interface LoadingProps {
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
  variant?: 'spinner' | 'dots' | 'pulse' | 'bars';
  color?: string;
  overlay?: boolean;
  text?: string;
  showText?: boolean;
}

interface SkeletonProps {
  variant?: 'text' | 'rectangular' | 'circular';
  width?: string | number;
  height?: string | number;
  animation?: 'pulse' | 'wave' | false;
}

// Usage Examples
<Loading size="md" variant="spinner" text="Saving changes..." />

<Skeleton variant="rectangular" width="100%" height={200} />
<Skeleton variant="text" width="60%" />
<Skeleton variant="circular" width={40} height={40} />
```

## EmptyState

```typescript
import { EmptyState } from '@/components/utilities/EmptyState';

interface EmptyStateProps {
  icon?: React.ComponentType;
  title: string;
  description?: string;
  size?: 'sm' | 'md' | 'lg';
  action?: {
    label: string;
    onClick: () => void;
    variant?: 'primary' | 'secondary';
    icon?: React.ComponentType;
  };
  secondaryAction?: {
    label: string;
    onClick: () => void;
  };
}

// Usage Example
<EmptyState
  icon={InboxIcon}
  title="No campaigns yet"
  description="Create your first email campaign to get started."
  action={{
    label: "Create Campaign",
    variant: "primary",
    icon: PlusIcon,
    onClick: () => navigate('/campaigns')
  }}
  secondaryAction={{
    label: "Learn More",
    onClick: () => openTutorial('creating-campaigns')
  }}
/>
```

## Badge

```typescript
import { Badge } from '@/components/utilities/Badge';

interface BadgeProps {
  variant?: 'default' | 'success' | 'warning' | 'error' | 'info' | 'neutral';
  size?: 'xs' | 'sm' | 'md';
  rounded?: boolean;
  dot?: boolean;
  icon?: React.ComponentType;
  children: React.ReactNode;
}

// Usage Examples
<Badge variant="success">Active</Badge>
<Badge variant="warning" size="sm">Pending</Badge>
<Badge variant="error" dot /> {/* Status dot only */}
<Badge variant="info" icon={InfoIcon}>New Feature</Badge>
```

## Component Status and Roadmap

### Current Component Status

| Component | Status | Version |
|-----------|--------|---------|
| Button | Stable | 2.1.0 |
| Input | Stable | 2.0.3 |
| Select | Stable | 1.8.2 |
| Table | Stable | 2.2.1 |
| Modal | Stable | 1.9.0 |
| Card | Stable | 1.7.1 |
| Alert | Stable | 1.6.0 |
| Tabs | Stable | 1.5.2 |
| Tooltip | Stable | 1.4.1 |

### Upcoming Components

- **DatePicker**: Advanced date selection with calendar
- **TimePicker**: Time selection with timezone support
- **ColorPicker**: Color selection for email templates
- **FileUpload**: Drag-and-drop file upload with progress
- **Stepper**: Multi-step process indicator
- **Accordion**: Collapsible content sections

### Import Patterns

```typescript
// Preferred: Import specific components
import { Button, Input, Modal } from '@/components';

// Alternative: Import with alias for complex components
import { Table as DataTable } from '@/components/data-display/Table';
import { Chart as AnalyticsChart } from '@/components/data-display/Chart';
```

## Related Documentation

- [Primitive Components](./primitives) - Button, Input
- [Data & Feedback](./data-feedback) - Table, Modal, Toast
- [Component Library Overview](./) - Full component catalog

{% endraw %}
