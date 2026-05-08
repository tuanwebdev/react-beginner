# Viết Markup với JSX

> Hướng dẫn từ [react.dev/learn/writing-markup-with-jsx](https://react.dev/learn/writing-markup-with-jsx)

---

## JSX là gì? Tại sao React dùng nó?

### Ngày xưa — HTML, CSS, JS tách rời nhau

Trước đây, lập trình web theo triết lý: mỗi thứ một chỗ — nội dung trong HTML, giao diện trong CSS, logic trong JavaScript. Ba file riêng biệt.

```
index.html        style.css         app.js
──────────        ──────────        ──────────
<div>             .sidebar {        if (isLoggedIn) {
  <p>...</p>        color: red;       showDashboard();
  <form>...       }                 }
</div>
```

### Bây giờ — Web ngày càng tương tác hơn

Web hiện đại không chỉ hiển thị nội dung — nó phản ứng theo hành vi người dùng. JavaScript ngày càng quyết định **cái gì** được hiển thị, **khi nào**, và **như thế nào**.

Tách HTML và JS ra hai file khác nhau lại trở nên bất tiện: khi nút bấm thay đổi, bạn phải sửa cả HTML lẫn JS, dễ bị không đồng bộ.

### React giải quyết bằng cách gom lại trong Component

```jsx
// Sidebar.js — logic và markup sống cùng nhau
function Sidebar() {
  const isLoggedIn = checkAuth();   // ← logic JS

  return (                          // ← markup JSX
    <div className="sidebar">
      {isLoggedIn ? <Dashboard /> : <LoginForm />}
    </div>
  );
}
```

**Logic render và markup liên quan đến nhau → đặt cùng chỗ.** Đó là triết lý của React, và JSX là công cụ để làm điều đó.

> 💡 **JSX và React là hai thứ khác nhau.** JSX là *cú pháp mở rộng* của JavaScript. React là *thư viện*. Bạn có thể dùng React mà không cần JSX (nhưng sẽ rất bất tiện), và JSX có thể dùng với các thư viện khác.

---

## JSX trông như HTML nhưng không phải HTML

JSX rất giống HTML khi nhìn qua, nhưng kỳ thực nó được **biên dịch thành JavaScript** trước khi chạy:

```jsx
// Bạn viết JSX:
<h1 className="title">Xin chào</h1>

// Trình biên dịch chuyển thành JS:
React.createElement('h1', { className: 'title' }, 'Xin chào')
```

Vì JSX là JavaScript, nó có **quy tắc chặt chẽ hơn HTML**. HTML khá dễ tính — bạn quên đóng thẻ, trình duyệt vẫn đoán được. JSX thì không — nó sẽ báo lỗi ngay.

---

## Ví dụ thực tế: Chuyển HTML sang JSX

Giả sử bạn có HTML hợp lệ sau muốn đưa vào React:

```html
<!-- HTML hợp lệ — hoạt động tốt trong trình duyệt -->
<h1>Danh sách việc cần làm</h1>
<img
  src="https://example.com/photo.jpg"
  alt="Hedy Lamarr"
  class="photo"
>
<ul>
    <li>Phát minh đèn giao thông mới
    <li>Tập dượt cảnh phim
    <li>Cải thiện công nghệ phổ
</ul>
```

Nếu copy thẳng vào component React → **lỗi ngay**:

```jsx
// ❌ KHÔNG HOẠT ĐỘNG
export default function TodoList() {
  return (
    <h1>Danh sách việc cần làm</h1>     // Lỗi: nhiều thẻ gốc
    <img
      src="https://example.com/photo.jpg"
      alt="Hedy Lamarr"
      class="photo"                      // Lỗi: dùng class thay className
    >                                    // Lỗi: thẻ img không tự đóng
    <ul>
      <li>Phát minh đèn giao thông mới  // Lỗi: thẻ li không đóng
      <li>Tập dượt cảnh phim
    </ul>
  );
}
```

Cần sửa theo **3 quy tắc JSX** dưới đây.

---

## 3 Quy tắc JSX bắt buộc phải nhớ

---

### Quy tắc 1: Chỉ được trả về MỘT phần tử gốc

Một component chỉ được `return` **một** thẻ bao ngoài cùng. Không thể return nhiều thẻ ngang hàng trực tiếp.

```jsx
// ❌ SAI — hai thẻ ngang hàng, không có thẻ bao
return (
  <h1>Tiêu đề</h1>
  <p>Nội dung</p>
);
```

**Cách 1 — Bọc bằng `<div>`** (thêm một thẻ vào DOM):

```jsx
// ✅ ĐÚNG
return (
  <div>
    <h1>Tiêu đề</h1>
    <p>Nội dung</p>
  </div>
);
```

**Cách 2 — Dùng Fragment `<>...</>`** (không thêm thẻ vào DOM — thường dùng hơn):

```jsx
// ✅ ĐÚNG — Fragment không để lại dấu vết trong HTML
return (
  <>
    <h1>Tiêu đề</h1>
    <p>Nội dung</p>
  </>
);
```

**Tại sao lại cần vậy?**

> JSX được biên dịch thành JavaScript object. Một hàm JavaScript không thể return hai object cùng lúc — cũng như vậy, JSX không thể return hai thẻ cùng lúc nếu không có thẻ bao ngoài.

```js
// Tương đương JS — không thể return 2 object
return { tag: 'h1' }
       { tag: 'p' }   // ← cái này bị bỏ qua!

// Phải return 1 object bao ngoài
return {
  tag: 'div',
  children: [{ tag: 'h1' }, { tag: 'p' }]
}
```

---

### Quy tắc 2: Đóng tất cả các thẻ

HTML thoải mái: `<li>`, `<br>`, `<img>` không cần đóng — trình duyệt tự đoán. JSX thì bắt buộc đóng hết.

**Thẻ tự đóng** (self-closing) — thêm `/` trước `>`:

```jsx
// ❌ HTML ổn, JSX lỗi
<img src="photo.jpg">
<br>
<input type="text">

// ✅ JSX đúng
<img src="photo.jpg" />
<br />
<input type="text" />
```

**Thẻ có nội dung** — phải có thẻ đóng tường minh:

```jsx
// ❌ HTML ổn, JSX lỗi
<li>Mục 1
<li>Mục 2

// ✅ JSX đúng
<li>Mục 1</li>
<li>Mục 2</li>
```

---

### Quy tắc 3: Thuộc tính dùng camelCase

JSX biến thuộc tính HTML thành key của JavaScript object. Mà JavaScript object không chấp nhận tên có dấu gạch ngang (`-`) và không chấp nhận từ khóa dành riêng như `class`.

**Vì vậy, hầu hết thuộc tính HTML được đổi sang camelCase trong JSX:**

| HTML (gốc) | JSX (viết lại) | Lý do |
|---|---|---|
| `class` | `className` | `class` là từ khóa JS |
| `for` | `htmlFor` | `for` là từ khóa JS |
| `onclick` | `onClick` | Quy ước camelCase |
| `onchange` | `onChange` | Quy ước camelCase |
| `tabindex` | `tabIndex` | Quy ước camelCase |
| `stroke-width` | `strokeWidth` | Không có dấu `-` trong JS |
| `background-color` | `backgroundColor` | Không có dấu `-` trong JS |
| `font-size` | `fontSize` | Không có dấu `-` trong JS |

```jsx
// ❌ HTML
<div class="card" onclick="handleClick()">
  <label for="email">Email</label>
  <input tabindex="1" />
</div>

// ✅ JSX
<div className="card" onClick={handleClick}>
  <label htmlFor="email">Email</label>
  <input tabIndex={1} />
</div>
```

> ⚠️ **Ngoại lệ — KHÔNG đổi sang camelCase:**
> - `aria-*` → vẫn giữ dấu gạch ngang: `aria-label`, `aria-hidden`
> - `data-*` → vẫn giữ dấu gạch ngang: `data-id`, `data-value`

---

## Kết quả sau khi sửa đủ 3 quy tắc

```jsx
// ✅ JSX hợp lệ hoàn toàn
export default function TodoList() {
  return (
    <>                                    {/* Quy tắc 1: Fragment bao ngoài */}
      <h1>Danh sách việc cần làm</h1>
      <img
        src="https://example.com/photo.jpg"
        alt="Hedy Lamarr"
        className="photo"               {/* Quy tắc 3: class → className */}
      />                                {/* Quy tắc 2: img tự đóng */}
      <ul>
        <li>Phát minh đèn giao thông mới</li>  {/* Quy tắc 2: li đóng */}
        <li>Tập dượt cảnh phim</li>
        <li>Cải thiện công nghệ phổ</li>
      </ul>
    </>
  );
}
```

---

## Bảng tóm tắt nhanh: HTML vs JSX

| Tình huống | HTML | JSX |
|---|---|---|
| Nhiều thẻ ngang hàng | ✅ Được | ❌ Phải bọc trong `<div>` hoặc `<>` |
| Thẻ không tự đóng | `<img>` | `<img />` |
| Thẻ không có nội dung | `<br>` | `<br />` |
| Gán CSS class | `class="..."` | `className="..."` |
| Gán label | `for="..."` | `htmlFor="..."` |
| Sự kiện click | `onclick="..."` | `onClick={...}` |
| Thuộc tính có gạch ngang | `stroke-width` | `strokeWidth` |
| Thuộc tính aria | `aria-label` | `aria-label` (giữ nguyên) |
| Thuộc tính data | `data-id` | `data-id` (giữ nguyên) |

---

## Mẹo: Dùng công cụ chuyển đổi tự động

Nếu có nhiều HTML cần chuyển sang JSX, thay vì sửa tay, dùng công cụ online:

🔗 **[transform.tools/html-to-jsx](https://transform.tools/html-to-jsx)**

Paste HTML vào → nhận JSX hợp lệ ngay lập tức. Rất hữu ích khi làm việc với HTML có sẵn.

---

## Bài tập tự luyện

### Chuyển đoạn HTML sau sang JSX hợp lệ

```html
<!-- HTML cần chuyển -->
<div class="intro">
  <h1>Chào mừng đến website của tôi!</h1>
</div>
<p class="summary">
  Bạn có thể tìm suy nghĩ của tôi ở đây.
  <br><br>
  <b>Và <i>hình ảnh</b></i> các nhà khoa học!
</p>
```

Hãy tìm tất cả lỗi trước khi xem đáp án. Có tất cả **5 lỗi** cần sửa.

<details>
<summary>👉 Gợi ý: Đây là danh sách các lỗi</summary>

1. Hai thẻ `<div>` và `<p>` ngang hàng — thiếu thẻ bao ngoài
2. `class="intro"` → cần đổi thành `className`
3. `class="summary"` → cần đổi thành `className`
4. `<br>` → cần đổi thành `<br />`
5. Thẻ `<b>` và `<i>` đóng sai thứ tự: `<b>...<i>...</b></i>` → phải là `<b>...<i>...</i></b>`
</details>

<details>
<summary>👉 Xem đáp án</summary>

```jsx
export default function Bio() {
  return (
    <>                                  {/* Thêm Fragment bao ngoài */}
      <div className="intro">           {/* class → className */}
        <h1>Chào mừng đến website của tôi!</h1>
      </div>
      <p className="summary">           {/* class → className */}
        Bạn có thể tìm suy nghĩ của tôi ở đây.
        <br /><br />                    {/* <br> → <br /> */}
        <b>Và <i>hình ảnh</i></b> các nhà khoa học!
        {/* Sửa thứ tự đóng thẻ: <b><i>...</i></b> */}
      </p>
    </>
  );
}
```
</details>

---

## Tóm tắt

JSX là cú pháp mở rộng của JavaScript cho phép viết markup trực tiếp trong component. Nó trông giống HTML nhưng chặt chẽ hơn với **3 quy tắc bắt buộc:**

```
Quy tắc 1: Return DUY NHẤT một thẻ gốc
           → Dùng <div> hoặc Fragment <> </> để bao

Quy tắc 2: Đóng TẤT CẢ thẻ
           → <img /> <br /> <li>...</li>

Quy tắc 3: Thuộc tính dùng camelCase
           → class→className, onclick→onClick, stroke-width→strokeWidth
           → Ngoại lệ: aria-* và data-* giữ nguyên dấu gạch ngang
```