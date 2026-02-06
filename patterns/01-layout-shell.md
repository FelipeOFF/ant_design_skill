# Pattern 01: Layout Shell

## Problem / Context

Modern admin dashboards and web applications require a consistent layout structure with navigation, header, and content areas. The challenge is creating a responsive layout that collapses gracefully on mobile while maintaining usability across all screen sizes.

## When to Use

- Admin dashboards and back-office applications
- Multi-section applications with persistent navigation
- When you need a responsive sidebar that collapses on smaller screens

## When NOT to Use

- Single-page landing pages with minimal navigation
- Mobile-first apps where drawer navigation is preferred
- Simple content sites where header-only navigation suffices

## AntD Components Involved

- `Layout` - Container component for layout structure
- `Layout.Sider` - Collapsible sidebar container
- `Layout.Header` - Top navigation/header area
- `Layout.Content` - Main content area
- `Layout.Footer` - Optional footer section
- `Menu` - Navigation items within Sider/Header
- `Button` - Toggle button for mobile/desktop collapse

## React Implementation Notes

### State Management

```tsx
const [collapsed, setCollapsed] = useState(false);
const [mobileOpen, setMobileOpen] = useState(false);
const [isMobile, setIsMobile] = useState(false);
```

### Responsive Detection

Use a resize observer or media query hook to detect mobile breakpoints:

```tsx
useEffect(() => {
  const checkMobile = () => setIsMobile(window.innerWidth < 768);
  checkMobile();
  window.addEventListener('resize', checkMobile);
  return () => window.removeEventListener('resize', checkMobile);
}, []);
```

### Collapse Behavior

- **Desktop (>768px)**: Collapse sider to 80px icon-only mode
- **Mobile (≤768px)**: Hide sider completely, show overlay drawer when toggled

## Accessibility / Keyboard

- Sider toggle must be keyboard accessible (Tab + Enter/Space)
- Focus should not be lost when collapsing/expanding
- Mobile drawer should trap focus when open
- Provide `aria-expanded` on toggle button
- Use `aria-label` for icon-only menu items

## Do / Don't

| Do | Don't |
|----|-------|
| Keep sider width consistent (200px default) | Vary sider widths across pages |
| Show tooltips on collapsed icon-only items | Hide all navigation hints on collapse |
| Animate transitions smoothly | Use instant jarring transitions |
| Persist collapse state in localStorage | Reset on every page load |
| Use breakpoints consistently | Hardcode pixel values everywhere |

## Minimal Code Snippet

```tsx
import { useState, useEffect } from 'react';
import { Layout, Menu, Button } from 'antd';
import { MenuFoldOutlined, MenuUnfoldOutlined, DashboardOutlined, UserOutlined, SettingOutlined } from '@ant-design/icons';

const { Header, Sider, Content } = Layout;

export function AppLayout() {
  const [collapsed, setCollapsed] = useState(false);
  const [isMobile, setIsMobile] = useState(false);

  useEffect(() => {
    const checkMobile = () => {
      const mobile = window.innerWidth < 768;
      setIsMobile(mobile);
      if (mobile) setCollapsed(true);
    };
    checkMobile();
    window.addEventListener('resize', checkMobile);
    return () => window.removeEventListener('resize', checkMobile);
  }, []);

  const menuItems = [
    { key: 'dashboard', icon: <DashboardOutlined />, label: 'Dashboard' },
    { key: 'users', icon: <UserOutlined />, label: 'Users' },
    { key: 'settings', icon: <SettingOutlined />, label: 'Settings' },
  ];

  return (
    <Layout style={{ minHeight: '100vh' }}>
      <Sider 
        trigger={null} 
        collapsible 
        collapsed={collapsed}
        breakpoint="md"
        collapsedWidth={isMobile ? 0 : 80}
        style={{ 
          position: isMobile ? 'fixed' : 'relative',
          height: '100vh',
          zIndex: 1000,
          display: isMobile && collapsed ? 'none' : 'block'
        }}
      >
        <div style={{ height: 64, background: 'rgba(255,255,255,0.1)', margin: 0 }} />
        <Menu theme="dark" mode="inline" items={menuItems} defaultSelectedKeys={['dashboard']} />
      </Sider>
      
      <Layout>
        <Header style={{ padding: 0, background: '#fff', display: 'flex', alignItems: 'center' }}>
          <Button
            type="text"
            icon={collapsed ? <MenuUnfoldOutlined /> : <MenuFoldOutlined />}
            onClick={() => setCollapsed(!collapsed)}
            style={{ fontSize: 16, width: 64, height: 64 }}
            aria-label={collapsed ? "Expand sidebar" : "Collapse sidebar"}
            aria-expanded={!collapsed}
          />
          <span style={{ marginLeft: 16, fontSize: 18, fontWeight: 600 }}>My App</span>
        </Header>
        
        <Content style={{ margin: '24px 16px', padding: 24, background: '#fff', minHeight: 280 }}>
          Page content goes here
        </Content>
      </Layout>
      
      {isMobile && !collapsed && (
        <div 
          style={{
            position: 'fixed',
            top: 0, left: 0, right: 0, bottom: 0,
            background: 'rgba(0,0,0,0.5)',
            zIndex: 999
          }}
          onClick={() => setCollapsed(true)}
          aria-hidden="true"
        />
      )}
    </Layout>
  );
}
```
