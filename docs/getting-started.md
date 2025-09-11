# Getting Started

After completing the installation of both Enfyra backend and app, follow these steps to get started.

## First Login

1. Navigate to your Enfyra app (default: `http://localhost:3000`)
2. You'll be redirected to the login page
3. Use the admin account created during backend setup:
   - The credentials you provided when setting up the backend
   - If you used default setup, check your backend console for the admin credentials

## After Login

Once logged in, you'll see the main interface with a sidebar containing:

### Sidebar Navigation

- **Dashboard** (Grid icon) - Overview and quick stats of your system
- **Data** (List icon) - Browse, create, edit and delete records in your tables
- **Collections** (Database icon) - Create and manage database tables/schemas
- **Settings** (Gear icon) - System configuration, users, roles, and permissions
- **Files Management** (Folder icon) - Upload and manage media files and documents

Clicking on sidebar items will expand submenus on the right side with additional options.

## Create Your First Table

1. Click on **Collections** in the sidebar
2. Click on **"+ Create New Table"** button in the left panel

### Important: Order of Creation

**You must create tables in the right order for relations to work:**
1. **Create target tables first** - If you want to create relations (like "Post → Category"), you must create the "Category" table before creating the "Post" table
2. **Add columns before constraints** - Unique constraints and indexes need existing columns to reference
3. **Add columns/relations before constraints** - Both unique constraints and indexes need existing fields to select from

### Table Creation Form

You'll see the table creation form with these sections:

   - **Table Name** - Enter a name for your table (e.g., "products", "posts")
   - **Description** - Optional description of what this table stores
   - **Unique Constraints** - Define unique field combinations (only appears after you add columns)
   - **Index** - Add database indexes for better performance (only appears after you add columns)  
   - **Columns** - Add fields to your table (an "id" field is added by default)
     - Click green **"+ Add Column"** button to add new fields
     - Configure column properties in the drawer:
       - **name** - Field name (required, cannot start with number or \_)
       - **type** - Data type:
         - `uuid` - Auto-generated UUID (sets isGenerated to true automatically)
         - `varchar` - Variable length text
         - `text` - Long text content
         - `int`, `bigint`, `number` - Numeric values
         - `boolean` - True/false values
         - `date`, `timestamp` - Date/time values
         - `enum` - Single selection from predefined options
         - `array-select` - Multiple selection from predefined options
         - `simple-json` - JSON data
         - `richtext` - Rich text with formatting
         - `code` - Code snippets with syntax highlighting
       - **isNullable** - Toggle to allow empty/null values
       - **defaultValue** - Default value for new records:
         - For `boolean`: true/false selector
         - For `int`/`number`: numeric input
         - For `date`: date picker
         - For `enum`: dropdown from options
         - For `array-select`: multi-select from options
         - For `uuid`: auto-generated (disabled input)
         - For text types: text input
       - **isUpdatable** - Toggle to allow field updates after creation
       - **isGenerated** - Auto-set for `uuid` type (system managed)
       - **isHidden** - Hide field from API responses
       - **description** - Field documentation with rich text editor (displays as help text under field labels in forms)
       - **placeholder** - Input placeholder text for forms
       - **options** - For `enum`/`array-select` types: define available choices
   - **Unique Constraints** - Ensure field combinations are unique across records
     - **Only appears after you add columns** - You need existing columns to select from
     - Click green **"+ Add"** button to create a constraint group
     - Select one or more fields that must be unique together
     - **Important**: Creating a unique constraint automatically creates an index, so you don't need to add a separate index for the same fields
   - **Index** - Improve query performance on specific fields
     - **Only appears after you add columns and relations** - You can index both table columns and relation fields
     - Click green **"+ Add"** button to create an index group
     - Select one or more fields to index together
     - **Tip**: Don't create indexes on fields that already have unique constraints
   - **Relations** - Define relationships with other tables
     - **Only shows existing tables** - You must create target tables first before you can relate to them
     - Displays existing relations with badges showing type, target table, and nullable status
     - Click on existing relation to edit, or click green **"+ Add Relation"** button for new
     - Configure relation properties in the drawer:
       - **type** - Relationship type (one-to-one, one-to-many, many-to-one, many-to-many)
       - **propertyName** - Name of the property in current table (required)
       - **inversePropertyName** - Name of the reverse property in target table (optional)
       - **targetTable** - Select the target table from available tables (only shows tables you've already created)
       - **isNullable** - Toggle to allow null relations
       - **description** - Relation documentation with rich text editor
     - Relations list shows: property name, type badge, target table badge (→ TargetTable), nullable badge if applicable
     - **Note**: Once created, these relations will appear as fields with pencil icons in forms - see [Relation Picker System](./relation-picker.md) for how to use them

4. After configuring your table, click the green **"+ Create New Table"** button at the top right to save it

Once saved, Enfyra will automatically generate REST and GraphQL APIs for your new table.
