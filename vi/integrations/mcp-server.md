---
slug: may-chu-mcp
---

# MCP Server

Dùng `@enfyra/mcp-server` để kết nối coding tool hỗ trợ MCP với Enfyra instance. Server cung cấp tool chuyên biệt cho schema change, route, handler, hook, permission, flow, websocket event, file, package, menu, extension, log và runtime check.

## Cài đặt và cấu hình

Trong project mà coding agent sẽ làm việc:

```bash
npx @enfyra/mcp-server config
```

Config helper ghi MCP config cục bộ cho Codex, Claude Code, Cursor, VS Code / GitHub Copilot và Google Antigravity. Nó hỏi Enfyra app/admin URL cùng programmatic API token lấy từ `/me` trong Enfyra admin UI.

Thiết lập không tương tác:

```bash
npx @enfyra/mcp-server config --yes \
  --app-url https://demo.enfyra.io \
  --api-token efy_pat_your-token
```

Helper tự suy ra runtime API base từ app URL. App và demo thông thường phải trỏ đến app/admin URL, không phải hidden backend host.

## Xác thực

`ENFYRA_API_TOKEN` không được dùng trực tiếp như Bearer JWT. MCP server đổi token qua:

```text
POST {ENFYRA_API_URL}/auth/token/exchange
```

Access token ngắn hạn sau exchange được dùng cho MCP tool call đã xác thực. Khi token hết hạn hoặc bị từ chối, MCP server đổi API token lại.

Non-root API token có thể dùng admin helper tool nếu admin route tương ứng có ordinary route permission. Dùng `get_permission_profile`, `audit_route_access`, `ensure_route_access` để kiểm tra hoặc cấp quyền.

## Required knowledge handshake

Trước khi agent lưu dynamic server code hoặc Enfyra extension code, phải gọi:

```text
get_enfyra_required_knowledge
```

```text
get_extension_theme_contract
```

Tool trả required contract và acknowledgement key. Code-writing tool kiểm tra key trước khi lưu:

- Truyền `dynamicCodeAckKey` làm `knowledgeAckKey` khi lưu handler, hook, websocket event script, script/condition flow step và script-backed record.
- Truyền `extensionAckKey` làm `extensionKnowledgeAckKey` khi lưu page, widget hoặc global extension code.

Discovery, validation và preview không cần acknowledgement; agent vẫn có thể inspect instance, đọc contract, validate source và preview patch trước khi lưu.

## Dynamic repository path

- `@REPOS.main`: secure repository cho main table của route hiện tại.
- `@REPOS.secure.<table>`: secure explicit-table repository cho public/user-facing custom handler, hook, websocket script, flow trả data và third-party integration.
- `@REPOS.<table>`: trusted internal repository cho server-owned maintenance hoặc admin logic cần hidden field.

Không trả raw trusted-repository record cho user. Khi cần trusted access, hãy project hoặc sanitize response trước khi trả. Lựa chọn repository không thay thế authorization: handler/hook vẫn cần route access, owner/tenant filter, membership check và mutation check tường minh.

## Hidden field query surface

Unpublished field và private relation nhạy cảm kể cả khi giá trị không được chọn. API hướng tới user không nên lộ filter dùng làm predicate oracle trên hidden field, aggregate (`sum`, `avg`, `max`, `min`) trên hidden field hoặc sort helper `_max(relation.field)`, `_min(relation.field)`, `_count(relation)` trên private relation trừ khi endpoint chủ ý lộ thông tin đó.

Nếu REST read bình thường trả `isPublished=false` qua `fields`, dotted relation field hoặc `deep`, coi đó là Enfyra core bug và xác nhận minimal REST repro.

## Quy tắc extension

Trước khi viết/review page, widget hoặc global extension UI, gọi `get_extension_theme_contract`; khi cần exact `eapp-*` class hoặc Nuxt UI color mapping, gọi `get_theme_class_reference`.

Extension nên theo app shell: dùng `eapp-surface-*`, `eapp-text-*`, `eapp-divide-y`, `eapp-divider` cho surface trung tính; dùng `eapp-primary-*` hoặc Nuxt UI `primary` cho runtime-primary identity/accent; semantic color chỉ cho trạng thái thật. Dùng `useMenuNotificationRegistry` cho sidebar count/dot và `useAccountPanelRegistry` `count`/`badgeColor` cho account notification. Không hard-code palette như `color="violet"`, không dùng raw CSS variable utility khi app token class đã có, không thêm root-level page padding và lưu extension như Vue SFC trong `enfyra_extension.code`, không dùng static import.

## Luồng tool khuyến nghị

Với custom API:

1. Gọi `inspect_route`, `inspect_table` hoặc `discover_script_contexts`.
2. Gọi `get_enfyra_required_knowledge`.
3. Validate source bằng `validate_dynamic_script`.
4. Dùng `api_endpoint_workflow` hoặc route/handler/hook tool chuyên biệt.
5. Test bằng `test_rest_endpoint`, `run_admin_test` hoặc `test_flow_step`.

Với extension:

1. Gọi `get_extension_theme_contract`.
2. Khi cần, gọi `get_theme_class_reference`.
3. Gọi `get_enfyra_required_knowledge`.
4. Validate bằng `validate_extension_code`.
5. Lưu với `ensure_page_extension`, `ensure_widget_extension` hoặc `ensure_global_extension`.

Ưu tiên operation tool chuyên biệt trước khi fallback về generic record CRUD.
