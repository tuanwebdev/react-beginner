# React Quick Start — Hướng dẫn đầy đủ (Tiếng Việt)

> Tóm tắt 10 khái niệm cốt lõi từ [react.dev/learn](https://react.dev/learn) kèm ví dụ thực tế.

---

## 1. Tạo và lồng Component

**Component** là khối xây dựng cơ bản của React. Mỗi component là một hàm JavaScript trả về JSX (giao diện). Component có thể nhỏ như một nút bấm hoặc lớn như toàn bộ trang.

**Quy tắc quan trọng:** Tên component phải bắt đầu bằng **chữ hoa** (`MyButton`), còn thẻ HTML thì viết thường (`div`, `button`).

```jsx
// Component con
function MyButton() {
  return <button>Tôi là một nút</button>;
}

// Component cha — lồng MyButton vào trong
export default function MyApp() {
  return (
    <div>
      <h1>Chào mừng đến app của tôi</h1>
      <MyButton />   {/* <-- chữ hoa! */}
    </div>
  );
}
```

**Giải thích:**
- `export default` đánh dấu component chính của file.
- `<MyButton />` là cách dùng component như thẻ HTML — React nhận ra vì tên bắt đầu bằng chữ hoa.

---

## 2. Viết markup với JSX

**JSX** là cú pháp cho phép viết HTML-like trực tiếp trong JavaScript. Hầu hết dự án React đều dùng JSX vì sự tiện lợi của nó.

JSX **chặt chẽ hơn HTML** ở 2 điểm:
- Phải đóng tất cả thẻ: `<br />`, `<img />`, `<input />`
- Chỉ được trả về **một** phần tử gốc duy nhất

```jsx
// ❌ SAI — trả về 2 thẻ ngang hàng, không có thẻ bao ngoài
function AboutPage() {
  return (
    <h1>Giới thiệu</h1>
    <p>Xin chào!</p>
  );
}

// ✅ ĐÚNG — bọc trong <div>
function AboutPage() {
  return (
    <div>
      <h1>Giới thiệu</h1>
      <p>Xin chào!</p>
    </div>
  );
}

// ✅ ĐÚNG — bọc trong Fragment <> </> (không tạo thẻ thừa trong DOM)
function AboutPage() {
  return (
    <>
      <h1>Giới thiệu</h1>
      <p>Xin chào!<br />Bạn có khỏe không?</p>
    </>
  );
}
```

> 💡 Nếu có nhiều HTML cần chuyển sang JSX, dùng công cụ [html-to-jsx converter](https://transform.tools/html-to-jsx) để tiết kiệm thời gian.

---

## 3. Thêm Style (CSS)

Trong JSX, dùng `className` thay vì `class` để gán CSS class (vì `class` là từ khóa trong JavaScript).

```jsx
// JSX — dùng className
<img className="avatar" src="photo.jpg" />
```

```css
/* File CSS riêng */
.avatar {
  border-radius: 50%;
  width: 80px;
  height: 80px;
}
```

**Inline style** dùng object JavaScript (thuộc tính viết camelCase):

```jsx
// style nhận vào một object, nên có 2 dấu ngoặc nhọn {{ }}
<div style={{ color: 'red', fontSize: 16, backgroundColor: '#f0f0f0' }}>
  Chữ đỏ
</div>
```

| CSS thường | JSX inline style |
|---|---|
| `font-size` | `fontSize` |
| `background-color` | `backgroundColor` |
| `border-radius` | `borderRadius` |
| `margin-top` | `marginTop` |

> 💡 React không quy định cách bạn thêm file CSS. Cách đơn giản nhất là dùng thẻ `<link>` trong HTML, hoặc theo hướng dẫn của framework bạn dùng (Next.js, Vite...).

---

## 4. Hiển thị Dữ liệu

Dùng dấu ngoặc nhọn `{}` để nhúng biến và biểu thức JavaScript vào JSX.

```jsx
const user = {
  name: 'Nguyễn Văn A',
  imageUrl: 'https://example.com/photo.jpg',
  age: 25,
};

export default function Profile() {
  return (
    <>
      {/* Nhúng biến vào nội dung */}
      <h1>Xin chào, {user.name}</h1>

      {/* Nhúng biến vào thuộc tính */}
      <img
        className="avatar"
        src={user.imageUrl}
        alt={'Ảnh của ' + user.name}
      />

      {/* Biểu thức tính toán */}
      <p>Năm sinh: {new Date().getFullYear() - user.age}</p>

      {/* Gọi hàm */}
      <p>Tên in hoa: {user.name.toUpperCase()}</p>
    </>
  );
}
```

**Trong `{}` bạn có thể đặt:**
- Biến: `{user.name}`
- Phép tính: `{price * quantity}`
- Gọi hàm: `{formatDate(date)}`
- Ternary: `{isOnline ? 'Online' : 'Offline'}`

**Không thể đặt trong `{}`:** câu lệnh `if`, vòng lặp `for`, khai báo biến `const`...

---

## 5. Render Có Điều Kiện

React không có cú pháp đặc biệt — dùng JavaScript thuần để render có điều kiện.

### Cách 1: `if / else` (rõ ràng nhất)

```jsx
function Greeting({ isLoggedIn }) {
  let content;
  if (isLoggedIn) {
    content = <AdminPanel />;
  } else {
    content = <LoginForm />;
  }
  return <div>{content}</div>;
}
```

### Cách 2: Toán tử ternary `? :` (ngắn gọn, dùng được trong JSX)

```jsx
function Greeting({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? (
        <AdminPanel />
      ) : (
        <LoginForm />
      )}
    </div>
  );
}
```

### Cách 3: Toán tử `&&` (khi không có nhánh `else`)

```jsx
function Notification({ hasMessage }) {
  return (
    <div>
      {hasMessage && <p>Bạn có tin nhắn mới!</p>}
    </div>
  );
}
```

> ⚠️ **Bẫy phổ biến:** `{0 && <X/>}` sẽ render ra số `0` thay vì không render gì!
> Dùng `{count > 0 && <X/>}` hoặc `{!!count && <X/>}` để tránh lỗi này.

---

## 6. Render Danh Sách

Dùng `.map()` để chuyển mảng thành danh sách JSX. Mỗi phần tử **bắt buộc** có thuộc tính `key` duy nhất.

```jsx
const products = [
  { id: 1, name: 'Táo',   isFruit: true,  price: 5000 },
  { id: 2, name: 'Cải',   isFruit: false, price: 3000 },
  { id: 3, name: 'Xoài',  isFruit: true,  price: 8000 },
];

export default function ShoppingList() {
  const listItems = products.map(product => (
    <li
      key={product.id}                       // key là bắt buộc!
      style={{ color: product.isFruit ? 'green' : 'black' }}
    >
      {product.name} — {product.price.toLocaleString()}đ
    </li>
  ));

  return <ul>{listItems}</ul>;
}
```

**Tại sao cần `key`?**

React dùng `key` để biết phần tử nào được thêm, xóa, hay di chuyển khi danh sách thay đổi — từ đó cập nhật DOM hiệu quả hơn.

| Nguồn key | Ví dụ | Dùng được không? |
|---|---|---|
| ID từ database | `key={user.id}` | ✅ Tốt nhất |
| Index của mảng | `key={index}` | ⚠️ Chỉ dùng khi danh sách không thay đổi thứ tự |
| Random | `key={Math.random()}` | ❌ Không bao giờ dùng |

---

## 7. Xử lý Sự kiện

Khai báo hàm handler bên trong component và truyền vào thuộc tính sự kiện.

```jsx
function MyButton() {
  // 1. Khai báo hàm handler
  function handleClick() {
    alert('Bạn đã nhấn nút!');
  }

  return (
    // 2. Truyền hàm — KHÔNG có () ở cuối
    <button onClick={handleClick}>
      Nhấn vào đây
    </button>
  );
}
```

**Sự kiện phổ biến:**

```jsx
<button onClick={handleClick}>Nhấn</button>
<input  onChange={handleChange} />
<form   onSubmit={handleSubmit}>...</form>
<div    onMouseEnter={handleHover} onMouseLeave={handleLeave} />
<input  onKeyDown={handleKeyDown} />
```

**Truyền tham số vào handler:**

```jsx
function ProductList() {
  function handleDelete(productId) {
    console.log('Xóa sản phẩm:', productId);
  }

  return (
    <button onClick={() => handleDelete(42)}>
      Xóa
    </button>
  );
}
```

> ⚡ **Nguyên tắc vàng:**
> - `onClick={handleClick}` ✅ — truyền hàm (React sẽ gọi khi click)
> - `onClick={handleClick()}` ❌ — gọi hàm ngay khi render, không phải khi click!

---

## 8. State — Trạng thái Component

`useState` cho phép component "nhớ" và cập nhật dữ liệu. Khi state thay đổi, React tự động render lại component.

```jsx
import { useState } from 'react';

function Counter() {
  // [giá_trị_hiện_tại, hàm_cập_nhật] = useState(giá_trị_ban_đầu)
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <p>Đã nhấn: {count} lần</p>
      <button onClick={handleClick}>+1</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

**Nhiều state trong một component:**

```jsx
function Form() {
  const [name, setName]       = useState('');
  const [email, setEmail]     = useState('');
  const [isSubmit, setSubmit] = useState(false);

  return (
    <form>
      <input value={name}  onChange={e => setName(e.target.value)}  />
      <input value={email} onChange={e => setEmail(e.target.value)} />
    </form>
  );
}
```

**Mỗi instance component có state riêng:**

```jsx
// Đặt 2 Counter cạnh nhau — chúng hoàn toàn độc lập
export default function App() {
  return (
    <>
      <Counter />   {/* count riêng của nó */}
      <Counter />   {/* count riêng của nó */}
    </>
  );
}
```

> ⚠️ **Không bao giờ** sửa state trực tiếp:
> ```jsx
> count = count + 1;     // ❌ Sai — React không biết để render lại
> setCount(count + 1);   // ✅ Đúng — dùng hàm setter
> ```

---

## 9. Hooks

**Hooks** là các hàm đặc biệt bắt đầu bằng `use`. Chúng cho phép dùng các tính năng của React bên trong function component.

### Quy tắc bắt buộc của Hooks

```jsx
// ✅ ĐÚNG — gọi ở đầu component
function MyComponent() {
  const [count, setCount] = useState(0);
  useEffect(() => { ... }, []);
  // ...
}

// ❌ SAI — không gọi Hook trong điều kiện
function MyComponent() {
  if (condition) {
    const [x, setX] = useState(0);  // LỖI!
  }
}

// ❌ SAI — không gọi Hook trong vòng lặp
function MyComponent() {
  for (let i = 0; i < 3; i++) {
    const [x, setX] = useState(0);  // LỖI!
  }
}
```

### Các Hook built-in quan trọng

**`useState`** — lưu trữ và cập nhật state:
```jsx
const [value, setValue] = useState(initialValue);
```

**`useEffect`** — chạy side effects sau khi render (gọi API, subscribe, thao tác DOM...):
```jsx
useEffect(() => {
  document.title = `Bạn có ${count} tin nhắn`;
}, [count]); // Chạy lại mỗi khi count thay đổi

useEffect(() => {
  fetchData(); // Chỉ chạy 1 lần khi component mount
}, []);
```

**`useRef`** — tham chiếu đến DOM element mà không trigger re-render:
```jsx
function TextInput() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

**`useContext`** — đọc dữ liệu từ Context (chia sẻ data không cần truyền props qua nhiều tầng):
```jsx
const theme = useContext(ThemeContext);
```

**Custom Hook** — tạo hook riêng bằng cách kết hợp các hook có sẵn:
```jsx
// Tạo custom hook
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handler = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler);
  }, []);

  return width;
}

// Dùng custom hook
function MyComponent() {
  const width = useWindowWidth();
  return <p>Chiều rộng cửa sổ: {width}px</p>;
}
```

---

## 10. Chia sẻ Dữ liệu — Lifting State Up

Khi nhiều component cần dùng **chung** dữ liệu và cập nhật cùng lúc, hãy đưa state lên component cha và truyền xuống các con qua **props**.

### Vấn đề: State độc lập (không chia sẻ)

```jsx
// Mỗi MyButton có count riêng — không liên quan nhau
export default function MyApp() {
  return (
    <>
      <MyButton />   {/* count: 0 */}
      <MyButton />   {/* count: 0 (riêng biệt) */}
    </>
  );
}

function MyButton() {
  const [count, setCount] = useState(0);  // state nằm ở đây
  return <button onClick={() => setCount(count + 1)}>Nhấn {count}</button>;
}
```

### Giải pháp: Lifting State Up

```jsx
// Bước 1: Đưa state lên component cha
export default function MyApp() {
  const [count, setCount] = useState(0);  // state chuyển lên đây

  function handleClick() {
    setCount(count + 1);
  }

  // Bước 2: Truyền state và handler xuống qua props
  return (
    <>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </>
  );
}

// Bước 3: Component con nhận và dùng props
function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Đã nhấn {count} lần
    </button>
  );
}
// Giờ cả 2 nút cùng hiển thị và cập nhật một count chung!
```

### Luồng dữ liệu một chiều

```
MyApp (state: count = 3)
  ├── MyButton (props: count=3, onClick=handleClick)
  └── MyButton (props: count=3, onClick=handleClick)
```

Khi bấm nút → `onClick` gọi `handleClick` → `setCount` cập nhật state ở `MyApp` → React render lại `MyApp` và cả 2 `MyButton` với count mới.

> 💡 **Nguyên tắc:** State nên đặt ở component thấp nhất có thể, nhưng đủ cao để tất cả component cần dùng đều có thể nhận qua props.

---

## Tổng kết

| Khái niệm | Dùng để | Ví dụ nhanh |
|---|---|---|
| **Component** | Tách UI thành mảnh nhỏ | `function MyBtn() { return <button/> }` |
| **JSX** | Viết HTML trong JS | `<h1>{user.name}</h1>` |
| **className** | Gán CSS class | `<div className="card">` |
| **`{}`** | Nhúng JS vào JSX | `<p>{price * qty}</p>` |
| **Ternary `?:`** | Render có điều kiện | `{ok ? <A/> : <B/>}` |
| **`.map()` + key** | Render danh sách | `items.map(i => <li key={i.id}>)` |
| **onClick, onChange** | Xử lý sự kiện | `<button onClick={fn}>` |
| **`useState`** | Lưu trữ state | `const [n, setN] = useState(0)` |
| **Hooks** | Dùng tính năng React | `useState`, `useEffect`, `useRef` |
| **Props + Lifting State** | Chia sẻ data | Đưa state lên cha, truyền xuống con |

---

## Bước tiếp theo

- 📖 [Tutorial: Tic-Tac-Toe](https://react.dev/learn/tutorial-tic-tac-toe) — xây app đầu tiên
- 📖 [Describing the UI](https://react.dev/learn/describing-the-ui) — đi sâu hơn về JSX và component
- 📖 [Adding Interactivity](https://react.dev/learn/adding-interactivity) — state và sự kiện
- 📖 [Managing State](https://react.dev/learn/managing-state) — quản lý state phức tạp