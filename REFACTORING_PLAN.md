# Kế Hoạch Refactoring Tài Liệu - Chi Tiết Theo Phase

## 📋 Tổng Quan

**Mục tiêu:** Loại bỏ trùng lặp trong tài liệu, giữ frontend và server docs tách biệt
**Phương pháp:** Xác định "Source of Truth" cho mỗi concept, loại bỏ duplicates, thêm cross-references

---

## 🎯 PHASE 1: Server Docs - Context Reference ($ctx)

### Mục tiêu
Loại bỏ duplicate `$ctx` explanations từ các file server khác, chỉ giữ trong `context-reference.md`

### Công việc chi tiết

#### 1.1. Sửa `server/api-lifecycle.md`

**Hiện tại:**
- Có section "Context Sharing ($ctx)" với full properties list
- Liệt kê tất cả `$ctx` properties như `$body`, `$params`, `$query`, `$repos`, etc.
- Có examples về context sharing

**Sẽ sửa thành:**
- Giữ section "Context Sharing ($ctx)" nhưng chỉ giải thích **concept** của context sharing
- **Loại bỏ** full properties list
- **Loại bỏ** detailed examples về từng property
- **Thêm link:** `[→ Complete $ctx Reference](./context-reference.md)` ngay sau section title
- Giữ examples về **context sharing giữa hooks** (vì đây là lifecycle concept, không phải context reference)

**Ví dụ sửa:**
```markdown
## Context Sharing ($ctx)

The `$ctx` object is the **same reference** throughout the entire request lifecycle. This means changes in preHooks affect handlers and afterHooks.

**📖 For complete context reference, see [Context Reference](./context-reference.md)**

### Persistent Reference
[Giữ examples về context sharing giữa hooks]

### Available Context Properties
[LOẠI BỎ - thay bằng link]
```

**Checklist:**
- [ ] Mở `server/api-lifecycle.md`
- [ ] Tìm section "Context Sharing ($ctx)"
- [ ] Xóa subsection "Available Context Properties" (hoặc toàn bộ properties list)
- [ ] Thêm link `[→ Complete $ctx Reference](./context-reference.md)` ngay sau title
- [ ] Giữ lại examples về context sharing giữa hooks (vì đây là lifecycle concept)
- [ ] Kiểm tra không còn duplicate properties list

---

#### 1.2. Sửa `server/hook-development.md`

**Hiện tại:**
- Có section "Hook Context ($ctx)" với properties list
- Có examples về `$ctx` usage
- Có best practices về `$ctx`

**Sẽ sửa thành:**
- Giữ section "Hook Context ($ctx)" nhưng chỉ có **brief overview** (2-3 sentences)
- **Loại bỏ** properties list
- **Loại bỏ** detailed examples về từng property
- **Thêm link:** `[→ Complete $ctx Reference](./context-reference.md)` ngay sau brief overview
- **Giữ** examples về hook-specific usage (nhưng không giải thích properties chi tiết)

**Ví dụ sửa:**
```markdown
## Hook Context ($ctx)

Hooks use the same context object as handlers, providing full access to request data and system functions.

**📖 For complete context reference, see [Context Reference](./context-reference.md)**

## PreHook Examples
[Giữ examples nhưng không giải thích properties chi tiết]
```

**Checklist:**
- [ ] Mở `server/hook-development.md`
- [ ] Tìm section "Hook Context ($ctx)"
- [ ] Rút gọn thành brief overview (2-3 sentences)
- [ ] Xóa properties list nếu có
- [ ] Thêm link `[→ Complete $ctx Reference](./context-reference.md)`
- [ ] Giữ examples nhưng đảm bảo không giải thích properties chi tiết
- [ ] Kiểm tra không còn duplicate properties list

---

#### 1.3. Sửa `server/template-syntax.md`

**Hiện tại:**
- Có bảng template mappings (đúng, cần giữ)
- Có thể có giải thích về properties (cần kiểm tra)

**Sẽ sửa thành:**
- **Giữ** bảng template mappings (đây là source of truth cho template syntax)
- **Loại bỏ** nếu có giải thích chi tiết về `$ctx` properties (chỉ giữ template syntax)
- **Thêm link:** `[→ Complete $ctx Reference](./context-reference.md)` ở section "Overview"

