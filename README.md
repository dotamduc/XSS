# XSS
## 1. Browser Security Model
- Frontend chạy trong sandbox của browser, chỉ làm được những gì browser cho phép qua API (fetch, DOM, WebAPI…); không chạm trực tiếp OS như backend
- Không được tự ý đọc file local: chỉ đọc được file khi user chọn (input file + FileReader). Nếu không thì web nào cũng có thể ăn cắp dữ liệu máy người dùng
- Không gọi được system API trực tiếp: JS chỉ là ngôn ngữ, còn quyền là do browser; các API nhạy cảm (camera, mic, Bluetooth…) đều phải được user cho phép
- Same-Origin Policy (SOP): script chỉ truy cập được dữ liệu cùng origin; không đọc DOM/URL/nội dung của tab/iframe origin khác
- Browser còn dùng site isolation để tách site theo process, giảm rủi ro leak dữ liệu giữa site
- Nếu vượt được các giới hạn này (bypass SOP, RCE trong browser) thì đó là lỗ hổng trình duyệt, không còn là bug ứng dụng web bình thường

## 2. Starting with XSS for Frontend Security
- XSS = attacker thực thi được JS trong context webstie của nạn nhân
- Từ XSS, attacker có thể:
  - Đọc token trong `localStorage`/cookie (nếu không HTttpOnly)
  - Thực hiện mọi hành động mà user làm được (gửi request, đổi setting, thao tác dữ liệu....)
- Vì vậy nhiều site khi đổi mật khẩu bắt nhập lại _current password_: dù có XSS + token thì attacker vẫn không chiếm tài khoản lâu dài nếu không biết current password
- Phân loại theo cách lưu trữ:
  - **Reflected XSS**: payload không lưu, thường nằm trên URL/form → cần dụ từng nạn nhân click
  - **Stored XSS**: payload được lưu (comment, profile, bài viết…) → ai mở là dính, có thể lan rộng như “worm”
  - **Self-XSS**: chỉ ảnh hưởng đến chính người nhập (hoặc chỉ xuất hiện ở trang chỉ mình họ xem)
  - **Blind XSS**: payload chạy ở nơi bạn không thấy (thường là admin/staff panel), phải log ngược ra server của attacker

## 3.Understanding XSS a Bit More
### Các cách thực thi JS khi đã kiểm soát HTML:
- Thẻ `<script>`: trực diện nhưng hay bị filter/WAF, nhiều nơi (innerHTML) không thực thi script
- Event handler: `onerror=`, `onload=`, `onclick=...
  - Ví dụ: `<img src=x onerror=alert(1)>`, `<svg onload=alert(1)>`

- `javascript:` URL: `{href, src, action}="javascript:alert(1)"`
### Ngữ cảnh injection (context): 
1. **HTML context**: input thành nội dung HTML → có thể chèn thẻ mới, ví dụ `<img ... onerror=...>`
  - Cần escape `<` và `>`
2. **Attribute context**: input nằm trong giá trị thuộc tính (`class="...user input..."`)
- Tấn công bằng cách đóng quote, thêm thuộc tính/event hoặc thêm thẻ mới
- Cần escape thêm cả `"` `'` và tránh viết attribute không có quote

3. **JavaScript context**: input chui vào trong JS (string, template literal…)
- Có thể thoát khỏi string/script hoặc dùng `${...}` trong template literal để inject code
- Cần escape đủ `< > " '` và không concat input thẳng vào JS

## 4. Dangerous `javascript:` pseudo protocol
- `javascript:` là pseudo-protocol dùng để chạy JS, giống như `mailto:`/`tel:` nhưng nguy hiểm hơn vì thực thi code
- Có thể dùng ở nhiều chỗ:
  - href của <a>: `<a href="javascript:alert(1)">`
  - src của `<iframe>`: `<iframe src="javascript:alert(1)">`
  - `action`/`formaction` của form/button
  - Gán cho `window.location` hoặc redirect sau login/đăng ký
- Lý do dễ dính bug:
  - Nhiều code chỉ encode HTML (` < > " ' `) hoặc chỉ check “chứa chuỗi `youtube.com`/`example.com`”, nhưng không kiểm tra scheme → `javascript:alert(1)` vẫn hợp lệ
  - Framework như React/Vue nếu `href`/`src` bind trực tiếp từ dữ liệu user (không filter) thì vẫn XSS với `javascript:...`
- Cách phòng:
  - Whitelist scheme: chỉ cho `http:`, `https:` (có thể thêm `mailto:`, `tel:` nếu cần)
  - Validate URL ở backend hoặc dùng library chuyên để sanitize URL
  - Cẩn thận với mọi nơi dùng input làm URL hoặc redirect target
