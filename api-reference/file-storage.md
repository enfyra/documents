# File & Storage

Use these endpoints to upload files, organize them in folders, and serve assets from your app. Base: `{appUrl}/api`

## Files

### List files

```
GET {appUrl}/api/enfyra_file?fields=id,filename,mimetype,size,folder&filter={"folder":{"_eq":123}}
```

### Upload file

Use `multipart/form-data` with a `file` field. Add `folder`, `title`, `description` in the form if needed.

```
POST {appUrl}/api/enfyra_file
```

### Get metadata / Update / Delete file

```
GET    {appUrl}/api/enfyra_file?filter={"id":{"_eq":123}}&limit=1
PATCH  {appUrl}/api/enfyra_file/{id}
DELETE {appUrl}/api/enfyra_file/{id}
```

---

## Folders

**List folders:**
```
GET {appUrl}/api/enfyra_folder?filter={"parent":{"_is_null":true}}
```

**Create folder:**
```
POST {appUrl}/api/enfyra_folder
Body: { "name": "My Folder", "parent": null }
```

**Folder tree:**
```
GET {appUrl}/api/enfyra_folder/tree
```
Returns nested folder structure.

---

## Download Files

Serve a file by ID (images, PDFs, etc.):

```
GET {appUrl}/api/assets/{id}
```

Returns the file binary with appropriate `Content-Type`. Use this URL directly in `<img src="...">` or download links.
