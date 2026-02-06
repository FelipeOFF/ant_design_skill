# Pattern 03: Forms

## Problem / Context

Forms are the primary way users input data. The challenge is handling validation, async submission, loading states, and error presentation in a consistent, user-friendly way.

## When to Use

- Data entry and editing interfaces
- Search and filter panels
- Configuration and settings forms
- Any user input requiring validation

## When NOT to Use

- Simple one-field inputs (use standalone Input)
- Real-time filtering without submission
- Read-only data display

## AntD Components Involved

- `Form` - Form container and state management
- `Form.Item` - Field wrapper with validation
- `Input`, `Input.Password`, `Input.TextArea` - Text inputs
- `Select`, `DatePicker`, `Checkbox`, `Radio` - Selection inputs
- `Button` - Submit actions
- `Alert` - Error summary display

## React Implementation Notes

### Form Instance

```tsx
const [form] = Form.useForm();
```

### Validation Rules

```tsx
const rules = {
  email: [
    { required: true, message: 'Email is required' },
    { type: 'email', message: 'Invalid email format' },
  ],
  password: [
    { required: true, message: 'Password is required' },
    { min: 8, message: 'Password must be at least 8 characters' },
  ],
};
```

### Async Submit Pattern

```tsx
const [submitting, setSubmitting] = useState(false);
const [submitError, setSubmitError] = useState<string | null>(null);

const handleSubmit = async (values: FormValues) => {
  setSubmitting(true);
  setSubmitError(null);
  
  try {
    await api.submitForm(values);
    message.success('Form submitted successfully');
    form.resetFields();
  } catch (error) {
    setSubmitError(error.message);
    // Optionally set field-specific errors
    form.setFields([
      { name: 'email', errors: ['This email is already registered'] },
    ]);
  } finally {
    setSubmitting(false);
  }
};
```

### Error Summary Pattern

Display a summary Alert above the form when submission fails:

```tsx
{submitError && (
  <Alert 
    type="error" 
    message="Submission failed" 
    description={submitError}
    style={{ marginBottom: 24 }}
    closable
    onClose={() => setSubmitError(null)}
  />
)}
```

## Accessibility / Keyboard

- All inputs must have associated labels
- Error messages linked via `aria-describedby`
- Submit on Enter key in single-field forms
- Focus management after validation errors
- Clear error announcements for screen readers

## Do / Don't

| Do | Don't |
|----|-------|
| Disable submit button during submission | Allow double-submit clicks |
| Show field-level and form-level errors | Only show console errors |
| Reset form after successful submit | Keep dirty data after success |
| Validate on blur for touched fields | Validate everything on every keystroke |
| Use `name` prop matching API fields | Map form names to API separately |
| Show loading state on submit | Leave user wondering if click worked |

## Minimal Code Snippet

```tsx
import { useState } from 'react';
import { Form, Input, Button, Alert, message } from 'antd';

interface FormValues {
  email: string;
  password: string;
  confirmPassword: string;
}

export function RegistrationForm() {
  const [form] = Form.useForm<FormValues>();
  const [submitting, setSubmitting] = useState(false);
  const [submitError, setSubmitError] = useState<string | null>(null);

  const handleSubmit = async (values: FormValues) => {
    setSubmitting(true);
    setSubmitError(null);

    try {
      // Simulate API call
      await new Promise((resolve, reject) => {
        setTimeout(() => {
          if (values.email === 'exists@example.com') {
            reject(new Error('Email already registered'));
          } else {
            resolve(null);
          }
        }, 1000);
      });

      message.success('Registration successful!');
      form.resetFields();
    } catch (error) {
      const message = error instanceof Error ? error.message : 'Unknown error';
      setSubmitError(message);
      
      // Set field-specific error if it's an email conflict
      if (message.includes('Email')) {
        form.setFields([{ name: 'email', errors: [message] }]);
      }
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <Form
      form={form}
      layout="vertical"
      onFinish={handleSubmit}
      autoComplete="off"
      style={{ maxWidth: 400 }}
    >
      {submitError && (
        <Alert
          type="error"
          message="Registration failed"
          description={submitError}
          style={{ marginBottom: 24 }}
          closable
          onClose={() => setSubmitError(null)}
        />
      )}

      <Form.Item
        label="Email"
        name="email"
        rules={[
          { required: true, message: 'Please enter your email' },
          { type: 'email', message: 'Please enter a valid email' },
        ]}
      >
        <Input placeholder="you@example.com" />
      </Form.Item>

      <Form.Item
        label="Password"
        name="password"
        rules={[
          { required: true, message: 'Please enter a password' },
          { min: 8, message: 'Password must be at least 8 characters' },
        ]}
      >
        <Input.Password placeholder="••••••••" />
      </Form.Item>

      <Form.Item
        label="Confirm Password"
        name="confirmPassword"
        dependencies={['password']}
        rules={[
          { required: true, message: 'Please confirm your password' },
          ({ getFieldValue }) => ({
            validator(_, value) {
              if (!value || getFieldValue('password') === value) {
                return Promise.resolve();
              }
              return Promise.reject(new Error('Passwords do not match'));
            },
          }),
        ]}
      >
        <Input.Password placeholder="••••••••" />
      </Form.Item>

      <Form.Item>
        <Button type="primary" htmlType="submit" loading={submitting} block>
          Register
        </Button>
      </Form.Item>
    </Form>
  );
}
```
