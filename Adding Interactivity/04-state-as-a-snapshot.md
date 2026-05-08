# State như một Bản Chụp (Snapshot)

> Hướng dẫn từ [react.dev/learn/state-as-a-snapshot](https://react.dev/learn/state-as-a-snapshot)

Đây là một trong những khái niệm **khó hiểu nhất** của React — nhưng khi hiểu được nó, bạn sẽ giải thích được rất nhiều hành vi "kỳ lạ" mà mình từng gặp.

---

## Ý tưởng cốt lõi: State không phải biến thông thường

Nhìn qua, state trông giống biến JavaScript bình thường:

```jsx
const [number, setNumber] = useState(0);
```

Nhưng thực ra nó hoạt động hoàn toàn khác. **State giống một bản chụp ảnh (snapshot) hơn là một biến.**

---

## Phép ẩn dụ: Chụp ảnh

Hãy tưởng tượng mỗi lần React render component, nó **chụp một bức ảnh** của toàn bộ giao diện tại thời điểm đó — bao gồm state, props, event handler, biến cục bộ.

```
Render lần 1 (number = 0):
┌─────────────────────────────────────┐
│  number = 0                         │
│  JSX: <h1>0</h1>                    │
│  onClick: setNumber(0+1) = set(1)   │  ← "bản chụp" lần 1
└─────────────────────────────────────┘

Render lần 2 (number = 1):
┌─────────────────────────────────────┐
│  number = 1                         │
│  JSX: <h1>1</h1>                    │
│  onClick: setNumber(1+1) = set(2)   │  ← "bản chụp" lần 2
└─────────────────────────────────────┘
```

Mỗi bản chụp là **hoàn toàn độc lập**. Event handler trong bản chụp lần 1 luôn thấy `number = 0`, dù state đã được cập nhật lên 1 ở bản chụp lần 2.

---

## Ví dụ kinh điển: +3 nhưng chỉ tăng 1

Đây là ví dụ làm hầu hết mọi người bị bất ngờ:

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);  // setNumber(0 + 1)
        setNumber(number + 1);  // setNumber(0 + 1)
        setNumber(number + 1);  // setNumber(0 + 1)
      }}>+3</button>
    </>
  );
}
```

**Bạn nghĩ kết quả là gì khi click?**

👉 Chỉ tăng lên **1**, không phải 3!

**Tại sao?**

Trong một lần render, `number` là một hằng số cố định — giống như bạn đóng băng giá trị đó vào bản chụp. Hãy thay `number` bằng giá trị thực của nó:

```jsx
// Lúc render, number = 0. React "đóng băng" số 0 vào event handler này:
<button onClick={() => {
  setNumber(0 + 1);  // React ghi nhớ: "sẽ đổi thành 1"
  setNumber(0 + 1);  // React ghi nhớ: "sẽ đổi thành 1" (lần nữa!)
  setNumber(0 + 1);  // React ghi nhớ: "sẽ đổi thành 1" (lần nữa!)
}}>+3</button>

// Kết quả: React chỉ nhận được lệnh "đổi thành 1" — dù 3 lần
// → number sau khi re-render = 1, không phải 3
```

Cả 3 lần `setNumber` đều dùng `number = 0` vì **trong một bản chụp render, `number` không bao giờ thay đổi** dù bạn gọi setter bao nhiêu lần.

---

## State "đóng băng" ngay cả trong code bất đồng bộ

Đây mới là phần thực sự gây ngạc nhiên. State giữ nguyên giá trị của bản chụp **kể cả khi code chạy bất đồng bộ** — sau `setTimeout`, sau `fetch`, sau bất kỳ delay nào.

### Ví dụ 1: alert ngay lập tức

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <button onClick={() => {
      setNumber(number + 5);
      alert(number);  // ← hiện số mấy?
    }}>+5</button>
  );
}
```

Bạn click khi `number = 0`. `alert` hiện **0**, không phải 5.

