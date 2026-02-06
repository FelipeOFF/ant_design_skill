# Pattern 02: Navigation Menus

## Problem / Context

Applications need navigation that stays synchronized with the current URL/route. Users expect visual feedback about their current location, and navigation should update when they use browser back/forward buttons.

## When to Use

- Multi-page applications with routing
- When users need to understand "where am I?"
- Apps where deep-linking to specific sections is required

## When NOT to Use

- Single-page apps with only one view
- Wizard/stepper flows where linear progression is enforced
- Contextual actions that change based on content selection

## AntD Components Involved

- `Menu` - Navigation container
- `Menu.Item` / `items` prop - Individual navigation items
- `Menu.SubMenu` - Nested navigation groups
- `Badge` - Notification counts on menu items

## React Implementation Notes

### URL Synchronization

The selected keys should derive from the current URL, not local state:

```tsx
const location = useLocation(); // react-router
const selectedKeys = [location.pathname.split('/')[1] || 'home'];
```

### Handling Navigation

Use your router's navigation method, not `window.location`:

```tsx
const navigate = useNavigate();

const handleMenuClick = ({ key }: { key: string }) => {
  navigate(`/${key}`);
};
```

### Open Keys Management

For submenus, control which sections stay expanded:

```tsx
const [openKeys, setOpenKeys] = useState<string[]>(['settings']);

const handleOpenChange = (keys: string[]) => {
  // Optional: accordion mode - only one submenu open
  setOpenKeys(keys.length ? [keys[keys.length - 1]] : []);
};
```

## Accessibility / Keyboard

- Menu should be navigable via arrow keys when focused
- Selected item should have `aria-current="page"`
- Submenu toggles should have `aria-expanded`
- Focus visible state must be clearly distinguishable

## Do / Don't

| Do | Don't |
|----|-------|
| Derive selected key from URL | Store selected key in React state only |
| Handle external navigation (back button) | Break browser history |
| Use `key` values that match route segments | Use display labels as keys |
| Provide fallback for unmatched routes | Leave menu unselected on 404 pages |
| Group related items with SubMenu | Flatten everything into one list |

## Minimal Code Snippet

```tsx
import { useState, useEffect } from 'react';
import { useLocation, useNavigate } from 'react-router-dom';
import { Menu, Badge } from 'antd';
import { DashboardOutlined, UserOutlined, SettingOutlined, MailOutlined } from '@ant-design/icons';
import type { MenuProps } from 'antd';

type MenuItem = Required<MenuProps>['items'][number];

export function NavigationMenu() {
  const location = useLocation();
  const navigate = useNavigate();
  const [openKeys, setOpenKeys] = useState<string[]>([]);

  // Derive selected key from current path
  const pathSegment = location.pathname.split('/')[1] || 'dashboard';
  const selectedKeys = [pathSegment];

  const items: MenuItem[] = [
    {
      key: 'dashboard',
      icon: <DashboardOutlined />,
      label: 'Dashboard',
    },
    {
      key: 'users',
      icon: <UserOutlined />,
      label: 'Users',
    },
    {
      key: 'messages',
      icon: <Badge count={5} size="small"><MailOutlined /></Badge>,
      label: 'Messages',
    },
    {
      key: 'settings',
      icon: <SettingOutlined />,
      label: 'Settings',
      children: [
        { key: 'settings/profile', label: 'Profile' },
        { key: 'settings/security', label: 'Security' },
        { key: 'settings/notifications', label: 'Notifications' },
      ],
    },
  ];

  const handleClick: MenuProps['onClick'] = ({ key }) => {
    navigate(`/${key}`);
  };

  const handleOpenChange = (keys: string[]) => {
    setOpenKeys(keys);
  };

  // Restore open keys from localStorage on mount
  useEffect(() => {
    const saved = localStorage.getItem('menuOpenKeys');
    if (saved) {
      try {
        setOpenKeys(JSON.parse(saved));
      } catch {}
    }
  }, []);

  // Persist open keys
  useEffect(() => {
    localStorage.setItem('menuOpenKeys', JSON.stringify(openKeys));
  }, [openKeys]);

  return (
    <Menu
      mode="inline"
      theme="dark"
      selectedKeys={selectedKeys}
      openKeys={openKeys}
      onOpenChange={handleOpenChange}
      onClick={handleClick}
      items={items}
    />
  );
}
```
