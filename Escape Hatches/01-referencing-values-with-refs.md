# Referencing Values with Refs trong React

## Ref là gì?

Ref là một "hộp bí mật" trong component — nơi bạn lưu trữ thông tin mà React **không theo dõi** và không gây re-render khi thay đổi.

Khai báo bằng `useRef`:

```js
import { useRef } from 'react';

const ref = useRef(0);
// ref = { current: 0 }
```

Bạn đọc/ghi qua `ref.current`:

```js
ref.current = ref.current + 1; // mutate trực tiếp, không cần setter
```

---

## Ví dụ thực tế: Đồng hồ bấm giờ (Stopwatch)

Đây là ví dụ kinh điển để thấy **khi nào dùng state, khi nào dùng ref**:

- `startTime` và `now` → dùng **state** vì cần hiển thị lên màn hình.
- `intervalRef` (ID của `setInterval`) → dùng **ref** vì chỉ cần khi `clearInterval`, không cần render.

```js
const [startTime, setStartTime] = useState(null);
const [now, setNow] = useState(null);
const intervalRef = useRef(null); // lưu interval ID

function handleStart() {
  intervalRef.current = setInterval(() => setNow(Date.now()), 10);
}

function handleStop() {
  clearInterval(intervalRef.current); // lấy ID từ ref
}
```

---

## useRef hoạt động như thế nào bên trong?

Thực ra `useRef` được xây dựng trên `useState`, nhưng bỏ đi setter:

```js
// Mô phỏng cách React triển khai useRef
function useRef(initialValue) {
  const [ref] = useState({ current: initialValue });
  return ref; // cùng object qua mọi render
}
```

Đó là lý do ref **giữ nguyên identity** qua các lần render — React luôn trả về cùng một object.

---

## Best practices

**Treat ref as an "escape hatch"** — dùng khi cần thoát ra khỏi luồng dữ liệu một chiều của React (thường là tương tác với browser API).

**Không đọc/ghi `ref.current` trong quá trình render** — vì React không biết khi nào nó thay đổi, code sẽ không dự đoán được.

```js
// ❌ Sai — đọc ref trong render
return <div>{myRef.current}</div>;

// ✅ Đúng — đọc trong event handler
function handleClick() {
  console.log(myRef.current);
}
```

**Ref và DOM** — trường hợp phổ biến nhất là trỏ ref đến một DOM element:

```jsx
const inputRef = useRef(null);

// React sẽ gán DOM node vào inputRef.current
<input ref={inputRef} />

// Rồi dùng trong event handler
inputRef.current.focus();
```

---

**Tóm lại một câu:** nếu thông tin cần hiển thị lên UI → dùng **state**; nếu chỉ cần trong event handler / tương tác ngoài React → dùng **ref**.