Dưới đây là phần trình bày kiến thức dựa trên nội dung từ liên kết bạn cung cấp, kèm theo các ví dụ minh họa và giải thích chi tiết.

---

### 📚 Thao tác DOM với Refs trong React

**Refs (viết tắt của references)** là một "lối thoát" (escape hatch) trong React, cho phép bạn truy cập trực tiếp vào các phần tử DOM hoặc các giá trị khác mà không cần thông qua luồng dữ liệu thông thường. Chúng ta thường dùng refs khi cần thực hiện các thao tác mà React không hỗ trợ sẵn như focus, scroll, đo kích thước phần tử, v.v.

### 1. Cách lấy ref đến một phần tử DOM

Để truy cập vào một node DOM do React quản lý, bạn cần thực hiện 3 bước:

1.  **Import `useRef` Hook:** `import { useRef } from 'react';`
2.  **Khai báo một ref:** `const myRef = useRef(null);` (Giá trị khởi tạo thường là `null`).
3.  **Gán ref cho phần tử JSX:** `<div ref={myRef}>`

Lúc này, `myRef` là một đối tượng có thuộc tính `current`. Ban đầu, `myRef.current` là `null`. Khi React tạo ra node DOM cho `<div>`, nó sẽ gán tham chiếu đến node đó vào `myRef.current`. Bạn có thể truy cập node DOM này trong các event handler để gọi các Web API gốc.

#### 🔍 Ví dụ: Focus vào ô input

```jsx
import { useRef } from 'react';

function Form() {
  const inputRef = useRef(null); // Bước 2: Khai báo ref

  function handleClick() {
    inputRef.current.focus(); // Truy cập DOM node và gọi API focus()
  }

  return (
    <>
      <input ref={inputRef} /> {/* Bước 3: Gán ref */}
      <button onClick={handleClick}>Focus Input</button>
    </>
  );
}
```

### 2. Ví dụ: Cuộn đến một phần tử

Bạn có thể sử dụng nhiều ref trong một component. Ví dụ sau đây tạo 3 ref cho 3 hình ảnh và các nút bấm để cuộn mượt mà đến từng ảnh bằng `scrollIntoView()`:

```jsx
import { useRef } from 'react';

function CatFriends() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function scrollToCat(ref) {
    ref.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center',
    });
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToCat(firstCatRef)}>Neo</button>
        <button onClick={() => scrollToCat(secondCatRef)}>Millie</button>
        <button onClick={() => scrollToCat(thirdCatRef)}>Bella</button>
      </nav>
      <div>
        <ul>
          <li><img src="neo.jpg" alt="Neo" ref={firstCatRef} /></li>
          <li><img src="millie.jpg" alt="Millie" ref={secondCatRef} /></li>
          <li><img src="bella.jpg" alt="Bella" ref={thirdCatRef} /></li>
        </ul>
      </div>
    </>
  );
}
```

### 3. Quản lý danh sách refs với Ref Callback

Khi cần tham chiếu đến một danh sách các phần tử không có số lượng cố định, bạn không thể gọi `useRef` trong vòng lặp. **Ref callback** là giải pháp: bạn truyền một hàm cho thuộc tính `ref`. React sẽ gọi hàm này với node DOM khi nó được tạo ra và trả về `null` khi nó bị hủy.

Ví dụ sau dùng ref callback để lưu trữ một `Map` giữa đối tượng dữ liệu và node DOM của nó, cho phép cuộn đến bất kỳ mục nào trong danh sách dài.

```jsx
<li
  key={cat.id}
  ref={(node) => {
    const map = getMap(); // Lấy Map từ ref chính
    if (node) {
      map.set(cat, node); // Thêm vào Map khi node được tạo
    } else {
      map.delete(cat); // Xóa khỏi Map khi node bị hủy
    }
  }}
>
```

### 4. Truy cập DOM Node của Component khác

Theo mặc định, React component ẩn các node DOM của nó. Để cho phép component cha truy cập node DOM của component con, bạn có thể sử dụng cơ chế **`forwardRef`** hoặc truyền ref như một prop thông thường (với tên gọi khác `ref`).

```jsx
// Component con nhận ref như một prop tên là 'ref'
function MyInput({ ref }) {
  return <input ref={ref} />;
}

// Component cha tạo ref và truyền xuống
function MyForm() {
  const inputRef = useRef(null);
  function handleClick() {
    inputRef.current.focus();
  }
  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus</button>
    </>
  );
}
```

**Lưu ý:** Việc này khiến mã dễ bị phụ thuộc. Để giới hạn quyền truy cập, bạn có thể dùng **`useImperativeHandle`** để chỉ phơi bày một số phương thức nhất định (ví dụ: chỉ cho phép `focus`, không cho thay đổi style).

### 5. Thời điểm React gán Refs

Quá trình cập nhật của React chia làm hai pha:
- **Render:** React gọi component để biết cần hiển thị gì.
- **Commit:** React cập nhật DOM thật.

**Bạn chỉ nên truy cập `ref.current` trong hoặc sau pha Commit.** Trong pha Render, DOM node chưa được tạo/cập nhật, nên `ref.current` có thể là `null` hoặc chứa giá trị cũ. Các event handler và Effect là nơi an toàn để dùng refs.

### 6. Đồng bộ cập nhật State với `flushSync`

Đôi khi bạn cần DOM được cập nhật **ngay lập tức** sau khi thay đổi state để thao tác (ví dụ: cuộn đến phần tử mới thêm vào). Theo mặc định, các cập nhật state được gom theo lô (batching). Để ép React cập nhật DOM đồng bộ, bạn dùng `flushSync` từ `react-dom`.

```jsx
import { flushSync } from 'react-dom';

function handleAdd() {
  const newTodo = { id: nextId++, text: text };
  flushSync(() => { // Ép cập nhật DOM ngay lập tức
    setTodos([...todos, newTodo]);
  });
  // Lúc này DOM đã có phần tử mới, có thể cuộn tới nó
  listRef.current.lastChild.scrollIntoView();
}
```

### 7. Các phương pháp tốt nhất

- **Chỉ dùng refs khi thực sự cần "bước ra ngoài React"**, chủ yếu cho các hành động không phá hủy như focus, scroll, đo lường.
- **Tránh sửa DOM do React quản lý.** Việc tự ý xóa, sửa, thêm node có thể gây lỗi nghiêm trọng vì React không còn kiểm soát được DOM đó nữa.
- **An toàn nếu chỉ sửa phần DOM mà React không có lý do gì để cập nhật.** Ví dụ: một `<div>` luôn trống trong JSX thì bạn có thể tự do thêm/xóa phần tử con vào đó.

### 📝 Tóm tắt

| Mục đích | Cách thực hiện |
| :--- | :--- |
| **Khai báo ref** | `const myRef = useRef(null);` |
| **Gán ref vào JSX** | `<div ref={myRef}>` |
| **Truy cập node DOM** | `myRef.current` (chỉ trong event handler, Effect, hoặc sau pha Commit) |
| **Xử lý danh sách động** | Dùng **ref callback**: `<li ref={(node) => map.set(item, node)}>` |
| **Cho phép cha truy cập DOM con** | Truyền ref như một prop hoặc dùng `useImperativeHandle` để giới hạn API. |
| **Ép cập nhật DOM đồng bộ** | Dùng `flushSync` bọc câu lệnh set state. |
| **Quy tắc an toàn** | Chỉ thao tác DOM không phá hủy (focus, scroll, đo). Không sửa cấu trúc DOM mà React đang quản lý. |
