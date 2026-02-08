# Pattern 9: Theming Strategy (ConfigProvider)

## Problem / Context
Customizing the look and feel (colors, radius, spacing) globally and switching between Light/Dark modes dynamically.

## When to use
- Branding requirements (Primary color isn't AntD Blue).
- Dark mode support.
- Component-specific overrides (e.g., all Buttons must be rounded).

## When NOT to use
- Inline styles (`style={{ ... }}`) for global changes.
- CSS `!important` overrides (Fragile).

## AntD Components
- `ConfigProvider`
- `theme` (algorithm: `defaultAlgorithm`, `darkAlgorithm`)

## React Implementation Notes
- Wrap the entire app (in `App.tsx` or `index.tsx`) with `ConfigProvider`.
- Use `theme.useToken()` hook in child components to access current design tokens (colors) instead of hardcoding hex values.

## Accessibility / Keyboard
- Dark mode should maintain sufficient contrast ratios. AntD's algorithm handles this generally, but verify custom colors.

## Do / Don't
- **Do**: Use Design Tokens (`colorPrimary`, `colorBgContainer`) for your custom components so they adapt to theme switches.
- **Do**: Nest `ConfigProvider` if a specific section needs a different theme (e.g., a dark sidebar in a light app).
- **Don't**: Overwrite CSS classes (`.ant-btn`) manually. Use the `token` or `components` prop in ConfigProvider.

## Minimal Code Snippet

```tsx
import React, { useState } from 'react';
import { ConfigProvider, Button, theme, Layout, Switch } from 'antd';

const { defaultAlgorithm, darkAlgorithm } = theme;

export const ThemeWrapper: React.FC = () => {
  const [isDark, setIsDark] = useState(false);

  return (
    <ConfigProvider
      theme={{
        algorithm: isDark ? darkAlgorithm : defaultAlgorithm,
        token: {
          colorPrimary: '#722ed1', // Custom Purple
          borderRadius: 2,
        },
        components: {
            Button: {
                fontWeight: 700
            }
        }
      }}
    >
      <AppContent isDark={isDark} toggleTheme={() => setIsDark(!isDark)} />
    </ConfigProvider>
  );
};

const AppContent: React.FC<{ isDark: boolean; toggleTheme: () => void }> = ({ isDark, toggleTheme }) => {
  const { token } = theme.useToken();
  
  return (
    <div style={{ background: token.colorBgContainer, padding: 50, height: '100vh', color: token.colorText }}>
      <h1>Theming Example</h1>
      <p>Current Primary Color: {token.colorPrimary}</p>
      <Button type="primary">Primary Button</Button>
      <div style={{ marginTop: 20 }}>
        <span>Dark Mode: </span>
        <Switch checked={isDark} onChange={toggleTheme} />
      </div>
    </div>
  );
};
```
