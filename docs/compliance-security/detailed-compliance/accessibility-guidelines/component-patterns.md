---
title: "Accessible Component Patterns"
description: "Component-specific accessibility guidelines and content guidelines for PenguinMails"
last_modified_date: "2025-10-28"
level: "2"
persona: "Documentation Users"
---

# Accessible Component Patterns

## Data Tables

```html
<table role="table" aria-label="Campaign performance data">
  <thead>
    <tr>
      <th scope="col">Campaign Name</th>
      <th scope="col">Sent</th>
      <th scope="col">Open Rate</th>
      <th scope="col">Click Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Welcome Series</th>
      <td>1,250</td>
      <td>42%</td>
      <td>8%</td>
    </tr>
  </tbody>
</table>
```

## Modal Dialogs

```html
<div role="dialog" aria-modal="true" aria-labelledby="modal-title">
  <div class="modal-header">
    <h2 id="modal-title">Confirm Campaign Deletion</h2>
    <button aria-label="Close dialog">
      <CloseIcon />
    </button>
  </div>
  <div class="modal-body">
    <p>Are you sure you want to delete this campaign?</p>
  </div>
  <div class="modal-footer">
    <button>Cancel</button>
    <button class="destructive">Delete Campaign</button>
  </div>
</div>
```

## Progress Indicators

```html
<div role="progressbar" aria-valuenow="75" aria-valuemin="0" aria-valuemax="100">
  <span class="sr-only">Sending emails: 75% complete</span>
  <div class="progress-bar" style="width: 75%"></div>
</div>
```

## Custom Components

- **ARIA attributes**: Use appropriate ARIA roles and properties

- **Keyboard support**: Custom components need keyboard event handlers

- **Focus management**: Proper focus behavior for complex components

- **Screen reader support**: Descriptive announcements and instructions

## Content Guidelines

### Writing for Accessibility

- **Clear headings** that describe section content

- **Short paragraphs** (3-5 sentences maximum)

- **Simple words** avoiding complex vocabulary

- **Active voice** instead of passive voice

- **Front-loaded content** (most important information first)

### Visual Design Guidelines

- **Sufficient contrast** for all text and UI elements

- **Meaningful icons** with text labels

- **Consistent spacing** for visual hierarchy

- **Clear focus indicators** that don't rely on color alone

- **Readable fonts** at appropriate sizes

### Multimedia Content

- **Audio content** requires transcripts

- **Video content** requires captions and audio descriptions

- **Animations** should be optional and respect user preferences

- **Interactive content** should be operable by multiple input methods

## Common Issues and Solutions

### Missing Alt Text

**Issue**: Images without alternative text
**Solution**: Implement alt text requirements in component library

### Poor Color Contrast

**Issue**: Text doesn't meet contrast requirements
**Solution**: Design system enforces contrast ratios

### Keyboard Navigation Problems

**Issue**: Interactive elements not keyboard accessible
**Solution**: Component library includes keyboard support by default

### Missing Form Labels

**Issue**: Form fields without proper labels
**Solution**: Form components require labels and enforce proper association

### Inaccessible Custom Components

**Issue**: Custom widgets not accessible to assistive technology
**Solution**: ARIA implementation guidelines and testing requirements

---

## Related Documents

- [Accessibility Overview](/docs/compliance-security/detailed-compliance/accessibility-guidelines/overview) - Strategic alignment and compliance

- [WCAG Guidelines](/docs/compliance-security/detailed-compliance/accessibility-guidelines/wcag-guidelines) - Detailed WCAG requirements

- [Implementation](/docs/compliance-security/detailed-compliance/accessibility-guidelines/implementation) - Development process and testing

- [Design System](/docs/design) - Complete design system overview

- [Design Tokens](/docs/design/tokens) - Design token specifications

- [Interaction Patterns](/docs/design/interaction-patterns) - User interaction frameworks
