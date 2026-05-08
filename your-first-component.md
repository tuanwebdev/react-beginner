# Component Đầu Tiên Của Bạn

> Hướng dẫn từ [react.dev/learn/your-first-component](https://react.dev/learn/your-first-component)  
> Component là khái niệm **cốt lõi nhất** của React — hiểu rõ nó là bạn đã có nền tảng vững chắc.

---

## Component là gì? Tại sao cần nó?

Trước đây, khi làm web, bạn viết HTML để dựng nội dung, rồi "rắc" JavaScript vào để thêm tương tác. Hai thứ tách rời nhau.

React thay đổi điều đó: **HTML, CSS và JavaScript được gom lại thành một khối gọi là Component** — một mảnh giao diện có thể tái sử dụng ở nhiều nơi.

Hãy tưởng tượng trang web như một tờ báo — nó được ghép từ nhiều mảnh:

```
Ngày xưa (HTML thuần):          React:
─────────────────────           ─────────────────────
<article>                       <PageLayout>
  <h1>Tiêu đề</h1>               <NavigationHeader>
  <ol>                               <SearchBar />
    <li>Mục 1</li>                   <Link>Docs</Link>
    <li>Mục 2</li>               </NavigationHeader>
  </ol>                          <Sidebar />
</article>                       <PageContent>
                                     <TableOfContents />
  ← Copy-paste ở mọi trang           <DocumentationText />
                                 </PageContent>
                               </PageLayout>
                                 ↑ Mỗi mảnh dùng lại được
```

Thay vì copy-paste HTML khắp nơi, bạn tạo `<TableOfContents />` một lần và dùng ở bất kỳ đâu.

---

## Cách tạo một Component — 3 bước đơn giản

### Bước 1: Export component

```jsx
export default function Profile() {
// ↑ export default = "đây là thứ chính của file này, cho file khác dùng được"
```

`export default` là cú pháp JavaScript tiêu chuẩn — không phải của riêng React. Nó giúp file khác có thể `import` component này vào dùng.

---

### Bước 2: Định nghĩa hàm

```jsx
function Profile() {
//       ↑ Tên bắt đầu bằng CHỮ HOA — bắt buộc!
```

> ⚠️ **Bẫy cực kỳ phổ biến:** Nếu tên component bắt đầu bằng chữ thường, React sẽ hiểu nhầm là thẻ HTML và **không hoạt động**!
>
> ```jsx
> function profile() { ... }  // ❌ React nghĩ đây là thẻ HTML <profile>
> function Profile() { ... }  // ✅ React biết đây là component
> ```

---

### Bước 3: Thêm markup (JSX) và return

```jsx
export default function Profile() {
  return (
    <img
      src="https://example.com/photo.jpg"
      alt="Katherine Johnson"
    />
  );
}
```

Phần bên trong `return (...)` trông như HTML nhưng thực ra là **JSX** — JavaScript có thể "nói chuyện" được với markup.

**Quy tắc về return:**

```jsx
// ✅ Một dòng — không cần ngoặc đơn
return <img src="photo.jpg" alt="Ảnh" />;

// ✅ Nhiều dòng — phải có ngoặc đơn ()
return (
  <div>
    <img src="photo.jpg" alt="Ảnh" />
  </div>
);

// ❌ Nhiều dòng nhưng KHÔNG có ngoặc — JavaScript sẽ bỏ qua mọi thứ sau return!
return
  <div>           // Dòng này bị bỏ qua hoàn toàn!
    <img />
  </div>
```

---

## Ví dụ hoàn chỉnh — Component đầu tiên

```jsx
// Profile là component hiển thị ảnh một nhà khoa học
export default function Profile() {
  return (
    <img
      src="https://example.com/scientist.jpg"
      alt="Katherine Johnson"
    />
  );
}
```

Chỉ vậy thôi! Bạn vừa tạo xong component React đầu tiên.

---

## Dùng Component — Lồng vào nhau

Sau khi định nghĩa `Profile`, bạn có thể dùng nó như một thẻ HTML tự tạo: `<Profile />`.

```jsx
// Component con
function Profile() {
  return (
    <img
      src="https://example.com/scientist.jpg"
      alt="Katherine Johnson"
    />
  );
}

// Component cha — dùng Profile nhiều lần
export default function Gallery() {
  return (
    <section>
      <h1>Các nhà khoa học vĩ đại</h1>
      <Profile />   {/* Dùng lần 1 */}
      <Profile />   {/* Dùng lần 2 */}
      <Profile />   {/* Dùng lần 3 */}
    </section>
  );
}
```

**Trình duyệt thực sự nhận được gì?** React "dịch" các component thành HTML thuần:

```html
<!-- Đây là HTML mà trình duyệt thực sự thấy -->
<section>
  <h1>Các nhà khoa học vĩ đại</h1>
  <img src="https://example.com/scientist.jpg" alt="Katherine Johnson" />
  <img src="https://example.com/scientist.jpg" alt="Katherine Johnson" />
  <img src="https://example.com/scientist.jpg" alt="Katherine Johnson" />
</section>
```

---

## React phân biệt Component với thẻ HTML như thế nào?

Rất đơn giản — **dựa vào chữ hoa/thường:**

| Viết trong JSX | React hiểu là | Kết quả |
|---|---|---|
| `<section>` | Thẻ HTML `<section>` | Render trực tiếp |
| `<Profile />` | Component tên `Profile` | Gọi hàm `Profile()`, lấy kết quả |
| `<img />` | Thẻ HTML `<img>` | Render trực tiếp |
| `<SearchBar />` | Component tên `SearchBar` | Gọi hàm `SearchBar()`, lấy kết quả |

---

## Quan hệ Cha — Con (Parent — Child)

Khi `Gallery` render `Profile` bên trong nó:
- `Gallery` là **component cha** (parent)
- `Profile` là **component con** (child)

```
Gallery (cha)
├── Profile (con)
├── Profile (con)
└── Profile (con)
```

Bạn định nghĩa `Profile` **một lần** và dùng bao nhiêu lần tùy thích — đó là sức mạnh của tái sử dụng.

---

## Lưu ý quan trọng: Không định nghĩa Component lồng nhau!

Đây là lỗi **rất hay gặp** và gây bug khó tìm:

```jsx
// ❌ SAI — định nghĩa Profile TRONG Gallery
export default function Gallery() {
  // 🔴 Đừng bao giờ làm thế này! Chậm và gây bug!
  function Profile() {
    return <img src="photo.jpg" alt="Ảnh" />;
  }

  return (
    <section>
      <Profile />
    </section>
  );
}
```

```jsx
// ✅ ĐÚNG — định nghĩa ở cùng cấp (top level)
export default function Gallery() {
  return (
    <section>
      <Profile />
    </section>
  );
}

// Profile ở ngoài Gallery, cùng cấp trong file
function Profile() {
  return <img src="photo.jpg" alt="Ảnh" />;
}
```

**Tại sao lại vậy?**

Mỗi lần `Gallery` render, React lại tạo ra một hàm `Profile` mới hoàn toàn. React không nhận ra đây là "cùng một component" nên sẽ **xóa và tạo lại từ đầu** mỗi lần — làm mất state, chậm hơn và tạo ra những bug khó hiểu.

> 💡 Nếu component con cần dữ liệu từ cha, hãy truyền qua **props** — không phải bằng cách lồng định nghĩa.

---

## Tổ chức Component trong file

Bạn có thể để nhiều component trong cùng một file — tiện khi chúng nhỏ hoặc liên quan chặt chẽ nhau:

```jsx
// file: Gallery.jsx

// Component con — nhỏ, liên quan trực tiếp → để cùng file
function Profile() {
  return <img src="photo.jpg" alt="Ảnh nhà khoa học" />;
}

// Component cha — export ra ngoài
export default function Gallery() {
  return (
    <section>
      <h1>Bộ sưu tập</h1>
      <Profile />
      <Profile />
    </section>
  );
}
```

Khi file bắt đầu quá dài hoặc `Profile` cần dùng ở nhiều nơi khác → chuyển nó sang file riêng `Profile.jsx`.

---

## Mọi thứ trong React app đều là Component

Trong React, **kể cả trang (page) cũng là component**. Không chỉ các nút nhỏ hay widget — sidebar, header, footer, form... tất cả đều là component.

```
App (root component)
├── Header
│   ├── Logo
│   ├── NavigationMenu
│   └── UserAvatar
├── Sidebar
│   ├── CategoryList
│   └── TagCloud
└── MainContent
    ├── ArticleCard
    ├── ArticleCard
    └── Pagination
```

Toàn bộ app là một cây component lồng nhau. Component ở gốc (root) thường được gọi là `App` — đây là điểm khởi đầu của mọi React app.

---

## Tóm tắt

| Khái niệm | Điểm cần nhớ |
|-----------|-------------|
| **Component là gì** | Hàm JavaScript trả về JSX — một mảnh UI có thể tái sử dụng |
| **Tên component** | Bắt buộc bắt đầu bằng **chữ hoa** (`MyButton`, không phải `myButton`) |
| **return nhiều dòng** | Phải bọc trong `( )` — nếu không JavaScript tự thêm `;` sau `return` |
| **Dùng component** | Viết như thẻ HTML: `<Profile />` hoặc `<Profile></Profile>` |
| **Lồng component** | Dùng component này bên trong component khác — nhưng KHÔNG lồng định nghĩa |
| **Tổ chức file** | Nhiều component có thể trong cùng file; tách ra khi file quá lớn |

---

## Bài tập tự luyện

### Bài 1: Sửa lỗi export

```jsx
// ❌ Code này không chạy được — tìm và sửa lỗi
function Profile() {
  return (
    <img src="https://example.com/photo.jpg" alt="Ảnh" />
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
// ✅ Thêm export default
export default function Profile() {
  return (
    <img src="https://example.com/photo.jpg" alt="Ảnh" />
  );
}
```

Thiếu `export default` nên file khác không import được component này.
</details>

---

### Bài 2: Sửa lỗi return

```jsx
// ❌ Code này không hiển thị gì — tại sao?
export default function Profile() {
  return
    <div>
      <img src="https://example.com/photo.jpg" alt="Ảnh" />
    </div>
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
// ✅ Thêm ngoặc đơn () sau return
export default function Profile() {
  return (
    <div>
      <img src="https://example.com/photo.jpg" alt="Ảnh" />
    </div>
  );
}
```

JavaScript tự động thêm `;` sau `return` trên một dòng riêng, khiến hàm trả về `undefined` thay vì JSX.
</details>

---

### Bài 3: Tìm lỗi

```jsx
// ❌ Code này có lỗi — bạn tìm được không?
export default function Gallery() {
  function profile() {
    return <img src="https://example.com/photo.jpg" alt="Ảnh" />;
  }

  return (
    <section>
      <profile />
    </section>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

Có **2 lỗi:**

1. `function profile()` — tên bắt đầu bằng chữ thường → React nghĩ là thẻ HTML
2. Định nghĩa `profile` lồng bên trong `Gallery` — không nên làm vậy

```jsx
// ✅ Sửa lại
export default function Gallery() {
  return (
    <section>
      <Profile />
    </section>
  );
}

function Profile() {   // chữ hoa, ở ngoài Gallery
  return <img src="https://example.com/photo.jpg" alt="Ảnh" />;
}
```
</details>
