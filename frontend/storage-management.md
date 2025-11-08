# Storage Management

Storage Management allows you to organize and manage files and folders in your application. You can upload files, create folders, and configure storage locations.

## Accessing Storage Management

1. Navigate to **Storage → Management** in the sidebar
2. You'll see the file management interface with folders and files

## Routes

- **Storage Management**: `/storage/management` - Main file management page
- **File Details**: `/storage/management/file/:id` - View and edit file information
- **Folder Details**: `/storage/management/folder/:id` - View and manage folder contents
- **Storage Configuration**: `/storage/config` - Manage storage configurations

## Uploading Files

### From Storage Management Page

1. Click the **"Upload Files"** button in the header (top right)
2. The upload modal will open
3. **Optional**: Select a storage location from the dropdown (if you have multiple storage configurations)
4. Choose files by either:
   - Clicking **"Choose File"** button to browse your computer
   - Dragging and dropping files into the upload area
5. Selected files will appear in a list below the upload area
6. Click **"Upload"** button to start uploading
7. Files will be uploaded to the root level (not in any folder)

### From Folder Details Page

1. Navigate into a folder by clicking on it
2. Click the **"Upload Files"** button in the header
3. Follow the same steps as above
4. Files will be uploaded into the current folder

### Upload Features

- **Multiple Files**: You can upload multiple files at once
- **File Size Limit**: Maximum file size is 50MB per file
- **Storage Selection**: You can optionally choose which storage configuration to use for the upload
- **File Types**: All file types are accepted by default

## Creating Folders

### From Storage Management Page

1. Click the **"New Folder"** button in the header (next to Upload Files)
2. A modal will open asking for folder name
3. Enter the folder name
4. Click **"Create"** to create the folder at root level

### From Folder Details Page

1. Navigate into a folder
2. Click the **"New Folder"** button
3. Enter the folder name
4. The new folder will be created inside the current folder

## Viewing Files and Folders

### Grid View (Default)

- Files and folders are displayed as cards
- Each card shows:
  - Icon (folder icon for folders, file type icon for files)
  - Name
  - Additional metadata (for files)

### List View

- Click the **"List View"** button in the subheader to switch to list view
- Files and folders are displayed in a table format
- Shows more detailed information

### Switching Views

- Use the view toggle button in the subheader (left side)
- Toggle between Grid View and List View

## Managing Files

### Viewing File Details

1. Click on any file card or name
2. You'll be taken to the file details page
3. You can see:
   - File preview (for images)
   - File icon (for other file types)
   - File information form
   - File permissions section

### Editing File Information

1. Navigate to file details page
2. Edit any field in the form
3. Click **"Save"** button in the header to save changes
4. Click **"Reset"** to discard changes

### Replacing a File

1. Navigate to file details page
2. Click **"Replace File"** button in the subheader
3. Upload modal will open
4. Select a new file to replace the current one
5. Click **"Upload"** to replace
6. **Warning**: The old file will be permanently lost

### Deleting Files

**From File Details Page:**
1. Navigate to file details
2. Click the **"Delete"** button in the header (red button)
3. Confirm the deletion

**From File Manager:**
1. Right-click on a file
2. Select **"Delete"** from the context menu
3. Confirm the deletion

**Multiple Files:**
1. Enter selection mode (click "Select Items" in subheader)
2. Select multiple files by clicking on them
3. Use the delete action from the context menu or toolbar

## Managing Folders

### Viewing Folder Contents

1. Click on any folder
2. You'll be taken to the folder details page
3. You can see:
   - All subfolders
   - All files in the folder
   - Folder information

### Editing Folder Information

1. Navigate to folder details page
2. Edit folder fields in the form
3. Click **"Save"** to save changes

### Deleting Folders

**From Folder Details:**
1. Right-click on a folder
2. Select **"Delete"** from the context menu
3. Confirm the deletion

**Warning**: Deleting a folder will also delete all files and subfolders inside it.

## Storage Configuration

### Viewing Storage Configurations

1. Navigate to **Storage → Config** in the sidebar
2. You'll see a list of all storage configurations
3. Each configuration shows:
   - Name
   - Driver type (S3, Local, GCS, etc.)
   - Status (Active/Inactive)
   - Description

### Creating Storage Configuration

1. Go to Storage Config page
2. Click **"Create Storage"** button in the header
3. Fill in the configuration form
4. Click **"Save"** to create

### Editing Storage Configuration

1. Click on any storage configuration card
2. Edit the fields in the form
3. Click **"Save"** to save changes
4. Click **"Reset"** to discard changes

### Enabling/Disabling Storage

1. On the Storage Config list page
2. Toggle the switch on any configuration card
3. Active configurations can be used for file uploads
4. Inactive configurations cannot be selected during upload

### Deleting Storage Configuration

1. On the Storage Config list page
2. Click the **"Delete"** button on a configuration card
3. Confirm the deletion

## File Actions

### Context Menu Actions

Right-click on any file to see available actions:
- **View**: Open file in new tab
- **Download**: Download the file
- **Copy URL**: Copy file URL to clipboard
- **Details**: Navigate to file details page
- **Delete**: Delete the file (if you have permission)

### Selection Mode

1. Click **"Select Items"** button in the subheader
2. Click on files/folders to select them
3. Selected items will be highlighted
4. Use bulk actions (delete, move, etc.)
5. Click **"Cancel Selection"** to exit selection mode

## Navigation

### Breadcrumbs

- Use breadcrumbs at the top to navigate back to parent folders
- Click on any folder name in the breadcrumb to go to that folder

### Back Navigation

- Use browser back button
- Or click on parent folder in breadcrumbs

## Pagination

When you have many files or folders:
- Pagination controls appear at the bottom
- Separate pagination for folders and files
- Use page numbers to navigate through pages

## Permissions

- **Upload Files**: Requires create permission on `/file_definition`
- **Create Folders**: Requires create permission on `/folder_definition`
- **Edit Files**: Requires update permission on `/file_definition`
- **Delete Files**: Requires delete permission on `/file_definition`
- **Manage Storage Config**: Requires permissions on `/storage_config_definition`

Actions will only appear if you have the required permissions.