Vì trong bản chụp này, `number = 0`. Dù `setNumber(5)` đã được gọi, giá trị `number` trong bản chụp **không bao giờ thay đổi**. `setNumber` chỉ lên lịch render lại với giá trị mới — nó không thay đổi biến hiện tại.

### Ví dụ 2: alert sau 3 giây (sau khi re-render xong)

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <button onClick={() => {
      setNumber(number + 5);
      setTimeout(() => {
        alert(number);  // ← hiện số mấy sau 3 giây?
      }, 3000);
    }}>+5</button>
  );
}
```

Bạn click khi `number = 0`. Sau 3 giây, `alert` vẫn hiện **0** — dù lúc này màn hình đã hiển thị 5.

**Tại sao?** Vì `setTimeout` callback được tạo ra trong bản chụp render lúc `number = 0`. Nó "đóng băng" số 0 vào trong closure — dù React đã cập nhật state bên trong của mình lên 5.

```
Thời điểm click:     number = 0 trong bản chụp này
                     → setNumber(0 + 5): báo React cập nhật
                     → setTimeout lưu lại number = 0 (từ bản chụp)

3 giây sau:          React đã re-render, màn hình hiện 5
                     Nhưng setTimeout vẫn dùng number = 0 từ bản chụp cũ
                     → alert(0)
