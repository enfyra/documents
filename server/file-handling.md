# File Handling

Enfyra supports request file uploads, storage-backed file records, metadata updates, and file deletion through dynamic script helpers.

## Uploaded File Context

Multipart requests expose the uploaded request file as `$ctx.$uploadedFile` or `@UPLOADED_FILE`.

```javascript
const file = @UPLOADED_FILE;

if (!file) {
  @THROW400("File is required");
}

return {
  originalname: file.originalname,
  mimetype: file.mimetype,
  size: file.size,
  fieldname: file.fieldname,
};
```

Available properties:

| Property | Type | Description |
| --- | --- | --- |
| `originalname` | `string` | Filename sent by the client |
| `mimetype` | `string` | Client-provided MIME type |
| `encoding` | `string` | Form upload encoding |
| `path` | `string` | Server temp-file path used internally by Enfyra helpers |
| `size` | `number` | File size in bytes |
| `fieldname` | `string` | Multipart field name, usually `file` |

Do not read `@UPLOADED_FILE.path` into a `Buffer` in script code. For normal request uploads, pass the file object directly to `@STORAGE.$upload` or `@STORAGE.$update` so Enfyra streams from disk to the selected storage backend.

## Upload Files

Use `$ctx.$storage.$upload` or `@STORAGE.$upload` to upload a file to storage and create a `enfyra_file` record.

### Upload The Request File

```javascript
if (!@UPLOADED_FILE) {
  @THROW400("File is required");
}

const saved = await @STORAGE.$upload({
  file: @UPLOADED_FILE,
  storageConfig: @BODY.storageConfig,
  folder: @BODY.folder,
  title: @BODY.title,
  description: @BODY.description,
});

return saved;
```

This path is streaming-safe. Enfyra writes the multipart upload to a temp file, then `$storage.$upload({ file })` opens a `Readable` stream from that temp file. The full file is not buffered into RAM.

### Upload A Generated Or Processed File

Use `buffer` only when the script itself creates or transforms a small file, such as an image thumbnail.

```javascript
const pngBuffer = await makeThumbnail();

const saved = await @STORAGE.$upload({
  filename: "thumbnail.png",
  mimetype: "image/png",
  buffer: pngBuffer,
  size: pngBuffer.length,
  description: "Generated thumbnail",
});

return saved;
```

Do not use the buffer form for large request uploads or database backups. For large files, use the `file: @UPLOADED_FILE` form.

### Upload Options

| Option | Type | Required | Description |
| --- | --- | --- | --- |
| `file` | `@UPLOADED_FILE` object | Required for request uploads | Streams the request upload from Enfyra's temp file |
| `buffer` | `Buffer` | Required for generated-file uploads | In-memory bytes for small generated or transformed files |
| `filename` / `originalname` | `string` | Required for buffer uploads | Filename to store |
| `mimetype` | `string` | Required for buffer uploads | MIME type to store |
| `size` | `number` | Optional | Defaults to request file size or buffer length |
| `folder` | `number \| { id: number }` | No | Folder relation |
| `storageConfig` | `number \| { id: number }` | No | Storage configuration relation |
| `title` | `string` | No | File title |
| `description` | `string` | No | File description |
| `isPublic` | `boolean` | No | Allow anonymous access through `/assets/:id` when true |

Pass either `file` or `buffer`, never both.

## Update Files

Use `$ctx.$storage.$update` or `@STORAGE.$update`.

### Replace With The Request File

```javascript
if (!@UPLOADED_FILE) {
  @THROW400("File is required");
}

const updated = await @STORAGE.$update(@PARAMS.fileId, {
  file: @UPLOADED_FILE,
  title: @BODY.title,
  description: @BODY.description,
});

return updated;
```

### Metadata-Only Update

```javascript
const updated = await @STORAGE.$update(@PARAMS.fileId, {
  title: @BODY.title,
  description: @BODY.description,
  folder: @BODY.folder,
  isPublic: @BODY.isPublic,
});

return updated;
```

Changing `storageConfig` requires replacing the blob in the same request. Metadata-only storage moves are rejected because Enfyra cannot point a record at a backend that does not contain the object.

## Delete Files

```javascript
await @STORAGE.$delete(@PARAMS.fileId);

return { success: true };
```

Deletion removes the `enfyra_file` record and the physical object from the configured storage backend.

## Public Asset Access

`enfyra_file.isPublic` controls anonymous access to stored assets. Set `isPublic: true` only when the asset can be served without authentication through `/assets/:id` or the app proxy equivalent. Use `isPublic: false` or omit it for private files that should require authenticated access.

This file-level public access flag is separate from column and relation visibility. Columns and relations still use `isPublished` as the field-permission baseline; file records use `isPublic`.

## Validation Pattern

Validate type and size before saving:

```javascript
const file = @UPLOADED_FILE;

if (!file) {
  @THROW400("File is required");
}

const allowed = ["image/jpeg", "image/png", "application/pdf"];
if (!allowed.includes(file.mimetype)) {
  @THROW400("Unsupported file type");
}

const maxBytes = 20 * 1024 * 1024;
if (file.size > maxBytes) {
  @THROW400("File is too large");
}

return await @STORAGE.$upload({
  file,
  folder: @BODY.folder,
  description: @BODY.description,
});
```

## Register Existing Storage Objects

Use `@STORAGE.$registerFile` when a trusted external process has already uploaded an object to the selected storage backend and the script only needs to create the `enfyra_file` record.

```javascript
return await @STORAGE.$registerFile({
  filename: "backup.sql.gz",
  mimetype: "application/gzip",
  location: @BODY.location,
  size: @BODY.size,
  storageConfig: @BODY.storageConfig,
  description: @BODY.description,
});
```

## Client Request

Send multipart data with field name `file`. Do not manually set `Content-Type`; the browser must set the multipart boundary.

```javascript
const form = new FormData();
form.append("file", file);
form.append("description", description);

const response = await fetch("/api/upload", {
  method: "POST",
  credentials: "include",
  body: form,
});
```

## Storage Behavior

File uploads use one stream contract internally. Request uploads are disk-backed and streamed to Local, S3, Cloudflare R2, or Google Cloud Storage. The buffer helper form exists only for small generated files and is converted to a stream at the helper edge.

## Next Steps

- [Context Reference](./context-reference/) for request context fields.
- [Error Handling](./error-handling.md) for `@THROW` patterns.
- [Repository Methods](./repository-methods/) for querying file records.
