# Render và Commit — React hoạt động "phía sau hậu trường" như thế nào?

> Hướng dẫn từ [react.dev/learn/render-and-commit](https://react.dev/learn/render-and-commit)

Trước khi component hiển thị lên màn hình, React phải thực hiện một quy trình gồm **3 bước**. Hiểu rõ quy trình này giúp bạn hiểu tại sao code hoạt động theo cách nó hoạt động — và giải thích được những hành vi "kỳ lạ" mà đôi khi bạn gặp phải.

---

## Phép ẩn dụ: React như một nhà hàng

Hãy tưởng tượng React hoạt động như một nhà hàng:

```
Khách hàng (User)
    ↓ đặt món
Bồi bàn (React)
    ↓ ghi order, mang vào bếp
Đầu bếp (Component)
    ↓ nấu món ăn
Bồi bàn (React)
    ↓ mang ra bàn cho khách
Màn hình (DOM)
```

Quy trình này gồm đúng **3 bước:**

| Bước | Nhà hàng | React |
|------|----------|-------|
| **1. Trigger** | Khách gọi món → bồi bàn ghi order | User tương tác → React nhận tín hiệu render |
| **2. Render** | Đầu bếp chuẩn bị món ăn trong bếp | React gọi component functions để tính toán JSX |
| **3. Commit** | Bồi bàn mang món ra bàn | React cập nhật DOM thực tế trên màn hình |

---

## Bước 1: Trigger — Khi nào React bắt đầu render?

Chỉ có **2 lý do** khiến React render một component:

### Lý do 1: Initial Render (Lần render đầu tiên)

Khi app khởi động, React cần render lần đầu. Việc này được thực hiện bằng `createRoot` và `render`:

```jsx
// index.js — điểm khởi đầu của mọi React app
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

// Tìm thẻ <div id="root"> trong HTML
const root = createRoot(document.getElementById('root'));

// Bắt đầu render — đây là "phát súng lệnh" đầu tiên
root.render(<Image />);
```

Thử comment dòng `root.render(...)` → component biến mất hoàn toàn khỏi màn hình.

### Lý do 2: Re-render khi State thay đổi

Sau lần render đầu, bất cứ khi nào **state được cập nhật** (qua setter function của `useState`), React tự động lên lịch render lại component đó:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}

// Khi user click:
// setCount(1) được gọi
//   → React nhận tín hiệu "Counter cần render lại"
//   → React lên lịch re-render (không render ngay lập tức)
//   → React render lại Counter với count = 1
```

> 💡 `setCount` không render ngay — nó **lên lịch** (queues) một render. React sẽ xử lý tất cả các update rồi render một lần duy nhất (gọi là batching).

---

## Bước 2: Render — React "nấu món" trong bếp

**"Rendering" = React gọi component function của bạn.**

React gọi hàm component để biết nó muốn hiển thị gì. Kết quả trả về là JSX — mô tả giao diện cần hiển thị.

### Initial Render

React gọi root component, rồi đệ quy gọi tất cả component con:

```jsx
// React sẽ gọi Gallery(), sau đó gọi Image() 3 lần
export default function Gallery() {
  return (
    <section>
      <h1>Tác phẩm điêu khắc</h1>
      <Image />   {/* → React gọi Image() lần 1 */}
      <Image />   {/* → React gọi Image() lần 2 */}
      <Image />   {/* → React gọi Image() lần 3 */}
    </section>
  );
}

function Image() {
  return (
    <img src="https://example.com/sculpture.jpg" alt="Tác phẩm" />
  );
}
```

**Kết quả:** React tạo ra một "bản vẽ" (virtual DOM) đầy đủ — toàn bộ cây DOM cần tạo.

### Re-render

React chỉ gọi **component có state thay đổi** (và các component con của nó):

```
App
├── Header          ← không re-render (state không đổi)
├── Counter         ← RE-RENDER (count thay đổi)
│   └── CountDisplay  ← re-render (vì cha re-render)
└── Footer          ← không re-render (state không đổi)
```

React **tính toán sự khác biệt** giữa JSX lần này và lần trước — nhưng chưa cập nhật DOM. Việc đó để bước 3 làm.

### ⚠️ Render phải là Pure Function (Hàm thuần túy)

```jsx
// ✅ ĐÚNG — cùng input → cùng output, không side effect
function ProductCard({ product }) {
  return (
    <div>
      <h2>{product.name}</h2>
      <p>{product.price}đ</p>
    </div>
  );
}

// ❌ SAI — thay đổi biến bên ngoài trong lúc render
let callCount = 0;
function ProductCard({ product }) {
  callCount++;          // ← side effect trong render — gây bug!
  return <div>{product.name}</div>;
}

// ❌ SAI — kết quả phụ thuộc vào thứ ngoài props/state
function Greeting() {
  return <h1>Xin chào, {Math.random() > 0.5 ? 'A' : 'B'}</h1>;
  // ← Mỗi lần render ra kết quả khác nhau — không thuần túy!
}
```

**2 nguyên tắc của render thuần túy:**

```
1. Cùng input → cùng output
   Với cùng props và state, component phải luôn trả về cùng JSX

2. Không can thiệp việc của người khác
   Không được thay đổi object/variable tồn tại trước khi render