**Checklist:**
- [ ] Mở `server/template-syntax.md`
- [ ] Kiểm tra có giải thích chi tiết về `$ctx` properties không
- [ ] Nếu có, xóa và thay bằng link đến context-reference.md
- [ ] Giữ bảng template mappings
- [ ] Thêm link `[→ Complete $ctx Reference](./context-reference.md)` ở đầu file hoặc Overview section

---

#### 1.4. Kiểm tra `server/context-reference.md`

**Mục tiêu:** Đảm bảo đây là source of truth hoàn chỉnh

**Checklist:**
- [ ] Mở `server/context-reference.md`
- [ ] Kiểm tra có đầy đủ tất cả `$ctx` properties không
- [ ] Kiểm tra có examples đầy đủ không
- [ ] Kiểm tra có best practices không
- [ ] Đảm bảo đây là file duy nhất có complete reference

---

## 🎯 PHASE 2: Server Docs - API Querying (Filter Operators)

### Mục tiêu
Consolidate filter operators vào `api-querying.md`, loại bỏ duplicates từ các file khác

### Công việc chi tiết

#### 2.1. Sửa `server/hook-development.md`

**Hiện tại:**
- Có examples về filtering trong hooks
- Có thể có giải thích về filter operators

**Sẽ sửa thành:**
- **Giữ** examples về filtering trong hooks (vì đây là hook-specific usage)
- **Loại bỏ** giải thích chi tiết về filter operators (`_eq`, `_gt`, `_contains`, etc.)
- **Thêm link:** `[→ Complete Filter Operators Guide](./api-querying.md#filter-operators)` trong examples
- Chỉ giữ brief mention trong examples

**Ví dụ sửa:**
```markdown
### Complex Validation with Database Access
```javascript
// Example using filter operators
// See [Filter Operators Guide](./api-querying.md#filter-operators) for complete syntax
const existingUser = await $ctx.$repos.user_definition.find({
  where: { email: { _eq: $ctx.$body.email } }
});
```
```

**Checklist:**
- [ ] Mở `server/hook-development.md`
- [ ] Tìm tất cả sections có filter operators
- [ ] Xóa giải thích chi tiết về operators (nếu có)
- [ ] Thêm link đến `api-querying.md` trong examples
- [ ] Giữ examples nhưng chỉ với brief comments

---

#### 2.2. Kiểm tra `server/api-querying.md`

**Mục tiêu:** Đảm bảo đây là source of truth hoàn chỉnh cho filter operators

**Checklist:**
- [ ] Mở `server/api-querying.md`
- [ ] Kiểm tra section "Filter Operators" có đầy đủ không
- [ ] Kiểm tra có tất cả operators: `_eq`, `_neq`, `_gt`, `_gte`, `_lt`, `_lte`, `_between`, `_in`, `_not_in`, `_contains`, `_starts_with`, `_ends_with`, `_is_null`
- [ ] Kiểm tra có logical operators: `_and`, `_or`, `_not`
- [ ] Kiểm tra có relation filtering examples
- [ ] Kiểm tra có aggregation filtering
- [ ] Đảm bảo đây là file duy nhất có complete filter operators guide

---

#### 2.3. Sửa `server/custom-handlers.md` (nếu có file này)

**Hiện tại:**
- Có examples về filtering trong handlers

**Sẽ sửa thành:**
- **Giữ** examples nhưng thêm link đến `api-querying.md`
- **Loại bỏ** giải thích chi tiết về operators

**Checklist:**
- [ ] Kiểm tra có file `server/custom-handlers.md` không
- [ ] Nếu có, tìm sections về filtering
- [ ] Thêm link đến `api-querying.md`
- [ ] Xóa giải thích chi tiết về operators

---

## 🎯 PHASE 3: Server Docs - Template Syntax

### Mục tiêu
Đảm bảo `template-syntax.md` là source of truth, loại bỏ duplicates

### Công việc chi tiết

#### 3.1. Sửa `server/context-reference.md`

**Hiện tại:**
- Có section về template syntax ở cuối file

**Sẽ sửa thành:**
- **Giữ** brief mention về template syntax (1-2 sentences)
- **Loại bỏ** bảng mappings hoặc detailed examples
- **Thêm link:** `[→ Complete Template Syntax Guide](./template-syntax.md)`

**Ví dụ sửa:**
```markdown
## Template Syntax (Optional)

You can use either the full `$ctx.$property` syntax or shorter template syntax - **both work exactly the same way**.

**📖 For complete template syntax guide, see [Template Syntax Guide](./template-syntax.md)**

[Giữ 1-2 examples ngắn, không cần bảng mappings]
```

