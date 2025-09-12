# Filter System

The Filter System helps you search and filter data in your tables. Instead of scrolling through many records, create search conditions to find exactly what you need. Look for the **Filter** button - it shows "Filter" normally, and "Filters (N)" when active.

## How to Filter Data

**Start Filtering:**
1. Click the **Filter** button in your table toolbar
2. Click **"+ Add Filter"** to create your first condition

**Build a Condition:**
1. **Choose a field** from the first dropdown (your table columns or related table fields)
2. **Pick how to compare** from the second dropdown (equals, contains, greater than, etc.)  
3. **Enter your search value** in the input box (if needed)

**Apply Your Filter:**
- Click **"Apply Filters"** to search your data
- Your table updates to show only matching records
- The Filter button now shows "Filters (X)" to indicate it's active

## Multiple Conditions and Logic

**Add More Conditions:**
- Click **"+ Add Filter"** again for additional search criteria
- Choose **"AND"** if all conditions must be true
- Choose **"OR"** if any condition can be true

**Group Complex Logic:**
- Click **"+ Add Group"** to create nested conditions
- Each group has its own AND/OR logic
- Use this for complex searches like: *(name contains "John" OR email contains "john") AND status = "active"*

**Remove Conditions:**
- Click the **X button** on any condition to remove it
- Click **"Clear All"** to remove everything and start over

## Using Saved Filters

**Auto-Save:**
- When you apply filters, they're automatically saved for later use
- Each filter gets a smart name like "status = active" or "name contains John"

**Reuse Filters:**
- Click any saved filter to instantly apply it
- Popular filters (most used) appear at the top
- Rename filters by clicking the edit button
- Delete unwanted filters with the trash button

**Search Saved Filters:**
- If you have many saved filters, use the search box to find specific ones

## Common Filter Examples

**Text Searches:**
- Find customers: `name contains "Smith"`
- Find emails: `email ends with "@company.com"`
- Find empty descriptions: `description is empty`

**Number Ranges:**
- Find expensive products: `price > 100`
- Find age range: `age between 25 and 65`
- Find recent orders: `total >= 50`

**Date Filtering:**
- Recent records: `created_date > "2024-01-01"`
- Date range: `order_date between "2024-01-01" and "2024-12-31"`

**Multiple Conditions:**
- Active customers in New York: `status = "active" AND city = "New York"`
- Premium or VIP customers: `plan = "premium" OR plan = "VIP"`

**Relation Filtering:**
- Orders from specific customers: `customer → name contains "John"`
- Products in certain categories: `category → name = "Electronics"`
- **Note**: These relation fields come from table relationships you create - see [Getting Started](../getting-started/getting-started.md) for setting up relations, and [Relation Picker System](./relation-picker.md) for selecting related records

## Tips for Better Filtering

**Keep It Simple:**
- Start with one condition, add more as needed
- Use clear, descriptive names when renaming saved filters
- Delete old filters you no longer use

**Performance:**
- Simple filters (like status = "active") are fastest
- Complex nested relations may take longer to load
- Use specific conditions rather than very broad searches

**Organization:**
- Save commonly used filters for quick access
- Use groups for complex "and/or" logic
- Name your custom filters clearly (e.g., "Active NY Customers" instead of "Filter 1")