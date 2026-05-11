# Chia Sẻ State Giữa Các Component

> Hướng dẫn từ [react.dev/learn/sharing-state-between-components](https://react.dev/learn/sharing-state-between-components)

Đôi khi bạn muốn state của hai component luôn thay đổi cùng nhau. Giải pháp: **đưa state lên component cha chung** rồi truyền xuống qua props. Kỹ thuật này gọi là **Lifting State Up** — một trong những thao tác phổ biến nhất khi viết React.

---

## Vấn đề: Hai component cần chia sẻ state

### Tình huống ban đầu — State độc lập

Mỗi `Panel` tự quản lý state `isActive` của mình → chúng hoàn toàn độc lập, mở/đóng riêng biệt:

```jsx
function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);  // ← state nằm trong con
  return (
    <section>
      <h3>{title}</h3>
      {isActive
        ? <p>{children}</p>
        : <button onClick={() => setIsActive(true)}>Hiện</button>
      }
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <Panel title="Giới thiệu">Nội dung A...</Panel>
      <Panel title="Lịch sử">Nội dung B...</Panel>
    </>
  );
}
```

```
Accordion
├── Panel [isActive=false]  ← state riêng
└── Panel [isActive=false]  ← state riêng (độc lập)

→ Mở Panel 1 KHÔNG ảnh hưởng Panel 2
```

**Yêu cầu mới:** Chỉ được mở một panel tại một thời điểm. Mở panel 2 → panel 1 phải tự đóng lại.

Hai panel **không thể tự đồng bộ** với nhau vì state nằm tách biệt ở mỗi con. Giải pháp duy nhất: đưa state lên cha.

---

## Giải pháp: Lifting State Up — 3 bước

### Bước 1: Xóa state khỏi component con

```jsx
// Trước
function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);  // ← xóa dòng này
  ...
}

// Sau — nhận isActive và onShow từ cha qua props
function Panel({ title, children, isActive, onShow }) {
  return (
    <section>
      <h3>{title}</h3>
      {isActive
        ? <p>{children}</p>
        : <button onClick={onShow}>Hiện</button>  // gọi handler từ cha
      }
    </section>
  );
}
```

Panel giờ không còn tự quyết định trạng thái của mình — nó hoàn toàn phụ thuộc vào cha.

---

### Bước 2 & 3: Thêm state vào component cha chung + truyền xuống

Tìm **component cha chung gần nhất** của tất cả component cần chia sẻ state — đó là `Accordion`.

```jsx
import { useState } from 'react';

export default function Accordion() {
  // State chuyển lên đây — một nguồn sự thật duy nhất
  const [activeIndex, setActiveIndex] = useState(0);
  // activeIndex=0 → Panel 1 mở; activeIndex=1 → Panel 2 mở

  return (
    <>
      <Panel
        title="Giới thiệu"
        isActive={activeIndex === 0}          // tính từ state cha
        onShow={() => setActiveIndex(0)}      // handler thay đổi state cha
      >
        Almaty là thành phố lớn nhất Kazakhstan...
      </Panel>
      <Panel
        title="Lịch sử"
        isActive={activeIndex === 1}          // tính từ state cha
        onShow={() => setActiveIndex(1)}      // handler thay đổi state cha
      >
        Tên thành phố xuất phát từ tiếng Kazakh...
      </Panel>
    </>
  );
}
```

**Sơ đồ luồng dữ liệu:**

```
Accordion (state: activeIndex = 0)
    │
    ├──→ Panel 1: isActive={0===0}=true,  onShow={() => setActiveIndex(0)}
    │              ↑ ĐANG MỞ
    │
    └──→ Panel 2: isActive={0===1}=false, onShow={() => setActiveIndex(1)}
                   ↑ đang đóng

User click "Hiện" ở Panel 2:
  Panel 2 gọi onShow()
    → setActiveIndex(1)
    → Accordion re-render với activeIndex=1
    → Panel 1: isActive={1===0}=false  ← tự đóng!
    → Panel 2: isActive={1===1}=true   ← mở ra!
```

---

## Tại sao đặt tên state thay đổi?

Lưu ý quan trọng: khi lift state up, **bản chất của state thường thay đổi**:

```
Trước (trong con): boolean isActive — "panel này có đang mở không?"
Sau (trong cha):   number activeIndex — "panel nào đang mở?"

Boolean → Enum/Number vì cha phải biết NHIỀU hơn, không chỉ một panel
```

---

## Controlled vs Uncontrolled Component

Đây là hai khái niệm quan trọng xuất hiện từ kỹ thuật lifting state:

### Uncontrolled Component — Tự quản lý

```jsx
// Panel ban đầu — không ai kiểm soát được từ bên ngoài
function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);  // ← tự quyết định
  ...
}

// Dùng: đơn giản, ít config
<Panel title="A">...</Panel>  // cha không kiểm soát được isActive
```

✅ Ưu điểm: Dễ dùng, ít props phải truyền

❌ Nhược điểm: Không thể phối hợp với component khác

---

### Controlled Component — Cha kiểm soát

```jsx
// Panel sau khi lift — cha hoàn toàn kiểm soát
function Panel({ title, children, isActive, onShow }) {
  // không có state riêng — hoàn toàn phụ thuộc props
  ...
}

// Dùng: cha phải truyền đủ props
<Panel
  title="A"
  isActive={activeIndex === 0}  // cha quyết định
  onShow={() => setActiveIndex(0)}
>...</Panel>
```

✅ Ưu điểm: Linh hoạt tối đa, có thể phối hợp nhiều component

❌ Nhược điểm: Cha phải cấu hình đầy đủ, nhiều props hơn

---

### So sánh nhanh

| | Uncontrolled | Controlled |
|---|---|---|
| **State ở đâu** | Trong component con | Cha truyền qua props |
| **Ai quyết định** | Bản thân component | Component cha |
| **Độ linh hoạt** | Thấp | Cao |
| **Dễ dùng** | Cao (ít config) | Thấp (nhiều props) |
| **Phối hợp** | Không được | Được |
| **Ví dụ** | `<input>` quản lý value riêng | `<input value={x} onChange={fn}>` |

> 💡 Trong thực tế, một component thường vừa có state local (uncontrolled) vừa nhận props (controlled) — không cần chọn một trong hai tuyệt đối.

---

## Single Source of Truth — Một nguồn sự thật duy nhất

Nguyên tắc cốt lõi: **mỗi piece of state chỉ "sống" ở một nơi duy nhất** trong cây component.

```
App
├── Header
│   └── UserAvatar         ← đọc user từ props
├── Sidebar
│   └── UserProfile        ← đọc user từ props
└── MainContent
    └── UserSettings       ← đọc user từ props

↑ Tất cả đọc user từ App — App là nguồn sự thật duy nhất cho user state
```

**Không** có nghĩa là tất cả state phải sống ở một nơi — mà là mỗi piece of state có một "chủ sở hữu" rõ ràng:

```
App              ← sở hữu: currentUser, theme
  Header         ← sở hữu: isMenuOpen
  Sidebar        ← sở hữu: (không có state riêng)
  Accordion      ← sở hữu: activeIndex
    Panel        ← sở hữu: (không có state riêng)
    Panel        ← sở hữu: (không có state riêng)
  TodoList       ← sở hữu: todos, filter
    TodoItem     ← sở hữu: isEditing (chỉ local UI)
```

State nên sống **thấp nhất có thể** trong cây, nhưng **đủ cao** để tất cả component cần dùng đều nhận được.

---

## Ví dụ thực tế thêm: Tab navigation

```jsx
import { useState } from 'react';

// Tab component — controlled
function Tab({ label, isActive, onClick }) {
  return (
    <button
      onClick={onClick}
      style={{ fontWeight: isActive ? 'bold' : 'normal' }}
    >
      {label}
    </button>
  );
}

// TabPanel — controlled
function TabPanel({ children, isActive }) {
  if (!isActive) return null;
  return <div className="tab-panel">{children}</div>;
}

// TabGroup — sở hữu state, kiểm soát tất cả
export default function TabGroup() {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div>
      {/* Tab headers */}
      <div className="tab-bar">
        <Tab label="Hồ sơ"   isActive={activeTab === 0} onClick={() => setActiveTab(0)} />
        <Tab label="Bài viết" isActive={activeTab === 1} onClick={() => setActiveTab(1)} />
        <Tab label="Cài đặt" isActive={activeTab === 2} onClick={() => setActiveTab(2)} />
      </div>

      {/* Tab contents */}
      <TabPanel isActive={activeTab === 0}>Nội dung hồ sơ...</TabPanel>
      <TabPanel isActive={activeTab === 1}>Danh sách bài viết...</TabPanel>
      <TabPanel isActive={activeTab === 2}>Tùy chọn cài đặt...</TabPanel>
    </div>
  );
}
```

---

## Quy trình tổng quát: Khi nào lift state up?

```
Dấu hiệu cần lift state up:
  □ Hai component cần hiển thị cùng một dữ liệu
  □ Thay đổi ở component A cần ảnh hưởng component B
  □ Chỉ một trong nhiều component cùng loại được active tại một thời điểm

Cách thực hiện:
  1. Tìm component cha chung gần nhất của tất cả component liên quan
  2. Xóa state khỏi các component con
  3. Thêm state vào cha
  4. Truyền state xuống con qua props
  5. Truyền handler (setter) xuống con qua props để con có thể cập nhật cha
```

---

## Bài tập tự luyện

### Bài 1: Đồng bộ hai ô input

Hai input hiện đang độc lập. Hãy lift state lên để khi gõ vào một ô, ô kia cũng cập nhật theo:

```jsx
// Hiện tại — mỗi input có state riêng, không đồng bộ
function Input({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}: <input value={text} onChange={e => setText(e.target.value)} />
    </label>
  );
}

export default function SyncedInputs() {
  return (
    <>
      <Input label="Input 1" />
      <Input label="Input 2" />
    </>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
import { useState } from 'react';

// Input giờ là controlled — nhận value và onChange từ cha
function Input({ label, value, onChange }) {
  return (
    <label>
      {label}: <input value={value} onChange={onChange} />
    </label>
  );
}

export default function SyncedInputs() {
  // State được lift lên cha
  const [text, setText] = useState('');

  return (
    <>
      <Input
        label="Input 1"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <Input
        label="Input 2"
        value={text}
        onChange={e => setText(e.target.value)}
      />
    </>
  );
}
```

Cả hai input dùng cùng `text` state → luôn đồng bộ.
</details>

---

### Bài 2: FilterableList

```jsx
// SearchBar và ProductList cần chia sẻ filterText
// Hãy thiết kế lại để chúng đồng bộ

function SearchBar() {
  const [filterText, setFilterText] = useState('');
  return <input value={filterText} onChange={e => setFilterText(e.target.value)} />;
}

function ProductList() {
  // Cần dùng filterText nhưng không có!
  return <ul>...</ul>;
}

export default function App() {
  return (
    <>
      <SearchBar />
      <ProductList />
    </>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
import { useState } from 'react';

// SearchBar — controlled
function SearchBar({ filterText, onFilterChange }) {
  return (
    <input
      value={filterText}
      onChange={e => onFilterChange(e.target.value)}
      placeholder="Tìm sản phẩm..."
    />
  );
}

// ProductList — controlled
function ProductList({ filterText, products }) {
  const filtered = products.filter(p =>
    p.name.toLowerCase().includes(filterText.toLowerCase())
  );
  return (
    <ul>
      {filtered.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}

// App — sở hữu filterText state
export default function App() {
  const [filterText, setFilterText] = useState('');
  const products = [
    { id: 1, name: 'Táo' },
    { id: 2, name: 'Chuối' },
    { id: 3, name: 'Cam' },
  ];

  return (
    <>
      <SearchBar
        filterText={filterText}
        onFilterChange={setFilterText}
      />
      <ProductList
        filterText={filterText}
        products={products}
      />
    </>
  );
}
```
</details>

---

### Bài 3: Tìm vị trí đúng cho state

```
Bạn có cây component sau:
  App
  ├── NavBar
  │   └── ThemeToggle    ← nút bật/tắt dark mode
  ├── Sidebar
  │   └── UserCard       ← hiển thị avatar theo theme
  └── MainContent
      └── Article        ← hiển thị text theo theme

Hỏi: State `isDarkMode` nên đặt ở đâu?
```

<details>
<summary>👉 Xem đáp án</summary>

**State `isDarkMode` nên đặt ở `App`** — đây là component cha chung gần nhất của tất cả component cần dùng theme (NavBar, Sidebar, MainContent).

```jsx
export default function App() {
  const [isDarkMode, setIsDarkMode] = useState(false);

  return (
    <div className={isDarkMode ? 'dark' : 'light'}>
      <NavBar isDarkMode={isDarkMode} onToggle={() => setIsDarkMode(!isDarkMode)} />
      <Sidebar isDarkMode={isDarkMode} />
      <MainContent isDarkMode={isDarkMode} />
    </div>
  );
}
```

> Trong thực tế với app lớn, state như `isDarkMode` thường được đặt trong Context để tránh prop drilling — sẽ học ở bài [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context).
</details>

---

## Tóm tắt

```
Lifting State Up:
  1. Xác định: component nào cần chia sẻ state?
  2. Tìm: component cha chung gần nhất của chúng
  3. Chuyển: xóa state khỏi con, thêm vào cha
  4. Truyền: state xuống qua props, handler (setter) xuống qua props

Controlled vs Uncontrolled:
  Controlled  = props điều khiển → linh hoạt, phối hợp được
  Uncontrolled = state local → dễ dùng, không phối hợp được

Single Source of Truth:
  Mỗi piece of state có MỘT component "sở hữu" duy nhất
  Không có nghĩa tất cả state ở một chỗ
  → Đặt state thấp nhất có thể, nhưng đủ cao để chia sẻ được
```

---