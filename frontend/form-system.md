# Form System

Enfyra's form system automatically generates forms from your database schema. Forms are dynamic, validated, and fully integrated with permissions and relations.

## How Forms Work

When you create a table in Enfyra, the system automatically generates forms for creating and editing records. These forms:

- Display appropriate input types based on field types
- Validate data according to schema rules
- Handle relations between tables
- Respect user permissions

## Field Types and Input Components

### Basic Field Types

| Field Type         | Input Component | Description                               |
| ------------------ | --------------- | ----------------------------------------- |
| **varchar/string** | Text input      | Single-line text field                    |
| **text**           | Textarea        | Multi-line text area                      |
| **int/number**     | Number input    | Numeric input with validation             |
| **boolean**        | Toggle switch   | On/off toggle                             |
| **date/timestamp** | Date picker     | Calendar date selector                    |
| **uuid**           | UUID field      | Auto-generated unique ID with copy button |

### Advanced Field Types

| Field Type       | Input Component  | Description                          |
| ---------------- | ---------------- | ------------------------------------ |
| **richtext**     | Rich text editor | WYSIWYG editor with formatting tools |
| **code**         | Code editor      | Syntax-highlighted code input        |
| **simple-json**  | JSON editor      | JSON input with validation           |
| **enum**         | Dropdown         | Single selection from options        |
| **array-select** | Multi-select     | Multiple selections from options     |

### Special Field Types

| Field Type     | Component                                     | Description            |
| -------------- | --------------------------------------------- | ---------------------- |
| **relation**   | [Relation Picker](./relation-picker.md)       | Select related records |
| **permission** | [Permission Builder](./permission-builder.md) | Configure access rules |

## Form Layout

### Responsive Grid Layout

- Forms use a 2-column grid on desktop, 1-column on mobile
- Full-width fields automatically span both columns:
  - Rich text editors
  - Code editors
  - JSON editors
  - Long text areas

### Field Ordering

1. Regular table columns appear first
2. Relation fields appear after column fields
3. Custom ordering via configuration

## Form Validation

### Required Fields

- Fields marked with red asterisk (\*) are required
- Validation runs before form submission
- Clear error messages explain what's missing

### Validation Rules

- **Required**: Field cannot be empty (based on `isNullable` setting)
- **Type**: Input must match field type (number, date, etc.)
- **Format**: Special formats like email, URL (when configured)
- **Custom**: Business logic validation via hooks

### Error Display

- Field-level error messages appear below inputs
- Errors clear automatically when corrected
- Toast notifications for form-level issues

## Working with Relations

### One-to-One / Many-to-One

- Single selection from related table
- Shows selected record as badge
- Click to change selection

### One-to-Many / Many-to-Many

- Multiple selection from related table
- Shows count of selected records
- Manage selections in modal

**See [Relation Picker System](./relation-picker.md) for detailed usage**

## Permission Integration

Forms respect user permissions automatically:

### Field-Level Permissions

- Fields only show if user has read permission
- Edit capability based on update permission
- Delete options require delete permission

### Form-Level Permissions

- Create forms require create permission on table
- Edit forms require update permission
- Save button disabled without proper permissions

**How Permissions Work:**

1. **Backend validates** all permissions on API calls
2. **Frontend checks** permissions before showing UI elements
3. **Menus hide** options user cannot access
4. **Forms disable** fields user cannot edit

**See [Permission System](../server/permission-system.md) for technical details**

## Special Form Features

### Copy Field Values

- Click copy icon next to any field
- Useful for debugging or sharing data
- Works with all field types

### Field Placeholders

- Helpful hints appear in empty fields
- Configured per-field in table settings
- Guide users on expected input

### Field Descriptions

- Rich text descriptions below field labels
- Provide context and instructions
- Set in table column configuration

### Default Values

- Pre-filled values for new records
- Different defaults per field type
- Can be dynamic (like current date)

## Form States

### Loading State

- Skeleton loaders while fetching data
- Type-specific loading animations
- Smooth transitions when ready

### Edit Mode

- Change detection tracks modifications
- Save button enables only with changes
- Confirmation on unsaved changes

## Form Change Detection

Enfyra automatically tracks form changes to provide better user experience:

### How It Works

