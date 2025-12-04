# File Handling

Enfyra provides file upload and management capabilities through helpers in the context object. You can upload, update, and delete files with support for folders and storage configurations.

## Quick Navigation

- [Uploaded File Information](#uploaded-file-information) - Access uploaded files
- [Upload Files](#upload-files) - Using $uploadFile helper
- [Update Files](#update-files) - Using $updateFile helper
- [Delete Files](#delete-files) - Using $deleteFile helper
- [Common Patterns](#common-patterns) - Real-world examples

## Uploaded File Information

When a file is uploaded via a request, it's available in the context.

### $ctx.$uploadedFile

Information about the uploaded file (if any).

```javascript
if ($ctx.$uploadedFile) {
  const filename = $ctx.$uploadedFile.originalname;  // Original filename
  const mimetype = $ctx.$uploadedFile.mimetype;      // MIME type
  const size = $ctx.$uploadedFile.size;              // File size in bytes
  const buffer = $ctx.$uploadedFile.buffer;          // File buffer
  const fieldname = $ctx.$uploadedFile.fieldname;    // Form field name
}
```

### File Properties

| Property | Type | Description |
|----------|------|-------------|
| `originalname` | `string` | Original filename from client |
| `mimetype` | `string` | MIME type (e.g., `image/jpeg`, `application/pdf`) |
| `size` | `number` | File size in bytes |
| `buffer` | `Buffer` | Raw file content as Node.js Buffer |
| `fieldname` | `string` | Form field name used for upload |

## Upload Files

Use `$ctx.$helpers.$uploadFile` to upload files to storage.

### Basic Upload

```javascript
if (!$ctx.$uploadedFile) {
  $ctx.$throw['400']('No file uploaded');
  return;
}

const fileResult = await $ctx.$helpers.$uploadFile({
  originalname: $ctx.$uploadedFile.originalname,
  mimetype: $ctx.$uploadedFile.mimetype,
  buffer: $ctx.$uploadedFile.buffer,
  size: $ctx.$uploadedFile.size
});

return fileResult;
```

### Upload with Options

```javascript
const fileResult = await $ctx.$helpers.$uploadFile({
  originalname: 'my-image.jpg',
  filename: 'custom-filename.jpg',  // Optional: custom filename
  mimetype: 'image/jpeg',
  buffer: fileBuffer,
  size: 1024000,
  folder: 123,  // Optional: folder ID
  storageConfig: 1,  // Optional: storage config ID
  title: 'My Image',  // Optional: custom title
  description: 'Image description'  // Optional
});
```

### Upload Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `originalname` | `string` | No | Original filename |
| `filename` | `string` | No | Custom filename (alternative to originalname) |
| `mimetype` | `string` | Yes | MIME type |
| `buffer` | `Buffer` | Yes | File buffer |
| `size` | `number` | Yes | File size in bytes |
| `encoding` | `string` | No | File encoding (default: 'utf8') |
| `folder` | `number \| { id: number }` | No | Folder ID to store file in |
| `storageConfig` | `number` | No | Storage configuration ID |
| `title` | `string` | No | Custom title (defaults to filename) |
| `description` | `string` | No | File description |

### Example: Upload from Request

```javascript
// In handler for file upload endpoint
if (!$ctx.$uploadedFile) {
  $ctx.$throw['400']('No file uploaded');
  return;
}

const fileResult = await $ctx.$helpers.$uploadFile({
  originalname: $ctx.$uploadedFile.originalname,
  mimetype: $ctx.$uploadedFile.mimetype,
  buffer: $ctx.$uploadedFile.buffer,
  size: $ctx.$uploadedFile.size,
  title: $ctx.$body.title || $ctx.$uploadedFile.originalname,
  description: $ctx.$body.description
});

$ctx.$logs(`File uploaded: ${fileResult.filename}`);

return {
  success: true,
  file: fileResult
};
```

## Update Files

Use `$ctx.$helpers.$updateFile` to update existing files.

### Basic Update

```javascript
await $ctx.$helpers.$updateFile(fileId, {
  buffer: newFileBuffer,
  mimetype: 'image/jpeg',
  size: newFileSize
});
```

### Update with Options

```javascript
await $ctx.$helpers.$updateFile(fileId, {
  buffer: newFileBuffer,
  originalname: 'new-name.jpg',
  filename: 'custom-new-name.jpg',
  mimetype: 'image/jpeg',
  size: 2048000,
  folder: 456,
  title: 'Updated Title',
  description: 'Updated description'
});
```

### Update Parameters

All parameters are optional - only include what you want to update.

| Parameter | Type | Description |
|-----------|------|-------------|
| `buffer` | `Buffer` | New file buffer |
| `originalname` | `string` | New original filename |
| `filename` | `string` | New custom filename |
| `mimetype` | `string` | New MIME type |
| `size` | `number` | New file size |
| `folder` | `number \| { id: number }` | Move to different folder |
| `title` | `string` | Update title |
| `description` | `string` | Update description |

### Example: Update File Metadata

```javascript
// Update only metadata, not file content
await $ctx.$helpers.$updateFile($ctx.$params.fileId, {
  title: $ctx.$body.title,
  description: $ctx.$body.description,
  folder: $ctx.$body.folderId
});

$ctx.$logs(`File metadata updated: ${$ctx.$params.fileId}`);

return { success: true };
```

## Delete Files

Use `$ctx.$helpers.$deleteFile` to delete files.

### Basic Delete

```javascript
await $ctx.$helpers.$deleteFile(fileId);
```

### Example: Delete File

```javascript
// Check if file exists
const fileResult = await $ctx.$repos.file_definition.find({
  where: { id: { _eq: $ctx.$params.fileId } }
});

if (fileResult.data.length === 0) {
  $ctx.$throw['404']('File not found');
  return;
}

// Delete the file
await $ctx.$helpers.$deleteFile($ctx.$params.fileId);

$ctx.$logs(`File deleted: ${$ctx.$params.fileId}`);

return { success: true, message: 'File deleted successfully' };
```

## Common Patterns

### Pattern 1: Upload and Create Related Record

```javascript
// Upload file
if (!$ctx.$uploadedFile) {
  $ctx.$throw['400']('No file uploaded');
  return;
}

const fileResult = await $ctx.$helpers.$uploadFile({
  originalname: $ctx.$uploadedFile.originalname,
  mimetype: $ctx.$uploadedFile.mimetype,
  buffer: $ctx.$uploadedFile.buffer,
  size: $ctx.$uploadedFile.size
});

// Create product with file reference
const productResult = await $ctx.$repos.products.create({
  data: {
    name: $ctx.$body.name,
    price: $ctx.$body.price,
    imageFileId: fileResult.id
  }
});

return {
  product: productResult.data[0],
  file: fileResult
};
```

### Pattern 2: Validate File Before Upload

```javascript
if (!$ctx.$uploadedFile) {
  $ctx.$throw['400']('No file uploaded');
  return;
}

// Validate file type
const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
if (!allowedTypes.includes($ctx.$uploadedFile.mimetype)) {
  $ctx.$throw['400']('Invalid file type. Only images are allowed.');
  return;
}

// Validate file size (max 5MB)
const maxSize = 5 * 1024 * 1024; // 5MB
if ($ctx.$uploadedFile.size > maxSize) {
  $ctx.$throw['400']('File too large. Maximum size is 5MB.');
  return;
}

// Upload file
const fileResult = await $ctx.$helpers.$uploadFile({
  originalname: $ctx.$uploadedFile.originalname,
  mimetype: $ctx.$uploadedFile.mimetype,
  buffer: $ctx.$uploadedFile.buffer,
  size: $ctx.$uploadedFile.size
});

return fileResult;
```

### Pattern 3: Upload to Specific Folder

```javascript
// Get or create folder
let folder = await $ctx.$repos.folders.find({
  where: { name: { _eq: 'User Uploads' } }
});

if (folder.data.length === 0) {
  const folderResult = await $ctx.$repos.folders.create({
    data: {
      name: 'User Uploads',
      userId: $ctx.$user.id
    }
  });
  folder = folderResult.data[0];
} else {
  folder = folder.data[0];
}

// Upload file to folder
const fileResult = await $ctx.$helpers.$uploadFile({
  originalname: $ctx.$uploadedFile.originalname,
  mimetype: $ctx.$uploadedFile.mimetype,
  buffer: $ctx.$uploadedFile.buffer,
  size: $ctx.$uploadedFile.size,
  folder: folder.id
});

return fileResult;
```

### Pattern 4: Replace File

```javascript
// Get existing file
const existingFile = await $ctx.$repos.file_definition.find({
  where: { id: { _eq: $ctx.$params.fileId } }
});

if (existingFile.data.length === 0) {
  $ctx.$throw['404']('File not found');
  return;
}

// Replace with new file
await $ctx.$helpers.$updateFile($ctx.$params.fileId, {
  buffer: $ctx.$uploadedFile.buffer,
  mimetype: $ctx.$uploadedFile.mimetype,
  size: $ctx.$uploadedFile.size,
  originalname: $ctx.$uploadedFile.originalname
});

$ctx.$logs(`File replaced: ${$ctx.$params.fileId}`);

return { success: true };
```

### Pattern 5: Upload Multiple Files

```javascript
// Note: This requires multiple file uploads in the request
// Each file would be processed separately

const uploadResults = [];

// Process each uploaded file
// (Implementation depends on how multiple files are sent in request)

for (const uploadedFile of uploadedFiles) {
  const fileResult = await $ctx.$helpers.$uploadFile({
    originalname: uploadedFile.originalname,
    mimetype: uploadedFile.mimetype,
    buffer: uploadedFile.buffer,
    size: uploadedFile.size
  });
  
  uploadResults.push(fileResult);
}

return {
  success: true,
  files: uploadResults,
  count: uploadResults.length
};
```

## Best Practices

1. **Validate files** - Always validate file type and size before uploading
2. **Handle missing files** - Check if file exists before processing
3. **Use appropriate folders** - Organize files using folders
4. **Set file metadata** - Include title and description for better organization
5. **Error handling** - Wrap file operations in try-catch blocks
6. **File size limits** - Enforce reasonable file size limits
7. **File type restrictions** - Only allow safe file types

## Next Steps

- Learn about [Context Reference](./context-reference/) for file-related properties
- See [Error Handling](./error-handling.md) for proper error handling
- Check [Repository Methods](./repository-methods/) for querying file records

