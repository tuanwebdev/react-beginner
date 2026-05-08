# Xử lý Sự kiện trong React

> Hướng dẫn từ [react.dev/learn/responding-to-events](https://react.dev/learn/responding-to-events)

React cho phép bạn thêm **event handler** vào JSX — đây là những hàm sẽ được gọi khi người dùng tương tác: click, hover, gõ phím, submit form...

---

## Thêm Event Handler — 3 bước

```jsx
// Bước 1: Nút chưa làm gì cả
export default function Button() {
  return <button>Tôi chưa làm gì</button>;
}
```

**Để nút làm gì đó khi click, làm theo 3 bước:**

```jsx
export default function Button() {
  // Bước 1: Khai báo hàm xử lý bên trong component
  function handleClick() {
    // Bước 2: Viết logic vào trong hàm
    alert('Bạn đã nhấn tôi!');
  }

  return (
    // Bước 3: Gắn hàm vào sự kiện onClick
    <button onClick={handleClick}>
      Nhấn vào đây
    </button>
  );
}
```

**Quy ước đặt tên:** Event handler thường được đặt tên theo dạng `handle` + tên sự kiện:

| Sự kiện | Tên hàm gợi ý |
|---|---|
| Click | `handleClick` |
| Mouse enter | `handleMouseEnter` |
| Form submit | `handleSubmit` |
| Input change | `handleChange` |

---

## 3 cách viết Event Handler

Cả 3 cách đều hoạt động như nhau — tùy tình huống mà chọn:

```jsx
// Cách 1: Hàm riêng biệt (rõ ràng, dễ đọc — dùng khi logic phức tạp)
function handleClick() {
  alert('Clicked!');
}
<button onClick={handleClick}>Click</button>


// Cách 2: Hàm inline (tiện khi logic đơn giản)
<button onClick={function handleClick() {
  alert('Clicked!');
}}>Click</button>


// Cách 3: Arrow function inline (ngắn gọn nhất — phổ biến nhất)
<button onClick={() => alert('Clicked!')}>Click</button>
```

---

## ⚠️ Bẫy Cực Kỳ Phổ Biến: Truyền hàm vs Gọi hàm

Đây là lỗi **mà hầu hết người mới đều mắc phải ít nhất một lần:**

```jsx
// ✅ ĐÚNG — truyền hàm (React gọi khi user click)
<button onClick={handleClick}>

// ❌ SAI — gọi hàm ngay lập tức (chạy khi render, không phải khi click!)
<button onClick={handleClick()}>
```

**Tại sao lại vậy?** Hãy nghĩ về dấu `()`:

```
handleClick     →  bản thân hàm đó (như một "địa chỉ")
handleClick()   →  kết quả của việc CHẠY hàm đó ngay bây giờ
```

Khi bạn viết `onClick={handleClick()}`, React nhận được **kết quả** của hàm (thường là `undefined`), không phải bản thân hàm — nên sẽ không có gì xảy ra khi click.

**Với arrow function inline cũng vậy:**

```jsx
// ✅ ĐÚNG — truyền một hàm arrow
<button onClick={() => alert('Hi!')}>

// ❌ SAI — gọi alert() ngay khi render
<button onClick={alert('Hi!')}>
```

**Quy tắc đơn giản để nhớ:** Thứ bạn truyền vào `onClick={...}` **phải là một hàm**, không phải kết quả của hàm.

---

## Event Handler có thể đọc Props

Vì event handler được định nghĩa bên trong component, nó có thể truy cập props của component đó:

```jsx
// AlertButton nhận prop 'message' và dùng trong handler
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      {/* Mỗi nút hiển thị thông báo khác nhau */}
      <AlertButton message="Đang phát phim!">
        Phát phim
      </AlertButton>
      <AlertButton message="Đang tải lên!">
        Tải ảnh lên
      </AlertButton>
    </div>
  );
}
```

---

## Truyền Event Handler qua Props

Đôi khi bạn muốn **component cha quyết định** hành động của component con. Ví dụ: một component `Button` dùng chung, nhưng mỗi nơi dùng lại làm việc khác nhau.

```jsx
// Button: chỉ lo giao diện, không biết mình sẽ làm gì
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

// PlayButton: quyết định hành động khi click
function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Đang phát: ${movieName}!`);
  }
  return (
    <Button onClick={handlePlayClick}>
      ▶ Phát "{movieName}"
    </Button>
  );
}

// UploadButton: quyết định hành động khác
function UploadButton() {
  return (
    <Button onClick={() => alert('Đang tải lên!')}>
      ⬆ Tải ảnh lên
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="Kiki's Delivery Service" />
      <UploadButton />
    </div>
  );
}
```

**Luồng truyền handler:**
```
Toolbar
  └── PlayButton (định nghĩa handlePlayClick)
        └── Button (nhận onClick={handlePlayClick}, gắn vào <button>)
                └── <button onClick={handlePlayClick}> ← trình duyệt thực thi