1. **Initial State**: Form loads with original data
2. **Change Detection**: Any modification triggers change tracking
3. **Save Button**: Only enabled when changes are detected
4. **Reset**: Changes reset after successful save

### Implementation Pattern

```typescript
// In your page component
const hasFormChanges = ref(false);
const formEditorRef = ref();

// Template
<FormEditorLazy
  ref="formEditorRef"
  v-model="form"
  v-model:errors="errors"
  @has-changed="(hasChanged) => hasFormChanges = hasChanged"
  table-name="user_definition"
  :loading="loading"
/>

// Save function
async function save() {
  // ... save logic
  await updateUser({ body: form.value });
  
  // Reset form changes after successful save
  formEditorRef.value?.confirmChanges();
}

// Header action
useHeaderActionRegistry([
  {
    id: "save-user",
    label: "Save",
    disabled: computed(() => !hasFormChanges.value),
    submit: save,
  },
]);
```

### Key Components

- **`hasFormChanges`**: Reactive boolean tracking form state
- **`formEditorRef`**: Reference to FormEditor component
- **`@has-changed`**: Event emitted when form state changes
- **`confirmChanges()`**: Method to reset change detection

### Benefits

- **Better UX**: Save button only appears when needed
- **Data Safety**: Prevents accidental saves
- **Clear Feedback**: Users know when changes exist
- **Automatic Reset**: No manual state management needed

## Reset Button Pattern

Enfyra provides a consistent reset button pattern to discard form changes:

### How It Works

1. **Change Detection**: Tracks when form data is modified
2. **Reset Button**: Appears only when changes exist
3. **Confirmation**: Asks user to confirm before discarding
4. **Restore**: Returns form to original state

### Implementation Pattern

```typescript
// In your page component
const { confirm } = useConfirm();
const hasFormChanges = ref(false);
const formEditorRef = ref();

// Reset function
async function handleReset() {
  const ok = await confirm({
    title: "Discard Changes",
    content: "Are you sure you want to discard all changes? This action cannot be undone.",
  });
  
  if (ok) {
    // Reset form to original state
    form.value = formChanges.discardChanges(form.value);
    formEditorRef.value?.confirmChanges();
  }
}

// Header actions
useHeaderActionRegistry([
  {
    id: "save-user",
    label: "Save",
    disabled: computed(() => !hasFormChanges.value),
    submit: save,
  },
  {
    id: "reset-user",
    label: "Reset",
    icon: "lucide:rotate-ccw",
    variant: "outline",
    color: "gray",
    side: "left",
    show: computed(() => hasFormChanges.value),
    onClick: handleReset,
  },
]);
```

### Using useSchema Pattern

For forms using `useSchema`, you can leverage the built-in `discardChanges` function:

```typescript
// Using useSchema composable
const { formChanges } = useSchema(tableName);

// Initialize form changes tracking
onMounted(async () => {
  const data = await fetchData();
  form.value = data;
  formChanges.update(data); // Track original state
});

// Reset function
async function handleReset() {
  const ok = await confirm({
    title: "Discard Changes",
    content: "Are you sure you want to discard all changes?",
  });
  
  if (ok) {
    // Use useSchema's discardChanges
    form.value = formChanges.discardChanges(form.value);
    formEditorRef.value?.confirmChanges();
  }
}
```

### Key Features

- **Conditional Display**: Reset button only shows when changes exist
- **Confirmation Dialog**: Prevents accidental data loss
- **Deep Reset**: Restores all form fields to original values
- **State Sync**: Updates form change detection state
- **Consistent UX**: Same pattern across all forms

### Best Practices

1. **Always Confirm**: Use confirmation dialog for destructive actions
2. **Clear Messaging**: Explain what will be lost
3. **Positioning**: Place reset button on the left side
4. **Icon Usage**: Use `lucide:rotate-ccw` for consistency
5. **State Management**: Always call `confirmChanges()` after reset

### Disabled Fields

- System fields cannot be edited
- Generated fields are read-only
- Conditional disabling via configuration

## Form Submission

### Create Flow

1. Fill in required fields
2. Validation runs automatically
3. Click "Create" to save
4. Success notification and redirect

### Update Flow

1. Modify existing values
2. Changes tracked automatically
3. Click "Save" when ready
4. Instant feedback on success

### Error Handling

- API errors show as notifications
- Field errors highlight specific inputs
- Retry capability for network issues