**Checklist:**
- [ ] Mở `server/context-reference.md`
- [ ] Tìm section "Template Syntax"
- [ ] Rút gọn thành brief mention (1-2 sentences)
- [ ] Xóa bảng mappings nếu có
- [ ] Thêm link `[→ Complete Template Syntax Guide](./template-syntax.md)`
- [ ] Giữ 1-2 examples ngắn nếu cần

---

#### 3.2. Kiểm tra `server/template-syntax.md`

**Mục tiêu:** Đảm bảo đây là source of truth hoàn chỉnh

**Checklist:**
- [ ] Mở `server/template-syntax.md`
- [ ] Kiểm tra có bảng template mappings đầy đủ không
- [ ] Kiểm tra có examples cho tất cả templates không
- [ ] Kiểm tra có best practices không
- [ ] Đảm bảo đây là file duy nhất có complete template syntax guide

---

## 🎯 PHASE 4: Getting Started - README Simplification

### Mục tiêu
Simplify README, loại bỏ duplicates với installation và architecture

### Công việc chi tiết

#### 4.1. Sửa `README.md` - Installation Section

**Hiện tại:**
- Có section "Installation" với quick setup steps
- Có configuration prompts details

**Sẽ sửa thành:**
- **Rút gọn** thành 1-2 paragraphs với quick overview
- **Loại bỏ** chi tiết về configuration prompts
- **Loại bỏ** detailed steps
- **Thêm link:** `[→ Complete Installation Guide](./getting-started/installation.md)`

**Ví dụ sửa:**
```markdown
## Installation

Enfyra requires both backend and frontend to work properly.

**Quick Setup:**
1. Install backend: `npx @enfyra/create-server@latest <project-name>`
2. Install frontend: `npx @enfyra/create-app@latest <project-name>`
3. Connect them together

**[→ Complete Installation Guide](./getting-started/installation.md)** - Detailed setup instructions, configuration prompts, and troubleshooting.
```

**Checklist:**
- [ ] Mở `README.md`
- [ ] Tìm section "Installation"
- [ ] Rút gọn thành 1-2 paragraphs
- [ ] Xóa configuration prompts details
- [ ] Xóa detailed steps
- [ ] Thêm link `[→ Complete Installation Guide](./getting-started/installation.md)`
- [ ] Giữ quick overview (3-4 bullet points)

---

#### 4.2. Sửa `README.md` - Architecture Overview Section

**Hiện tại:**
- Có section "Architecture Overview" với diagram và explanations

**Sẽ sửa thành:**
- **Rút gọn** thành 1 paragraph overview
- **Loại bỏ** detailed diagram và explanations
- **Thêm link:** `[→ Complete Architecture Overview](./architecture-overview.md)`

**Ví dụ sửa:**
```markdown
### 🏗️ Architecture Overview

**Two-Component System:**
- **Backend (Port 1105)**: Generates & serves all REST & GraphQL APIs from your database schema
- **Frontend (Port 3000)**: Pure client application consuming APIs from backend URL

**[→ Complete Architecture Overview](./architecture-overview.md)** - Detailed diagrams, component responsibilities, and data flow.
```

**Checklist:**
- [ ] Mở `README.md`
- [ ] Tìm section "Architecture Overview" hoặc "🏗️ Architecture Overview"
- [ ] Rút gọn thành 1 paragraph
- [ ] Xóa detailed diagram
- [ ] Xóa detailed explanations
- [ ] Thêm link `[→ Complete Architecture Overview](./architecture-overview.md)`
- [ ] Giữ brief overview (2-3 sentences)

---

#### 4.3. Sửa `README.md` - Backend-Frontend Separation (nếu lặp lại nhiều lần)

**Hiện tại:**
- Có thể có nhiều mentions về "Backend riêng, Frontend riêng" trong README

**Sẽ sửa thành:**
- **Giữ** 1-2 mentions chính (ở Architecture section)
- **Loại bỏ** duplicate mentions ở các section khác
- **Thêm link** đến architecture-overview.md nếu cần chi tiết

**Checklist:**
- [ ] Mở `README.md`
- [ ] Tìm tất cả mentions về "Backend riêng, Frontend riêng"
- [ ] Giữ 1-2 mentions chính (ở Architecture section)
- [ ] Xóa duplicate mentions ở các section khác
- [ ] Thêm link đến architecture-overview.md nếu cần

