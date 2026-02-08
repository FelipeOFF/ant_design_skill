# Pattern 8: Upload Management

## Problem / Context
Uploading files with preview, progress tracking, validation (type/size), and handling server responses.

## When to use
- Avatar changes.
- Document attachments.
- Bulk imports.

## When NOT to use
- Large file chunking (needs specialized logic, though AntD UI helps).

## AntD Components
- `Upload`
- `ImgCrop` (optional, for avatars)
- `Button`, `message`

## React Implementation Notes
- **Controlled vs Uncontrolled**: `fileList` prop makes it controlled. Prefer this for better state management.
- **Mocking**: Use `customRequest` to override the default XHR if using a library like Axios or doing client-side signing (S3).
- **Validation**: `beforeUpload` hook to check `file.type` and `file.size`.

## Accessibility / Keyboard
- Standard `<input type="file">` is hidden but accessible via the trigger button.
- Provide text alternatives for image previews.

## Do / Don't
- **Do**: Validate size/type *before* upload starts.
- **Do**: Handle the `removed` status to clean up server files if needed.
- **Don't**: Rely solely on the `action` prop URL if you need headers (Auth) - use `headers` prop or `customRequest`.

## Minimal Code Snippet

```tsx
import React, { useState } from 'react';
import { Upload, Button, message } from 'antd';
import { UploadOutlined } from '@ant-design/icons';
import type { UploadFile, UploadProps } from 'antd/es/upload/interface';

export const FileUploader: React.FC = () => {
  const [fileList, setFileList] = useState<UploadFile[]>([]);

  const handleChange: UploadProps['onChange'] = (info) => {
    let newFileList = [...info.fileList];

    // Limit to 1 file
    newFileList = newFileList.slice(-1);

    // Read response from server
    newFileList = newFileList.map((file) => {
      if (file.response) {
        // Component will show file.url as link
        file.url = file.response.url;
      }
      return file;
    });

    setFileList(newFileList);

    if (info.file.status === 'done') {
      message.success(`${info.file.name} file uploaded successfully`);
    } else if (info.file.status === 'error') {
      message.error(`${info.file.name} file upload failed.`);
    }
  };

  const props: UploadProps = {
    action: 'https://run.mocky.io/v3/435e224c-44fb-4773-9faf-380c5e6a2188',
    onChange: handleChange,
    multiple: false,
    fileList,
    beforeUpload: (file) => {
        const isPNG = file.type === 'image/png';
        if (!isPNG) {
            message.error(`${file.name} is not a png file`);
        }
        return isPNG || Upload.LIST_IGNORE;
    }
  };

  return (
    <Upload {...props}>
      <Button icon={<UploadOutlined />}>Click to Upload</Button>
    </Upload>
  );
};
```
