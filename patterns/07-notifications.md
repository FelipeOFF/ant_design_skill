# Pattern 07: Notifications

## Problem / Context

Users need feedback about action results, errors, and system events. Notifications must be informative without being intrusive, and errors need clear recovery paths.

## When to Use

- `message`: Brief success/error feedback (3s auto-dismiss)
- `notification`: Detailed information with actions
- `Modal.confirm`: Destructive action confirmations

## When NOT to Use

- For inline validation (use Form errors)
- For loading states in buttons (use Button loading)
- For persistent alerts (use Alert component)

## AntD Components Involved

- `message.success/error/warning/info/loading` - Brief toast notifications
- `notification.open` - Rich notifications with actions
- `Modal.confirm` - Confirmation dialogs
- `Alert` - Inline notification banners
- `Badge` - Count indicators
- `Progress` - Upload/operation progress

## React Implementation Notes

### Message Usage

```tsx
import { message } from 'antd';

// Simple usage
message.success('Item saved');

// With duration
message.error('Failed to save', 5);

// Loading that updates
const key = 'updatable';
message.loading({ content: 'Saving...', key });
await saveData();
message.success({ content: 'Saved!', key, duration: 2 });
```

### Notification with Actions

```tsx
import { notification, Button } from 'antd';

notification.open({
  message: 'Upload Complete',
  description: 'Your file has been processed successfully.',
  btn: (
    <Button type="primary" size="small" onClick={() => viewResults()}>
      View Results
    </Button>
  ),
  onClose: () => console.log('Notification closed'),
});
```

### Error Handling Pattern

```tsx
const handleError = (error: Error, context: string) => {
  console.error(`${context}:`, error);
  
  if (error.code === 'NETWORK_ERROR') {
    message.error('Network connection failed. Please check your connection.');
  } else if (error.code === 'UNAUTHORIZED') {
    message.error('Session expired. Please log in again.');
    redirectToLogin();
  } else {
    notification.error({
      message: 'An error occurred',
      description: error.message,
      duration: 0, // Don't auto-dismiss
    });
  }
};
```

### Global Error Boundary Integration

```tsx
// In your error boundary
componentDidCatch(error, errorInfo) {
  notification.error({
    message: 'Application Error',
    description: 'Something went wrong. Please refresh the page.',
    duration: 0,
  });
}
```

## Accessibility / Keyboard

- Notifications should be announced via ARIA live regions
- Action buttons in notifications must be keyboard accessible
- Error messages should describe how to recover
- Don't auto-dismiss error notifications

## Do / Don't

| Do | Don't |
|----|-------|
| Use `message` for brief feedback | Use `notification` for one-word messages |
| Keep messages under 60 characters | Write paragraphs in toast messages |
| Position consistently (usually top-right) | Jump positions randomly |
| Allow manual dismissal of important notices | Auto-dismiss critical errors |
| Use `key` to prevent duplicate messages | Show 10 identical messages |
| Include recovery actions in notifications | Leave users stuck after errors |

## Minimal Code Snippet

```tsx
import { message, notification, Modal, Button, Space } from 'antd';
import { CheckCircleOutlined, ExclamationCircleOutlined } from '@ant-design/icons';

export function NotificationExamples() {
  // Success message
  const showSuccess = () => {
    message.success('Changes saved successfully!');
  };

  // Error with description
  const showError = () => {
    notification.error({
      message: 'Failed to save changes',
      description: 'There was a problem connecting to the server. Please try again.',
      duration: 0,
    });
  };

  // Loading state that updates
  const showLoading = async () => {
    const key = 'loading';
    message.loading({ content: 'Processing...', key });
    
    await new Promise((resolve) => setTimeout(resolve, 2000));
    
    message.success({ content: 'Done!', key, duration: 2 });
  };

  // Notification with action
  const showActionNotification = () => {
    notification.open({
      message: 'New update available',
      description: 'A new version of the application is ready to install.',
      icon: <CheckCircleOutlined style={{ color: '#52c41a' }} />,
      btn: (
        <Space>
          <Button size="small" onClick={() => notification.destroy()}>
            Later
          </Button>
          <Button type="primary" size="small" onClick={() => {
            notification.destroy();
            message.success('Update installed!');
          }}>
            Install Now
          </Button>
        </Space>
      ),
      duration: 0,
    });
  };

  // Confirmation modal
  const showConfirm = () => {
    Modal.confirm({
      title: 'Delete this item?',
      icon: <ExclamationCircleOutlined />,
      content: 'This action cannot be undone. The item will be permanently removed.',
      okText: 'Delete',
      okType: 'danger',
      cancelText: 'Cancel',
      onOk: async () => {
        await new Promise((resolve) => setTimeout(resolve, 1000));
        message.success('Item deleted');
      },
    });
  };

  return (
    <Space direction="vertical">
      <Button onClick={showSuccess}>Show Success Message</Button>
      <Button onClick={showError} danger>Show Error Notification</Button>
      <Button onClick={showLoading}>Show Loading (2s)</Button>
      <Button onClick={showActionNotification}>Show Action Notification</Button>
      <Button onClick={showConfirm} danger>Show Confirm Dialog</Button>
    </Space>
  );
}
```
