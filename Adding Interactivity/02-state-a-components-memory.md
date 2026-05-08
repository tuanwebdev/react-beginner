# State: Bộ Nhớ của Component

> Hướng dẫn từ [react.dev/learn/state-a-components-memory](https://react.dev/learn/state-a-components-memory)

Component thường cần **ghi nhớ** thông tin để hiển thị đúng — ô input đang có giá trị gì, ảnh nào đang được hiện, giỏ hàng có những gì... Trong React, loại bộ nhớ này được gọi là **state**.

---

## Tại sao biến thông thường không đủ?

Hãy xem ví dụ gallery ảnh — mỗi lần nhấn "Next" muốn hiển thị ảnh tiếp theo:

```jsx
// ❌ Code này KHÔNG hoạt động — tại sao?
import { sculptureList } from './data.js';

export default function Gallery() {
  let index = 0;  // ← biến thông thường

  function handleClick() {
    index = index + 1;  // ← thay đổi biến
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>Next</button>
      <h2>{sculpture.name}</h2>
      <img src={sculpture.url} alt={sculpture.alt} />
    </>
  );
}
```

Bạn click nút, nhưng ảnh **không thay đổi**. Vì sao?

Có **2 vấn đề** với biến thông thường:

```
Vấn đề 1: Biến cục bộ KHÔNG tồn tại giữa các lần render
           Mỗi lần React render lại component, nó chạy lại hàm từ đầu
           → index được đặt lại = 0

Vấn đề 2: Thay đổi biến cục bộ KHÔNG kích hoạt render lại
           React không biết index đã thay đổi
           → Không render lại → màn hình không cập nhật
```

Để UI cập nhật, cần **2 thứ** xảy ra đồng thời:
1. **Giữ lại** dữ liệu giữa các lần render
2. **Kích hoạt** React render lại với dữ liệu mới

`useState` cung cấp đúng 2 thứ đó.

---

## Thêm State Variable với `useState`

### Bước 1: Import useState

```jsx
import { useState } from 'react';
```

### Bước 2: Thay biến thông thường bằng state

```jsx
// ❌ Trước — biến thông thường
let index = 0;

// ✅ Sau — state variable
const [index, setIndex] = useState(0);
```

### Kết quả hoàn chỉnh

```jsx
// ✅ Code này HOẠT ĐỘNG
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);  // ← state thay biến thường

  function handleClick() {
    setIndex(index + 1);  // ← dùng setter thay vì gán trực tiếp
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>Next</button>
      <h2>{sculpture.name}</h2>
      <img src={sculpture.url} alt={sculpture.alt} />
    </>
  );
}
```

Bây giờ khi click → ảnh thay đổi!

---

## Giải phẫu `useState`

```jsx
const [index, setIndex] = useState(0);
//     ↑       ↑                   ↑
//  giá trị  hàm cập nhật      giá trị ban đầu
```

`useState` trả về một **mảng 2 phần tử** — bạn dùng array destructuring để lấy ra:

| Phần tử | Tên | Vai trò |
|---|---|---|
| Phần tử thứ 1 | `index` | **State variable** — giá trị hiện tại, được giữ lại giữa các lần render |
| Phần tử thứ 2 | `setIndex` | **Setter function** — cập nhật state VÀ kích hoạt render lại |

**Quy ước đặt tên:** `[something, setSomething]` — tên setter = `set` + tên state.

### useState hoạt động như thế nào qua từng bước?

```
Lần render đầu tiên:
  useState(0) → React lưu 0, trả về [0, setIndex]
  index = 0, hiển thị sculptureList[0]

User click "Next":
  setIndex(0 + 1) = setIndex(1)
  → React lưu 1, kích hoạt render lại

Lần render thứ 2:
  useState(0) → React thấy đã có state = 1, trả về [1, setIndex]
  index = 1, hiển thị sculptureList[1]

...cứ tiếp tục như vậy
```

> 💡 **Lưu ý:** Tham số `0` trong `useState(0)` chỉ được dùng cho **lần đầu tiên**. Từ lần thứ 2 trở đi, React bỏ qua nó và trả về giá trị đã lưu.

---

## Hook là gì?

`useState` — và bất kỳ hàm nào bắt đầu bằng `use` — đều gọi là **Hook**.

Hook là những hàm đặc biệt cho phép bạn "móc vào" (hook into) các tính năng của React. State chỉ là một trong số đó.

### ⚠️ Quy tắc bắt buộc của Hook

```jsx
// ✅ ĐÚNG — gọi Hook ở đầu component
function MyComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  // ...
}

// ❌ SAI — gọi Hook trong điều kiện
function MyComponent() {
  if (someCondition) {
    const [count, setCount] = useState(0);  // LỖI!
  }
}

// ❌ SAI — gọi Hook trong vòng lặp
function MyComponent() {
  for (let i = 0; i < 3; i++) {
    const [x, setX] = useState(0);  // LỖI!
  }
}

// ❌ SAI — gọi Hook trong hàm lồng nhau
function MyComponent() {
  function helper() {
    const [x, setX] = useState(0);  // LỖI!
  }
}
```

**Tại sao phải gọi ở đầu?**

React nhận diện state không phải bằng tên biến mà bằng **thứ tự gọi Hook**. Nội bộ React dùng một mảng để lưu các state theo thứ tự:

```
Render lần 1:
  useState(0)     → slot[0] = 0   (index)
  useState(false) → slot[1] = false (showMore)

Render lần 2:
  useState(0)     → đọc slot[0] → 1   (index đã được update)
  useState(false) → đọc slot[1] → true (showMore đã được update)
```

Nếu bạn gọi Hook trong điều kiện, thứ tự có thể thay đổi → React đọc sai slot → bug khó tìm!

---

## Nhiều State Variables trong một Component

Một component có thể có bao nhiêu state tùy ý, với nhiều kiểu dữ liệu khác nhau:

```jsx
import { useState } from 'react';

export default function Gallery() {
  const [index, setIndex]       = useState(0);      // number
  const [showMore, setShowMore] = useState(false);  // boolean

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);  // toggle: true→false, false→true
  }

  let sculpture = sculptureList[index];

  return (
    <>
      <button onClick={handleNextClick}>Next</button>
      <h2>{sculpture.name} by {sculpture.artist}</h2>
      <p>({index + 1} of {sculptureList.length})</p>

      <button onClick={handleMoreClick}>
        {showMore ? 'Ẩn' : 'Xem'} chi tiết
      </button>

      {showMore && <p>{sculpture.description}</p>}

      <img src={sculpture.url} alt={sculpture.alt} />
    </>
  );
}
```

**Khi nào nên tách/gộp state?**

```
✅ Tách thành nhiều state khi:
   - Các giá trị KHÔNG liên quan đến nhau
   - Ví dụ: index (ảnh nào) và showMore (có hiện mô tả không)

✅ Gộp thành một state (object) khi:
   - Các giá trị thường thay đổi CÙNG LÚC
   - Ví dụ: form có nhiều trường → gộp vào { name, email, phone }
```

```jsx
// Ví dụ gộp state cho form
const [formData, setFormData] = useState({
  name: '',
  email: '',
  phone: '',
});

function handleNameChange(e) {
  setFormData({
    ...formData,        // giữ lại các trường khác
    name: e.target.value  // chỉ cập nhật name
  });
}
```

---

## State là riêng tư và độc lập

State **chỉ thuộc về** component instance đó trên màn hình. Nếu render cùng một component hai lần, mỗi bản có state hoàn toàn độc lập:

```jsx
// Page.js
export default function Page() {
  return (
    <div>
      <Gallery />   {/* Gallery này có state riêng */}
      <Gallery />   {/* Gallery này có state riêng — hoàn toàn độc lập! */}
    </div>
  );
}
```

```
Page
├── Gallery #1  [index=0, showMore=false]  ← state riêng
└── Gallery #2  [index=3, showMore=true]   ← state riêng, không liên quan
```

Click "Next" ở Gallery #1 không ảnh hưởng gì đến Gallery #2.

**State khác với biến module ở điểm này:**

```jsx
// ❌ Biến module — DÙNG CHUNG cho mọi instance
let sharedIndex = 0;  // cả 2 Gallery đều dùng cái này!

// ✅ State — RIÊNG cho từng instance
const [index, setIndex] = useState(0);  // mỗi Gallery có index riêng
```

### State hoàn toàn riêng tư — cha không thể can thiệp

```jsx
// Page KHÔNG biết Gallery đang ở index nào
// Page KHÔNG thể thay đổi state của Gallery
export default function Page() {
  return <Gallery />;  // chỉ render thôi, không kiểm soát state bên trong
}
```

Điều này giúp bạn thêm/xóa state trong bất kỳ component nào mà không lo ảnh hưởng đến component khác.

> 💡 **Nếu muốn 2 Gallery đồng bộ state?** Hãy **đưa state lên component cha** (`Page`) và truyền xuống qua props — đây là kỹ thuật "Lifting State Up" đã học trong Thinking in React.

---

## So sánh: Biến thường vs State

| | Biến thông thường (`let x = 0`) | State (`useState(0)`) |
|---|---|---|
| **Tồn tại qua re-render** | ❌ Bị reset mỗi lần render | ✅ Được giữ lại |
| **Kích hoạt re-render** | ❌ Không | ✅ Có (khi dùng setter) |
| **Riêng tư với instance** | ❌ Không (nếu là biến module) | ✅ Có |
| **Cha có thể đọc** | Tùy | ❌ Không trực tiếp |
| **Cách cập nhật** | `x = x + 1` | `setX(x + 1)` |

---

## Tóm tắt

```
1. Dùng useState khi component cần "nhớ" thông tin giữa các lần render

2. Cú pháp:
   const [value, setValue] = useState(initialValue);
   //     ↑         ↑                  ↑
   // đọc giá trị  cập nhật        giá trị lần đầu

3. Hook chỉ được gọi ở đầu component — không trong if/for/hàm lồng

4. Nhiều state: gọi useState nhiều lần — React phân biệt bằng thứ tự gọi

5. State là riêng tư — mỗi instance component có state độc lập
   → Cùng component render 2 lần = 2 bộ state khác nhau
```

---

## Bài tập tự luyện

### Bài 1: Hoàn thiện Gallery

Code hiện tại bị crash khi nhấn "Next" ở ảnh cuối cùng. Hãy:
1. Ngăn crash khi đã ở ảnh cuối
2. Thêm nút "Previous" để quay lại ảnh trước (không crash ở ảnh đầu)

```jsx
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);

  function handleNextClick() {
    setIndex(index + 1);  // ← crash khi index = sculptureList.length - 1
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleNextClick}>Next</button>
      {/* Thêm nút Previous ở đây */}
      <h2>{sculpture.name}</h2>
      <img src={sculpture.url} alt={sculpture.alt} />
    </>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);

  function handleNextClick() {
    // Ngăn vượt quá ảnh cuối
    if (index < sculptureList.length - 1) {
      setIndex(index + 1);
    }
  }

  function handlePrevClick() {
    // Ngăn xuống dưới 0
    if (index > 0) {
      setIndex(index - 1);
    }
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handlePrevClick} disabled={index === 0}>
        Previous
      </button>
      <button onClick={handleNextClick} disabled={index === sculptureList.length - 1}>
        Next
      </button>
      <h2>{sculpture.name} by {sculpture.artist}</h2>
      <p>({index + 1} / {sculptureList.length})</p>
      <img src={sculpture.url} alt={sculpture.alt} />
    </>
  );
}
```
</details>

---

### Bài 2: Tìm lỗi — tại sao state không cập nhật?

```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    count = count + 1;  // ← lỗi ở đây
  }

  return (
    <button onClick={handleClick}>
      Đã nhấn {count} lần
    </button>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);  // ✅ dùng setter, không gán trực tiếp
  }

  return (
    <button onClick={handleClick}>
      Đã nhấn {count} lần
    </button>
  );
}
```

**Lỗi:** Gán trực tiếp `count = count + 1` không kích hoạt React render lại. Phải dùng `setCount(count + 1)`.
</details>

---

### Bài 3: Xóa state không cần thiết

```jsx
import { useState } from 'react';

// Có gì sai với component này?
export default function Greeting({ firstName, lastName }) {
  const [fullName, setFullName] = useState(firstName + ' ' + lastName);

  return <h1>Xin chào, {fullName}!</h1>;
}
```

<details>
<summary>👉 Xem đáp án</summary>

`fullName` không cần là state vì nó **tính được từ props**. State không cần thiết làm code phức tạp hơn và dễ gây bug (khi props thay đổi, state không tự cập nhật).

```jsx
// ✅ Đúng — tính trực tiếp từ props
export default function Greeting({ firstName, lastName }) {
  const fullName = firstName + ' ' + lastName;  // biến thường, không phải state
  return <h1>Xin chào, {fullName}!</h1>;
}
```
</details>