```

---

## Đặt tên Props cho Event Handler

Với thẻ HTML thuần (`<button>`, `<div>`...) phải dùng tên sự kiện chuẩn như `onClick`, `onChange`.

Nhưng với **component tự tạo**, bạn đặt tên props tùy ý — **theo quy ước** nên bắt đầu bằng `on` + chữ hoa:

```jsx
// Component Toolbar nhận props tên tùy ý — bắt đầu bằng 'on'
function Toolbar({ onPlayMovie, onUploadImage }) {
  return (
    <div>
      <Button onClick={onPlayMovie}>Phát phim</Button>
      <Button onClick={onUploadImage}>Tải ảnh</Button>
    </div>
  );
}

// App dùng Toolbar — đặt tên rõ ràng theo nghĩa nghiệp vụ
export default function App() {
  return (
    <Toolbar
      onPlayMovie={() => alert('Đang phát!')}
      onUploadImage={() => alert('Đang tải lên!')}
    />
  );
}
```

Đặt tên theo nghiệp vụ (`onPlayMovie`) thay vì tên sự kiện (`onClick`) giúp code dễ hiểu hơn và linh hoạt hơn khi thay đổi sau này.

> 💡 **Luôn dùng thẻ HTML ngữ nghĩa đúng** khi xử lý sự kiện. Dùng `<button>` cho click thay vì `<div onClick>` — vì `<button>` hỗ trợ điều hướng bằng bàn phím và accessibility tốt hơn.

---

## Event Propagation (Sự kiện lan truyền)

Khi bạn click vào một phần tử, sự kiện không dừng lại ở đó — nó **nổi bọt lên** (bubble up) qua toàn bộ cây component từ dưới lên trên.

```jsx
export default function Toolbar() {
  return (
    <div
      className="toolbar"
      onClick={() => alert('Bạn click vào toolbar!')}  // ← handler của div cha
    >
      <button onClick={() => alert('Đang phát!')}>
        Phát phim
      </button>
      <button onClick={() => alert('Đang tải lên!')}>
        Tải ảnh
      </button>
    </div>
  );
}
```

**Điều gì xảy ra khi click nút "Phát phim"?**

```
Click nút "Phát phim"
  → alert('Đang phát!')          ← handler của button chạy trước
  → alert('Bạn click vào toolbar!')  ← handler của div cha chạy sau
```

Sự kiện **lan truyền từ dưới lên trên**, nghĩa là handler của con chạy trước, rồi đến cha.

> ⚠️ **Ngoại lệ:** `onScroll` là sự kiện DUY NHẤT trong React không lan truyền lên.

---

## Dừng Lan Truyền: `e.stopPropagation()`

Nếu bạn không muốn sự kiện lan lên cha, dùng `e.stopPropagation()`:

```jsx
function Button({ onClick, children }) {
  return (
    <button
      onClick={e => {
        e.stopPropagation();  // ← Dừng tại đây, không lan lên div cha
        onClick();            // ← Vẫn gọi handler được truyền vào
      }}
    >
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div onClick={() => alert('Click vào toolbar!')}>
      <Button onClick={() => alert('Đang phát!')}>
        Phát phim
      </Button>
    </div>
  );
}
// Kết quả: Click nút → chỉ alert 'Đang phát!' — toolbar không bị trigger
```

**Thứ tự thực thi:**

```
1. React gọi onClick handler của <button>
2. Handler gọi e.stopPropagation() → chặn lan truyền
3. Handler gọi onClick() → hiện alert 'Đang phát!'
4. div cha KHÔNG nhận được sự kiện → không chạy handler của nó
```

### Capture Phase — Bắt sự kiện từ trên xuống

Trong trường hợp hiếm (ví dụ: analytics), bạn cần bắt sự kiện **ngay cả khi con đã stopPropagation**. Dùng `Capture`:

```jsx
<div onClickCapture={() => {
  // Chạy đầu tiên, dù con có stopPropagation hay không
  logAnalytics('div_clicked');
}}>
  <button onClick={e => e.stopPropagation()}>
    Nút con
  </button>
</div>
```

**3 giai đoạn của một sự kiện:**

```
Giai đoạn 1 (Capture — đi xuống):  div ← onClickCapture chạy
Giai đoạn 2 (Target):               button ← onClick của button chạy
Giai đoạn 3 (Bubble — đi lên):     div ← onClick chạy (nếu không bị stop)
```

---

## Chặn Hành vi Mặc định: `e.preventDefault()`

Một số thẻ HTML có hành vi mặc định. Ví dụ phổ biến nhất: `<form>` tự **reload trang** khi submit.

```jsx
// ❌ Trang sẽ reload mỗi khi nhấn Send
export default function Signup() {
  return (
    <form onSubmit={() => alert('Đang gửi!')}>
      <input />
      <button>Gửi</button>
    </form>
  );
}

