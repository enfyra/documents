# Enfyra Configuration

The `enfyra.config.ts` file is the central configuration file for your Enfyra application. It allows you to customize various aspects of the application, including the Rich Text Editor, and can be extended with additional configuration options as needed.

## File Location

The configuration file is located at the root of your Enfyra app:

```
app/
├── enfyra.config.ts          # Main configuration file
├── enfyra.config.types.ts    # TypeScript type definitions
└── ...
```

## Configuration Structure

The `enfyra.config.ts` file exports a single `enfyraConfig` object that follows the `EnfyraConfig` interface:

```typescript
import type { EnfyraConfig } from "./enfyra.config.types";

export const enfyraConfig: EnfyraConfig = {
  richText: {
    // Rich Text Editor configuration
  },
  // Additional configuration sections can be added here
};
```

All configuration properties are **optional**, allowing you to configure only what you need.

## Rich Text Editor Configuration

The `richText` section configures the TinyMCE Rich Text Editor used throughout the application.

### Basic Configuration

```typescript
export const enfyraConfig: EnfyraConfig = {
  richText: {
    plugins: ['link', 'lists', 'code', 'table'],
    toolbar: 'undo redo | bold italic underline | codeinline | bullist numlist | link table | code',
  },
};
```

### Available Options

#### `plugins?: string[]`

Array of TinyMCE plugins to enable. Available plugins include:
- `'link'` - Link insertion and editing
- `'lists'` - Bulleted and numbered lists
- `'code'` - Code block viewing
- `'table'` - Table creation and editing

**Example:**
```typescript
plugins: ['link', 'lists', 'code', 'table']
```

#### `toolbar?: string`

Toolbar configuration string. Defines which buttons appear in the editor toolbar, separated by `|` for groups.

**Example:**
```typescript
toolbar: 'undo redo | bold italic underline | codeinline | bullist numlist | link table | code'
```

**Available built-in buttons:**
- `undo`, `redo` - Undo/redo operations
- `bold`, `italic`, `underline` - Text formatting
- `bullist`, `numlist` - Lists
- `link` - Insert/edit links
- `table` - Table operations
- `code` - Code block viewer

#### `customButtons?: RichTextEditorButtonConfig[]`

Array of custom buttons to add to the toolbar.

**Button Configuration:**
```typescript
{
  name: string;                    // Required: Unique button identifier (used in toolbar string)
  text?: string;                    // Optional: Button label text
  tooltip?: string;                 // Optional: Tooltip on hover
  format?: string;                  // Optional: Shorthand - format name to toggle (e.g., 'code')
  onAction?: string | ((editor: any, params?: any) => void);  // Optional: Custom action handler
  params?: any[];                  // Optional: Parameters for command-based actions
}
```

**Shorthand - Format Toggle (Recommended):**
If you only need to toggle a format, use the `format` property:
```typescript
customButtons: [
  {
    name: 'codeinline',
    text: 'Code',
    tooltip: 'Inline code',
    format: 'code',  // ← Automatically toggles format 'code'
  },
]
```

**Custom Action Handler:**
For custom logic, use `onAction`:
```typescript
customButtons: [
  {
    name: 'codeinline',
    text: 'Code',
    tooltip: 'Inline code',
    onAction: (editor: any) => {
      editor.execCommand('mceToggleFormat', false, 'code');
    },
  },
]
```

**Example - Custom Insert Button:**
```typescript
customButtons: [
  {
    name: 'inserttimestamp',
    text: 'Timestamp',
    tooltip: 'Insert current timestamp',
    onAction: (editor: any) => {
      const now = new Date().toLocaleString();
      editor.insertContent(`<span>${now}</span>`);
    },
  },
]
```

**Using Command Strings:**
You can also use TinyMCE command strings directly:
```typescript
{
  name: 'mybutton',
  text: 'My Button',
  onAction: 'mceInsertContent',  // TinyMCE command
  params: ['<p>Hello</p>'],       // Command parameters
}
```

**Note:** Make sure to include the button name in your `toolbar` string:
```typescript
toolbar: '... | codeinline | inserttimestamp | ...'
```

#### `formats?: Record<string, FormatConfig>`

Custom format definitions for text styling. **The format key automatically becomes the HTML tag name** (e.g., key `'code'` creates `<code>` tag).

**Format Configuration:**
```typescript
{
  classes?: string | string[] | ((theme: 'light' | 'dark') => string | string[]);  // CSS classes to add to the tag
  css?: Record<string, string> | ((theme: 'light' | 'dark') => Record<string, string>);  // CSS styles for the tag (injected as stylesheet, not inline)
  classStyles?: Record<string, Record<string, string> | ((theme: 'light' | 'dark') => Record<string, string>)>;  // CSS styles for specific classes
  attributes?: Record<string, string>; // HTML attributes
}
```

