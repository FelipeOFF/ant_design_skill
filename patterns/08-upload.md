# Pattern 08: Upload

## Problem / Context

File uploads require handling progress tracking, validation, preview, and error states. Users need clear feedback about upload status and the ability to retry failed uploads.

## When to Use

- Avatar/profile picture uploads
- Document attachments
- Bulk file imports
- Image galleries

## When NOT to Use

- For files that should be pasted/dragged directly into content
- When files need immediate server-side processing without user confirmation
- For extremely large files (>500MB) - consider chunked upload

## AntD Components Involved

- `Upload` - Main upload component
- `Upload.Dragger` - Drag-and-drop zone
- `Progress` - Upload progress indicator
- `Image` / `Avatar` - File previews
- `Button` - Upload trigger
- `message` - Upload status feedback

## React Implementation Notes

### Controlled Upload

```tsx
const [fileList, setFileList] = useState<UploadFile[]>([]);

const handleChange: UploadProps['onChange'] = ({ fileList: newFileList }) => {
  setFileList(newFileList);
};
```

### Custom Request

```tsx
const customRequest: UploadProps['customRequest'] = async (options) => {
  const { file, onSuccess, onError, onProgress } = options;
  
  const formData = new FormData();
  formData.append('file', file);

  try {
    const response = await axios.post('/api/upload', formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
      onUploadProgress: (progressEvent) => {
        const percent = Math.round((progressEvent.loaded * 100) / (progressEvent.total || 1));
        onProgress?.({ percent });
      },
    });
    onSuccess?.(response.data);
  } catch (error) {
    onError?.(error as Error);
  }
};
```

### Before Upload Validation

```tsx
const beforeUpload = (file: File) => {
  const isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png';
  if (!isJpgOrPng) {
    message.error('You can only upload JPG/PNG files!');
  }
  
  const isLt2M = file.size / 1024 / 1024 < 2;
  if (!isLt2M) {
    message.error('Image must be smaller than 2MB!');
  }
  
  return isJpgOrPng && isLt2M;
};
```

### Preview Handling

```tsx
const [previewOpen, setPreviewOpen] = useState(false);
const [previewImage, setPreviewImage] = useState('');

const handlePreview = async (file: UploadFile) => {
  if (!file.url && !file.preview) {
    file.preview = await getBase64(file.originFileObj!);
  }
  setPreviewImage(file.url || file.preview || '');
  setPreviewOpen(true);
};
```

## Accessibility / Keyboard

- Upload button must be keyboard accessible
- Drag-and-drop must have keyboard alternative
- Progress should be announced to screen readers
- Error messages must describe what went wrong and how to fix

## Do / Don't

| Do | Don't |
|----|-------|
| Show progress for uploads >1s | Leave user wondering if upload started |
| Validate before upload starts | Reject after full upload completes |
| Allow retry of failed uploads | Force user to reselect file |
| Show file size limits upfront | Reject with "too large" after selection |
| Support drag-and-drop AND button | Only support one method |
| Generate previews for images | Show broken image icons |

## Minimal Code Snippet

```tsx
import { useState } from 'react';
import { Upload, Button, Modal, message, Image } from 'antd';
import type { UploadFile, UploadProps } from 'antd';
import { UploadOutlined, EyeOutlined, DeleteOutlined } from '@ant-design/icons';

const getBase64 = (file: File): Promise<string> =>
  new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => resolve(reader.result as string);
    reader.onerror = (error) => reject(error);
  });

export function UploadExample() {
  const [fileList, setFileList] = useState<UploadFile[]>([]);
  const [previewOpen, setPreviewOpen] = useState(false);
  const [previewImage, setPreviewImage] = useState('');
  const [uploading, setUploading] = useState(false);

  const beforeUpload = (file: File) => {
    const isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png';
    if (!isJpgOrPng) {
      message.error('You can only upload JPG/PNG files!');
      return false;
    }
    
    const isLt2M = file.size / 1024 / 1024 < 2;
    if (!isLt2M) {
      message.error('Image must be smaller than 2MB!');
      return false;
    }
    
    return true;
  };

  const handlePreview = async (file: UploadFile) => {
    if (!file.url && !file.preview) {
      file.preview = await getBase64(file.originFileObj as File);
    }
    setPreviewImage(file.url || (file.preview as string));
    setPreviewOpen(true);
  };

  const handleChange: UploadProps['onChange'] = ({ fileList: newFileList }) => {
    setFileList(newFileList);
  };

  const customRequest: UploadProps['customRequest'] = async (options) => {
    const { file, onSuccess, onError, onProgress } = options;
    
    setUploading(true);
    
    // Simulate upload with progress
    try {
      for (let i = 0; i <= 100; i += 10) {
        await new Promise((resolve) => setTimeout(resolve, 100));
        onProgress?.({ percent: i });
      }
      
      // Simulate success
      onSuccess?.({ url: URL.createObjectURL(file as File) });
      message.success('Upload successful!');
    } catch (error) {
      onError?.(error as Error);
      message.error('Upload failed');
    } finally {
      setUploading(false);
    }
  };

  const handleRemove = (file: UploadFile) => {
    const index = fileList.indexOf(file);
    const newFileList = fileList.slice();
    newFileList.splice(index, 1);
    setFileList(newFileList);
    message.info('File removed');
  };

  return (
    <div>
      <Upload
        listType="picture-card"
        fileList={fileList}
        onPreview={handlePreview}
        onChange={handleChange}
        customRequest={customRequest}
        beforeUpload={beforeUpload}
        onRemove={handleRemove}
        accept="image/png,image/jpeg"
        maxCount={5}
      >
        {fileList.length >= 5 ? null : (
          <div>
            <UploadOutlined />
            <div style={{ marginTop: 8 }}>Upload</div>
          </div>
        )}
      </Upload>

      <Modal
        open={previewOpen}
        title="Image Preview"
        footer={null}
        onCancel={() => setPreviewOpen(false)}
      >
        <Image alt="preview" style={{ width: '100%' }} src={previewImage} />
      </Modal>

      <p style={{ marginTop: 16, color: '#666' }}>
        Supports: JPG, PNG (max 2MB, max 5 files)
      </p>
    </div>
  );
}
```
