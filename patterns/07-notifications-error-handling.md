# Pattern 7: Notification & Error Handling

## Problem / Context
Providing consistent feedback for user actions (success, error, warning) and handling global API errors.

## When to use
- Post-action feedback (Saved, Deleted).
- System alerts (Connection lost).
- Form validation errors (if not inline).

## When NOT to use
- Critical confirmation required (use `Modal.confirm`).
- Persistent status that shouldn't disappear (use `Alert` component).

## AntD Components
- `message` (Global, centered toast)
- `notification` (Top-right corner, richer content)
- `App` (New in v5 for context-aware static methods)

## React Implementation Notes
- **v5 Context**: Wrap your app in `<App>` to use `App.useApp()` hooks (`message`, `notification`, `modal`). This fixes context issues (e.g., themes not applying to static methods).
- **Centralize**: Create a helper/hook `useNotify` or interceptors (Axios) to trigger these standardly.

## Accessibility / Keyboard
- Messages should be screen-reader announcements (`role="alert"` or `status`). AntD handles this.
- Duration: Don't auto-dismiss critical errors too quickly. Allow user to close them.

## Do / Don't
- **Do**: Use `message` for simple one-liners ("Saved!").
- **Do**: Use `notification` for complex errors with stack traces or descriptions.
- **Don't**: Overuse `duration: 0` (sticky) unless user interaction is strictly required to dismiss.

## Minimal Code Snippet

```tsx
import React from 'react';
import { Button, App, Space } from 'antd';

// IMPORTANT: This component must be inside an <App> wrapper
export const FeedbackExample: React.FC = () => {
  const { message, notification, modal } = App.useApp();

  const handleSuccess = () => {
    message.success('Data saved successfully!');
  };

  const handleError = () => {
    notification.error({
      message: 'Submission Failed',
      description: 'The server responded with 500. Please try again later.',
      placement: 'topRight',
    });
  };

  const handleConfirm = () => {
    modal.confirm({
      title: 'Are you sure?',
      content: 'This action cannot be undone.',
      onOk: () => message.info('Deleted!'),
    });
  };

  return (
    <Space>
      <Button onClick={handleSuccess}>Show Success</Button>
      <Button danger onClick={handleError}>Show Error</Button>
      <Button type="dashed" onClick={handleConfirm}>Confirm Action</Button>
    </Space>
  );
};

/* 
Usage Root:
import { App } from 'antd';
<App>
  <FeedbackExample />
</App>
*/
```
