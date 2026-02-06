# Pattern 09: Theming

## Problem / Context

Applications need consistent visual styling that can adapt to user preferences (light/dark mode) and brand requirements. Design tokens provide a single source of truth for colors, spacing, and typography.

## When to Use

- Supporting light/dark mode toggle
- White-label/multi-tenant applications
- Maintaining consistent design across components
- Customizing Ant Design to match brand

## When NOT to Use

- For one-off style overrides (use CSS-in-JS or style prop)
- When no customization is needed (use default theme)

## AntD Components Involved

- `ConfigProvider` - Global theme configuration
- `theme` - Theme utilities from antd
- `Button`, `Switch` - Theme toggle controls

## React Implementation Notes

### Basic ConfigProvider

```tsx
import { ConfigProvider, theme } from 'antd';

<ConfigProvider
  theme={{
    token: {
      colorPrimary: '#1890ff',
      borderRadius: 4,
    },
  }}
>
  <App />
</ConfigProvider>
```

### Theme Tokens

Key tokens to customize:

```tsx
const customTokens = {
  // Colors
  colorPrimary: '#1890ff',
  colorSuccess: '#52c41a',
  colorWarning: '#faad14',
  colorError: '#f5222d',
  colorInfo: '#1890ff',
  
  // Typography
  fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto',
  fontSize: 14,
  
  // Spacing
  paddingSM: 12,
  padding: 16,
  paddingLG: 24,
  
  // Border
  borderRadius: 6,
  colorBorder: '#d9d9d9',
};
```

### Algorithm for Dark Mode

```tsx
import { theme } from 'antd';
const { darkAlgorithm, defaultAlgorithm } = theme;

const [isDark, setIsDark] = useState(false);

<ConfigProvider
  theme={{
    algorithm: isDark ? darkAlgorithm : defaultAlgorithm,
    token: {
      colorPrimary: '#1890ff',
    },
  }}
>
```

### Persisting User Preference

```tsx
const [isDark, setIsDark] = useState(() => {
  return localStorage.getItem('theme') === 'dark' ||
    (!localStorage.getItem('theme') && 
     window.matchMedia('(prefers-color-scheme: dark)').matches);
});

useEffect(() => {
  localStorage.setItem('theme', isDark ? 'dark' : 'light');
}, [isDark]);
```

### Component-Specific Overrides

```tsx
const components = {
  Button: {
    colorPrimary: '#00b96b',
    algorithm: true, // Enable algorithmic color generation
  },
  Table: {
    headerBg: '#fafafa',
  },
};
```

## Accessibility / Keyboard

- Ensure sufficient color contrast in both themes
- Don't rely solely on color to convey meaning
- Test focus indicators in both light and dark modes
- Respect `prefers-color-scheme` media query

## Do / Don't

| Do | Don't |
|----|-------|
| Use design tokens consistently | Hardcode colors in components |
| Test both themes thoroughly | Only test one theme |
| Provide manual toggle + system preference | Force one theme always |
| Use algorithm for derived colors | Manually set every shade |
| Persist user preference | Reset on every reload |
| Animate theme transitions | Switch instantly (jarring) |

## Minimal Code Snippet

```tsx
import { useState, useEffect } from 'react';
import { ConfigProvider, Button, Switch, Card, Space, theme } from 'antd';
import { MoonOutlined, SunOutlined } from '@ant-design/icons';

const { darkAlgorithm, defaultAlgorithm } = theme;

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [isDark, setIsDark] = useState(() => {
    // Check localStorage first, then system preference
    const saved = localStorage.getItem('theme');
    if (saved) return saved === 'dark';
    return window.matchMedia('(prefers-color-scheme: dark)').matches;
  });

  useEffect(() => {
    localStorage.setItem('theme', isDark ? 'dark' : 'light');
  }, [isDark]);

  const customTokens = {
    colorPrimary: '#722ed1',
    colorSuccess: '#52c41a',
    colorWarning: '#faad14',
    colorError: '#f5222d',
    borderRadius: 8,
  };

  return (
    <ConfigProvider
      theme={{
        algorithm: isDark ? darkAlgorithm : defaultAlgorithm,
        token: customTokens,
        components: {
          Button: {
            colorPrimary: '#722ed1',
            algorithm: true,
          },
          Card: {
            borderRadius: 12,
          },
        },
      }}
    >
      <div style={{ 
        minHeight: '100vh', 
        background: isDark ? '#141414' : '#f0f2f5',
        transition: 'background 0.3s ease'
      }}>
        <Space style={{ padding: 16 }}>
          <Switch
            checked={isDark}
            onChange={setIsDark}
            checkedChildren={<MoonOutlined />}
            unCheckedChildren={<SunOutlined />}
          />
          <span>{isDark ? 'Dark' : 'Light'} Mode</span>
        </Space>
        {children}
      </div>
    </ConfigProvider>
  );
}

// Usage example
export function ThemedApp() {
  return (
    <ThemeProvider>
      <div style={{ padding: 24 }}>
        <Card title="Theme Demo" style={{ maxWidth: 600, margin: '0 auto' }}>
          <Space direction="vertical">
            <Button type="primary">Primary Button</Button>
            <Button>Default Button</Button>
            <Button type="dashed">Dashed Button</Button>
            <Button danger>Danger Button</Button>
          </Space>
        </Card>
      </div>
    </ThemeProvider>
  );
}
```
