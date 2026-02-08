# Pattern 1: Layout Shell (Responsive)

## Problem / Context
Applications need a consistent structural wrapper that handles header, sidebar, content area, and footer, while adapting to different screen sizes (mobile/desktop).

## When to use
- Every dashboard or admin application.
- Apps requiring persistent navigation.

## When NOT to use
- Marketing landing pages (use simple CSS/Divs).
- Single-page tools with no navigation structure.

## AntD Components
- `Layout` (Wrapper)
- `Layout.Header`
- `Layout.Sider`
- `Layout.Content`
- `Layout.Footer`

## React Implementation Notes
- Use `useState` for `collapsed` state of the Sider.
- Use `onBreakpoint` prop on Sider for automatic responsiveness.
- Wrap the Layout in a context provider if navigation state needs to be shared globaly, or keep it local to the shell component.
- Use CSS Modules or styled-components to set `minHeight: 100vh`.

## Accessibility / Keyboard
- Ensure the `Sider` trigger is keyboard accessible (AntD handles this default).
- Use proper semantic HTML5 tags (AntD outputs `<header>`, `<main>`, `<footer>` if configured or use semantic elements inside).
- Skip-to-content link should be added manually before the Layout for better a11y.

## Do / Don't
- **Do**: Set a `minHeight: 100vh` on the outer Layout to prevent footer floating.
- **Do**: Use `trigger={null}` if you implement a custom toggle button in the Header.
- **Don't**: Mix multiple nested Layouts unnecessarily; keep the tree shallow.

## Minimal Code Snippet

```tsx
import React, { useState } from 'react';
import { Layout, Menu, Button, theme } from 'antd';
import { MenuFoldOutlined, MenuUnfoldOutlined } from '@ant-design/icons';

const { Header, Sider, Content } = Layout;

export const AppShell: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [collapsed, setCollapsed] = useState(false);
  const {
    token: { colorBgContainer, borderRadiusLG },
  } = theme.useToken();

  return (
    <Layout style={{ minHeight: '100vh' }}>
      <Sider trigger={null} collapsible collapsed={collapsed} breakpoint="lg" onBreakpoint={setCollapsed}>
        <div className="logo" style={{ height: 32, margin: 16, background: 'rgba(255,255,255,0.2)' }} />
        <Menu theme="dark" mode="inline" defaultSelectedKeys={['1']} items={[{ key: '1', label: 'Home' }]} />
      </Sider>
      <Layout>
        <Header style={{ padding: 0, background: colorBgContainer }}>
          <Button
            type="text"
            icon={collapsed ? <MenuUnfoldOutlined /> : <MenuFoldOutlined />}
            onClick={() => setCollapsed(!collapsed)}
            style={{ fontSize: '16px', width: 64, height: 64 }}
          />
        </Header>
        <Content
          style={{
            margin: '24px 16px',
            padding: 24,
            minHeight: 280,
            background: colorBgContainer,
            borderRadius: borderRadiusLG,
          }}
        >
          {children}
        </Content>
      </Layout>
    </Layout>
  );
};
```
