# Import và Export Component

> Hướng dẫn từ [react.dev/learn/importing-and-exporting-components](https://react.dev/learn/importing-and-exporting-components)

Sức mạnh của component nằm ở khả năng **tái sử dụng**. Nhưng khi app lớn dần, để tất cả component trong một file sẽ rất khó quản lý. Bài này hướng dẫn cách tách component ra nhiều file và import/export chúng đúng cách.

---

## Root Component File là gì?

Khi bạn tạo một React app, thường có một file gọi là **root component file** — thường là `App.js`. Đây là điểm xuất phát của toàn bộ app.

```
App.js  ← root component file
```

Ban đầu bạn hay để mọi thứ trong `App.js`:

```jsx
// App.js — tất cả trong một file, ổn khi app còn nhỏ
function Profile() {
  return (
    <img src="https://example.com/photo.jpg" alt="Katherine Johnson" />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Các nhà khoa học</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

Khi app lớn dần → cần **tách ra nhiều file** cho dễ quản lý.

---

## 3 bước để tách Component ra file riêng

Giả sử bạn muốn chuyển `Gallery` và `Profile` ra file `Gallery.js`:

```
Bước 1: Tạo file mới       →  Gallery.js
Bước 2: Export component    →  export default function Gallery() { ... }
Bước 3: Import vào nơi dùng →  import Gallery from './Gallery.js'
```

**Sau khi tách — Gallery.js:**

```jsx
// Gallery.js

function Profile() {
  return (
    <img src="https://example.com/photo.jpg" alt="Katherine Johnson" />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Các nhà khoa học</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

**Sau khi tách — App.js:**

```jsx
// App.js
import Gallery from './Gallery.js';  // ← import từ file khác

export default function App() {
  return (
    <Gallery />
  );
}
```

Kết quả hoàn toàn như trước, nhưng code giờ được **tổ chức gọn gàng hơn**.

> 💡 Bạn có thể viết `'./Gallery'` hoặc `'./Gallery.js'` — cả hai đều hoạt động với React.

---

## Default Export vs Named Export — Điểm khác biệt cốt lõi

Đây là phần **quan trọng nhất** của bài. JavaScript có 2 cách export:

### Default Export — "Xuất mặc định"

```jsx
// ✅ Một file chỉ được có DUY NHẤT một default export
export default function Gallery() {
  return <section>...</section>;
}
```

```jsx
// Khi import — đặt tên TÙY Ý
import Gallery from './Gallery.js';
import BatKyTenGi from './Gallery.js';   // cũng hoạt động!
import XYZ from './Gallery.js';          // cũng hoạt động!
```

> Vì là "mặc định" nên JavaScript biết ngay bạn muốn lấy cái gì — không cần tên khớp.

---

### Named Export — "Xuất có tên"

```jsx
// Một file có thể có NHIỀU named export
export function Profile() { ... }
export function Avatar() { ... }
export const API_URL = 'https://...';
```

```jsx
// Khi import — phải dùng đúng TÊN, trong dấu ngoặc nhọn { }
import { Profile } from './Gallery.js';
import { Avatar } from './Gallery.js';
import { Profile, Avatar } from './Gallery.js';  // import nhiều cùng lúc
```

> Vì là "có tên" nên JavaScript cần biết chính xác bạn muốn lấy cái gì — tên phải khớp.

---

### So sánh cạnh nhau

| | Default Export | Named Export |
|--|---|---|
| **Số lượng mỗi file** | Tối đa **1** | Không giới hạn |
| **Cú pháp export** | `export default function Foo()` | `export function Foo()` |
| **Cú pháp import** | `import Foo from './file'` | `import { Foo } from './file'` |
| **Tên khi import** | Tùy đặt | Phải khớp với tên export |
| **Khi nào dùng** | File chỉ export 1 thứ | File export nhiều thứ |

**Ví dụ minh họa rõ nhất:**

```jsx
// ✅ Default — tên import tùy ý
import Gallery from './Gallery.js';
import MyGallery from './Gallery.js';   // cũng được!

// ✅ Named — tên phải khớp
import { Profile } from './Gallery.js';
import { Profile as P } from './Gallery.js';  // dùng alias nếu muốn đổi tên

// ❌ Sai — named export nhưng import không có { }
import Profile from './Gallery.js';  // Sẽ lấy default export, không phải Profile!
```

---

## Export nhiều Component từ một File

Một file hoàn toàn có thể vừa có default export vừa có named export:

**Gallery.js:**

```jsx
// Named export — Profile
export function Profile() {
  return (
    <img src="https://example.com/photo.jpg" alt="Ảnh" />
  );
}

// Default export — Gallery
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

**App.js — import cả hai:**

```jsx
import Gallery from './Gallery.js';          // default import
import { Profile } from './Gallery.js';      // named import

// Hoặc gộp lại một dòng:
import Gallery, { Profile } from './Gallery.js';

export default function App() {
  return (
    <>
      <Profile />   {/* Hiện 1 profile riêng lẻ */}
      <Gallery />   {/* Hiện cả gallery */}
    </>
  );
}
```

---

## Tách ra nhiều file riêng biệt

Khi `Gallery.js` có quá nhiều thứ, tách tiếp ra file `Profile.js`:

```
Trước:                          Sau:
─────────────────               ─────────────────────────
App.js                          App.js
  └─ Gallery.js                   ├─ Gallery.js
      ├─ Gallery (default)         │    └─ Gallery (default)
      └─ Profile (named)           └─ Profile.js
                                        └─ Profile (default)
```

**Profile.js:**

```jsx
// Profile.js — chỉ có Profile, dùng default export vì chỉ có 1 thứ
export default function Profile() {
  return (
    <img src="https://example.com/photo.jpg" alt="Ảnh nhà khoa học" />
  );
}
```

**Gallery.js — import Profile từ file riêng:**

```jsx
// Gallery.js
import Profile from './Profile.js';  // import Profile từ file riêng

export default function Gallery() {
  return (
    <section>
      <h1>Bộ sưu tập</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

**App.js — import cả hai:**

```jsx
// App.js
import Gallery from './Gallery.js';
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile />
      <Gallery />
    </>
  );
}
```

---

## Sơ đồ tổng quan — Luồng import/export

```
Profile.js                  Gallery.js                  App.js
──────────────              ──────────────              ──────────────
export default              import Profile              import Gallery
function Profile()    →     from './Profile.js'         from './Gallery.js'
                                                        import Profile
                            export default        →     from './Profile.js'
                            function Gallery()
                                                        export default
                                                        function App()
```

---

## Khi nào dùng Default, khi nào dùng Named?

Không có quy tắc bắt buộc, nhưng đây là **thông lệ phổ biến:**

| Tình huống | Nên dùng |
|-----------|----------|
| File chỉ export **1 component chính** | Default export |
| File export **nhiều component hoặc hàm tiện ích** | Named export |
| Muốn đổi tên khi import | Default (tên tùy ý) |
| Muốn đảm bảo tên nhất quán khắp project | Named (tên phải khớp) |

> ⚠️ **Tránh:** `export default () => {}` — component không tên rất khó debug vì React DevTools sẽ hiển thị là "Anonymous".

---

## Tóm tắt

```
Default Export                          Named Export
──────────────────────────────          ──────────────────────────────
export default function Foo() {}        export function Foo() {}
export default Foo;                     export { Foo };

import Foo from './file'                import { Foo } from './file'
import BatKyTen from './file'           import { Foo as AliasName } from './file'

Tối đa 1 per file ✓                    Không giới hạn ✓
Tên import tùy ý ✓                     Tên phải khớp ✓
```

---

## Bài tập tự luyện

### Tách Profile ra file riêng

Hiện tại `Gallery.js` chứa cả `Profile` lẫn `Gallery`. Hãy:
1. Tạo file `Profile.js` và chuyển `Profile` vào đó
2. Cập nhật `Gallery.js` để import `Profile` từ `Profile.js`
3. Cập nhật `App.js` để render cả `<Profile />` và `<Gallery />`

**Trạng thái ban đầu:**

```jsx
// Gallery.js (hiện tại)
export function Profile() {
  return (
    <img src="https://example.com/alan.jpg" alt="Alan L. Hart" />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Các nhà khoa học</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

<details>
<summary>👉 Xem đáp án (dùng Named Export cho Profile)</summary>

```jsx
// Profile.js
export function Profile() {
  return (
    <img src="https://example.com/alan.jpg" alt="Alan L. Hart" />
  );
}
```

```jsx
// Gallery.js
import { Profile } from './Profile.js';

export default function Gallery() {
  return (
    <section>
      <h1>Các nhà khoa học</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```jsx
// App.js
import Gallery from './Gallery.js';
import { Profile } from './Profile.js';

export default function App() {
  return (
    <>
      <Profile />
      <Gallery />
    </>
  );
}
```
</details>

<details>
<summary>👉 Xem đáp án (dùng Default Export cho Profile)</summary>

```jsx
// Profile.js
export default function Profile() {
  return (
    <img src="https://example.com/alan.jpg" alt="Alan L. Hart" />
  );
}
```

```jsx
// Gallery.js
import Profile from './Profile.js';

export default function Gallery() {
  return (
    <section>
      <h1>Các nhà khoa học</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```jsx
// App.js
import Gallery from './Gallery.js';
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile />
      <Gallery />
    </>
  );
}
```
</details>