**Key Behavior:**
- The format **key automatically becomes the HTML tag name**
- Example: `code: { ... }` creates `<code>text</code>`
- Example: `span: { ... }` creates `<span>text</span>`
- Example: `highlight: { ... }` creates `<highlight>text</highlight>`

**Understanding Format Properties:**

- **Format Key**: Automatically becomes the HTML tag name
  - `code: { ... }`  creates `<code>text</code>`
  - `span: { ... }`  creates `<span>text</span>`
  - `strong: { ... }`  creates `<strong>text</strong>`

- **`classes`**: Adds CSS classes to the tag (not inline styles)
  - **String**: `classes: 'my-class'`  creates `<code class="my-class">`
  - **Array**: `classes: ['class1', 'class2']`  creates `<code class="class1 class2">`
  - **Function**: `classes: (theme) => `code-${theme}``  creates `<code class="code-light">` or `<code class="code-dark">`

- **`css`**: CSS styles injected as stylesheet (not inline styles)
  - Styles are applied to the tag directly (e.g., `code { ... }`)
  - **Static object**: `css: { backgroundColor: '#f4f4f4' }`
  - **Function**: `css: (theme) => ({ ... })` - Dynamically generates styles based on current theme

- **`classStyles`**: CSS styles for specific classes
  - Styles are applied to the tag with specific class (e.g., `code.my-class { ... }`)
  - Each key in `classStyles` is a class name, value is the CSS styles
  - Can override or extend base `css` styles

- **`attributes`**: Adds HTML attributes to the element
  - Example: `attributes: { title: 'Tooltip', 'data-custom': 'value' }`

**Example - Basic Format with CSS:**
```typescript
formats: {
  code: {
    css: (theme: 'light' | 'dark') => ({
      backgroundColor: theme === 'dark' ? '#2d2d2d' : '#f4f4f4',
      color: theme === 'dark' ? '#f8f8f2' : '#333',
      padding: '2px 4px',
      borderRadius: '3px',
      fontFamily: 'monospace',
    }),
  },
}
```
Result: Creates `<code>` tag with CSS styles injected as stylesheet (`code { ... }`)

**Example - Format with Classes:**
```typescript
formats: {
  highlight: {
    classes: 'text-highlight',
    css: (theme: 'light' | 'dark') => ({
      backgroundColor: theme === 'dark' ? '#3d3d3d' : '#f0f0f0',
    }),
    classStyles: {
      'text-highlight': (theme: 'light' | 'dark') => ({
        backgroundColor: theme === 'dark' ? '#ffeb3b' : '#fff59d',
        color: theme === 'dark' ? '#000' : '#333',
        padding: '2px 6px',
        borderRadius: '4px',
      }),
    },
  },
}
```
Result: Creates `<highlight class="text-highlight">` tag with:
- Base styles: `highlight { ... }`
- Class-specific styles: `highlight.text-highlight { ... }`

**Example - Multiple Classes:**
```typescript
formats: {
  highlight: {
    classes: (theme) => [`highlight-${theme}`, 'text-highlight'],
    classStyles: {
      'highlight-light': {
        backgroundColor: '#fff59d',
      },
      'highlight-dark': {
        backgroundColor: '#ffeb3b',
      },
      'text-highlight': {
        fontWeight: 'bold',
      },
    },
  },
}
```
Result: Creates `<highlight class="highlight-light text-highlight">` or `<highlight class="highlight-dark text-highlight">` with styles for each class

**Complete Example:**
```typescript
formats: {
  code: {
    css: (theme: 'light' | 'dark') => ({
      backgroundColor: theme === 'dark' ? '#2d2d2d' : '#f4f4f4',
      color: theme === 'dark' ? '#f8f8f2' : '#333',
      padding: '2px 4px',
      borderRadius: '3px',
      fontFamily: 'monospace',
    }),
  },
  highlight: {
    classes: 'text-highlight',
    css: (theme: 'light' | 'dark') => ({
      backgroundColor: theme === 'dark' ? '#3d3d3d' : '#f0f0f0',
    }),
    classStyles: {
      'text-highlight': (theme: 'light' | 'dark') => ({
        backgroundColor: theme === 'dark' ? '#ffeb3b' : '#fff59d',
        color: theme === 'dark' ? '#000' : '#333',
        padding: '2px 6px',
        borderRadius: '4px',
      }),
    },
  },
}
```

### Complete Rich Text Editor Example

```typescript
import type { EnfyraConfig } from "./enfyra.config.types";

export const enfyraConfig: EnfyraConfig = {
  richText: {
    plugins: ['link', 'lists', 'code', 'table'],
    toolbar: 'undo redo | bold italic underline | codeinline | bullist numlist | link table | code',
    customButtons: [
      {
        name: 'codeinline',
        text: 'Code',
        tooltip: 'Inline code',
        format: 'code',  // ← Shorthand: automatically toggles format 'code'
      },
      {
        name: 'inserttimestamp',
        text: 'Timestamp',
        tooltip: 'Insert current timestamp',
        onAction: (editor: any) => {
          const now = new Date().toLocaleString();
          editor.insertContent(`<span>${now}</span>`);
        },
      },
    ],
    formats: {
      code: {  // ← Key 'code' automatically becomes <code> tag
        styles: (theme: 'light' | 'dark') => ({
          backgroundColor: theme === 'dark' ? '#2d2d2d' : '#f4f4f4',
          color: theme === 'dark' ? '#f8f8f2' : '#333',
          padding: '2px 4px',
          borderRadius: '3px',
          fontFamily: 'monospace',
        }),
      },
    },
  },
};
```