## Advanced Configuration

### TypeMap System

The `typeMap` prop allows you to customize form behavior for specific fields without modifying the database schema. Pass it to `FormEditorLazy` component:

```typescript
<FormEditorLazy
  v-model="form"
  :table-name="tableName"
  :type-map="typeMap"
/>
```

#### Basic TypeMap Usage

```typescript
// Example: Change field type
const typeMap = computed(() => ({
  description: {
    type: 'richtext'  // Make description a rich text field
  }
}));

// Example: Span full width
const typeMap = computed(() => ({
  content: {
    fieldProps: {
      class: 'col-span-2'
    }
  }
}));

// Example: Custom placeholder
const typeMap = computed(() => ({
  email: {
    placeholder: 'user@example.com'
  }
}));

// Example: Disable a field
const typeMap = computed(() => ({
  type: {
    disabled: true  // Field cannot be edited
  }
}));
```

#### Filtering Enum Options

For enum and array-select fields, you can filter which options appear in the dropdown:

**Exclude Options:**
```typescript
// Hide specific options from enum dropdown
const typeMap = computed(() => ({
  storageType: {
    excludedOptions: ['Local Storage', 'Legacy System']
  }
}));
```

**Include Only Specific Options:**
```typescript
// Show only specific options in enum dropdown
const typeMap = computed(() => ({
  status: {
    includedOptions: ['Active', 'Pending', 'Review']
  }
}));
```

**Complete Example - Storage Configuration:**
```typescript
// In storage config create page
const typeMap = computed(() => ({
  type: {
    excludedOptions: ['Local Storage']  // Hide Local Storage from create form
  }
}));

// In storage config detail page
const typeMap = computed(() => ({
  type: {
    disabled: true,  // Cannot change type after creation
    excludedOptions: ['Local Storage']  // Still hide it
  }
}));
```

#### TypeMap Options Reference

| Option | Type | Description |
|--------|------|-------------|
| `type` | `string` | Override field input type (richtext, code, etc.) |
| `disabled` | `boolean` | Make field read-only |
| `placeholder` | `string` | Custom placeholder text |
| `excludedOptions` | `string[]` | Hide these options from enum/array-select dropdowns |
| `includedOptions` | `string[]` | Show only these options in enum/array-select dropdowns |
| `fieldProps` | `object` | Additional props for field wrapper (e.g., `class: 'col-span-2'`) |
| `componentProps` | `object` | Additional props for input component |
| `options` | `any[]` | Override all options (use with excludedOptions/includedOptions for filtering) |

**Note:** `excludedOptions` and `includedOptions` work with the original enum options from your schema. They filter the options, not replace them.

### Field Visibility

- Include/exclude specific fields
- Conditional visibility based on values
- Role-based field display

## Integration with Header Actions

### Save Button

- Appears in header when editing
- Disabled when no changes
- Shows loading state during save

### Delete Button

- Available when viewing existing record
- Requires delete permission
- Confirmation dialog prevents accidents

### Custom Actions

- Add custom buttons via configuration
- Integrate with form data
- Permission-controlled visibility

## Best Practices

### Form Design

- Keep forms focused and concise
- Group related fields together
- Use descriptions for complex fields
- Provide helpful placeholders

### Validation

- Validate early and often
- Provide clear error messages
- Use frontend and backend validation
- Consider user experience

### Performance

- Lazy load heavy components
- Use pagination for relation pickers
- Optimize validation logic
- Cache schema when possible

## Common Scenarios

### Multi-Step Forms

While Enfyra uses single-page forms by default, you can:

- Use hooks to create wizard-style flows
- Conditionally show/hide sections
- Save drafts between steps

### Dependent Fields

- Use hooks to update fields based on others
- Calculate values automatically
- Validate field combinations

### File Uploads

- File fields integrate with file system
- Drag-and-drop support
- Progress indicators for uploads

## Troubleshooting

### Form Not Loading

- Check table schema exists
- Verify user has read permission
- Ensure API endpoint is accessible

### Validation Errors

- Review field requirements
- Check data types match schema
- Verify custom validation logic

### Save Failures

- Confirm user has update permission
- Check network connectivity
- Review API error messages

The form system provides everything needed for data entry while maintaining security, validation, and excellent user experience.
