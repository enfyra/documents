# File Handling & Streaming

This guide covers file upload handling and response streaming in Enfyra, including advanced use cases like image processing and efficient memory management.

## Table of Contents
- [File Upload](#file-upload)
- [Response Streaming](#response-streaming)
- [Image Processing with Sharp](#image-processing-with-sharp)
- [Performance Considerations](#performance-considerations)

---

## File Upload

### Basic File Upload

When a handler receives a file upload, the file is available via `@UPLOADED` (or `$ctx.$uploadedFile`):

```javascript
// Handler for file upload
const file = @UPLOADED;

console.log(file.filename);   // Original filename
console.log(file.mimetype);   // MIME type (e.g., 'image/jpeg')
console.log(file.filesize);   // File size in bytes
console.log(file.buffer);     // Buffer containing file data
```

### File Properties

| Property | Type | Description |
|----------|------|-------------|
| `filename` | `string` | Original filename from client |
| `mimetype` | `string` | MIME type (e.g., `image/jpeg`, `application/pdf`) |
| `filesize` | `number` | File size in bytes |
| `buffer` | `Buffer` | Raw file content as Node.js Buffer |

### Example: Text Extraction from Upload

```javascript
// Parse uploaded text file
const content = @UPLOADED.buffer.toString('utf-8');
const lines = content.split('\n');

return {
  totalLines: lines.length,
  preview: lines.slice(0, 10)
};
```

### Example: Save File to Database

```javascript
// Save uploaded file to file_definition table
const savedFile = await #file_definition.create({
  filename: @UPLOADED.filename,
  mimetype: @UPLOADED.mimetype,
  filesize: @UPLOADED.filesize,
  buffer: @UPLOADED.buffer,
  uploadedById: @USER.id
});

return savedFile;
```

---

## Response Streaming

### Why Use Streaming?

**Traditional approach (load entire file into memory):**
```javascript
// ❌ BAD: 100MB file = 100MB RAM usage
const response = await fetch(imageUrl);
const buffer = Buffer.from(await response.arrayBuffer());
@RES.send(buffer);  // Memory spike!
```

**Streaming approach (process chunks):**
```javascript
// ✅ GOOD: 100MB file = only ~64KB RAM usage (per chunk)
const response = await fetch(imageUrl);
const stream = Readable.fromWeb(response.body);
@RES.stream(stream);  // Efficient memory usage!
```

### Basic Streaming

The `@RES.stream()` method allows you to send data efficiently by streaming chunks instead of loading everything into memory at once.

**Signature:**
```typescript
@RES.stream(stream: Readable | AsyncIterable, options?: {
  mimetype?: string;
  filename?: string;
})
```

**Parameters:**
- `stream`: A Node.js `Readable` stream or any async iterable
- `options.mimetype`: Content-Type header (e.g., `'image/jpeg'`)
- `options.filename`: Filename for Content-Disposition header (for downloads)

### Example: Stream File from URL

```javascript
const { Readable } = require('stream');

// Download and stream image from external URL
const imageUrl = 'https://example.com/large-photo.jpg';
const response = await fetch(imageUrl);
const stream = Readable.fromWeb(response.body);

// Stream directly to client
@RES.stream(stream, {
  mimetype: 'image/jpeg',
  filename: 'downloaded-photo.jpg'
});
```

### Example: Stream from Database

```javascript
const { Readable } = require('stream');

// Get file from database
const file = await #file_definition.findOne({ id: @PARAMS.id });

// Create stream from buffer
const stream = Readable.from(file.buffer);

// Stream to client
@RES.stream(stream, {
  mimetype: file.mimetype,
  filename: file.filename
});
```

---

## Image Processing with Sharp

Sharp is a high-performance image processing library that works excellently with streams.

### Install Sharp

Sharp is already available in Enfyra. Access it via `@PKGS`:

```javascript
const sharp = @PKGS.sharp;
```

### Example: Download and Resize Image

```javascript
const { Readable } = require('stream');
const sharp = @PKGS.sharp;

// Download image from URL
const imageUrl = 'https://example.com/large-image.jpg';
const response = await fetch(imageUrl);
const inputStream = Readable.fromWeb(response.body);

// Create Sharp transform stream
const transformer = sharp()
  .resize(800, 600, { fit: 'inside' })  // Resize to max 800x600
  .jpeg({ quality: 85 });                // Convert to JPEG at 85% quality

// Pipe through Sharp and stream to client
const outputStream = inputStream.pipe(transformer);

@RES.stream(outputStream, {
  mimetype: 'image/jpeg',
  filename: 'resized-image.jpg'
});
```

### Example: Resize Uploaded Image

```javascript
const { Readable } = require('stream');
const sharp = @PKGS.sharp;

// Get uploaded image
const uploadedFile = @UPLOADED;

// Create stream from buffer
const inputStream = Readable.from(uploadedFile.buffer);

// Resize and optimize
const transformer = sharp()
  .resize(1024, 1024, {
    fit: 'inside',
    withoutEnlargement: true  // Don't upscale small images
  })
  .jpeg({ quality: 80, progressive: true });

// Stream result
const outputStream = inputStream.pipe(transformer);

@RES.stream(outputStream, {
  mimetype: 'image/jpeg',
  filename: `resized-${uploadedFile.filename}`
});
```

### Example: Create Thumbnail

```javascript
const { Readable } = require('stream');
const sharp = @PKGS.sharp;

// Get file from database
const file = await #file_definition.findOne({ id: @PARAMS.id });

// Create thumbnail stream
const inputStream = Readable.from(file.buffer);
const transformer = sharp()
  .resize(200, 200, {
    fit: 'cover',           // Crop to fill dimensions
    position: 'center'
  })
  .jpeg({ quality: 70 });

const outputStream = inputStream.pipe(transformer);

@RES.stream(outputStream, {
  mimetype: 'image/jpeg',
  filename: `thumb-${file.filename}`
});
```

### Example: Add Watermark

```javascript
const { Readable } = require('stream');
const sharp = @PKGS.sharp;

const imageUrl = @BODY.imageUrl;
const response = await fetch(imageUrl);
const inputStream = Readable.fromWeb(response.body);

// Add text watermark
const transformer = sharp()
  .resize(1200, 800)
  .composite([
    {
      input: Buffer.from(
        '<svg><text x="10" y="30" font-size="24" fill="white" opacity="0.5">© Your Company</text></svg>'
      ),
      top: 10,
      left: 10
    }
  ])
  .jpeg({ quality: 85 });

const outputStream = inputStream.pipe(transformer);

@RES.stream(outputStream, {
  mimetype: 'image/jpeg',
  filename: 'watermarked-image.jpg'
});
```

### Available Sharp Transformations

```javascript
const sharp = @PKGS.sharp;

const transformer = sharp()
  // Resize
  .resize(width, height, options)

  // Format conversion
  .jpeg({ quality: 80, progressive: true })
  .png({ compressionLevel: 9 })
  .webp({ quality: 80 })

  // Image operations
  .rotate(90)                    // Rotate
  .flip()                        // Flip vertically
  .flop()                        // Flip horizontally
  .blur(5)                       // Gaussian blur
  .sharpen()                     // Sharpen
  .grayscale()                   // Convert to grayscale
  .normalize()                   // Auto-enhance

  // Cropping
  .extract({ left: 0, top: 0, width: 100, height: 100 })

  // Composition
  .composite([{ input: buffer, top: 0, left: 0 }]);
```

---

## Performance Considerations

### Memory Usage Comparison

**Loading file into memory:**
```javascript
// 100MB file → 100MB RAM + processing overhead
const buffer = Buffer.from(await response.arrayBuffer());
const resized = await sharp(buffer).resize(800, 600).toBuffer();
// Peak RAM usage: ~200MB
```

**Streaming approach:**
```javascript
// 100MB file → only 64KB per chunk in RAM
const stream = Readable.fromWeb(response.body);
const transformer = sharp().resize(800, 600);
@RES.stream(stream.pipe(transformer));
// Peak RAM usage: ~5MB
```

### Best Practices

1. **Always use streaming for large files** (> 10MB)
   ```javascript
   // ✅ Stream large files
   if (filesize > 10_000_000) {
     @RES.stream(stream, options);
   }
   ```

2. **Use streaming for external URLs**
   ```javascript
   // ✅ Stream from external URL (unknown size)
   const response = await fetch(externalUrl);
   @RES.stream(Readable.fromWeb(response.body));
   ```

3. **Chain transformations efficiently**
   ```javascript
   // ✅ Single pipeline - very efficient
   const output = inputStream
     .pipe(sharp().resize(1024, 768))
     .pipe(sharp().jpeg({ quality: 80 }));

   @RES.stream(output);
   ```

4. **Set appropriate headers**
   ```javascript
   // ✅ Good user experience
   @RES.stream(stream, {
     mimetype: 'image/webp',
     filename: 'optimized-image.webp'  // Browser will suggest this name
   });
   ```

5. **Handle errors gracefully**
   ```javascript
   try {
     const stream = await createImageStream(url);
     @RES.stream(stream, { mimetype: 'image/jpeg' });
   } catch (error) {
     @THROW400(`Failed to process image: ${error.message}`);
   }
   ```

### When NOT to Use Streaming

For very small files or when you need to modify the entire buffer:

```javascript
// Small files (< 1MB) - buffer is fine
if (@UPLOADED.filesize < 1_000_000) {
  const text = @UPLOADED.buffer.toString('utf-8');
  return { content: text };
}

// Need entire file for processing
const allData = [];
for await (const chunk of stream) {
  allData.push(chunk);
}
const buffer = Buffer.concat(allData);
// Now process entire buffer...
```

---

## Common Patterns

### Pattern: Download → Transform → Stream

```javascript
const { Readable } = require('stream');
const sharp = @PKGS.sharp;

// 1. Fetch from external source
const response = await fetch(@QUERY.imageUrl);

// 2. Create stream
const stream = Readable.fromWeb(response.body);

// 3. Transform
const transformer = sharp()
  .resize(1024, 1024, { fit: 'inside' })
  .webp({ quality: 80 });

// 4. Stream to client
@RES.stream(stream.pipe(transformer), {
  mimetype: 'image/webp',
  filename: 'optimized.webp'
});
```

### Pattern: Upload → Process → Save → Stream

```javascript
const { Readable } = require('stream');
const sharp = @PKGS.sharp;

// 1. Get upload
const uploaded = @UPLOADED;

// 2. Process to buffer (need to save to DB)
const inputStream = Readable.from(uploaded.buffer);
const processedBuffer = await inputStream
  .pipe(sharp().resize(800, 600).jpeg({ quality: 85 }))
  .toBuffer();

// 3. Save to database
const savedFile = await #file_definition.create({
  filename: uploaded.filename,
  mimetype: 'image/jpeg',
  filesize: processedBuffer.length,
  buffer: processedBuffer
});

// 4. Stream saved file back to client
const outputStream = Readable.from(processedBuffer);
@RES.stream(outputStream, {
  mimetype: 'image/jpeg',
  filename: savedFile.filename
});
```

### Pattern: Conditional Processing

```javascript
const { Readable } = require('stream');
const sharp = @PKGS.sharp;

const file = await #file_definition.findOne({ id: @PARAMS.id });
const stream = Readable.from(file.buffer);

// Only resize if requested
if (@QUERY.resize) {
  const [width, height] = @QUERY.resize.split('x').map(Number);
  const transformer = sharp().resize(width, height, { fit: 'inside' });
  @RES.stream(stream.pipe(transformer), {
    mimetype: file.mimetype,
    filename: `${width}x${height}-${file.filename}`
  });
} else {
  // Stream original
  @RES.stream(stream, {
    mimetype: file.mimetype,
    filename: file.filename
  });
}
```

---

## Multi Cloud & Multi Account Support

Enfyra provides a unified file serving endpoint `/assets/:id` that seamlessly handles files stored across multiple cloud providers and multiple accounts. This system automatically routes requests to the correct storage backend based on each file's configuration.

### Overview

The `/assets/:id` endpoint is a **single unified interface** that:
- Automatically detects where each file is stored (local, GCS, S3)
- Uses the correct credentials for each storage account
- Streams files efficiently regardless of storage location
- Supports image processing with query parameters for all storage types
- Handles permissions and access control transparently

### How It Works

When you request a file via `/assets/:id`, the system:

1. **Fetches file metadata** from the `file_definition` table, including the associated `storageConfig`
2. **Determines storage type** (local, GCS, or S3) from the file's storage configuration
3. **Retrieves credentials** from the storage config (each config can have different credentials)
4. **Streams the file** using the appropriate method for that storage type
5. **Applies image processing** if query parameters are provided (works for all storage types)

### Uploading Files with Folder and Storage Config

When uploading files, you can specify both the target folder and storage configuration:

```http
POST /file_definition
Content-Type: multipart/form-data

file: <file>
folder: 123                    # Folder ID (optional)
storageConfig: 2               # Storage config ID (optional)
title: "My Document"            # Title (optional)
description: "File description" # Description (optional)
```

**Parameters:**
- `file` (required): The file to upload
- `folder` (optional): ID of the folder to organize the file. Can be:
  - A number: `folder: 123`
  - An object: `folder: { id: 123 }`
- `storageConfig` (optional): ID of the storage configuration to use. Can be:
  - A number: `storageConfig: 2`
  - An object: `storageConfig: { id: 2 }`
  - If not specified, uses default local storage
- `title` (optional): Custom title for the file (defaults to original filename)
- `description` (optional): File description

**Example: Upload to specific folder and storage config**

```javascript
// Using FormData in JavaScript
const formData = new FormData();
formData.append('file', fileInput.files[0]);
formData.append('folder', '123');              // Upload to folder ID 123
formData.append('storageConfig', '2');         // Use storage config ID 2

const response = await fetch('/file_definition', {
  method: 'POST',
  body: formData
});

const uploadedFile = await response.json();
// File is now stored using storage config #2 in folder #123
```

**Example: Upload to default storage (local)**

```javascript
const formData = new FormData();
formData.append('file', fileInput.files[0]);
// No storageConfig specified → uses default local storage

const response = await fetch('/file_definition', {
  method: 'POST',
  body: formData
});
```

### Single Endpoint, Multiple Backends

The beauty of `/assets/:id` is that **you don't need to know where the file is stored**. The endpoint handles everything:

```http
# File stored locally
GET /assets/123
→ Automatically streams from local filesystem

# File stored in GCS Account #1
GET /assets/456
→ Automatically streams from GCS using Account #1 credentials

# File stored in GCS Account #2 (different project)
GET /assets/789
→ Automatically streams from GCS using Account #2 credentials

# File stored in S3
GET /assets/101
→ Automatically streams from S3 (when implemented)
```

### Image Processing Across All Storage Types

The endpoint supports image processing query parameters regardless of storage location:

```http
# Local file with image processing
GET /assets/123?width=800&height=600&format=webp&quality=85

# GCS file with image processing
GET /assets/456?width=800&height=600&format=webp&quality=85

# Both work identically - the system handles streaming and processing
```

**Supported Query Parameters:**
- `width`: Resize width (pixels)
- `height`: Resize height (pixels)
- `format`: Output format (`jpeg`, `png`, `webp`, etc.)
- `quality`: Compression quality (1-100)
- `cache`: Cache-Control max-age (seconds)

### Multi-Account Support

Each storage configuration can use **different cloud accounts**. Storage configurations are managed through the admin interface, where you can:
- Create multiple storage configs for different environments (dev, staging, prod)
- Configure different GCS accounts with their own credentials
- Set up client-specific storage accounts
- Enable or disable storage configs

When uploading a file, specify the `storageConfig` parameter to choose which storage backend to use. The `/assets/:id` endpoint automatically uses the correct credentials when serving that file, regardless of which storage config was used during upload.

### How It Works

The `/assets/:id` endpoint automatically handles all the complexity behind the scenes:

1. **Fetches file metadata** including storage configuration
2. **Checks permissions** (if file is private)
3. **Routes to correct storage** (local, GCS, or S3) based on file's configuration
4. **Uses correct credentials** for the storage account associated with the file
5. **Streams the file** efficiently, applying image processing if requested

You don't need to worry about where files are stored or which credentials to use - the system handles everything transparently.

### Performance Features

**Caching:**
- Image processing results are cached in Redis
- Cache keys include file path, dimensions, format, and quality
- Frequently accessed images are kept in "hot keys" for faster access

**Streaming:**
- Files are streamed directly from cloud storage (no local download)
- Image processing happens on-the-fly during streaming
- Memory-efficient: only processes chunks, not entire file

**Storage Config Cache:**
- Storage configurations are cached in memory
- Cache is synchronized across multiple server instances via Redis pub/sub
- Automatic reload when configurations change

### Example: Uploading and Serving Files

```javascript
// Upload file to production storage config in a specific folder
const formData = new FormData();
formData.append('file', fileInput.files[0]);
formData.append('folder', '10');           // Organize in folder #10
formData.append('storageConfig', '1');     // Use production storage config
formData.append('title', 'Product Photo');

const response = await fetch('/file_definition', {
  method: 'POST',
  body: formData
});

const uploadedFile = await response.json();

// Serve the file - system automatically uses correct storage config
// GET /assets/{uploadedFile.id}
// → Automatically streams from production GCS using config #1 credentials
```

### Permissions

The endpoint respects file permissions:
- **Published files** (`isPublished: true`): Publicly accessible
- **Private files** (`isPublished: false`): Requires permission check
  - Checks `file_permission_definition` table
  - Validates user/role permissions
  - Returns 403 if access denied

### Error Handling

The endpoint handles various error scenarios:

```http
# File not found
GET /assets/999
→ 404 Not Found

# Permission denied
GET /assets/123 (private file, user not authorized)
→ 403 Forbidden

# Storage config not found
GET /assets/456 (file references invalid storage config)
→ 400 Bad Request

# Cloud storage error
GET /assets/789 (GCS credentials invalid or bucket not accessible)
→ 500 Internal Server Error
```

### Best Practices

1. **Use storage configs for organization**
   - Separate configs for different environments (dev, staging, prod)
   - Separate configs for different clients/tenants
   - Separate configs for different file types if needed

2. **Leverage image processing**
   - Use query parameters instead of storing multiple sizes
   - Cache frequently accessed transformations
   - Optimize quality based on use case

3. **Monitor storage usage**
   - Track which storage configs are being used
   - Monitor costs per account
   - Set up alerts for storage limits

4. **Security**
   - Store credentials securely in database
   - Use service accounts with minimal required permissions
   - Regularly rotate credentials

---

## Related Documentation

- [Context Reference](./context-reference.md) - Full `$ctx` object reference
- [Template Syntax](./template-syntax.md) - Shortcuts like `@UPLOADED`, `@RES`
- [Hook Development](./hook-development.md) - Using file handling in hooks

---

## API Reference Summary

### `@UPLOADED` (File Upload)
```typescript
{
  filename: string;    // Original filename
  mimetype: string;    // MIME type
  filesize: number;    // Bytes
  buffer: Buffer;      // File content
}
```

### `@RES.stream()` (Response Streaming)
```typescript
@RES.stream(
  stream: Readable | AsyncIterable,
  options?: {
    mimetype?: string;    // Content-Type header
    filename?: string;    // Content-Disposition filename
  }
): Promise<void>
```

### `@PKGS.sharp` (Image Processing)
```typescript
const sharp = @PKGS.sharp;

sharp()
  .resize(width?, height?, options?)
  .jpeg(options?)
  .png(options?)
  .webp(options?)
  .rotate(angle?)
  .flip()
  .flop()
  .blur(sigma?)
  .sharpen()
  .grayscale()
  .composite(overlays?)
  // ... and more
```
