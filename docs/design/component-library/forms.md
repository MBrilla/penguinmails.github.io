---
title: Form Components
description: FormField, Select, Checkbox, and Radio components
last_modified_date: 2025-01-15
level: 2
persona: ["developer"]
keywords: [forms, select, checkbox, radio, form-field]
---

# Form Components

Components for building accessible, validated forms.

## FormField

```typescript
import { FormField } from '@/components/forms/FormField';

interface FormFieldProps {
  label?: string;
  required?: boolean;
  helperText?: string;
  errorMessage?: string;
  disabled?: boolean;
  children: React.ReactNode;
}

// Usage Example
<FormField
  label="Campaign Name"
  required
  helperText="Choose a descriptive name for your campaign"
  errorMessage={errors.name}
>
  <Input
    value={campaignName}
    onChange={setCampaignName}
    placeholder="e.g., Summer Sale 2025"
  />
</FormField>
```

## Select

```typescript
import { Select } from '@/components/forms/Select';

interface SelectOption {
  value: string | number;
  label: string;
  disabled?: boolean;
  icon?: React.ComponentType;
}

interface SelectProps {
  options: SelectOption[];
  value?: string | number;
  placeholder?: string;
  multiple?: boolean;
  searchable?: boolean;
  clearable?: boolean;
  loading?: boolean;
  disabled?: boolean;
  size?: 'sm' | 'md' | 'lg';
  onChange: (value: string | number | (string | number)[]) => void;
}

// Usage Example
<Select
  options={[
    { value: 'draft', label: 'Draft', icon: EditIcon },
    { value: 'scheduled', label: 'Scheduled', icon: CalendarIcon },
    { value: 'sending', label: 'Sending', icon: SendIcon },
    { value: 'completed', label: 'Completed', icon: CheckIcon }
  ]}
  value={status}
  placeholder="Select campaign status"
  onChange={(value) => setStatus(value as string)}
/>
```

## Checkbox & Radio

```typescript
import { Checkbox, Radio, RadioGroup } from '@/components/forms/Checkbox';

interface CheckboxProps {
  checked: boolean;
  disabled?: boolean;
  label?: string;
  helperText?: string;
  onChange: (checked: boolean) => void;
}

interface RadioGroupProps {
  value: string;
  onChange: (value: string) => void;
  disabled?: boolean;
  children: React.ReactNode;
}

// Usage Examples
<Checkbox
  checked={agreeToTerms}
  onChange={setAgreeToTerms}
  label="I agree to the Terms of Service"
  helperText="You must agree to continue"
/>

<RadioGroup value={planType} onChange={setPlanType}>
  <Radio value="starter" label="Starter Plan" />
  <Radio value="professional" label="Professional Plan" />
  <Radio value="enterprise" label="Enterprise Plan" />
</RadioGroup>
```

## Related Documentation

- [Primitive Components](./primitives) - Button, Input
- [Layout Components](./layout) - Container, Grid
- [Data & Feedback Components](./data-feedback) - Table, Modal
- [Component Library Overview](./) - Full component catalog
