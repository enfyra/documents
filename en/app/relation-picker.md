# Relation Picker System

The Relation Picker lets you select related records when working with forms. When you see a **pencil icon** next to a form field, that's a relation field that connects to records in another table. Instead of typing IDs, the Relation Picker provides a user-friendly way to browse and select related data.

**Schema vs. forms:** The picker is for **choosing related rows in create/edit forms**. Defining relations, **incoming** links from other tables, and **Create Inverse** (reverse property name on the current table) are done in **table schema  Relations** — see [Table Creation Guide](../getting-started/table-creation.md#relations-setup).

## How to Use Relation Fields

**Select Related Records:**
1. **Click the field** with the pencil icon
2. **Choose records** from the list that appears
3. **Click "Apply"** to confirm your selection

**For Single Selection** (like choosing one customer):
- Click a record to select it
- Clicking another record replaces your selection

**For Multiple Selection** (like choosing multiple categories):
- Click records to add/remove them from your selection  
- Selected records show a checkmark

## Additional Actions

The picker only selects existing records. To create a record, use the related table's own page, then reload the picker.

**Filter Records:**  
- Use **Filter** to narrow the available records; the picker has no separate quick-search field
- Use the same filtering system as described in the [Filter System](./filter-system.md) guide

**Browse Pages:**
- The list is a table with cursor pagination; use **Previous** / **Next** to move through pages
- Moving between pages does not clear your selection
- The table header shows the total number of records matching the current filter, not just the current page; paging keeps that total unchanged

**View Record Details:**
- Click the **external-link icon** next to any record to open its detail page
- This doesn't affect your current selection

## Common Examples

**Blog Post  Categories:**
- **"Categories" field** shows pencil icon
- **Click field**  see list of all categories
- **Click multiple categories**  they get selected with checkmarks
- **Click "Apply"**  field shows "3 categories selected"

**Order  Customer:**
- **"Customer" field** shows pencil icon  
- **Click field**  see list of all customers
- **Click one customer**  previous selection is replaced
- **Click "Apply"**  field shows the selected customer's name

**Product  Supplier:**
- **"Supplier" field** shows pencil icon
- **Click field**  see many suppliers
- **Click Filter button**  search by location or name
- **Select supplier**  field shows the chosen supplier

## Tips

**Finding Records:**
- Use the Filter button if you have many records to choose from
- Look for meaningful names (the system shows name, title, or other identifying fields)
- Use the external-link icon to open a record's detail page before selecting

**Working Efficiently:**
- For multiple selections, you can select several records before clicking Apply
- Click Apply to keep your choices. Closing without applying discards draft changes; reopening restores the last confirmed selection.