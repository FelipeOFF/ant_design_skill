# Pattern 4: Smart Data Table (Pagination & Actions)

## Problem / Context
Displaying tabular data with server-side pagination, sorting, filtering, and row-level actions.

## When to use
- User lists, order history, inventory management.
- Any dataset larger than ~50 records.

## When NOT to use
- Small static lists (use `List` component).
- Complex grid layouts (use CSS Grid or `Card` list).

## AntD Components
- `Table`
- `Tag`, `Space`, `Button`, `Dropdown` (for actions)

## React Implementation Notes
- **State**: Keep `pagination`, `filters`, and `sorter` in state.
- **Fetching**: Use a `useEffect` that triggers when table params change.
- **TableProps**: Pass `loading`, `dataSource`, `pagination`, and `onChange` to the Table.
- **Columns**: Define columns outside the component or memoize them if they rely on closure state.

## Accessibility / Keyboard
- AntD Table handles pagination focus.
- Ensure action buttons have distinct labels (e.g., "Edit User 123" via `aria-label`) if the visual label is just an icon.

## Do / Don't
- **Do**: Map your backend pagination format (e.g., `page`, `limit`) to AntD's `current`, `pageSize` structure.
- **Do**: Use `rowKey="id"` to prevent render issues.
- **Don't**: Fetch all data and paginate client-side for large datasets; use server-side pagination logic.

## Minimal Code Snippet

```tsx
import React, { useEffect, useState } from 'react';
import { Table, Tag, Space, Button } from 'antd';
import type { TablePaginationConfig, TableProps } from 'antd/es/table';

interface DataType {
  id: string;
  name: string;
  role: string;
}

export const UserTable: React.FC = () => {
  const [data, setData] = useState<DataType[]>([]);
  const [loading, setLoading] = useState(false);
  const [pagination, setPagination] = useState<TablePaginationConfig>({ current: 1, pageSize: 10 });

  const fetchData = async (params: TablePaginationConfig) => {
    setLoading(true);
    // Simulate server call using params.current and params.pageSize
    setTimeout(() => {
        setData([{ id: '1', name: 'John Doe', role: 'Admin' }, { id: '2', name: 'Jane Smith', role: 'User' }]);
        setLoading(false);
        setPagination({ ...params, total: 50 }); // Update total from server response
    }, 500);
  };

  useEffect(() => {
    fetchData(pagination);
  }, []); // Mount only, or add deps if needed

  const handleTableChange: TableProps<DataType>['onChange'] = (newPagination) => {
    fetchData(newPagination);
    setPagination(newPagination);
  };

  const columns = [
    { title: 'Name', dataIndex: 'name', key: 'name' },
    { 
      title: 'Role', 
      dataIndex: 'role', 
      key: 'role',
      render: (role: string) => <Tag color={role === 'Admin' ? 'blue' : 'green'}>{role}</Tag> 
    },
    {
      title: 'Action',
      key: 'action',
      render: (_: any, record: DataType) => (
        <Space size="middle">
          <Button type="link">Edit</Button>
          <Button type="link" danger>Delete</Button>
        </Space>
      ),
    },
  ];

  return (
    <Table
      columns={columns}
      rowKey="id"
      dataSource={data}
      pagination={pagination}
      loading={loading}
      onChange={handleTableChange}
    />
  );
};
```
