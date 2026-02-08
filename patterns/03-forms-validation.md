# Pattern 3: Robust Forms (Validation & Async)

## Problem / Context
Forms require validation, loading states during submission, error handling, and initial data population (edit mode).

## When to use
- Login/Register screens.
- CRUD creation/edit modals.
- Settings pages.

## When NOT to use
- Simple search inputs (use Input.Search).
- Non-data entry interactions.

## AntD Components
- `Form`
- `Form.Item`
- `Input`, `Select`, `DatePicker`, etc.

## React Implementation Notes
- Use `Form.useForm()` to get the form instance.
- **Async Submit**: The `onFinish` handler should return a Promise or be `async`.
- **Loading State**: Bind the submit button's `loading` prop to a state variable (e.g., `isSubmitting`).
- **Initial Values**: Use `initialValues` for static defaults, or `form.setFieldsValue` inside a `useEffect` for data fetched asynchronously.

## Accessibility / Keyboard
- Use `htmlType="submit"` on the primary button to enable "Enter to submit".
- Ensure `label` prop is used on `Form.Item` for correct `aria-label` / `for` attribute association.
- Error messages are automatically linked via `aria-describedby` by AntD.

## Do / Don't
- **Do**: Use `rules={[{ required: true }]}` for declarative validation.
- **Do**: Reset fields with `form.resetFields()` after successful submission if it's a "create new" flow.
- **Don't**: Manually manage input value state (`value={state}`) unless you need real-time control (controlled components). AntD Forms work best as uncontrolled or "form-controlled" components.

## Minimal Code Snippet

```tsx
import React, { useState } from 'react';
import { Form, Input, Button, message } from 'antd';

interface FormValues {
  username: string;
  email: string;
}

export const RegisterForm: React.FC = () => {
  const [form] = Form.useForm();
  const [loading, setLoading] = useState(false);

  const onFinish = async (values: FormValues) => {
    setLoading(true);
    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 1000));
      console.log('Success:', values);
      message.success('Registered successfully!');
      form.resetFields();
    } catch (error) {
      message.error('Registration failed.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <Form form={form} layout="vertical" onFinish={onFinish} autoComplete="off">
      <Form.Item
        name="username"
        label="Username"
        rules={[{ required: true, message: 'Please input your username!' }]}
      >
        <Input />
      </Form.Item>
      
      <Form.Item
        name="email"
        label="Email"
        rules={[{ type: 'email', message: 'The input is not valid E-mail!' }]}
      >
        <Input />
      </Form.Item>

      <Form.Item>
        <Button type="primary" htmlType="submit" loading={loading} block>
          Register
        </Button>
      </Form.Item>
    </Form>
  );
};
```