---

## 🎯 PHASE 5: Getting Started - Table Creation

### Mục tiêu
Loại bỏ duplicate giữa `getting-started.md` và `table-creation.md`

### Công việc chi tiết

#### 5.1. Sửa `getting-started/getting-started.md`

**Hiện tại:**
- Có section "Next Steps: Create Your First Table"
- Có thể có details về table creation

**Sẽ sửa thành:**
- **Rút gọn** thành brief mention (1-2 sentences)
- **Loại bỏ** details về table creation
- **Thêm link:** `[→ Complete Table Creation Guide](./table-creation.md)`

**Ví dụ sửa:**
```markdown
## Next Steps: Create Your First Table

Now that you're familiar with the interface, it's time to create your first table and start building your application.

**→ [Table Creation Guide](./table-creation.md)** - Complete step-by-step guide to creating tables with all field types, relations, and constraints.
```

**Checklist:**
- [ ] Mở `getting-started/getting-started.md`
- [ ] Tìm section về table creation
- [ ] Rút gọn thành brief mention
- [ ] Xóa details nếu có
- [ ] Thêm link `[→ Complete Table Creation Guide](./table-creation.md)`
- [ ] Giữ brief overview (1-2 sentences)

---

## 🎯 PHASE 6: Frontend Docs - Filter System

### Mục tiêu
Consolidate filter UI vào `filter-system.md`, loại bỏ duplicates

### Công việc chi tiết

#### 6.1. Sửa `frontend/relation-picker.md`

**Hiện tại:**
- Có mention về filter system khi sử dụng relation picker

**Sẽ sửa thành:**
- **Giữ** brief mention (1 sentence)
- **Thêm link:** `[→ Complete Filter System Guide](./filter-system.md)`
- **Loại bỏ** detailed explanation về filter UI

**Checklist:**
- [ ] Mở `frontend/relation-picker.md`
- [ ] Tìm mentions về filter system
- [ ] Rút gọn thành brief mention
- [ ] Thêm link `[→ Complete Filter System Guide](./filter-system.md)`
- [ ] Xóa detailed explanation nếu có

---

#### 6.2. Sửa `frontend/data-management.md`

**Hiện tại:**
- Có mention về Filter button

**Sẽ sửa thành:**
- **Giữ** brief mention về Filter button (1-2 sentences)
- **Thêm link:** `[→ Complete Filter System Guide](./filter-system.md)`
- **Loại bỏ** detailed explanation về filter UI

**Checklist:**
- [ ] Mở `frontend/data-management.md`
- [ ] Tìm section về Filter button
- [ ] Rút gọn thành brief mention (1-2 sentences)
- [ ] Thêm link `[→ Complete Filter System Guide](./filter-system.md)`
- [ ] Xóa detailed explanation nếu có

---

## 🎯 PHASE 7: Frontend Docs - Form System

### Mục tiêu
Consolidate form UI vào `form-system.md`, loại bỏ duplicates

### Công việc chi tiết

#### 7.1. Sửa `frontend/relation-picker.md`

**Hiện tại:**
- Có mention về forms khi sử dụng relation picker

**Sẽ sửa thành:**
- **Giữ** brief mention (1 sentence)
- **Thêm link:** `[→ Complete Form System Guide](./form-system.md)`
- **Loại bỏ** detailed explanation về form UI

**Checklist:**
- [ ] Mở `frontend/relation-picker.md`
- [ ] Tìm mentions về forms
- [ ] Rút gọn thành brief mention
- [ ] Thêm link `[→ Complete Form System Guide](./form-system.md)`
- [ ] Xóa detailed explanation nếu có

---

#### 7.2. Sửa `frontend/data-management.md`

**Hiện tại:**
- Có mention về create/edit forms

**Sẽ sửa thành:**
- **Giữ** brief mention (1-2 sentences)
- **Thêm link:** `[→ Complete Form System Guide](./form-system.md)`
- **Loại bỏ** detailed explanation về form UI

**Checklist:**
- [ ] Mở `frontend/data-management.md`
- [ ] Tìm sections về create/edit forms
- [ ] Rút gọn thành brief mention
- [ ] Thêm link `[→ Complete Form System Guide](./form-system.md)`
- [ ] Xóa detailed explanation nếu có

