# Phản Ứng với Input bằng State

> Hướng dẫn từ [react.dev/learn/reacting-to-input-with-state](https://react.dev/learn/reacting-to-input-with-state)

React cung cấp cách **khai báo** (declarative) để quản lý UI. Thay vì chỉ tay từng phần tử DOM, bạn mô tả các *trạng thái* mà component có thể ở trong — rồi để React tự lo việc cập nhật.

---

## Imperative vs Declarative — Sự khác biệt cốt lõi

### Imperative (mệnh lệnh) — "Làm từng bước này"

Giống như ngồi trong xe và chỉ đường từng khúc:
*"Rẽ trái, đi 200m, rẽ phải, dừng lại..."*

```js
// ❌ Imperative — JavaScript thuần, không dùng React
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);         // ← tắt textarea
  disable(button);           // ← tắt button
  show(loadingMessage);      // ← hiện loading
  hide(errorMessage);        // ← ẩn error
  try {
    await submitForm(textarea.value);
    show(successMessage);    // ← hiện thành công
    hide(form);              // ← ẩn form
  } catch (err) {
    show(errorMessage);      // ← hiện lỗi
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);    // ← ẩn loading
    enable(textarea);        // ← bật lại textarea
    enable(button);          // ← bật lại button
  }
}
```

App có nhiều form như thế này → code ngày càng phức tạp, dễ quên bật/tắt một thứ nào đó → bug khó tìm.

### Declarative (khai báo) — "Đưa tôi đến chỗ này"

Giống như đi taxi và nói địa chỉ đích:
*"Đưa tôi đến 123 Nguyễn Huệ"* — tài xế tự lo đường đi.

```jsx
// ✅ Declarative — React
// Bạn chỉ mô tả: "Khi status = 'submitting', hiển thị như này"
// React tự lo việc cập nhật DOM
<textarea disabled={status === 'submitting'} />
<button disabled={answer.length === 0 || status === 'submitting'}>
  Submit
</button>
{status === 'error' && <p>{error.message}</p>}
```

---

## 5 Bước thiết kế UI với State

Lấy ví dụ thực tế: **Form quiz** có các trạng thái: rỗng → đang nhập → đang gửi → thành công / lỗi.

---

### Bước 1: Xác định tất cả visual states

Trước khi viết logic, hãy liệt kê **tất cả trạng thái** mà UI có thể hiển thị:

```
Form quiz có 5 trạng thái:

┌─────────────┬───────────────────────────────────────────────────┐
│  State      │  Giao diện trông như thế nào                      │
├─────────────┼───────────────────────────────────────────────────┤
│  empty      │  Textarea rỗng, nút Submit bị disable             │
│  typing     │  Textarea có chữ, nút Submit được enable          │
│  submitting │  Toàn bộ form disabled, hiện spinner              │
│  success    │  Ẩn form, hiện "Chúc mừng bạn đúng rồi!"         │
│  error      │  Giống typing, thêm thông báo lỗi màu đỏ         │
└─────────────┴───────────────────────────────────────────────────┘
```

**Mẹo hay:** Mock từng trạng thái bằng prop tĩnh trước — không cần logic:

```jsx
// Mock để xem giao diện từng trạng thái — rất hữu ích khi làm việc với designer
export default function Form({ status = 'empty' }) {
  if (status === 'success') {
    return <h1>Chúc mừng bạn đúng rồi!</h1>;
  }
  return (
    <form>
      <textarea disabled={status === 'submitting'} />
      <button disabled={status === 'empty' || status === 'submitting'}>
        Gửi
      </button>
      {status === 'error' && <p className="error">Sai rồi, thử lại!</p>}
    </form>
  );
}

// Hiển thị tất cả trạng thái cùng lúc để kiểm tra (Living Styleguide)
const allStatuses = ['empty', 'typing', 'submitting', 'success', 'error'];

export default function App() {
  return (
    <>
      {allStatuses.map(status => (
        <section key={status}>
          <h4>Form ({status}):</h4>
          <Form status={status} />
        </section>
      ))}
    </>
  );
}
```

> 💡 Kỹ thuật hiển thị tất cả visual states cùng lúc gọi là **"living styleguide"** hoặc **"storybook"** — rất phổ biến trong các team lớn.

---

### Bước 2: Xác định điều gì kích hoạt thay đổi state

Có 2 loại trigger (tác nhân kích hoạt):

```
Trigger từ người dùng (Human inputs):
  • Gõ text vào input
  • Click nút
  • Chọn trong dropdown
  • Điều hướng link

Trigger từ máy tính (Computer inputs):
  • Network request thành công / thất bại
  • setTimeout hoàn thành
  • Ảnh tải xong
```

**Vẽ sơ đồ chuyển đổi state (State Machine):**

```
                    gõ text
    [empty] ─────────────────→ [typing]
                                   │
                              click Submit
                                   ↓
                            [submitting]
                           /             \
              network success         network error
                   ↓                       ↓
              [success]               [error]
                                          │
                                      gõ lại
                                          ↓
                                      [typing]
```

Vẽ sơ đồ này trước khi code giúp phát hiện bug sớm — ví dụ: từ `submitting` có thể chuyển sang đâu? Có bỏ sót trường hợp nào không?

---

### Bước 3: Biểu diễn state bằng `useState`

Liệt kê tất cả state cần thiết — **bắt đầu nhiều hơn rồi cắt bớt**:

```jsx
// Phiên bản ban đầu — nhiều state, có thể thừa
const [answer, setAnswer]         = useState('');
const [error, setError]           = useState(null);
const [isEmpty, setIsEmpty]       = useState(true);
const [isTyping, setIsTyping]     = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess]   = useState(false);
const [isError, setIsError]       = useState(false);
// ← 7 state variables — quá nhiều!
```

---

### Bước 4: Loại bỏ state không cần thiết

Đây là bước quan trọng nhất để tránh bug. Dùng 3 câu hỏi để lọc:

#### Câu hỏi 1: State này có tạo ra "paradox" không?

```
isTyping = true  VÀ  isSubmitting = true  → vô lý! Không thể vừa gõ vừa đang gửi.
isSuccess = true VÀ  isError = true       → vô lý! Không thể vừa thành công vừa lỗi.

→ Gộp thành một biến status duy nhất:
   status: 'typing' | 'submitting' | 'success'
   (loại bỏ được isTyping, isSubmitting, isSuccess)
```

#### Câu hỏi 2: State này đã có trong state khác chưa?

```
isEmpty = true  ↔  answer.length === 0
→ Hoàn toàn tính được từ 'answer'
→ Xóa isEmpty đi, dùng answer.length === 0 trực tiếp
```

#### Câu hỏi 3: State này tính được từ nghịch đảo của state khác không?

```
isError = true  ↔  error !== null
→ Tính được từ 'error'
→ Xóa isError đi, dùng error !== null trực tiếp
```

**Kết quả sau khi lọc: 7 → 3 state variables!**

```jsx
// ✅ Chỉ còn 3 state thực sự cần thiết
const [answer, setAnswer] = useState('');      // nội dung người dùng nhập
const [error, setError]   = useState(null);    // lỗi từ server (nếu có)
const [status, setStatus] = useState('typing'); // 'typing' | 'submitting' | 'success'
```

**Tại sao 3 cái này là "bất khả giảm"?**

```
Xóa 'answer'  → không biết người dùng nhập gì, không gửi được
Xóa 'error'   → không biết lỗi là gì để hiển thị
Xóa 'status'  → không biết đang ở trạng thái nào
→ Không thể xóa thêm cái nào nữa!
```

---

### Bước 5: Kết nối event handlers với state

```jsx
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError]   = useState(null);
  const [status, setStatus] = useState('typing');

  // Trạng thái success — hiện thông báo thay vì form
  if (status === 'success') {
    return <h1>Chúc mừng bạn đúng rồi!</h1>;
  }

  // Xử lý submit — chuyển đổi giữa các state
  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');   // → disable form, hiện spinner
    try {
      await submitForm(answer);
      setStatus('success');    // → ẩn form, hiện thành công
    } catch (err) {
      setStatus('typing');     // → bật lại form
      setError(err);           // → lưu lỗi để hiển thị
    }
  }

  // Xử lý typing — cập nhật nội dung input
  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        value={answer}
        onChange={handleTextareaChange}
        disabled={status === 'submitting'}
      />
      <button disabled={
        answer.length === 0 ||    // chưa nhập gì
        status === 'submitting'   // đang gửi
      }>
        Gửi
      </button>
      {error !== null && (
        <p className="error">{error.message}</p>
      )}
    </form>
  );
}

// Giả lập gọi API
function submitForm(answer) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() === 'lima') {
        resolve();
      } else {
        reject(new Error('Sai rồi, thử lại!'));
      }
    }, 1500);
  });
}
```

---

## So sánh Imperative vs Declarative — Cùng một tính năng

```jsx
// IMPERATIVE (JavaScript thuần) — 30 dòng, dễ bỏ sót
async function handleSubmit(e) {
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}
// Mỗi lần thêm tính năng → phải đọc lại toàn bộ, dễ quên bật/tắt thứ gì đó

// DECLARATIVE (React) — rõ ràng, dễ mở rộng
// Chỉ cần thay đổi state, React lo việc còn lại:
setStatus('submitting');   // → cả form tự disable
setStatus('success');      // → form ẩn, success hiện
setStatus('typing');       // → form bật lại
setError(err);             // → error message hiện
```

**Lợi ích của Declarative:**
- Thêm trạng thái mới (ví dụ: 'paused') không phá vỡ code cũ
- Đổi giao diện của một trạng thái không cần sửa logic
- Không bao giờ quên bật/tắt một phần tử nào đó

---

## Checklist loại bỏ state thừa

Dùng checklist này khi review state của bất kỳ component nào:

```
□ State này có tạo ra trạng thái "không thể xảy ra" cùng lúc với state khác không?
  → Nếu có: gộp chúng thành một biến enum (status: 'a' | 'b' | 'c')

□ State này có tính được từ state/props khác không?
  → Nếu có: xóa đi, tính trực tiếp khi dùng

□ State này có thể thay bằng sự kiện (event) không?
  → Nếu có: theo dõi event thay vì lưu thêm state

□ State này có trùng lặp thông tin từ state khác không?
  → Nếu có: xóa bản sao đi
```

---

## Tóm tắt 5 bước

```
Bước 1 — Liệt kê visual states:
  Tất cả trạng thái UI người dùng có thể thấy
  → Mock từng cái bằng prop tĩnh trước khi viết logic

Bước 2 — Xác định triggers:
  Human: click, gõ phím, điều hướng
  Computer: network, timeout, load
  → Vẽ sơ đồ chuyển đổi state (state machine diagram)

Bước 3 — useState:
  Bắt đầu với nhiều state hơn cần
  → Sẽ cắt bớt ở bước 4

Bước 4 — Loại bỏ state thừa:
  Xóa state gây paradox → gộp thành enum
  Xóa state tính được từ state khác
  Xóa state tính được từ nghịch đảo state khác

Bước 5 — Kết nối handlers:
  Mỗi event → setState đúng trạng thái
  JSX → đọc state, render tương ứng
```

---

## Bài tập tự luyện

### Bài 1: Thêm/xóa CSS class bằng state

```jsx
// Yêu cầu:
// - Click vào ảnh → xóa class 'background--active' khỏi div, thêm 'picture--active' vào img
// - Click ra ngoài → khôi phục lại ban đầu
export default function Picture() {
  return (
    <div className="background background--active">
      <img
        className="picture"
        alt="Ngôi nhà màu sắc"
        src="https://example.com/houses.jpg"
      />
    </div>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
import { useState } from 'react';

export default function Picture() {
  const [isActive, setIsActive] = useState(false);

  return (
    <div
      className={isActive ? 'background' : 'background background--active'}
      onClick={() => setIsActive(false)}
    >
      <img
        className={isActive ? 'picture picture--active' : 'picture'}
        alt="Ngôi nhà màu sắc"
        src="https://example.com/houses.jpg"
        onClick={e => {
          e.stopPropagation();  // ngăn event lan lên div
          setIsActive(true);
        }}
      />
    </div>
  );
}
```
</details>

---

### Bài 2: Tìm state thừa

```jsx
// Component này có state nào thừa không? Giải thích và sửa lại.
const [firstName, setFirstName] = useState('');
const [lastName, setLastName]   = useState('');
const [fullName, setFullName]   = useState('');  // ← nghi vấn

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
  setFullName(e.target.value + ' ' + lastName);
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
  setFullName(firstName + ' ' + e.target.value);
}
```

<details>
<summary>👉 Xem đáp án</summary>

`fullName` là state thừa — nó **tính được** từ `firstName` và `lastName`.

```jsx
// ✅ Xóa fullName state, tính trực tiếp
const [firstName, setFirstName] = useState('');
const [lastName, setLastName]   = useState('');

const fullName = firstName + ' ' + lastName;  // biến thường, không phải state

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
  // không cần setFullName nữa!
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
}

return <p>Xin chào, {fullName}!</p>;
```

Lợi ích: Không bao giờ bị out-of-sync giữa `firstName`, `lastName` và `fullName`.
</details>

---

### Bài 3: Thiết kế state cho Wizard (multi-step form)

```
Form đặt vé máy bay có 4 bước:
  Bước 1: Chọn điểm đi/đến
  Bước 2: Chọn ngày bay
  Bước 3: Chọn chỗ ngồi
  Bước 4: Thanh toán

Hỏi: Bạn cần những state gì?
```

<details>
<summary>👉 Xem gợi ý thiết kế</summary>

```jsx
// State cho Wizard form đặt vé
const [currentStep, setCurrentStep] = useState(1);  // 1 | 2 | 3 | 4
const [formData, setFormData] = useState({
  from: '',
  to: '',
  date: null,
  seat: null,
});
const [status, setStatus] = useState('filling');  // 'filling' | 'submitting' | 'success' | 'error'
const [error, setError]   = useState(null);

// Tại sao KHÔNG dùng:
// isStep1Done, isStep2Done... → tính được từ formData
// isLoading → tính được từ status === 'submitting'
// hasError  → tính được từ error !== null
// totalPrice → tính được từ formData (seat, date)
```
</details>
