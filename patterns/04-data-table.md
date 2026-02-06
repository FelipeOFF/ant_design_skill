# Pattern 04: Data Table

## Problem / Context

Displaying large datasets requires efficient pagination, sorting, and filtering. Users need to perform actions on rows while understanding the current data state (loading, empty, error).

## When to Use

- Displaying tabular data with many rows
- When users need to sort and filter data
- When row-level actions are required
- Data that doesn't fit naturally in cards

## When NOT to Use

- Very small datasets (≤5 items) - use cards or list
- When mobile experience is primary (consider cards)
- For complex hierarchical data (consider tree)

## AntD Components Involved

- `Table` - Main data display component
- `Pagination` - Built-in or separate pagination
- `Space`, `Button` - Row action buttons
- `Tag`, `Badge` - Status indicators
- `Empty` - Empty state display
- `Spin` - Loading indicator
- `Tooltip` - Action button hints

## React Implementation Notes

### Controlled vs Uncontrolled

Prefer controlled mode for server-side data:

```tsx
const [data, setData] = useState([]);
const [loading, setLoading] = useState(false);
const [pagination, setPagination] = useState({ current: 1, pageSize: 10, total: 0 });
const [filters, setFilters] = useState({});
const [sorter, setSorter] = useState({});
```

### Server-Side Data Fetching

```tsx
const fetchData = async (params: TableParams) => {
  setLoading(true);
  try {
    const response = await api.getUsers({
      page: params.current,
      pageSize: params.pageSize,
      sortField: params.sorter?.field,
      sortOrder: params.sorter?.order,
      ...params.filters,
    });
    setData(response.data);
    setPagination({ ...pagination, total: response.total });
  } finally {
    setLoading(false);
  }
};
```

### Table Change Handler

```tsx
const handleTableChange: TableProps['onChange'] = (pagination, filters, sorter) => {
  setPagination(pagination);
  setFilters(filters);
  setSorter(sorter);
  fetchData({ pagination, filters, sorter });
};
```

## Accessibility / Keyboard

- Table headers must be sortable via keyboard
- Row actions must be focusable
- Provide row selection checkboxes with proper labels
- Empty state must be announced to screen readers

## Do / Don't

| Do | Don't |
|----|-------|
| Show loading state during data fetch | Show stale data while loading |
| Preserve scroll position on page change | Scroll to top unexpectedly |
| Use `rowKey` for stable row identity | Use array index as key |
| Handle empty state explicitly | Show blank or broken table |
| Debounce filter inputs | Filter on every keystroke |
| Show skeleton while loading first page | Show nothing while loading |

## Minimal Code Snippet

```tsx
import { useState, useEffect } from 'react';
import { Table, Button, Space, Tag, Tooltip, Empty, Spin } from 'antd';
import type { TableProps } from 'antd';
import { EditOutlined, DeleteOutlined, EyeOutlined } from '@ant-design/icons';

interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'editor';
  status: 'active' | 'inactive';
  createdAt: string;
}

interface TableParams {
  pagination: { current: number; pageSize: number };
  filters: Record<string, string[]>;
  sorter: { field?: string; order?: 'ascend' | 'descend' };
}

export function UserTable() {
  const [data, setData] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  const [pagination, setPagination] = useState({
    current: 1,
    pageSize: 10,
    total: 0,
  });

  const fetchData = async (page: number, pageSize: number) => {
    setLoading(true);
    try {
      // Simulate API call
      await new Promise((resolve) => setTimeout(resolve, 500));
      
      const mockData: User[] = Array.from({ length: pageSize }, (_, i) => ({
        id: `${page}-${i}`,
        name: `User ${(page - 1) * pageSize + i + 1}`,
        email: `user${(page - 1) * pageSize + i + 1}@example.com`,
        role: ['admin', 'user', 'editor'][Math.floor(Math.random() * 3)] as User['role'],
        status: Math.random() > 0.3 ? 'active' : 'inactive',
        createdAt: new Date().toISOString(),
      }));

      setData(mockData);
      setPagination((prev) => ({ ...prev, total: 100 }));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData(pagination.current, pagination.pageSize);
  }, []);

  const handleTableChange: TableProps<User>['onChange'] = (newPagination) => {
    setPagination(newPagination as typeof pagination);
    fetchData(newPagination.current!, newPagination.pageSize!);
  };

  const handleEdit = (user: User) => {
    console.log('Edit user:', user);
  };

  const handleDelete = (user: User) => {
    console.log('Delete user:', user);
  };

  const handleView = (user: User) => {
    console.log('View user:', user);
  };

  const columns: TableProps<User>['columns'] = [
    {
      title: 'Name',
      dataIndex: 'name',
      key: 'name',
      sorter: true,
    },
    {
      title: 'Email',
      dataIndex: 'email',
      key: 'email',
    },
    {
      title: 'Role',
      dataIndex: 'role',
      key: 'role',
      filters: [
        { text: 'Admin', value: 'admin' },
        { text: 'User', value: 'user' },
        { text: 'Editor', value: 'editor' },
      ],
      render: (role: string) => {
        const colors = { admin: 'red', user: 'blue', editor: 'green' };
        return <Tag color={colors[role as keyof typeof colors]}>{role.toUpperCase()}</Tag>;
      },
    },
    {
      title: 'Status',
      dataIndex: 'status',
      key: 'status',
      render: (status: string) => (
        <Tag color={status === 'active' ? 'success' : 'default'}>
          {status}
        </Tag>
      ),
    },
    {
      title: 'Actions',
      key: 'actions',
      render: (_, user) => (
        <Space size="small">
          <Tooltip title="View">
            <Button icon={<EyeOutlined />} onClick={() => handleView(user)} size="small" />
          </Tooltip>
          <Tooltip title="Edit">
            <Button icon={<EditOutlined />} onClick={() => handleEdit(user)} size="small" />
          </Tooltip>
          <Tooltip title="Delete">
            <Button 
              icon={<DeleteOutlined />} 
              danger 
              onClick={() => handleDelete(user)} 
              size="small" 
            />
          </Tooltip>
        </Space>
      ),
    },
  ];

  return (
    <Table
      rowKey="id"
      columns={columns}
      dataSource={data}
      loading={loading}
      pagination={pagination}
      onChange={handleTableChange}
      locale={{
        emptyText: (
          <Empty
            image={Empty.PRESENTED_IMAGE_SIMPLE}
            description="No users found"
          />
        ),
      }}
    />
  );
}
```
