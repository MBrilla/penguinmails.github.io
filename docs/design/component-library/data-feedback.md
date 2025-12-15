---
title: Data Display & Feedback Components
description: Table, Card, Alert, Toast, and Modal components
last_modified_date: 2025-01-15
level: 2
persona: ["developer"]
keywords: [table, card, alert, toast, modal, data-display, feedback]
---

{% raw %}

# Data Display & Feedback Components

Components for displaying data and providing user feedback.

## Table

```typescript
import { Table } from '@/components/data-display/Table';

interface TableColumn<T> {
  key: keyof T;
  header: string;
  sortable?: boolean;
  filterable?: boolean;
  width?: string | number;
  align?: 'left' | 'center' | 'right';
  render?: (value: any, row: T) => React.ReactNode;
}

interface TableProps<T> {
  data: T[];
  columns: TableColumn<T>[];
  loading?: boolean;
  selectable?: boolean;
  pagination?: boolean;
  pageSize?: number;
  emptyState?: React.ReactNode;
  onSort?: (key: keyof T, direction: 'asc' | 'desc') => void;
  onSelect?: (selectedRows: T[]) => void;
  onRowClick?: (row: T) => void;
}

// Usage Example
<Table
  data={campaigns}
  columns={[
    {
      key: 'name',
      header: 'Campaign Name',
      sortable: true,
      render: (value, row) => (
        <Link to={`/campaigns/${row.id}`}>{value}</Link>
      )
    },
    {
      key: 'status',
      header: 'Status',
      filterable: true,
      render: (value) => <StatusBadge status={value} />
    },
    {
      key: 'openRate',
      header: 'Open Rate',
      sortable: true,
      align: 'right',
      render: (value) => `${(value * 100).toFixed(1)}%`
    }
  ]}
  pagination
  pageSize={25}
  selectable
  onSelect={handleSelection}
/>
```

## Card

```typescript
import { Card, CardHeader, CardContent, CardFooter } from '@/components/data-display/Card';

interface CardProps {
  variant?: 'default' | 'elevated' | 'outlined' | 'filled';
  size?: 'sm' | 'md' | 'lg';
  padding?: 'none' | 'sm' | 'md' | 'lg';
  hoverable?: boolean;
  clickable?: boolean;
  fullWidth?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}

// Usage Example
<Card variant="elevated" hoverable clickable onClick={handleCardClick}>
  <CardHeader>
    <Avatar src={campaign.creator.avatar} size="md" />
    <div>
      <h3 className="card-title">{campaign.name}</h3>
      <p className="card-subtitle">{campaign.description}</p>
    </div>
  </CardHeader>

  <CardContent>
    <div className="metrics-grid">
      <Metric label="Recipients" value={campaign.recipientCount} />
      <Metric label="Open Rate" value={`${campaign.openRate}%`} />
    </div>
  </CardContent>

  <CardFooter>
    <Button variant="secondary" size="sm">View Details</Button>
    <Button variant="primary" size="sm">Edit Campaign</Button>
  </CardFooter>
</Card>
```

## Alert

```typescript
import { Alert } from '@/components/feedback/Alert';

interface AlertProps {
  variant: 'info' | 'success' | 'warning' | 'error';
  title?: string;
  message: string;
  dismissible?: boolean;
  action?: {
    label: string;
    onClick: () => void;
    variant?: 'primary' | 'secondary';
  };
  icon?: React.ComponentType;
  onDismiss?: () => void;
}

// Usage Examples
<Alert
  variant="warning"
  title="Action Required"
  message="Your Stripe account setup is incomplete."
  action={{
    label: "Complete Setup",
    onClick: () => navigate('/settings')
  }}
  dismissible
/>

<Alert
  variant="success"
  message="Campaign sent successfully!"
/>
```

## Toast

```typescript
import { useToast } from '@/components/feedback/Toast';

interface ToastOptions {
  variant?: 'info' | 'success' | 'warning' | 'error';
  title?: string;
  message: string;
  duration?: number;
  persistent?: boolean;
  action?: {
    label: string;
    onClick: () => void;
  };
}

// Usage with Hook
const toast = useToast();

const handleSave = async () => {
  try {
    await saveCampaign(campaignData);
    toast.success({
      title: "Campaign Saved",
      message: "Your campaign has been saved successfully.",
      action: {
        label: "View Campaign",
        onClick: () => navigate(`/campaigns/${campaignData.id}`)
      }
    });
  } catch (error) {
    toast.error({
      title: "Save Failed",
      message: "Unable to save campaign. Please try again.",
      persistent: true
    });
  }
};
```

## Modal

```typescript
import { Modal } from '@/components/feedback/Modal';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  description?: string;
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full';
  closable?: boolean;
  closeOnOverlayClick?: boolean;
  closeOnEscape?: boolean;
  footer?: React.ReactNode;
  children: React.ReactNode;
}

// Usage Example
<Modal
  isOpen={showDeleteModal}
  onClose={() => setShowDeleteModal(false)}
  title="Delete Campaign"
  description="This action cannot be undone."
  size="md"
  footer={
    <div className="modal-footer-actions">
      <Button variant="secondary" onClick={() => setShowDeleteModal(false)}>
        Cancel
      </Button>
      <Button variant="danger" onClick={handleDelete} loading={isDeleting}>
        Delete Campaign
      </Button>
    </div>
  }
>
  <p>Are you sure you want to delete <strong>"{campaignName}"</strong>?</p>
</Modal>
```

## Related Documentation

- [Form Components](./forms) - FormField, Select
- [Navigation & Utility](./navigation-utility) - Breadcrumb, Tabs, Loading
- [Component Library Overview](./) - Full component catalog

{% endraw %}
