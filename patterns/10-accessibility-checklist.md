# Pattern 10: Accessibility (A11y) Checklist

## Problem / Context
Ensuring Ant Design applications are usable by keyboard users, screen readers, and compliant with WCAG standards.

## When to use
- ALWAYS. Throughout development, not just at the end.

## AntD Components
- Most AntD components are accessible by default (keyboard support, roles).
- **Attention needed**: `Tooltip`, `Modal`, `Dropdown` (focus management), `Form` (labelling).

## React Implementation Notes
- **Focus Management**: When a Modal closes, where does focus go? (AntD handles return focus usually, but verify).
- **Semantic HTML**: Don't use `<div>` for buttons. AntD uses correct tags, but your wrappers might not.

## Checklist

### 1. Keyboard Navigation
- [ ] Can you Tab through all interactive elements?
- [ ] Is the focus outline visible? (Do not remove `outline` via CSS without replacement).
- [ ] Do `Enter` and `Space` activate buttons/links?

### 2. Forms & Labels
- [ ] Does every `Input` have a label?
- [ ] If visual design hides the label, use `aria-label` or `aria-labelledby`.
- [ ] Are error messages linked via `aria-describedby`? (AntD Form does this).

### 3. Modals & Popovers
- [ ] Does focus get trapped inside the Modal when open?
- [ ] Does `Esc` close the overlay?

### 4. Color Contrast
- [ ] Do text/background combinations meet WCAG AA (4.5:1)?
- [ ] Note: Standard AntD Blue on White is compliant. Custom themes might not be.

## Do / Don't
- **Do**: Use `<ConfigProvider theme={{ cssVar: true }}>` if you need CSS variables for high-contrast mode tweaking.
- **Do**: Test with a screen reader (NVDA, VoiceOver) at least once per feature.
- **Don't**: Use `Tooltip` for vital instructions that screen readers might miss if not focused/hovered. Use visible text or `aria-describedby`.

## Minimal Code Snippet (Manual Fixes)

```tsx
import React, { useRef, useEffect } from 'react';
import { Button, Input } from 'antd';

export const AccessibleSearch = () => {
  const inputRef = useRef<any>(null);

  // Focus management example
  useEffect(() => {
    // Focus input on mount
    inputRef.current?.focus();
  }, []);

  return (
    <div role="search">
      {/* 
         If no visible label, provide aria-label.
         AntD Input passes rest props to the native input 
      */}
      <Input 
        ref={inputRef}
        placeholder="Search..." 
        aria-label="Global Site Search" 
      />
      <Button onClick={() => {}}>Search</Button>
    </div>
  );
};
```