```

> 💡 **Strict Mode:** Khi bật `<React.StrictMode>`, React **gọi mỗi component 2 lần** trong development để giúp phát hiện các hàm render không thuần túy. Nếu component của bạn có side effect trong render, nó sẽ bị phát hiện ngay.

---

## Bước 3: Commit — Mang món ra bàn

Sau khi tính toán xong, React **cập nhật DOM thực tế**.

### Initial Render

React dùng `appendChild()` để tạo tất cả DOM nodes mới và đưa vào trang:

```
JSX từ React:              DOM được tạo:
<section>          →       <section>
  <h1>Tiêu đề</h1>  →       <h1>Tiêu đề</h1>
  <img src="..." />  →       <img src="...">
</section>                 </section>
```

### Re-render — Điều đặc biệt nhất của React

React **chỉ cập nhật những phần thực sự thay đổi** — không render lại toàn bộ trang:

```jsx
// Component này re-render mỗi giây với 'time' mới
function Clock({ time }) {
  return (
    <>
      <h1>{time}</h1>   {/* ← thay đổi mỗi giây */}
      <input />          {/* ← KHÔNG thay đổi */}
    </>
  );
}
```

**Điều xảy ra:**
- `<h1>` được cập nhật mỗi giây ✅
- `<input>` **không bị đụng đến** — nên nếu bạn đang gõ vào input, chữ bạn gõ **không bị xóa** dù component re-render liên tục ✅

Đây chính là lý do React nhanh — nó không xóa và vẽ lại toàn bộ, mà **chỉ vá đúng chỗ thay đổi**.

```
Render lần trước:      Render lần này:       React làm gì?
<h1>12:00:00</h1>  →  <h1>12:00:01</h1>  →  Cập nhật text của h1
<input value="hi"/>→  <input value="hi"/>→  Không làm gì cả ✓
```

---

## Bước 4 (Bonus): Browser Paint

Sau khi React commit xong, trình duyệt **vẽ lại màn hình** (browser paint/repaint).

```
Trigger → Render → Commit → Browser Paint
                              ↑
                    Đây là lúc bạn thực sự thấy thay đổi trên màn hình
```

> ⚠️ Trong tài liệu React, "render" chỉ đề cập đến bước 2 (React gọi component). "Browser rendering" hay "painting" là bước cuối cùng hoàn toàn do trình duyệt xử lý.

---

## Sơ đồ tổng quan toàn bộ quy trình

```
╔══════════════════════════════════════════════════════════╗
║                    REACT RENDER CYCLE                    ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  [Trigger]                                               ║
║   ├── App khởi động → Initial render                     ║
║   └── setState() được gọi → Re-render                    ║
║           ↓                                              ║
║  [Render]                                                ║
║   ├── React gọi component function                       ║
║   ├── Component trả về JSX                               ║
║   ├── React đệ quy gọi các component con                 ║
║   └── React tính toán diff (khác gì so với lần trước?)   ║
║           ↓                                              ║
║  [Commit]                                                ║
║   ├── Initial: appendChild() — tạo toàn bộ DOM           ║
║   └── Re-render: chỉ cập nhật phần thay đổi              ║
║           ↓                                              ║
║  [Browser Paint]                                         ║
║   └── Trình duyệt vẽ lại màn hình                        ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

---

## Những điều quan trọng cần nhớ

### React không cập nhật DOM nếu kết quả giống nhau

```jsx
// Component này không thay đổi gì cả qua mỗi render
function StaticCard() {
  return <div className="card">Nội dung cố định</div>;
}

// Dù StaticCard được re-render (vì cha re-render),
// React thấy JSX output giống hệt lần trước
// → Không chạm vào DOM → Không repaint → Rất nhanh!
```

### Render không đồng bộ với việc hiển thị

```jsx
function MyComponent() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(1);
    // Tại đây, count vẫn là 0!
    // React chưa re-render — nó mới chỉ "lên lịch" render
    console.log(count); // → 0, không phải 1
  }

  return <button onClick={handleClick}>{count}</button>;
}
```

Lý do sẽ được giải thích kỹ hơn trong bài **State as a Snapshot**.

### Strict Mode giúp phát hiện lỗi

```jsx
// index.js — bật Strict Mode
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>      {/* ← bọc app trong StrictMode */}
    <App />
  </StrictMode>
);

// Strict Mode gọi component function 2 lần trong development
// → Nếu render có side effect, sẽ bị lộ ngay
// → Chỉ ảnh hưởng development, không ảnh hưởng production
```

---

## Tóm tắt

```
Bước 1 — TRIGGER: 2 lý do kích hoạt render
  • Initial render (app khởi động)
  • setState() được gọi

Bước 2 — RENDER: React gọi component function
  • Đệ quy gọi tất cả component con
  • Tính toán JSX output
  • Render PHẢI thuần túy: cùng input → cùng output, không side effect

Bước 3 — COMMIT: Cập nhật DOM
  • Initial: tạo toàn bộ DOM nodes
  • Re-render: chỉ cập nhật phần thực sự thay đổi
  • Nếu output giống lần trước → không đụng đến DOM

Bước 4 — BROWSER PAINT: Trình duyệt vẽ lại màn hình
  • React đã xong việc — trình duyệt tiếp quản
```

> 💡 **Insight quan trọng nhất của bài này:** React không render lại toàn bộ trang mỗi khi có thay đổi — nó chỉ cập nhật **đúng những gì thay đổi**. Đây là lý do React nhanh và tại sao input của bạn không bị reset khi component re-render.
