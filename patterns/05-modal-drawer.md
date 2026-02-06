# Pattern 05: Modal / Drawer

## Problem / Context

Modals and Drawers provide focused contexts for forms, confirmations, and detail views without navigating away from the current page. The challenge is managing their state, animations, and focus correctly.

## When to Use

- **Modal**: Critical confirmations, small forms (≤3 fields), alerts
- **Drawer**: Complex forms, detail views, multi-step wizards, lists

## When NOT to Use

- For content that should have a dedicated URL
- When users need to reference background content while editing
- For very long scrolling content (use full page)

## AntD Components Involved

- `Modal` - Centered dialog overlay
- `Drawer` - Slide-in panel from edge
- `Form` - Forms inside modals/drawers
- `Space`, `Button` - Action buttons in footer
- `Spin` - Loading state for async operations

## React Implementation Notes

### State Management Pattern

```tsx
const [isOpen, setIsOpen] = useState(false);
const [record, setRecord] = useState<Record | null>(null);

const openModal = (item?: Record) => {
  setRecord(item || null);
  setIsOpen(true);
};

const closeModal = () => {
  setIsOpen(false);
  // Optional: delay clearing record for exit animation
  setTimeout(() => setRecord(null), 300);
};
```

### Form in Modal Pattern

```tsx
const [form] = Form.useForm();

// Reset form when modal opens with new data
useEffect(() => {
  if (isOpen) {
    if (record) {
      form.setFieldsValue(record);
    } else {
      form.resetFields();
    }
  }
}, [isOpen, record, form]);

const handleOk = async () => {
  const values = await form.validateFields();
  await submit(values);
  closeModal();
};
```

### Async Submit with Loading

```tsx
const [confirmLoading, setConfirmLoading] = useState(false);

const handleOk = async () => {
  setConfirmLoading(true);
  try {
    await handleSubmit();
  } finally {
    setConfirmLoading(false);
  }
};
```

## Accessibility / Keyboard

- Focus must be trapped inside modal/drawer when open
- `ESC` key should close (unless confirm in progress)
- Return focus to trigger element on close
- First focusable element should receive focus on open
- Use `aria-labelledby` pointing to title

## Do / Don't

| Do | Don't |
|----|-------|
| Close on backdrop click for non-critical modals | Close on backdrop for destructive confirmations |
| Show loading state during async operations | Allow double-submit while loading |
| Animate open/close smoothly | Appear/disappear instantly |
| Reset form when opening for create | Show previous user's data |
| Use Drawer for forms with many fields | Cram 10+ fields into a Modal |
| Disable primary action while loading | Allow interaction during submit |

## Minimal Code Snippet

```tsx
import { useState, useEffect } from 'react';
import { Modal, Drawer, Form, Input, Button, Space, message } from 'antd';

interface User {
  id?: string;
  name: string;
  email: string;
  phone: string;
}

export function UserModalExample() {
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [isDrawerOpen, setIsDrawerOpen] = useState(false);
  const [editingUser, setEditingUser] = useState<User | null>(null);
  const [confirmLoading, setConfirmLoading] = useState(false);
  const [form] = Form.useForm();

  // Reset form when modal opens
  useEffect(() => {
    if (isModalOpen || isDrawerOpen) {
      if (editingUser) {
        form.setFieldsValue(editingUser);
      } else {
        form.resetFields();
      }
    }
  }, [isModalOpen, isDrawerOpen, editingUser, form]);

  const openModal = (user?: User) => {
    setEditingUser(user || null);
    setIsModalOpen(true);
  };

  const openDrawer = (user?: User) => {
    setEditingUser(user || null);
    setIsDrawerOpen(true);
  };

  const closeModal = () => {
    setIsModalOpen(false);
    setTimeout(() => setEditingUser(null), 300);
  };

  const closeDrawer = () => {
    setIsDrawerOpen(false);
    setTimeout(() => setEditingUser(null), 300);
  };

  const handleSubmit = async () => {
    try {
      const values = await form.validateFields();
      setConfirmLoading(true);

      // Simulate API call
      await new Promise((resolve) => setTimeout(resolve, 1000));
      
      message.success(editingUser ? 'User updated' : 'User created');
      closeModal();
      closeDrawer();
    } catch (error) {
      console.error('Validation failed:', error);
    } finally {
      setConfirmLoading(false);
    }
  };

  const formContent = (
    <Form form={form} layout="vertical">
      <Form.Item
        name="name"
        label="Name"
        rules={[{ required: true, message: 'Please enter name' }]}
      >
        <Input placeholder="John Doe" />
      </Form.Item>
      <Form.Item
        name="email"
        label="Email"
        rules={[
          { required: true, message: 'Please enter email' },
          { type: 'email', message: 'Invalid email' },
        ]}
      >
        <Input placeholder="john@example.com" />
      </Form.Item>
      <Form.Item name="phone" label="Phone">
        <Input placeholder="+1 234 567 890" />
      </Form.Item>
    </Form>
  );

  return (
    <div>
      <Space>
        <Button type="primary" onClick={() => openModal()}>
          New User (Modal)
        </Button>
        <Button onClick={() => openDrawer()}>
          New User (Drawer)
        </Button>
      </Space>

      {/* Modal for simple forms */}
      <Modal
        title={editingUser ? 'Edit User' : 'Create User'}
        open={isModalOpen}
        onOk={handleSubmit}
        onCancel={closeModal}
        confirmLoading={confirmLoading}
        destroyOnClose
      >
        {formContent}
      </Modal>

      {/* Drawer for complex forms */}
      <Drawer
        title={editingUser ? 'Edit User' : 'Create User'}
        open={isDrawerOpen}
        onClose={closeDrawer}
        width={400}
        footer={
          <Space>
            <Button onClick={closeDrawer}>Cancel</Button>
            <Button type="primary" loading={confirmLoading} onClick={handleSubmit}>
              Save
            </Button>
          </Space>
        }
        destroyOnClose
      >
        {formContent}
      </Drawer>
    </div>
  );
}
```
