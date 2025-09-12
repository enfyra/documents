# Data Management

After creating tables, you can manage your records through the Data section. Access it via **Data** in the sidebar, then select your table.

## Navigating to Your Table Data

**How to Access:**
1. Click **Data** in the sidebar
2. Select your table from the submenu
3. You'll see the data management page for that table

## Data Table View

**What You'll See:**
- **Table header** with your table name
- **Action buttons** in the top-right corner
- **Data table** showing all records with pagination
- **Empty state** if no records exist yet

## Main Actions

### Filter Button
- Shows **"Filter"** when no filters are active
- Changes to **"Filters (N)"** when filters are applied
- Click to open the filter drawer - see [Filter System](./filter-system.md)
- Active filters show as a badge above the table with a "Clear" button

### Create Button
- Blue **"Create"** button with plus icon
- Click to go to the record creation form
- Only appears if you have create permissions

### Select Items Mode
- Click **"Select Items"** to enable selection mode
- Button changes to **"Cancel Selection"** when active
- Select multiple records by clicking checkboxes
- **"Delete Selected (N)"** button appears when records are selected
- Only appears if you have delete permissions

### Column Selector
- Dropdown to show/hide table columns
- Select which fields to display in the table
- Your preferences are saved locally

## Viewing Records

**Table Display:**
- Each row represents one record
- **Click any row** to view and edit that record's details
- Fields display based on their type:
  - **Dates/Timestamps** show formatted date/time
  - **Booleans** show as "Yes/No" badges
  - **Long text** truncates at 50 characters
  - **Empty values** show as "-"

**Actions Column:**
- Three-dot menu on the right of each row
- **Delete** option (if you have permission)
- More actions may appear based on your permissions

**Pagination:**
- Shows at the bottom when you have multiple pages
- Navigate with Previous/Next buttons or page numbers
- URL updates with page number for bookmarking

## Creating New Records

**Click "Create" Button:**
1. Opens the creation form page
2. Form shows all fields based on your table schema
3. Required fields are marked with asterisks
4. Relation fields show with pencil icons - see [Relation Picker](./relation-picker.md)

**Form Behavior:**
- Fields appear based on column configuration
- Default values are pre-filled
- Field descriptions show as help text below labels
- Validation runs when you save

**Save Process:**
1. Click **"Save"** button in the top-right
2. Validation checks all required fields
3. If invalid, error messages appear under fields
4. If valid, record is created and you're redirected to the edit page
5. Success notification appears

## Editing Records

**Click Any Row in the Table:**
1. Opens the edit form for that record
2. All current values are loaded
3. Form looks similar to create form

**Edit Form Features:**
- **Save button** - Only enabled when you make changes
- **Delete button** - Red button to delete the record
- Field validation works the same as creation
- Changes are tracked automatically

**Making Changes:**
1. Modify any fields you need
2. Save button becomes enabled when changes are detected
3. Click **"Save"** to update the record
4. Success notification confirms the update

**Deleting a Record:**
1. Click the red **"Delete"** button
2. Confirmation dialog appears
3. Confirm to delete the record
4. You're redirected back to the table view

## Bulk Operations

**Selecting Multiple Records:**
1. Click **"Select Items"** to enable selection
2. Checkboxes appear on each row
3. Click checkboxes to select records
4. Selected count shows in the button

**Bulk Delete:**
1. Select records you want to delete
2. Click **"Delete Selected (N)"** button
3. Confirm the bulk deletion
4. All selected records are deleted
5. Table refreshes automatically

## Working with Related Data

**Relation Fields in Forms:**
- Show with a pencil icon
- Click to open the relation picker
- Select related records from other tables
- See [Relation Picker System](./relation-picker.md) for details

**Viewing Relations:**
- Related data may show as IDs or names in the table
- Click the row to see full relation details in edit form
- Use filters to search by related data

## Tips for Data Management

**Performance:**
- Use filters to find records quickly
- Hide unnecessary columns for cleaner view
- Pagination loads data in chunks for speed

**Organization:**
- Save commonly used filters
- Customize visible columns per table
- Use bulk operations for multiple changes

**Navigation:**
- URLs update with filters and pages - bookmark specific views
- Browser back/forward works with navigation
- Table state persists when you return