---
name: web-accessibility-testing
description: >-
  Web accessibility rules and guidelines to ensure inclusive experiences for all users.
  Based on WCAG 2.2 AA compliance with practical implementation guidance.
priority: CRITICAL
---

# Web Accessibility Rules

> This file extends [common/accessibility.md](../common/accessibility.md) with web-specific content.

## Core Principles

### POUR Framework

1. **Perceivable** - Information is presentable to users in ways they can perceive
2. **Operable** - User interface components are operable
3. **Understandable** - Information and operation are understandable
4. **Robust** - Content is robust across current and future technologies

### WCAG 2.2 AA Compliance

Mandatory for all public-facing web applications. Must pass automated and manual testing.

## Keyboard Navigation

### Tab Order

- Logical, meaningful sequence through content
- Follow visual flow (left to right, top to bottom)
- Skip links must be available for bypassing repetitive content
- `tabindex="0"` for focusable elements, `tabindex="-1"` for programmatic focus only

### Navigation Components

```tsx
// Skip navigation
<a href="#main-content" className="skip-link">
  Skip to main content
</a>

// Navigation with proper landmarks
<nav aria-label="Main navigation">
  <ul>
    <li><a href="/home">Home</a></li>
    <li><a href="/about">About</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>
```

### Form Controls

- All form fields must have associated labels
- Use explicit label elements with `for` attributes
- For visual-only labels, use `aria-label` or `aria-labelledby`
- Group related controls with `fieldset` and `legend`

```tsx
// Proper form labeling
<label htmlFor="email">Email address</label>
<input type="email" id="email" name="email" required />

// Fieldset for radio groups
<fieldset>
  <legend>Choose your preferred contact method</legend>
  <div>
    <input type="radio" id="email-contact" name="contact" value="email" />
    <label htmlFor="email-contact">Email</label>
  </div>
  <div>
    <input type="radio" id="phone-contact" name="contact" value="phone" />
    <label htmlFor="phone-contact">Phone</label>
  </div>
</fieldset>
```

## Screen Reader Compatibility

### ARIA Implementation

- Use semantic HTML first
- Apply ARIA roles, states, and properties only when necessary
- Ensure ARIA attributes are programmatically updated
- Use `aria-live` regions for dynamic content

```tsx
// Live region for notifications
<div aria-live="polite" aria-atomic="true">
  {notification && <div className="alert">{notification}</div>}
</div>

// Dynamic content updates
<div
  role="alert"
  aria-atomic="true"
  aria-live="assertive"
  className={notification.type === 'error' ? 'error' : 'success'}
>
  {notification.message}
</div>
```

### Landmark Regions

- Use appropriate ARIA landmarks: banner, navigation, main, complementary, contentinfo
- Ensure only one `role="main"` per page
- Provide unique labels for landmark regions when multiple exist

```tsx
// HTML5 semantic elements with ARIA fallbacks
<header role="banner">
  <h1>Site Name</h1>
</header>

<nav role="navigation" aria-label="Main navigation">
  <!-- navigation content -->
</nav>

<main id="main-content" role="main">
  <!-- main page content -->
</main>

<footer role="contentinfo">
  <!-- footer content -->
</footer>
```

## Color and Contrast

### Contrast Requirements

- Text and images of text: minimum 4.5:1 contrast ratio
- Large text (18pt+ or 14pt bold+): minimum 3:1 contrast ratio
- Non-text contrast (UI components, charts): minimum 3:1 contrast ratio
- Test in grayscale to ensure information isn't conveyed by color alone

### Color Usage

- Never use color as the only means of conveying information
- Provide text labels or patterns as alternatives
- Support both light and dark color schemes
- Respect `prefers-color-scheme` media query

```css
/* Respecting color scheme preference */
@media (prefers-color-scheme: dark) {
  :root {
    --surface-bg: #1a1a1a;
    --text-primary: #ffffff;
  }
}

/* High contrast mode support */
@media (prefers-contrast: high) {
  :root {
    --border-thick: 2px;
  }
}
```

## Motion and Animation

### Reduced Motion

- Respect `prefers-reduced-motion` media query
- Provide alternatives for motion-based interactions
- Eliminate non-essential animations
- Allow users to control animation duration

```tsx
// Conditional animation based on user preference
const useReducedMotion = () => {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReducedMotion(mediaQuery.matches);
    
    const handleChange = (e) => setPrefersReducedMotion(e.matches);
    mediaQuery.addEventListener('change', handleChange);
    
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);

  return prefersReducedMotion;
};

// Component using reduced motion preference
const AnimatedSection = () => {
  const prefersReducedMotion = useReducedMotion();
  
  return (
    <div
      className={prefersReducedMotion ? 'static-layout' : 'animated-layout'}
      style={{
        transition: prefersReducedMotion 
          ? 'none' 
          : 'transform 0.5s cubic-bezier(0.16, 1, 0.3, 1)'
      }}
    >
      {/* content */}
    </div>
  );
};
```

### Focus Indicators

- Visible focus styles on all interactive elements
- Do not remove `outline` property without providing equivalent visual indicator
- Ensure focus indicators have sufficient contrast
- Test keyboard navigation flow

```css
/* Preserve focus visibility */
*:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}

/* Alternative focus style if outline is removed */
button:focus {
  outline: none;
  box-shadow: 0 0 0 2px var(--color-accent);
}
```

## Testing Checklist

- [ ] All interactive elements are keyboard accessible
- [ ] Tab order follows logical sequence
- [ ] Skip links are present and functional
- [ ] All form fields have associated labels
- [ ] ARIA landmarks are properly implemented
- [ ] Semantic HTML is used appropriately
- [ ] Color contrast meets WCAG requirements
- [ ] Information is not conveyed by color alone
- [ ] `prefers-reduced-motion` is respected
- [ ] Focus indicators are visible and clear
- [ ] Screen reader testing performed with NVDA, VoiceOver
- [ ] Keyboard-only testing completed
- [ ] Automated accessibility scanning passed (axe, Lighthouse)