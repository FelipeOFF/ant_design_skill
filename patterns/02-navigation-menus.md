# Pattern 2: Navigation Menus (URL Sync)

## Problem / Context
Menu selection state needs to stay in sync with the browser URL (React Router) so that refreshing the page or navigating back/forward highlights the correct item.

## When to use
- Any app using `react-router-dom` or Next.js with an Ant Design `Menu`.

## When NOT to use
- Static menus that don't change routes (e.g., in-page settings toggle).

## AntD Components
- `Menu`
- `Menu.Item` (or `items` prop in v5)

## React Implementation Notes
- **Key Strategy**: Do NOT manage `selectedKeys` with local `useState` alone.
- Derive `selectedKeys` from `useLocation().pathname`.
- Use `useEffect` or direct derivation during render to match the current path to the menu key.
- Pass `openKeys` statefully if implementing an Accordion style sidebar.

## Accessibility / Keyboard
- AntD Menu handles arrow key navigation.
- Ensure menu items are semantic links (`<Link>` from router) or handle `onClick` to navigate. Ideally, wrap the label in a `<Link>` or use the `onClick` handler of the Menu to `navigate()`.

## Do / Don't
- **Do**: Use the `items` prop API (v5 recommended) instead of JSX children for better performance and type safety.
- **Do**: Create a helper function to map routes to keys if paths are complex.
- **Don't**: Force `selectedKeys` to a value that doesn't exist in the menu (causes warnings).

## Minimal Code Snippet

```tsx
import React, { useEffect, useState } from 'react';
import { Menu } from 'antd';
import { useLocation, useNavigate } from 'react-router-dom';
import type { MenuProps } from 'antd';

const items: MenuProps['items'] = [
  { key: '/dashboard', label: 'Dashboard' },
  { key: '/users', label: 'Users' },
  { key: '/settings', label: 'Settings' },
];

export const NavMenu: React.FC = () => {
  const location = useLocation();
  const navigate = useNavigate();
  const [current, setCurrent] = useState(location.pathname);

  // Sync state with URL
  useEffect(() => {
    if (location) {
        // Handle nested paths logic here if needed, e.g. /users/123 -> /users
        const activeKey = items.find(i => location.pathname.startsWith(String(i.key)))?.key || location.pathname;
        setCurrent(String(activeKey));
    }
  }, [location]);

  const onClick: MenuProps['onClick'] = (e) => {
    navigate(e.key);
  };

  return <Menu onClick={onClick} selectedKeys={[current]} mode="inline" items={items} />;
};
```
