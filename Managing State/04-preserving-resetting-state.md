# Giữ và Reset State

> Hướng dẫn từ [react.dev/learn/preserving-and-resetting-state](https://react.dev/learn/preserving-and-resetting-state)

State được cô lập giữa các component. React theo dõi state thuộc về component nào dựa trên **vị trí của component đó trong cây UI** — không phải tên biến, không phải vị trí trong code JSX. Hiểu điều này giúp bạn kiểm soát khi nào state được giữ lại, khi nào bị reset.

---

## Nguyên tắc nền tảng: State gắn với vị trí trong cây

Bạn có thể nghĩ state "sống bên trong component", nhưng thực ra **React giữ state** — và liên kết nó với component dựa trên vị trí trong cây render.

```jsx
// Dù chỉ có 1 biến 'counter', render 2 lần → 2 instance độc lập
const counter = <Counter />;
return (
  <div>
    {counter}   {/* vị trí 1 → state riêng */}
    {counter}   {/* vị trí 2 → state riêng */}
  </div>
);
```

```
Cây render:
  div
  ├── Counter [score=0]   ← vị trí 1, state riêng
  └── Counter [score=5]   ← vị trí 2, state riêng (độc lập)
```

Hai Counter này hoàn toàn độc lập — dù được tạo từ cùng một biến JSX.

---

## Quy tắc 1: Cùng component, cùng vị trí → GIỮ state

**React giữ state miễn là cùng loại component ở cùng một vị trí trong cây.**

```jsx
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} />   // ← vẫn là Counter, vị trí 1
      ) : (
        <Counter isFancy={false} />  // ← vẫn là Counter, vị trí 1
      )}
    </div>
  );
}
```

Dù `isFancy` thay đổi, `Counter` vẫn ở **vị trí 1** trong cây → React coi đây là **cùng một Counter** → **state KHÔNG bị reset!**

```
Trước (isFancy=false):   div → Counter[score=3]
Sau  (isFancy=true):     div → Counter[score=3]  ← score vẫn còn!
```

> 💡 **Quan trọng:** React nhìn vào **cây output**, không phải code JSX của bạn. Với React, Counter ở vị trí 1 trong cả hai trường hợp → giống nhau.

---

## ⚠️ Bẫy: Vị trí trong cây ≠ Vị trí trong JSX

Đây là điều **gây ngạc nhiên nhất** trong bài:

```jsx
// Trông như 2 Counter khác nhau trong code, nhưng React thấy giống nhau!
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />   // ← vị trí 1 trong cây
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />    // ← vị trí 1 trong cây (CÙNG VỊ TRÍ!)
    </div>
  );
}
```

Dù code có 2 nhánh `if` riêng biệt, React thấy output trong cả 2 trường hợp đều là: `div → Counter[vị trí 1]`. **State vẫn được giữ!**

```
React "thấy":
  Trường hợp 1: div → Counter (vị trí 1)
  Trường hợp 2: div → Counter (vị trí 1)
  → Cùng vị trí → giữ state
```

---

## Quy tắc 2: Khác loại component, cùng vị trí → RESET state

Khi React thấy **loại component khác** ở cùng vị trí → xóa component cũ và toàn bộ state bên dưới:

```jsx
export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused
        ? <p>Hẹn gặp lại!</p>    // ← thẻ <p>
        : <Counter />             // ← component Counter
      }
    </div>
  );
}
```

```
isPaused=false:  div → Counter[score=5]
isPaused=true:   div → p              ← Counter bị XÓA, state mất
isPaused=false:  div → Counter[score=0] ← Counter MỚI, reset từ đầu
```

**Điều tương tự xảy ra khi thẻ bao bên ngoài thay đổi:**

```jsx
// ❌ Thay đổi thẻ bao → reset state của toàn bộ cây bên trong!
{isFancy
  ? <div><Counter /></div>       // ← div bao Counter
  : <section><Counter /></section>  // ← section bao Counter
}
```

```
Trước: div → div → Counter[score=5]
Sau:   div → section → Counter[score=0]  ← section ≠ div → reset toàn bộ!
```

> ⚠️ **Vì vậy:** Không bao giờ thay đổi thẻ bao HTML (div ↔ section ↔ article...) của một component nếu bạn muốn giữ state bên trong nó.

---

## ⚠️ Bẫy: Đừng định nghĩa component lồng nhau!

Đây là lý do **không được lồng định nghĩa component** — đây là hệ quả trực tiếp từ quy tắc trên:

```jsx
// ❌ BUG nghiêm trọng — MyTextField được định nghĩa bên trong MyComponent
export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {              // ← tạo hàm MỚI mỗi lần render!
    const [text, setText] = useState('');
    return <input value={text} onChange={e => setText(e.target.value)} />;
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => setCounter(counter + 1)}>
        Clicked {counter} times
      </button>
    </>
  );
}
// Mỗi lần click → MyComponent re-render → tạo hàm MyTextField MỚI
// → React thấy component KHÁC ở cùng vị trí → reset state input!
// → Text bạn vừa gõ bị XÓA mỗi lần click nút
```

```jsx
// ✅ Đúng — định nghĩa ngoài, ở top level
function MyTextField() {                // ← cùng hàm qua mọi render
  const [text, setText] = useState('');
  return <input value={text} onChange={e => setText(e.target.value)} />;
}

export default function MyComponent() {
  const [counter, setCounter] = useState(0);
  return (
    <>
      <MyTextField />  {/* ← cùng component type → giữ state */}
      <button onClick={() => setCounter(counter + 1)}>
        Clicked {counter} times
      </button>
    </>
  );
}
```

---

## Reset state tại cùng vị trí — 2 cách

Đôi khi bạn **muốn** reset state dù component ở cùng vị trí. Ví dụ: scoreboard cho 2 người chơi — chuyển người phải reset điểm.

```jsx
// ❌ Vấn đề — Counter cùng vị trí → state được giữ khi chuyển người
export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA
        ? <Counter person="Taylor" />   // ← vị trí 1
        : <Counter person="Sarah" />    // ← vị trí 1 (giống!)
      }
      <button onClick={() => setIsPlayerA(!isPlayerA)}>
        Người tiếp theo
      </button>
    </div>
  );
}
// Chuyển sang Sarah → Counter vẫn giữ điểm của Taylor → BUG!
```

### Cách 1: Render ở các vị trí khác nhau

```jsx
// ✅ Mỗi người một vị trí trong cây → state độc lập
export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA  && <Counter person="Taylor" />}  {/* vị trí 1 */}
      {!isPlayerA && <Counter person="Sarah" />}   {/* vị trí 2 */}
      <button onClick={() => setIsPlayerA(!isPlayerA)}>
        Người tiếp theo
      </button>
    </div>
  );
}
// Taylor ở vị trí 1, Sarah ở vị trí 2
// Chuyển người → Counter cũ bị xóa, Counter mới xuất hiện → state reset ✓
```

**Phù hợp khi:** Chỉ có vài component cần độc lập, không nhiều.

---

### Cách 2: Dùng `key` — cách phổ biến và linh hoạt hơn

`key` không chỉ dùng cho list! Bạn có thể dùng `key` để cho React biết đây là **một instance cụ thể**, không phải "Counter thứ nhất" hay "Counter thứ hai":

```jsx
// ✅ Mỗi người một key khác nhau → React tạo Counter hoàn toàn mới
export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA
        ? <Counter key="Taylor" person="Taylor" />  // ← key="Taylor"
        : <Counter key="Sarah"  person="Sarah" />   // ← key="Sarah"
      }
      <button onClick={() => setIsPlayerA(!isPlayerA)}>
        Người tiếp theo
      </button>
    </div>
  );
}
// key="Taylor" và key="Sarah" → React thấy đây là 2 Counter khác nhau
// Dù ở cùng vị trí → state vẫn bị reset khi chuyển ✓
```

**Cách React xử lý key:**

```
Thay vì dùng thứ tự (vị trí 1, vị trí 2...),
React dùng key như một "địa chỉ" nhận dạng:

key="Taylor" → Counter của Taylor → state của Taylor
key="Sarah"  → Counter của Sarah  → state của Sarah

Khi key thay đổi → React coi đây là component hoàn toàn mới → reset state
```

---

## Ứng dụng thực tế của key: Reset form chat

Đây là ví dụ quan trọng nhất — bug rất hay gặp trong thực tế:

```jsx
// ❌ BUG — chuyển người nhận nhưng text vẫn còn trong input
export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList onSelect={contact => setTo(contact)} />
      <Chat contact={to} />  {/* ← Chat cùng vị trí → state input được giữ! */}
    </div>
  );
}
// User gõ "Xin chào Taylor..." → chuyển sang Alice → text vẫn còn!
```

```jsx
// ✅ Dùng key để reset Chat khi chuyển người nhận
export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList onSelect={contact => setTo(contact)} />
      <Chat key={to.id} contact={to} />  {/* ← key thay đổi → Chat reset! */}
    </div>
  );
}
// Chuyển từ Taylor (id=0) sang Alice (id=1):
// → key đổi từ 0 sang 1
// → React tạo Chat mới hoàn toàn
// → input rỗng, đúng như mong đợi ✓
```

---

## Giữ state của component đã bị xóa

Đôi khi bạn muốn giữ state dù component tạm thời bị xóa — ví dụ: chat với nhiều người, mỗi người có draft riêng.

### 3 cách tiếp cận:

**Cách 1: Render tất cả, ẩn bằng CSS** — đơn giản nhưng tốn tài nguyên

```jsx
// Render cả 3 chat, ẩn 2 cái không active bằng CSS
return (
  <div>
    {contacts.map(contact => (
      <div
        key={contact.id}
        style={{ display: contact.id === to.id ? 'block' : 'none' }}
      >
        <Chat contact={contact} />
      </div>
    ))}
  </div>
);
// ✅ State luôn tồn tại (vì component không bị unmount)
// ❌ Chậm nếu có nhiều chat — tất cả đều được render
```

**Cách 2: Lift state lên cha** — React-idiomatic nhất

```jsx
// Lưu draft của từng người trong state của cha
export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  const [drafts, setDrafts] = useState({});  // { contactId: draftText }

  return (
    <div>
      <ContactList onSelect={contact => setTo(contact)} />
      <Chat
        key={to.id}           // vẫn reset component
        contact={to}
        draft={drafts[to.id] ?? ''}    // truyền draft xuống
        onDraftChange={text => setDrafts({ ...drafts, [to.id]: text })}
      />
    </div>
  );
}
// ✅ State sống ở cha → không bị mất khi Chat unmount
// ✅ Cách được React khuyến nghị
```

**Cách 3: Dùng external storage** — localStorage, Redux, Zustand...

```jsx
function Chat({ contact }) {
  const [text, setText] = useState(
    () => localStorage.getItem(`draft-${contact.id}`) ?? ''  // đọc từ storage
  );

  function handleChange(e) {
    setText(e.target.value);
    localStorage.setItem(`draft-${contact.id}`, e.target.value);  // lưu vào storage
  }

  return <textarea value={text} onChange={handleChange} />;
}
// ✅ State tồn tại kể cả khi reload trang
// ❌ Phức tạp hơn, cần đồng bộ cẩn thận
```

---

## Tổng kết: Sơ đồ quyết định

```
React có RESET state không?

Cùng loại component?
  ├── CÓ → Cùng vị trí trong cây?
  │         ├── CÓ → Cùng key?
  │         │         ├── CÓ (hoặc không có key) → GIỮ STATE ✓
  │         │         └── KHÁC → RESET STATE ✗
  │         └── KHÁC → RESET STATE ✗
  └── KHÁC → RESET STATE ✗

Khi component bị unmount (xóa khỏi cây):
  → STATE BỊ XÓA NGAY LẬP TỨC
  → Khi mount lại → STATE MỚI HOÀN TOÀN (từ initialValue)
```

---

## Bảng tóm tắt nhanh

| Tình huống | Kết quả |
|---|---|
| Cùng component, cùng vị trí, cùng key | ✅ **GIỮ** state |
| Cùng component, cùng vị trí, **khác key** | 🔄 **RESET** state |
| **Khác component**, cùng vị trí | 🔄 **RESET** state + toàn bộ cây con |
| Component bị **unmount** (xóa khỏi cây) | 💥 **XÓA** state hoàn toàn |
| Component được **mount lại** | 🆕 State **mới từ đầu** |
| Thẻ bao HTML thay đổi (div→section) | 🔄 **RESET** toàn bộ cây bên trong |

---

## Bài tập tự luyện

### Bài 1: Tại sao input bị reset mỗi lần click?

```jsx
export default function App() {
  const [count, setCount] = useState(0);

  function Field() {   // ← Tìm lỗi ở đây
    const [text, setText] = useState('');
    return <input value={text} onChange={e => setText(e.target.value)} />;
  }

  return (
    <>
      <Field />
      <button onClick={() => setCount(count + 1)}>
        Clicked {count} times
      </button>
    </>
  );
}
```

<details>
<summary>👉 Xem giải thích và đáp án</summary>

**Nguyên nhân:** `Field` được định nghĩa **bên trong** `App`. Mỗi lần `App` re-render (khi click), JavaScript tạo ra một hàm `Field` **mới hoàn toàn**. React thấy component khác ở vị trí đó → reset state.

```jsx
// ✅ Đúng — định nghĩa Field ở top level, ngoài App
function Field() {
  const [text, setText] = useState('');
  return <input value={text} onChange={e => setText(e.target.value)} />;
}

export default function App() {
  const [count, setCount] = useState(0);
  return (
    <>
      <Field />
      <button onClick={() => setCount(count + 1)}>
        Clicked {count} times
      </button>
    </>
  );
}
```
</details>

---

### Bài 2: Chuyển tab không reset form

```jsx
// Yêu cầu: Chuyển qua lại giữa các tab nhưng giữ lại nội dung form đã nhập
export default function TabContainer() {
  const [tab, setTab] = useState('form');
  return (
    <>
      <button onClick={() => setTab('form')}>Form</button>
      <button onClick={() => setTab('preview')}>Preview</button>
      {tab === 'form' ? <ContactForm /> : <Preview />}
    </>
  );
}
// Vấn đề: Chuyển sang Preview rồi quay lại Form → form bị reset!
```

<details>
<summary>👉 Xem đáp án</summary>

**Cách 1: Render cả hai, ẩn bằng CSS**

```jsx
export default function TabContainer() {
  const [tab, setTab] = useState('form');
  return (
    <>
      <button onClick={() => setTab('form')}>Form</button>
      <button onClick={() => setTab('preview')}>Preview</button>
      <div style={{ display: tab === 'form' ? 'block' : 'none' }}>
        <ContactForm />
      </div>
      <div style={{ display: tab === 'preview' ? 'block' : 'none' }}>
        <Preview />
      </div>
    </>
  );
}
```

**Cách 2: Lift state lên cha**

```jsx
export default function TabContainer() {
  const [tab, setTab]       = useState('form');
  const [formData, setFormData] = useState({ name: '', email: '' });

  return (
    <>
      <button onClick={() => setTab('form')}>Form</button>
      <button onClick={() => setTab('preview')}>Preview</button>
      {tab === 'form'
        ? <ContactForm data={formData} onChange={setFormData} />
        : <Preview data={formData} />
      }
    </>
  );
}
```
</details>

---

### Bài 3: Nhận biết kết quả

```jsx
// score hiện tại = 5. Tick checkbox → score = ?
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy
        ? <section><Counter /></section>
        : <div><Counter /></div>
      }
      <input type="checkbox" onChange={e => setIsFancy(e.target.checked)} />
    </div>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

**Score = 0** — bị reset!

Khi `isFancy` thay đổi, thẻ bao thay đổi từ `<div>` sang `<section>`. React thấy `section ≠ div` → xóa toàn bộ cây bên trong (bao gồm Counter) → tạo lại từ đầu → score = 0.

Để giữ score, cần đảm bảo thẻ bao giống nhau:

```jsx
// ✅ Cùng thẻ bao → giữ state
{isFancy
  ? <div><Counter isFancy={true} /></div>
  : <div><Counter isFancy={false} /></div>
}
```
</details>