## TypeScript Support

The configuration file is fully typed. Type definitions are available in `enfyra.config.types.ts`:

```typescript
export interface RichTextEditorButtonConfig {
  name: string;
  text?: string;
  tooltip?: string;
  format?: string;  // Shorthand: format name to toggle
  onAction?: string | ((editor: any, params?: any) => void);
  params?: any[];
}

export interface RichTextEditorConfig {
  plugins?: string[];
  toolbar?: string;
  customButtons?: RichTextEditorButtonConfig[];
  buttonActions?: Record<string, (editor: any, params?: any) => void>;
  formats?: Record<string, {
    classes?: string | string[] | ((theme: 'light' | 'dark') => string | string[]);
    css?: Record<string, string> | ((theme: 'light' | 'dark') => Record<string, string>);
    classStyles?: Record<string, Record<string, string> | ((theme: 'light' | 'dark') => Record<string, string>)>;
    attributes?: Record<string, string>;
  }>;
}

export interface EnfyraConfig {
  richText?: RichTextEditorConfig;
}
```

## Best Practices

1. **Keep it minimal**: Only configure what you need. All properties are optional.

2. **Use `format` shorthand**: For simple format toggles, use `format: 'code'` instead of writing `onAction`:
   ```typescript
   //  Recommended
   { name: 'codeinline', format: 'code' }
   
   //  Unnecessary
   { name: 'codeinline', onAction: (editor) => editor.execCommand('mceToggleFormat', false, 'code') }
   ```

3. **Use theme-aware styles**: When defining formats, use function-based `css` or `classStyles` to support both light and dark themes:
   ```typescript
   css: (theme: 'light' | 'dark') => ({
     // Theme-aware styles
   })
   ```

4. **Format key = HTML tag**: The format key automatically becomes the HTML tag name, so choose meaningful keys:
   ```typescript
   formats: {
     code: { ... },      //  <code>
     highlight: { ... }, //  <highlight>
     span: { ... },      //  <span>
   }
   ```

5. **CSS vs Inline Styles**: Use `css` for stylesheet-based styling (recommended). Styles are injected into the editor's iframe, not as inline styles:
   ```typescript
   //  Recommended - CSS stylesheet
   css: { backgroundColor: '#f4f4f4' }
   
   //  Not supported - inline styles are not used
   ```

6. **Classes and Class Styles**: Use `classes` to add CSS classes to tags, and `classStyles` to style specific classes:
   ```typescript
   formats: {
     highlight: {
       classes: 'text-highlight',  // Adds class to tag
       classStyles: {
         'text-highlight': { ... }  // Styles for that class
       }
     }
   }
   ```

5. **Button naming**: Use descriptive names for custom buttons that match their purpose (e.g., `codeinline`, `inserttimestamp`).

6. **Toolbar organization**: Group related buttons together using `|` separators for better UX.

## Extending the Configuration

The `EnfyraConfig` interface can be extended to include additional configuration sections as needed. Simply add new properties to the interface and use them in your configuration:

```typescript
// In enfyra.config.types.ts
export interface EnfyraConfig {
  richText?: RichTextEditorConfig;
  // Add more configuration sections here
  // customFeature?: CustomFeatureConfig;
}
```

## Troubleshooting

### Button not appearing in toolbar

- Ensure the button `name` is included in the `toolbar` string
- Check that the button is properly defined in `customButtons`
- If using `format` shorthand, make sure the format exists in `formats`
- If using `onAction`, verify it's correctly implemented

### Format not applying

- Make sure the format name in `execCommand` matches the key in `formats`
- The format key automatically becomes the HTML tag name (e.g., `code: { ... }` creates `<code>`)
- Verify `css` or `classStyles` are properly formatted (object or function)

### Theme not updating

- Ensure `css` or `classStyles` is a function that accepts `theme: 'light' | 'dark'`
- The theme is automatically detected from the app's color mode

### Styles not appearing

- Styles are injected as CSS stylesheet, not inline styles
- Check browser DevTools to see if CSS rules are being applied
- Verify CSS selectors match the generated HTML (e.g., `code { ... }` for `<code>` tag, `code.my-class { ... }` for `<code class="my-class">`)

## Related Documentation

- [Form System](../app/form-system.md) - How Rich Text Editor is used in forms
- [TinyMCE Documentation](https://www.tiny.cloud/docs/) - Official TinyMCE documentation

