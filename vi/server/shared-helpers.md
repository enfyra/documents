---
slug: may-chu/helper-dung-chung
---

# Tái sử dụng thao tác server bằng Shared Helpers

Shared Helpers cho phép handler, hook, flow script và các dynamic script khác gọi một thao tác đã lưu theo tên, không cần dispatch route khác hay tạo queue job. Dùng helper cho thao tác tái sử dụng có đầu vào rõ ràng và kết quả xác định; giữ điều phối route và response riêng của request ở caller.

## Điều kiện sử dụng

Server phải có `enfyra_helper` và shared helper runner; App phải có **Settings → Helpers**. Kiểm tra release đã deploy hỗ trợ cả hai trước khi lưu script gọi helper. Database đã khởi tạo cần một Server release mới hơn có bootstrap cấp system table; boot cùng phiên bản không tạo bảng này. Không reset trạng thái khởi tạo hoặc hạ phiên bản đã ghi để bỏ qua kiểm tra.

Viết helper là quyền viết server code có đặc quyền. Route CRUD `/enfyra_helper` là private, không cấp mặc định cho anonymous hay non-root. Chỉ cấp cho người viết script đáng tin cậy.

## Tạo và kiểm tra helper

1. Mở **Settings → Helpers** (`/settings/helpers`) và tạo draft chưa bật.
2. Đặt `name` lowercase duy nhất, mô tả đầu vào/kết quả/tác động phụ và chọn JavaScript hoặc TypeScript ở `scriptLanguage`.
3. Viết thân script trong `sourceCode`. Đọc đầu vào tường minh qua `@PARAMS` rồi trả kết quả. Không có biến `input` được khai báo ngầm.
4. Dùng tab **Test** với params dạng JSON. Kiểm tra kết quả trước khi bật và lưu helper.

Tên có 1–128 chữ lowercase, chữ số hoặc dấu gạch dưới; được bắt đầu bằng chữ số. Các tên `__proto__`, `prototype`, `constructor`, `run`, `then`, `catch` và `finally` được dành riêng. Helper đã bật phải có source thực thi được. Server sinh `compiledCode`; không gửi hoặc sửa field này.

Ví dụ helper tên `normalize_query`:

```typescript
const query: string = String(@PARAMS?.query ?? '').trim();
if (!query) @THROW400('query is required');
return { query };
```

Editor có các tab **Editor**, **Usage** và **Test**. Test chạy source chưa lưu hiện tại với giới hạn năm giây, không lưu draft. Test có thể thực sự thay đổi repository/network/storage và không tự rollback. Lời gọi lồng nhau dùng helper đã lưu và được activate, không dùng draft khác chưa lưu. Save, Reset, Delete và điều hướng vẫn giữ guard cho trạng thái dirty/pending.

## Gọi theo tên

```typescript
const result = await @HELPERS.run('normalize_query', {
  query: @BODY.query,
});
return result;
```

API tương đương là `await $ctx.$helpers.run(name, params)`. Helper có thể gọi helper khác. Params có thể là object, array, primitive, null hoặc undefined; runner không deep-clone hay làm chúng bất biến. Luôn await lời gọi, đặc biệt trong transaction.

Mỗi lần gọi có biến cục bộ và data context mới. `@PARAMS` là đối số đã truyền; body, query, share và env là object rỗng, user là null. Không sao chép request/API/result/flow data hoặc biến cục bộ của caller. Hãy truyền tường minh thông tin cần dùng.

Built-in helper, repository, transaction, package, cache, storage, socket, log và throw giữ các capability hiện có. Helper dùng chung task, JavaScript realm/global, deadline và cancellation. Đây là ranh giới data context, **không phải sandbox riêng để chạy code không đáng tin cậy**. Không tạo child executor, HTTP request hay chạy lại vòng đời route.

## Giữ đúng phân quyền và atomicity

Helper giữ **quyền repository do host xác định cho caller**, dù user trong data context là null. Truyền user hoặc role qua params không cấp thêm quyền. Helper không chạy lại guard, validation, hook hoặc route-executed trigger của route khác. Đặt phân quyền và business validation bắt buộc ở ranh giới thao tác, dùng secure repository khi phù hợp.

```typescript
return await @TRANSACTION.run(async () => {
  await #orders.create({ data: { reference: @BODY.reference } });
  return await @HELPERS.run('reserve_inventory', {
    productId: @BODY.productId,
    quantity: @BODY.quantity,
  });
});
```

Các lời gọi repository đã await từ caller/helper tham gia transaction đang hoạt động và đọc được thay đổi chưa commit. Lỗi thoát khỏi callback ngoài cùng sẽ rollback các database mutation cùng nhau. Transaction lồng tham gia scope hiện tại; không tạo savepoint hay commit riêng. Nuốt lỗi không bảo đảm rollback.

Database rollback không hoàn tác network, cache, socket, storage hoặc queued-trigger. Tránh giữ database lock trong lúc làm việc bên ngoài nếu atomicity không yêu cầu.

## Cập nhật runtime và giới hạn

Helper được compile và activate sau metadata, trước các script cache khác. Lời gọi dùng snapshot đã activate thay vì đọc database mỗi lần. Một task giữ snapshot nhất quán; sửa đổi đã activate có hiệu lực với task tiếp theo.

- Helper không tồn tại hoặc bị tắt sẽ báo lỗi rõ ràng. Lỗi thông thường nêu tên helper; structured throw giữ status/carrier. Không tự retry helper.
- Giới hạn CPU, I/O, deadline và cancellation của task vẫn có hiệu lực.
- Catalogue có tối đa 1.024 definition; mỗi task có tối đa 1.024 lời gọi và độ sâu chuỗi gọi 32.
- Toàn bộ executable source của helper đã bật, caller và wrapper phải nằm trong giới hạn 4 MiB. Catalogue lớn vẫn có thể vượt giới hạn dù chỉ gọi một helper.
- Tham chiếu package vẫn chịu cơ chế chọn package và phân quyền hiện có.

Nếu thiếu trang Helpers hoặc runner, kiểm tra App/Server đã deploy và system metadata trước. Nếu không tìm thấy helper khi gọi, kiểm tra đúng tên và trạng thái enabled. Không tự tạo route dispatch dự phòng, retry hay mở rộng quyền.

Xem [Template Syntax](./template-syntax.md), [Helpers và Cache](./context-reference/helpers-cache.md) và [Phương thức Repository](./repository-methods/README.md) để tra cứu script API nền tảng.
