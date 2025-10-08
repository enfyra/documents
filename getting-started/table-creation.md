# Table Creation Guide

This guide walks you through creating your first table in Enfyra, from basic setup to understanding what happens after creation.

## Accessing Table Creation

1. Click on **Collections** in the sidebar
2. Click on **"+ Create New Table"** button in the left panel

## Important: Order of Creation

**You must create tables in the right order for relations to work:**

1. **Create target tables first** - If you want to create relations (like "Post → Category"), you must create the "Category" table before creating the "Post" table
2. **Add columns before constraints** - Unique constraints and indexes need existing columns to reference
3. **Add columns/relations before constraints** - Both unique constraints and indexes need existing fields to select from

## Table Creation Form

You'll see the table creation form with these sections:

### Basic Information
- **Table Name** - Enter a name for your table (e.g., "products", "posts")
- **Description** - Optional description of what this table stores

### Columns Configuration

**Columns** - Add fields to your table (an "id" field is added by default)
- Click green **"+ Add Column"** button to add new fields
- Configure column properties in the drawer:

#### Column Properties

**Basic Settings:**
- **name** - Field name (required, cannot start with number or \_)
- **type** - Data type (see [Field Types](#field-types) below)
- **isNullable** - Toggle to allow empty/null values
- **isUpdatable** - Toggle to allow field updates after creation
- **isGenerated** - Auto-set for `uuid` type (system managed)
- **isHidden** - Hide field from API responses

**Display & UX:**
- **description** - Field documentation with rich text editor (displays as help text under field labels in forms)
- **placeholder** - Input placeholder text for forms
- **options** - For `enum`/`array-select` types: define available choices

**Default Values:**
- **defaultValue** - Default value for new records:
  - For `boolean`: true/false selector
  - For `int`/`number`: numeric input
  - For `date`: date picker
  - For `enum`: dropdown from options
  - For `array-select`: multi-select from options
  - For `uuid`: auto-generated (disabled input)
  - For text types: text input

#### Field Types

| Type | Description | Use Case |
|------|-------------|----------|
| `uuid` | Auto-generated UUID (sets isGenerated to true automatically) | Unique identifiers |
| `varchar` | Variable length text | Names, titles, short text |
| `text` | Long text content | Descriptions, articles |
| `int`, `bigint`, `number` | Numeric values | Prices, quantities, IDs |
| `boolean` | True/false values | Flags, status indicators |
| `date`, `timestamp` | Date/time values | Created dates, deadlines |
| `enum` | Single selection from predefined options | Status, categories |
| `array-select` | Multiple selection from predefined options | Tags, multiple categories |
| `simple-json` | JSON data | Complex structured data |
| `richtext` | Rich text with formatting | Articles, descriptions with formatting |
| `code` | Code snippets with syntax highlighting | Code examples, scripts |

### Advanced Configuration

#### Unique Constraints
- **Only appears after you add columns** - You need existing columns to select from
- Click green **"+ Add"** button to create a constraint group
- Select one or more fields that must be unique together
- **Important**: Creating a unique constraint automatically creates an index, so you don't need to add a separate index for the same fields

#### Index Configuration
- **Only appears after you add columns and relations** - You can index both table columns and relation fields
- Click green **"+ Add"** button to create an index group
- Select one or more fields to index together
- **Tip**: Don't create indexes on fields that already have unique constraints

#### Relations Setup
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
- **Note**: Once created, these relations will appear as fields with pencil icons in forms - see [Relation Picker System](../frontend/relation-picker.md) for how to use them

## Saving Your Table

After configuring your table, click the green **"+ Create New Table"** button at the top right to save it.

## What Happens After Creating a Table

Once saved, Enfyra automatically sets up several things for your new table:

### 1. API Endpoints Generated on Backend

- REST and GraphQL APIs are instantly available on your **backend server** (port 1105)
- **4 CRUD endpoints** are automatically created:
  - `GET /[your-table-name]` - List all records
  - `POST /[your-table-name]` - Create new record
  - `PATCH /[your-table-name]/:id` - Update record
  - `DELETE /[your-table-name]/:id` - Delete record
- **Important**: APIs exist on backend, frontend consumes them via HTTP requests

### 2. Route Automatically Created

- A new route `/[your-table-name]` is automatically generated on the backend
- This route handles the 4 CRUD operations above
- **You can view this route**: Go to **Settings** → **Routings** in the sidebar to see all your table routes

### 3. Frontend Integration

- Your new table appears in the **Data** section of the sidebar
- Frontend makes HTTP requests to backend APIs to interact with data
- Click **Data** to see your table listed and start adding records

## Next Steps

- Navigate to **Data** → **[Your Table Name]** to start adding records - see [Data Management](./data-management.md) for complete guide
- **Remember**: All data operations flow through backend APIs, frontend never touches database directly
- Use the [Relation Picker System](../frontend/relation-picker.md) when working with relation fields
- Use the [Filter System](../frontend/filter-system.md) to search and filter your data

## Best Practices

### Table Design
- **Plan relationships first** - Create target tables before tables that reference them
- **Use descriptive names** - Clear table and column names improve maintainability
- **Set appropriate constraints** - Use unique constraints and indexes for data integrity and performance

### Column Configuration
- **Add descriptions** - Help users understand field purposes
- **Set sensible defaults** - Reduce data entry effort
- **Use appropriate types** - Match field types to data requirements

### Performance Considerations
- **Index frequently queried fields** - Improve query performance
- **Avoid over-indexing** - Too many indexes can slow down writes
- **Consider data volume** - Plan for growth in your table design

## Common Patterns

### User Management Table
```
Table: user_definition (built-in, but as example)
- id (uuid, generated)
- email (varchar, unique)
- name (varchar)
- isActive (boolean, default: true)
- createdAt (timestamp, default: now)
```

### Product Catalog
```
Table: products
- id (uuid, generated)
- name (varchar)
- description (text)
- price (number)
- categoryId (relation to categories)
- isActive (boolean, default: true)
```

### Blog System
```
Table: categories (create first)
- id (uuid, generated)
- name (varchar, unique)
- slug (varchar, unique)

Table: posts (create after categories)
- id (uuid, generated)
- title (varchar)
- content (richtext)
- categoryId (relation to categories)
- publishedAt (timestamp, nullable)
```

## Troubleshooting

### Common Issues

**Cannot create relation:**
- Ensure target table exists first
- Check that you're selecting the correct target table

**Unique constraint errors:**
- Add columns before creating constraints
- Ensure constraint fields are properly selected

**Index creation fails:**
- Add columns and relations before creating indexes
- Avoid indexing fields that already have unique constraints

**Table creation fails:**
- Check table name doesn't conflict with existing tables
- Ensure all required fields are properly configured
- Verify relation configurations are valid

## Related Documentation

- **[Data Management](./data-management.md)** - Working with records in your tables
- **[Relation Picker System](../frontend/relation-picker.md)** - Using relation fields in forms
- **[Filter System](../frontend/filter-system.md)** - Searching and filtering data
- **[Form System](../frontend/form-system.md)** - How forms are generated from tables
- **[API Querying](../server/api-querying.md)** - Advanced querying capabilities