---

## 🎯 PHASE 8: Examples - User Registration

### Mục tiêu
Đảm bảo `examples/user-registration-example.md` là complete example, các file khác chỉ link đến đây

### Công việc chi tiết

#### 8.1. Sửa `server/hook-development.md`

**Hiện tại:**
- Có examples về hooks (có thể có user registration pattern)

**Sẽ sửa thành:**
- **Giữ** brief examples về hooks
- **Thêm link:** `[→ Complete User Registration Example](../examples/user-registration-example.md)` nếu có user registration pattern
- **Loại bỏ** duplicate user registration code nếu có

**Checklist:**
- [ ] Mở `server/hook-development.md`
- [ ] Tìm examples về user registration (nếu có)
- [ ] Nếu có duplicate với examples/user-registration-example.md, xóa và thay bằng link
- [ ] Thêm link đến complete example

---

#### 8.2. Sửa `frontend/custom-handlers.md`

**Hiện tại:**
- Có examples về handlers (có thể có user registration pattern)

**Sẽ sửa thành:**
- **Giữ** brief examples về handlers
- **Thêm link:** `[→ Complete User Registration Example](../examples/user-registration-example.md)` nếu có user registration pattern
- **Loại bỏ** duplicate user registration code nếu có

**Checklist:**
- [ ] Mở `frontend/custom-handlers.md`
- [ ] Tìm examples về user registration (nếu có)
- [ ] Nếu có duplicate với examples/user-registration-example.md, xóa và thay bằng link
- [ ] Thêm link đến complete example

---

## 🎯 PHASE 9: Cross-References & Related Documentation

### Mục tiêu
Thêm "Related Documentation" sections và cross-references rõ ràng

### Công việc chi tiết

#### 9.1. Thêm "Related Documentation" section vào tất cả files

**Template:**
```markdown
## Related Documentation

- **[Context Reference](./context-reference.md)** - Complete $ctx object reference
- **[API Querying](./api-querying.md)** - Filter operators and query syntax
- **[Template Syntax](./template-syntax.md)** - Template syntax guide
```

**Checklist cho từng file:**

**Server Docs:**
- [ ] `server/context-reference.md` - Thêm Related Documentation section
- [ ] `server/api-querying.md` - Thêm Related Documentation section
- [ ] `server/template-syntax.md` - Thêm Related Documentation section
- [ ] `server/api-lifecycle.md` - Thêm Related Documentation section
- [ ] `server/hook-development.md` - Thêm Related Documentation section
- [ ] `server/permission-system.md` - Thêm Related Documentation section
- [ ] `server/graphql-api.md` - Thêm Related Documentation section
- [ ] `server/swagger-api.md` - Thêm Related Documentation section

**Frontend Docs:**
- [ ] `frontend/filter-system.md` - Thêm Related Documentation section
- [ ] `frontend/form-system.md` - Thêm Related Documentation section
- [ ] `frontend/permission-builder.md` - Thêm Related Documentation section
- [ ] `frontend/custom-handlers.md` - Thêm Related Documentation section
- [ ] `frontend/hooks.md` - Thêm Related Documentation section

**Getting Started:**
- [ ] `getting-started/installation.md` - Thêm Related Documentation section
- [ ] `getting-started/getting-started.md` - Thêm Related Documentation section
- [ ] `getting-started/table-creation.md` - Thêm Related Documentation section
- [ ] `architecture-overview.md` - Thêm Related Documentation section

---

#### 9.2. Standardize link format

**Format chuẩn:**
- `[→ Complete Guide](./path)` - Cho detailed guides
- `[→ UI Guide](./path)` - Cho frontend workflows
- `[→ Technical Details](./path)` - Cho server docs
- `[→ Examples](./path)` - Cho examples

**Checklist:**
- [ ] Review tất cả links trong các file đã sửa
- [ ] Đảm bảo format nhất quán
- [ ] Sửa các links không đúng format

---

## 🎯 PHASE 10: Final Review & Verification

### Mục tiêu
Kiểm tra lại toàn bộ, đảm bảo không còn duplicates

### Công việc chi tiết

#### 10.1. Review Server Docs