```

---

## Ứng dụng thực tế: Gửi tin nhắn với delay

Đây là ví dụ quan trọng nhất, rất gần với thực tế:

```jsx
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Xin chào');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      // Sau 5 giây mới hiện alert
      alert(`Bạn đã gửi "${message}" tới ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <select value={to} onChange={e => setTo(e.target.value)}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
      </select>
      <textarea
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Gửi</button>
    </form>
  );
}
```

**Kịch bản:**
1. Bạn chọn **Alice**, nhập "Xin chào", nhấn **Gửi**
2. Trong vòng 5 giây, bạn đổi sang **Bob**
3. Alert hiện ra — nó nói gì?

👉 Alert hiện: **"Bạn đã gửi 'Xin chào' tới Alice"** — không phải Bob!

**Tại sao đây lại là điều tốt?**

Đây thực ra là hành vi **đúng và an toàn**. Event handler được tạo ra ở thời điểm submit — nó "chụp" đúng state lúc user bấm nút. Nếu nó dùng state mới nhất, bạn có thể vô tình gửi tin nhắn sai người chỉ vì user vô tình chạm vào dropdown sau khi bấm gửi.

---

## Quy tắc vàng để ghi nhớ

> **Trong một lần render (một bản chụp), giá trị state là hằng số — không bao giờ thay đổi, kể cả trong code bất đồng bộ.**

Để hiểu code sẽ thấy gì, hãy **thay thế biến state bằng giá trị cụ thể của nó tại thời điểm render đó:**

```jsx
// Thay vì đọc:
onClick={() => {
  setNumber(number + 1);
  setNumber(number + 1);
  alert(number);
}}

// Hãy đọc như này (khi number = 0):
onClick={() => {
  setNumber(0 + 1);  // yêu cầu đổi thành 1
  setNumber(0 + 1);  // yêu cầu đổi thành 1 (lại)
  alert(0);          // hiện 0
}}
```

---

## State "sống" bên ngoài component

Một điểm quan trọng: state **không biến mất** khi hàm component trả về kết quả. State sống trong bộ nhớ của React, bên ngoài hàm của bạn — như thể trên một cái giá (shelf):

```
React's shelf (bộ nhớ của React):
┌──────────────────────────────┐
│  Counter #1: number = 3      │  ← React giữ state ở đây
│  Form: to = "Alice"          │
│  Form: message = "Xin chào"  │
└──────────────────────────────┘

Mỗi lần render, React lấy state từ shelf
và "tặng" cho component dưới dạng snapshot
```

Khi bạn gọi `setNumber(5)`:
- React **cập nhật shelf** (lưu 5 thay cho 0)
- React **lên lịch render lại**
- Lần render tiếp theo, component nhận `number = 5` từ shelf

---

## Tóm tắt bằng sơ đồ

```
User click nút
    ↓
Event handler chạy
(Dùng state từ BẢN CHỤP HIỆN TẠI — số đã "đóng băng")
    ↓
setXxx() được gọi
→ React cập nhật shelf
→ React lên lịch re-render
    ↓
React render lại component
→ Tạo BẢN CHỤP MỚI với state mới từ shelf
    ↓
DOM được cập nhật
User thấy giao diện mới
```

---

## Kiểm tra hiểu bài

### Câu hỏi 1: Alert hiện gì?

```jsx
const [count, setCount] = useState(10);

<button onClick={() => {
  setCount(count + 1);
  setCount(count + 1);
  alert(count);
}}>Click</button>
// count hiện tại = 10
```

<details>
<summary>👉 Xem đáp án</summary>

- `alert` hiện **10** (giá trị tại bản chụp hiện tại)
- Sau khi re-render, màn hình hiện **11** (không phải 12 — vì cả 2 lần đều `setCount(10 + 1)`)
</details>

---

### Câu hỏi 2: setTimeout alert hiện gì?

```jsx
const [name, setName] = useState('Alice');

<button onClick={() => {
  setName('Bob');
  setTimeout(() => alert(name), 2000);
}}>Đổi tên</button>
// name hiện tại = 'Alice'
```

<details>
<summary>👉 Xem đáp án</summary>

Alert hiện **"Alice"** — dù màn hình đã hiển thị "Bob" từ trước đó 2 giây.

`setTimeout` callback được tạo trong bản chụp lúc `name = 'Alice'`, nên nó luôn thấy `name = 'Alice'` bất kể state đã đổi.
</details>

---

### Câu hỏi 3: Đèn giao thông

```jsx
const [walk, setWalk] = useState(true);

function handleClick() {
  setWalk(!walk);
  alert(walk ? 'Tiếp theo: Dừng' : 'Tiếp theo: Đi');
}
```

Khi đèn đang **Walk (walk = true)**, bạn click. Alert hiện gì?

<details>
<summary>👉 Xem đáp án</summary>

Alert hiện **"Tiếp theo: Dừng"** — đây là hành vi đúng!

Lúc click, bản chụp có `walk = true`. Dù `setWalk(false)` đã được gọi, trong bản chụp này `walk` vẫn là `true`. Vì vậy:

```jsx
alert(true ? 'Tiếp theo: Dừng' : 'Tiếp theo: Đi')
// → alert('Tiếp theo: Dừng')
```

Alert đúng vì nó mô tả trạng thái **sắp đến** — rất hợp lý về mặt UX.

```jsx
// Code hoàn chỉnh:
function handleClick() {
  setWalk(!walk);
  // walk vẫn là giá trị CŨ trong bản chụp này
  // → alert mô tả đúng trạng thái tiếp theo
  alert(walk ? 'Tiếp theo: Dừng' : 'Tiếp theo: Đi');
}
```
</details>

---

## Tóm tắt

```
State là "bản chụp" — không phải biến thông thường

1. setXxx() KHÔNG thay đổi state ngay
   → Nó lên lịch render lại với giá trị mới

2. Trong một lần render, state là HẰNG SỐ
   → Không thay đổi dù gọi setter bao nhiêu lần
   → Không thay đổi dù code chạy bất đồng bộ (setTimeout, fetch...)

3. Mỗi render có bản chụp RIÊNG
   → Event handler "thấy" state tại thời điểm render tạo ra nó
   → Không phải state mới nhất

4. Để hiểu code: thay biến state bằng giá trị cụ thể
   number + 1 (khi number=0) → 0 + 1
```

> 💡 Muốn đọc state **mới nhất** mà không cần đợi re-render? Dùng **updater function**: `setNumber(n => n + 1)` — sẽ học trong bài tiếp theo.
