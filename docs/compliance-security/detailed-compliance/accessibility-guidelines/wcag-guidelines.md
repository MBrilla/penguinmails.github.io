---
title: "WCAG Guidelines"
description: "Perceivable, Operable, Understandable, and Robust accessibility guidelines"
last_modified_date: "2025-10-28"
level: "2"
persona: "Documentation Users"
---

# WCAG Guidelines

## Perceivable Guidelines

### Text Alternatives

- **All images must have alt text** describing the image's purpose or content

- **Decorative images** use empty alt text (alt="")

- **Complex images** (charts, graphs) include detailed descriptions

- **Icons** use descriptive alt text or aria-label attributes

```html
<!-- Good examples -->
<img src="logo.png" alt="PenguinMails - Email Infrastructure Platform">
<img src="chart.png" alt="Monthly user growth: January 100 users, February 250 users, March 500 users">
<button aria-label="Close dialog">
  <svg aria-hidden="true"><!-- icon --></svg>
</button>

<!-- Avoid -->
<img src="logo.png" alt="Logo">
<img src="chart.png"> <!-- Missing alt text -->
```

### Color and Contrast

- **Normal text**: 4.5:1 contrast ratio minimum

- **Large text** (18pt+ or 14pt+ bold): 3:1 contrast ratio minimum

- **UI components**: 3:1 contrast ratio minimum

- **Focus indicators**: 3:1 contrast ratio against adjacent colors

```css
/* Color contrast compliance */
--text-primary: hsl(222, 84%, 5%);    /* On white background: 20.6:1 */
--text-secondary: hsl(215, 16%, 47%); /* On white background: 6.8:1 */
--text-muted: hsl(215, 16%, 47%);     /* On gray background: 4.6:1 */

/* Focus indicators */
.focus-visible {
  outline: 2px solid hsl(199, 89%, 48%);
  outline-offset: 2px;
}
```

### Audio and Video

- **Captions** for all video content

- **Transcripts** for audio-only content

- **Audio descriptions** for video content with important visual information

- **Pause/stop controls** for auto-playing media

### Structure and Semantics

- **Heading hierarchy**: H1 → H2 → H3 (no skipping levels)

- **Semantic HTML**: Use proper elements (`<main>`, `<nav>`, `<article>`, etc.)

- **Landmarks**: ARIA landmarks for screen reader navigation

- **Lists**: Use proper `<ul>`, `<ol>`, `<dl>` elements

## Operable Guidelines

### Keyboard Navigation

- **All interactive elements** must be keyboard accessible

- **Logical tab order** following reading order

- **Keyboard shortcuts** don't interfere with screen readers

- **No keyboard traps** (can't tab out of a component)

```html
<!-- Keyboard navigation examples -->
<button tabindex="0">Primary Action</button>
<a href="#main-content">Skip to main content</a>

<!-- Custom components need keyboard support -->
<div role="tablist" aria-label="Campaign tabs">
  <button role="tab" aria-selected="true" tabindex="0">
    Active Campaigns
  </button>
  <button role="tab" aria-selected="false" tabindex="-1">
    Draft Campaigns
  </button>
</div>
```

### Touch Targets

- **Minimum size**: 44px × 44px for touch targets

- **Adequate spacing**: 8px minimum between interactive elements

- **Touch gestures**: Support common gestures with alternatives

### Timing and Movement

- **No auto-advancing** carousels or timed content without pause controls

- **Adjustable timeouts**: At least 20 seconds, preferably 10 minutes

- **Reduced motion**: Respect `prefers-reduced-motion` setting

- **Pause animations** that last more than 5 seconds

```css
/* Respect user motion preferences */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Error Prevention

- **Clear error messages** explaining what went wrong

- **Suggestions for correction** when possible

- **Confirmation** for destructive actions

- **Undo functionality** where appropriate

## Understandable Guidelines

### Clear Language

- **Plain language** avoiding jargon and technical terms

- **Consistent terminology** throughout the interface

- **Progressive disclosure** of complex information

- **Helpful error messages** with actionable guidance

### Predictable Behavior

- **Consistent navigation** across all pages

- **Expected interactions** following platform conventions

- **Clear focus indicators** for keyboard navigation

- **No unexpected context changes**

### Input Assistance

- **Field labels** clearly associated with inputs

- **Input formats** clearly specified (e.g., "MM/DD")

- **Help text** available for complex fields

- **Autocomplete** for repetitive inputs

```html
<!-- Proper form labeling -->
<label for="email">Email Address</label>
<input id="email" type="email" aria-describedby="email-help">
<span id="email-help">We'll use this to send you campaign updates</span>

<!-- Error states -->
<div class="error-message" role="alert" aria-live="polite">
  Please enter a valid email address
</div>
```

## Robust Guidelines

### Compatible Technologies

- **Standards compliance**: Valid HTML5, CSS3, and JavaScript

- **Progressive enhancement**: Core functionality works without JavaScript

- **Cross-browser support**: Modern browsers with fallbacks

- **API accessibility**: REST APIs designed for assistive technology

### Assistive Technology Support

- **Screen readers**: JAWS, NVDA, VoiceOver, TalkBack

- **Screen magnifiers**: ZoomText, built-in browser zoom

- **Voice control**: Dragon NaturallySpeaking

- **Alternative input**: Switch devices, head pointers

---

## Related Documents

- [Accessibility Overview](/docs/compliance-security/detailed-compliance/accessibility-guidelines/overview) - Strategic alignment and compliance

- [Implementation](/docs/compliance-security/detailed-compliance/accessibility-guidelines/implementation) - Development process and testing

- [Component Patterns](/docs/compliance-security/detailed-compliance/accessibility-guidelines/component-patterns) - Accessible component patterns
