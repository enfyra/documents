---
slug: lich-su-code
---

# Audit code và phiên bản

Lịch sử code xác định record bằng đúng tên bảng và ID, ghi nhận người thao tác và giữ source trước thay đổi để xem lại hoặc phục hồi. Source hiện tại nằm trên chính record.

## Record có version

Chín bảng sau bắt buộc gửi `version` mới mỗi lần update, kể cả khi chỉ đổi metadata như bật hoặc tắt record:

| Bảng | Field source |
|---|---|
| `enfyra_route_handler` | `sourceCode` |
| `enfyra_pre_hook` | `sourceCode` |
| `enfyra_post_hook` | `sourceCode` |
| `enfyra_helper` | `sourceCode` |
| `enfyra_websocket` | `sourceCode` |
| `enfyra_websocket_event` | `sourceCode` |
| `enfyra_oauth_config` | `sourceCode` |
| `enfyra_flow_step` | `sourceCode` |
| `enfyra_extension` | `code` |

Menu và các bảng khác không thuộc tính năng này. Source của flow step nằm trong `sourceCode`, không nằm trong `config`.

## Chọn version

Server chỉ kiểm tra version, không tự sinh version khi update.

- **Create:** bỏ `version` để dùng default `1.0.0` của cột, hoặc gửi version khởi đầu hợp lệ.
- **Update:** gửi version hợp lệ, lớn hơn hẳn giá trị đang lưu. Đổi metadata không liên quan source vẫn phải tuân thủ yêu cầu này.
- **Update hàng loạt:** nếu bảng hỗ trợ thao tác bulk đó, version dùng chung phải lớn hơn version hiện tại của mọi record đích.

Dùng ba segment số, ví dụ `1.0.0` hoặc `3.0.2`. Các segment được so sánh theo số: `1.0.10` mới hơn `1.0.9`. Bộ so sánh của Enfyra cũng hỗ trợ thêm segment số và hậu tố có dấu gạch ngang. Giá trị có hậu tố đứng sau giá trị gốc, nên `1.0.0-patch-1` mới hơn `1.0.0`; đây không phải thứ tự prerelease của SemVer. Tiền tố `v` được bỏ khi chuẩn hóa. Không hỗ trợ build metadata có dấu `+`.

Enfyra Admin đề xuất patch kế tiếp trong modal cho phép sửa. MCP đọc version hiện tại và tăng patch nếu update không gửi `version`. Nếu tự điền version qua MCP, giá trị phải hợp lệ và lớn hơn; MCP từ chối giá trị sai thay vì âm thầm thay thế. Khi gọi REST hoặc repository trực tiếp, caller tự gửi version.

```http
PATCH /api/enfyra_route_handler/12
Content-Type: application/json

{
  "version": "1.0.1",
  "sourceCode": "return $ctx.$body.id;"
}
```

Update thiếu version, sai định dạng, bằng hoặc thấp hơn version hiện tại trả `400`. Nếu người khác đã cập nhật record, đọc lại source và version trước khi thử tiếp. Không tự nâng version rồi gửi lại source cũ một cách mù quáng.

## Đọc lịch sử

`GET /api/enfyra_audit` là endpoint đọc chuẩn. Thao tác ứng dụng qua repository không thể tạo hoặc sửa dòng audit; runtime ghi thêm bằng đường nội bộ. `DELETE /api/enfyra_audit` là ngoại lệ duy nhất được phép, để quản trị viên dọn bớt revision qua bulk delete ở trang Audit, và route chỉ mở `GET` cùng `DELETE`. Quản trị viên database nằm ngoài cơ chế bảo vệ ở tầng ứng dụng này.

| Field | Ý nghĩa |
|---|---|
| `tableName`, `recordId` | Đúng bảng và ID của record đích; ID được lưu dạng text |
| `action` | `create`, `update`, `delete`, hoặc `revert` |
| `fromVersion`, `toVersion` | Version trước và sau mutation |
| `restoredFrom` | Version cũ được chọn để phục hồi |
| `previous` | Giá trị cột đã khai báo và được publish trước mutation |
| `changedBy` | Relation đến caller trong `enfyra_user`, nếu có |
| `changedByEmail` | Email caller tại thời điểm ghi |
| `changedBySource` | `pat` hoặc `jwt` với HTTP caller đã xác thực |
| `createdAt` | Thời điểm ghi audit |

`previous` không chứa định danh record, timestamp, `compiledCode` được sinh tự động, cột không publish hoặc relation mở rộng. Vì vậy OAuth `clientId` và `clientSecret` không được chép vào history. Revert không khôi phục những giá trị bị loại này hoặc cách gán relation. Dòng create không có trạng thái trước đó; dòng delete không có version tiếp theo.

Đọc một trang có giới hạn, dùng query parameter `filter` dạng JSON:

```json
{
  "tableName": { "_eq": "enfyra_route_handler" },
  "recordId": { "_eq": "12" }
}
```

Với danh sách, chọn các field như `id,tableName,recordId,action,fromVersion,toVersion,changedByEmail,createdAt`, `sort=-createdAt,-id`, `limit=20` và `page=1`. Chỉ lấy `previous` khi xem một dòng cụ thể vì source có thể lớn. Source lịch sử là dữ liệu đặc quyền: chỉ cấp quyền đọc history cho người được tin cậy để viết code.

## Phục hồi phiên bản cũ

```http
POST /api/admin/code-audit/revert
Content-Type: application/json

{
  "auditId": 41,
  "version": "1.0.4"
}
```

Caller cần quyền gọi endpoint revert, quyền đọc dòng audit và quyền PATCH trên route của bảng đích. Phục hồi dùng secure repository, giữ nguyên quyền ghi từng field cùng validation, biên dịch source, reload và hành vi mutation thông thường.

Server ghi các giá trị cột trước thay đổi vào record còn tồn tại, với version **mới** được gửi lên. Một dòng `revert` mới có `restoredFrom` xác định phiên bản cũ được phục hồi. Lịch sử đã có không bị đặt lại hay xóa đi.

Dòng create không có phiên bản trước để phục hồi. Dòng delete không thể revert, và record đã mất không được tạo lại. Tính năng này không hoàn tác quyết định xóa.

Xem [Audit trong Enfyra Admin](../app/code-history.md) để thao tác trên UI, hoặc [workflow với trợ lý AI](../integrations/mcp-server.md) để dùng MCP.

## Giới hạn độ tin cậy

Audit được ghi sau khi ghi record. Nếu ghi audit thất bại, request trả lỗi; ngoài transaction database bao ngoài, record có thể đã thay đổi. Cần đọc lại trạng thái trước khi thử lại. Đây chưa phải bảo đảm nguyên tử giữa code và history trên mọi database; tiến trình dừng giữa hai lần ghi có thể để lại khoảng trống.

Kiểm tra version không phải khóa compare-and-swap. Cập nhật đồng thời, mutation relation lồng nhau, xóa cascade và các đường reorder quản trị cần được kiểm chứng transaction riêng; không xem history này là audit toàn bộ database hay giải pháp thay thế backup. Version được phép nhảy, nên thiếu một số version không chứng minh history đã mất.