**Checklist:**
- [ ] `server/context-reference.md` - Đảm bảo là source of truth duy nhất cho $ctx
- [ ] `server/api-querying.md` - Đảm bảo là source of truth duy nhất cho filter operators
- [ ] `server/template-syntax.md` - Đảm bảo là source of truth duy nhất cho template syntax
- [ ] `server/api-lifecycle.md` - Đảm bảo không còn duplicate $ctx properties
- [ ] `server/hook-development.md` - Đảm bảo không còn duplicate $ctx properties hoặc filter operators

---

#### 10.2. Review Frontend Docs

**Checklist:**
- [ ] `frontend/filter-system.md` - Đảm bảo là source of truth cho filter UI
- [ ] `frontend/form-system.md` - Đảm bảo là source of truth cho form UI
- [ ] `frontend/relation-picker.md` - Đảm bảo chỉ có brief mentions và links
- [ ] `frontend/data-management.md` - Đảm bảo chỉ có brief mentions và links

---

#### 10.3. Review Getting Started

**Checklist:**
- [ ] `README.md` - Đảm bảo không còn duplicates với installation và architecture
- [ ] `getting-started/installation.md` - Đảm bảo có đầy đủ chi tiết
- [ ] `architecture-overview.md` - Đảm bảo có đầy đủ chi tiết
- [ ] `getting-started/getting-started.md` - Đảm bảo không còn duplicates với table-creation

---

#### 10.4. Verify Links

**Checklist:**
- [ ] Tất cả links đều hoạt động (không broken links)
- [ ] Tất cả links đều đúng format
- [ ] Cross-references rõ ràng giữa related files

---

#### 10.5. Verify No Duplicates

**Checklist:**
- [ ] Không còn duplicate $ctx properties list (chỉ có trong context-reference.md)
- [ ] Không còn duplicate filter operators explanation (chỉ có trong api-querying.md)
- [ ] Không còn duplicate template syntax mappings (chỉ có trong template-syntax.md)
- [ ] Không còn duplicate installation details (chỉ có trong installation.md)
- [ ] Không còn duplicate architecture details (chỉ có trong architecture-overview.md)

---

## 📊 Tổng Kết Checklist

### Phase 1: Server Docs - Context Reference
- [ ] Sửa `server/api-lifecycle.md`
- [ ] Sửa `server/hook-development.md`
- [ ] Sửa `server/template-syntax.md`
- [ ] Kiểm tra `server/context-reference.md`

### Phase 2: Server Docs - API Querying
- [ ] Sửa `server/hook-development.md`
- [ ] Kiểm tra `server/api-querying.md`
- [ ] Sửa `server/custom-handlers.md` (nếu có)

### Phase 3: Server Docs - Template Syntax
- [ ] Sửa `server/context-reference.md`
- [ ] Kiểm tra `server/template-syntax.md`

### Phase 4: Getting Started - README
- [ ] Sửa `README.md` - Installation section
- [ ] Sửa `README.md` - Architecture section
- [ ] Sửa `README.md` - Backend-Frontend separation

### Phase 5: Getting Started - Table Creation
- [ ] Sửa `getting-started/getting-started.md`

### Phase 6: Frontend Docs - Filter System
- [ ] Sửa `frontend/relation-picker.md`
- [ ] Sửa `frontend/data-management.md`

### Phase 7: Frontend Docs - Form System
- [ ] Sửa `frontend/relation-picker.md`
- [ ] Sửa `frontend/data-management.md`

### Phase 8: Examples - User Registration
- [ ] Sửa `server/hook-development.md`
- [ ] Sửa `frontend/custom-handlers.md`

### Phase 9: Cross-References
- [ ] Thêm Related Documentation sections
- [ ] Standardize link format

### Phase 10: Final Review
- [ ] Review Server Docs
- [ ] Review Frontend Docs
- [ ] Review Getting Started
- [ ] Verify Links
- [ ] Verify No Duplicates

---

## 🎯 Thứ Tự Thực Hiện

**Recommended order:**
1. Phase 1 (Context Reference) - Quan trọng nhất
2. Phase 2 (API Querying) - Quan trọng
3. Phase 3 (Template Syntax) - Quan trọng
4. Phase 4 (README) - Dễ thấy nhất
5. Phase 5-8 (Các phase khác) - Theo thứ tự
6. Phase 9 (Cross-References) - Cuối cùng
7. Phase 10 (Final Review) - Verification

---

**Lưu ý:** 
- Mỗi phase có thể làm độc lập
- Có thể làm nhiều phase cùng lúc nếu không conflict
- Nên commit sau mỗi phase để dễ rollback nếu cần