// ✅ Dùng e.preventDefault() để ngăn reload
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault();     // ← Ngăn reload trang
      alert('Đang gửi!');     // ← Xử lý theo ý mình
    }}>
      <input />
      <button>Gửi</button>
    </form>
  );
}
```

**Các hành vi mặc định thường gặp cần chặn:**

| Thẻ / Sự kiện | Hành vi mặc định | Khi nào cần `preventDefault()` |
|---|---|---|
| `<form onSubmit>` | Reload trang | Hầu như luôn luôn |
| `<a href onClick>` | Chuyển trang | Khi dùng SPA routing |
| `<input onKeyDown>` | Nhập ký tự | Khi cần chặn một số phím |
| `<input type="checkbox">` | Toggle check | Khi tự quản lý state |

---

## Phân biệt `stopPropagation` và `preventDefault`

Hai hàm này hay bị nhầm lẫn nhưng hoàn toàn khác nhau:

| | `e.stopPropagation()` | `e.preventDefault()` |
|---|---|---|
| **Làm gì** | Dừng sự kiện **lan lên** component cha | Chặn **hành vi mặc định** của trình duyệt |
| **Ảnh hưởng** | Cha không nhận được sự kiện | Trình duyệt không thực hiện hành động mặc định |
| **Dùng khi** | Không muốn cha bị trigger | Không muốn form reload, link chuyển trang... |
| **Ví dụ** | Click nút trong toolbar không trigger toolbar | Form submit không reload trang |

```jsx
// Dùng cả hai cùng lúc — hoàn toàn ổn
<form onSubmit={e => {
  e.preventDefault();     // Không reload trang
  e.stopPropagation();    // Không lan lên component cha
  handleSubmit();
}}>
```

---

## Event Handler có thể có Side Effect không?

**Hoàn toàn được!** Thực ra đây chính là nơi **lý tưởng nhất** để đặt side effect.

```
Hàm render (return JSX)  →  phải THUẦN TÚY, không side effect
Event handler            →  được phép có side effect ✅
```

```jsx
function Form() {
  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Tất cả những việc này đều được làm trong event handler:
    saveToDatabase(formData);      // gọi API
    sendEmail(userEmail);          // gửi email
    updateLocalStorage(userData);  // thao tác với storage
    setIsSubmitted(true);          // cập nhật state
  }

  return <form onSubmit={handleSubmit}>...</form>;
}
```

Để lưu lại thay đổi sau khi xử lý sự kiện, bạn cần **state** — sẽ học ở bài tiếp theo.

---

## Tóm tắt

```
1. Thêm handler     →  <button onClick={handleClick}>
                        Đặt tên: handle + TênSựKiện

2. Truyền hàm       →  onClick={handleClick}    ✅
   KHÔNG gọi hàm   →  onClick={handleClick()}   ❌

3. Viết inline      →  onClick={() => alert('Hi')}

4. Đọc props        →  handler bên trong component truy cập được props

5. Truyền handler   →  cha → con qua props (đặt tên on + TênSựKiện)
   qua props

6. Lan truyền       →  Sự kiện nổi bọt từ con lên cha
   Dừng lại         →  e.stopPropagation()

7. Chặn mặc định    →  e.preventDefault() (form reload, link chuyển trang...)

8. Side effect      →  event handler là nơi lý tưởng để có side effect
```

---

## Bài tập tự luyện

### Bài 1: Tìm lỗi Event Handler

```jsx
// ❌ Code này không hoạt động — tìm lỗi và sửa
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={handleClick()}>
      Bật/tắt đèn
    </button>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
// ✅ Bỏ dấu () sau handleClick
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={handleClick}>   {/* Bỏ () */}
      Bật/tắt đèn
    </button>
  );
}
```

**Lỗi:** `onClick={handleClick()}` gọi hàm ngay khi render (lúc trang load). Cần truyền tham chiếu hàm `onClick={handleClick}` để React gọi khi user click.
</details>

---

### Bài 2: Ngăn sự kiện lan truyền

```jsx
// Yêu cầu: Click vào nút không được trigger alert của div cha
export default function App() {
  return (
    <div onClick={() => alert('Div cha được click!')}>
      <button onClick={() => alert('Nút được click!')}>
        Nhấn tôi
      </button>
    </div>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
export default function App() {
  return (
    <div onClick={() => alert('Div cha được click!')}>
      <button onClick={e => {
        e.stopPropagation();         // Dừng lan truyền lên div cha
        alert('Nút được click!');
      }}>
        Nhấn tôi
      </button>
    </div>
  );
}
```
</details>

---

### Bài 3: Ngăn Form Reload

```jsx
// Yêu cầu: Submit form không được reload trang, chỉ hiện alert
export default function ContactForm() {
  return (
    <form onSubmit={() => alert('Đã gửi!')}>
      <input placeholder="Tên của bạn" />
      <button type="submit">Gửi</button>
    </form>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
export default function ContactForm() {
  return (
    <form onSubmit={e => {
      e.preventDefault();    // Ngăn reload trang
      alert('Đã gửi!');
    }}>
      <input placeholder="Tên của bạn" />
      <button type="submit">Gửi</button>
    </form>
  );
}
```
</details>