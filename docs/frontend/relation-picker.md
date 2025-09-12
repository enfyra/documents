# Relation Picker System

The Relation Picker lets you select related records when working with forms. When you see a **pencil icon** next to a form field, that's a relation field that connects to records in another table. Instead of typing IDs, the Relation Picker provides a user-friendly way to browse and select related data.

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

**Create New Records:**
- Click the **"+ Create"** button to add new records to the related table
- Fill out the form that appears
- New records are automatically selected when created

**Filter Records:**  
- Click the **Filter** button to search through available records
- Use the same filtering system as described in the [Filter System](./filter-system.md) guide

**View Record Details:**
- Click the **eye icon** next to any record to see its full details
- This doesn't affect your current selection

## Common Examples

**Blog Post → Categories:**
- **"Categories" field** shows pencil icon
- **Click field** → see list of all categories
- **Click multiple categories** → they get selected with checkmarks
- **Click "Apply"** → field shows "3 categories selected"

**Order → Customer:**
- **"Customer" field** shows pencil icon  
- **Click field** → see list of all customers
- **Click one customer** → previous selection is replaced
- **Click "Apply"** → field shows the selected customer's name

**Product → Supplier:**
- **"Supplier" field** shows pencil icon
- **Click field** → see many suppliers
- **Click Filter button** → search by location or name
- **Select supplier** → field shows the chosen supplier

## Tips

**Finding Records:**
- Use the Filter button if you have many records to choose from
- Look for meaningful names (the system shows name, title, or other identifying fields)
- Use the eye icon to preview a record's full details before selecting

**Working Efficiently:**
- Create new related records directly from the picker without leaving your form
- For multiple selections, you can select several records before clicking Apply
- Selected records persist if you close and reopen the picker (until you Apply or Cancel)