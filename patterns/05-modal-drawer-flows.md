# Pattern 5: Modal/Drawer with Internal Forms

## Problem / Context
Editing or creating entities without navigating away from the current page. Requires handling visibility, loading states, and form submission from the parent context.

## When to use
- "Quick Edit" actions.
- Creation flows where context (background page) matters.
- Drawers are better for long forms or detail views. Modals for quick confirmations or short forms.

## When NOT to use
- Extremely complex workflows (wizards) - prefer a dedicated page or a full-screen mode.

## AntD Components
- `Modal` or `Drawer`
- `Form` (inside the modal)

## React Implementation Notes
- **Separation**: Create a separate component `UserEditModal` that accepts `open`, `onCancel`, `onSuccess`.
- **Form Reset**: Use `useEffect` inside the modal to `form.resetFields()` or `form.setFieldsValue(data)` when the `open` prop changes or `data` prop changes.
- **Submission**: The Modal's "OK" button should trigger the form's submit. This is tricky. Use `form.submit()` connected to the Modal `onOk`.

## Accessibility / Keyboard
- AntD manages focus trapping inside Modal/Drawer.
- Esc key should close the modal (default behavior).

## Do / Don't
- **Do**: Expose `confirmLoading` on the Modal to show async progress.
- **Do**: Use `destroyOnClose` if you want to reset the state completely on close.
- **Don't**: Put the Modal definitions inside a loop (e.g., inside `Table.render`). Render one Modal and switch the `activeRecord` state.

## Minimal Code Snippet

```tsx
import React, { useEffect, useState } from 'react';
import { Modal, Form, Input } from 'antd';

interface UserModalProps {
  open: boolean;
  onCreate: (values: any) => Promise<void>;
  onCancel: () => void;
  initialValues?: any;
}

export const UserFormModal: React.FC<UserModalProps> = ({ open, onCreate, onCancel, initialValues }) => {
  const [form] = Form.useForm();
  const [loading, setLoading] = useState(false);

  // Reset or set values when modal opens/changes
  useEffect(() => {
    if (open) {
        form.setFieldsValue(initialValues || { name: '', email: '' });
    } else {
        form.resetFields();
    }
  }, [open, initialValues, form]);

  const handleOk = async () => {
    try {
      const values = await form.validateFields();
      setLoading(true);
      await onCreate(values);
      setLoading(false);
    } catch (info) {
      console.log('Validate Failed:', info);
    }
  };

  return (
    <Modal
      open={open}
      title={initialValues ? "Edit User" : "Create User"}
      okText={initialValues ? "Update" : "Create"}
      cancelText="Cancel"
      onCancel={onCancel}
      onOk={handleOk}
      confirmLoading={loading}
    >
      <Form form={form} layout="vertical" name="user_form_in_modal">
        <Form.Item name="name" label="Name" rules={[{ required: true }]}>
          <Input />
        </Form.Item>
        <Form.Item name="email" label="Email" rules={[{ type: 'email' }]}>
          <Input />
        </Form.Item>
      </Form>
    </Modal>
  );
};
```
