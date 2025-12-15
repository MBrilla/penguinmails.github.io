---
title: Primitive Components
description: Button, Input, and core primitive UI components
last_modified_date: 2025-01-15
level: 2
persona: ["developer"]
keywords: [button, input, primitives, react-components]
---

# Primitive Components

Core building blocks for the PenguinMails component library.

## Button

```typescript
import { Button } from '@/components/primitives/Button';

interface ButtonProps {
  variant: 'primary' | 'secondary' | 'tertiary' | 'danger' | 'success';
  size: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
  disabled?: boolean;
  loading?: boolean;
  fullWidth?: boolean;
  icon?: React.ComponentType<{ className?: string }>;
  iconPosition?: 'left' | 'right';
  children: React.ReactNode;
  onClick: () => void;
}

// Usage Examples
<Button variant="primary" size="md" onClick={handleSubmit}>
  Create Campaign
</Button>

<Button variant="secondary" icon={PlusIcon} iconPosition="left">
  Add Recipient
</Button>

<Button variant="danger" size="sm" loading={isDeleting}>
  Delete
</Button>
```

### Design Specifications

| Property | Small | Medium | Large |
|----------|-------|--------|-------|
| Height | 32px | 40px | 48px |
| Border radius | 6px | 6px | 8px |
| Font weight | 500 | 500 | 600 |

Focus ring: 2px solid primary color, 2px offset

## Input

```typescript
import { Input } from '@/components/primitives/Input';

interface InputProps {
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url' | 'search';
  size?: 'sm' | 'md' | 'lg';
  variant?: 'default' | 'error' | 'success';
  disabled?: boolean;
  required?: boolean;
  readOnly?: boolean;
  placeholder?: string;
  value: string;
  onChange: (value: string) => void;
  onBlur?: () => void;
  onFocus?: () => void;
  leftIcon?: React.ComponentType;
  rightIcon?: React.ComponentType;
  helperText?: string;
  errorMessage?: string;
}

// Usage Example
<Input
  type="email"
  placeholder="Enter your email address"
  value={email}
  onChange={setEmail}
  onBlur={() => validateEmail(email)}
  errorMessage={emailError}
  leftIcon={MailIcon}
  required
/>
```

### Design Specifications

| Property | Small | Medium | Large |
|----------|-------|--------|-------|
| Height | 32px | 40px | 48px |
| Padding | 8px 12px | 12px 16px | 16px 20px |
| Font size | 14px | 16px | 18px |

- Border: 1px solid neutral-300 (default)
- Border radius: 6px

## Related Documentation

- [Layout Components](./layout) - Container and Grid
- [Form Components](./forms) - FormField, Select, Checkbox
- [Component Library Overview](./) - Full component catalog
