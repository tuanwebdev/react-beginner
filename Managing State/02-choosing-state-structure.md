# Chọn Cấu Trúc State

> Hướng dẫn từ [react.dev/learn/choosing-the-state-structure](https://react.dev/learn/choosing-the-state-structure)

Cấu trúc state tốt tạo ra sự khác biệt lớn: component dễ sửa, dễ debug — hay là nguồn bug bất tận. Bài này trình bày **5 nguyên tắc** để thiết kế state đúng cách.

---

## Tổng quan 5 nguyên tắc

| # | Nguyên tắc | Mục đích |
|---|---|---|
| 1 | **Gom state liên quan** | Tránh quên cập nhật đồng bộ |
| 2 | **Tránh mâu thuẫn trong state** | Loại bỏ trạng thái "không thể xảy ra" |
| 3 | **Tránh state thừa** | State tính được từ state/props khác → không lưu |
| 4 | **Tránh trùng lặp state** | Một nguồn sự thật duy nhất |
| 5 | **Tránh state lồng quá sâu** | Cập nhật dễ hơn khi state phẳng |

> 💡 Mục tiêu cuối cùng: **state dễ cập nhật mà không gây bug.** Giống như database engineer "normalize" database để giảm lỗi.

---

## Nguyên tắc 1: Gom state liên quan

**Khi hai state luôn thay đổi cùng lúc → gom thành một.**

```jsx
// ❌ Tách rời — phải nhớ cập nhật cả hai cùng lúc
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// Khi di chuyển chuột: phải gọi cả setX lẫn setY
// Nếu quên một cái → x và y bị lệch nhau → bug!
onPointerMove={e => {
  setX(e.clientX);
  setY(e.clientY);  // ← dễ quên cái này
}}

// ✅ Gom lại — một lần cập nhật, không bao giờ lệch
const [position, setPosition] = useState({ x: 0, y: 0 });

onPointerMove={e => {
  setPosition({ x: e.clientX, y: e.clientY });  // cập nhật một lần
}}
```

**Khi nào nên gom:**

```
✅ Gom khi:
  • Hai state luôn thay đổi CÙNG LÚC (x và y của tọa độ)
  • Không biết trước cần bao nhiêu state (form có trường động)

✅ Giữ riêng khi:
  • Hai state thay đổi ĐỘC LẬP (tab đang mở và text đang nhập)
```

> ⚠️ **Lưu ý khi dùng object state:** Không thể cập nhật chỉ một trường mà không copy các trường khác.
>
> ```jsx
> // ❌ Sai — mất trường y!
> setPosition({ x: 100 });
>
> // ✅ Đúng — giữ lại y bằng spread
> setPosition({ ...position, x: 100 });
> ```

---

## Nguyên tắc 2: Tránh mâu thuẫn trong state

**Khi hai state boolean có thể cùng `true` một lúc → trạng thái vô lý có thể xảy ra.**

```jsx
// ❌ Có thể gây bug: isSending=true VÀ isSent=true cùng lúc?
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent]       = useState(false);

async function handleSubmit(e) {
  setIsSending(true);
  await sendMessage(text);
  setIsSending(false);
  setIsSent(true);       // ← Nếu quên dòng này, state sẽ sai
}
// Nếu ai đó refactor và quên một setIsSending(false)
// → isSending=true và isSent=true cùng tồn tại → UI hiển thị sai
```

```jsx
// ✅ Một biến status duy nhất — không thể mâu thuẫn
const [status, setStatus] = useState('typing'); // 'typing'|'sending'|'sent'

async function handleSubmit(e) {
  setStatus('sending');
  await sendMessage(text);
  setStatus('sent');      // ← một lần set, không bao giờ mâu thuẫn
}

// Vẫn có thể dùng tên gợi nhớ — nhưng là biến thường, không phải state
const isSending = status === 'sending';
const isSent    = status === 'sent';
```

**Quy tắc:** Nếu hai trạng thái **không thể cùng true một lúc** → gộp thành một enum.

```
isSending=true  +  isSent=true   → IMPOSSIBLE STATE → gộp thành status
isTyping=true   +  isSubmitting=true → IMPOSSIBLE STATE → gộp thành status
isSuccess=true  +  isError=true  → IMPOSSIBLE STATE → gộp thành status
```

---

## Nguyên tắc 3: Tránh state thừa (Redundant State)

**Nếu một giá trị có thể tính từ state/props khác trong lúc render → không cần lưu vào state.**

### Ví dụ 1: fullName tính được từ firstName + lastName

```jsx
// ❌ fullName là state thừa
const [firstName, setFirstName] = useState('');
const [lastName, setLastName]   = useState('');
const [fullName, setFullName]   = useState('');  // ← thừa!

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
  setFullName(e.target.value + ' ' + lastName);  // phải nhớ cập nhật cả đây
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
  setFullName(firstName + ' ' + e.target.value); // phải nhớ cập nhật cả đây
}
// Nếu quên một cái → fullName bị stale (lỗi thời) → bug!
```

```jsx
// ✅ fullName là biến thường, tính trong lúc render
const [firstName, setFirstName] = useState('');
const [lastName, setLastName]   = useState('');

const fullName = firstName + ' ' + lastName;  // tự động luôn đúng

function handleFirstNameChange(e) { setFirstName(e.target.value); }  // ngắn gọn hơn!
function handleLastNameChange(e)  { setLastName(e.target.value); }
```

### Ví dụ 2: Các giá trị tính được khác

```jsx
// ❌ State thừa
const [items, setItems]       = useState([...]);
const [itemCount, setCount]   = useState(0);    // tính được từ items.length
const [totalPrice, setTotal]  = useState(0);    // tính được từ items.reduce(...)
const [isEmpty, setIsEmpty]   = useState(true); // tính được từ items.length === 0

// ✅ Tính trong render — luôn đồng bộ
const [items, setItems] = useState([...]);
const itemCount  = items.length;
const totalPrice = items.reduce((sum, item) => sum + item.price, 0);
const isEmpty    = items.length === 0;
```

### ⚠️ Đừng copy props vào state (Don't mirror props)

Đây là lỗi **rất phổ biến với người mới**:

```jsx
// ❌ Sao chép prop vào state — sẽ không cập nhật khi prop thay đổi!
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
  // Nếu parent đổi messageColor='red' → color state vẫn là giá trị cũ!
  // state chỉ được khởi tạo 1 lần từ prop ban đầu
  return <p style={{ color }}> ... </p>;
}

// ✅ Dùng prop trực tiếp
function Message({ messageColor }) {
  return <p style={{ color: messageColor }}> ... </p>;
}
```

**Ngoại lệ duy nhất được phép:** Bạn **cố ý** muốn bỏ qua tất cả update của prop (chỉ lấy giá trị ban đầu). Khi đó đặt tên prop với tiền tố `initial` hoặc `default` để báo hiệu rõ:

```jsx
// ✅ Chấp nhận được — cố ý chỉ lấy giá trị khởi tạo
function ColorPicker({ initialColor }) {
  const [color, setColor] = useState(initialColor);
  // Các lần parent re-render với initialColor mới → bị bỏ qua (cố ý)
  return <input value={color} onChange={e => setColor(e.target.value)} />;
}
```

---

## Nguyên tắc 4: Tránh trùng lặp state (Duplication)

**Lưu cùng dữ liệu ở nhiều nơi → khó giữ đồng bộ → bug.**

### Ví dụ: selectedItem bị trùng lặp với items

```jsx
// ❌ selectedItem là bản sao của một phần tử trong items
const [items, setItems]               = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(items[0]); // ← trùng lặp!

// Bug: Khi edit tên item trong items → selectedItem không tự cập nhật theo!
function handleItemChange(id, e) {
  setItems(items.map(item =>
    item.id === id ? { ...item, title: e.target.value } : item
  ));
  // Quên cập nhật selectedItem → hiển thị tên cũ ở "Bạn đã chọn: ..."
}
```

```jsx
// ✅ Chỉ lưu selectedId — tính selectedItem từ items và id
const [items, setItems]       = useState(initialItems);
const [selectedId, setSelectedId] = useState(0); // ← chỉ lưu ID

// selectedItem tự động luôn đúng vì tính từ nguồn gốc
const selectedItem = items.find(item => item.id === selectedId);

function handleItemChange(id, e) {
  setItems(items.map(item =>
    item.id === id ? { ...item, title: e.target.value } : item
  ));
  // selectedItem tự cập nhật vì nó tính từ items mới nhất!
}
```

**Trước và sau:**

```
TRƯỚC (trùng lặp):
  items       = [{ id: 0, title: 'pretzels' }, ...]
  selectedItem = { id: 0, title: 'pretzels' }  ← copy của items[0]
  → Khi đổi title trong items, selectedItem vẫn giữ title cũ → DESYNC

SAU (chỉ lưu ID):
  items      = [{ id: 0, title: 'pretzels' }, ...]
  selectedId = 0
  selectedItem = items.find(i => i.id === 0)   ← tính tự động, luôn đúng
```

**Nguyên tắc tổng quát:** Khi lưu selection, hãy lưu **ID** thay vì toàn bộ object.

```jsx
// ✅ Chỉ lưu ID/index
const [selectedUserId, setSelectedUserId] = useState(null);
const [activeTabIndex, setActiveTabIndex] = useState(0);
const [checkedIds, setCheckedIds]         = useState(new Set());

// Tính object từ ID khi cần
const selectedUser = users.find(u => u.id === selectedUserId);
const activeTab    = tabs[activeTabIndex];
```

---

## Nguyên tắc 5: Tránh state lồng quá sâu

**State lồng nhiều tầng rất khó cập nhật** — phải spread từ vị trí thay đổi tất cả đường lên tới gốc.

### Vấn đề: Cấu trúc cây lồng nhau

```jsx
// ❌ State lồng nhiều tầng — khó cập nhật
const travelPlan = {
  id: 0,
  title: 'Earth',
  childPlaces: [
    {
      id: 1,
      title: 'Africa',
      childPlaces: [
        { id: 2, title: 'Botswana', childPlaces: [] },
        { id: 3, title: 'Egypt',    childPlaces: [] },
      ]
    },
    {
      id: 4,
      title: 'Asia',
      childPlaces: [
        { id: 5, title: 'Vietnam',  childPlaces: [] },
        { id: 6, title: 'Thailand', childPlaces: [] },
      ]
    }
  ]
};

// Để xóa 'Egypt' (id=3), phải spread từ Earth → Africa → lọc Egypt
// Code cực kỳ dài và dễ sai!
```

### Giải pháp: Làm phẳng (Normalize / Flatten)

Thay vì lồng object vào nhau, lưu mỗi node theo ID và chỉ giữ mảng ID con:

```jsx
// ✅ Cấu trúc phẳng — giống database table
const travelPlan = {
  0: { id: 0, title: 'Earth',    childIds: [1, 4] },
  1: { id: 1, title: 'Africa',   childIds: [2, 3] },
  2: { id: 2, title: 'Botswana', childIds: [] },
  3: { id: 3, title: 'Egypt',    childIds: [] },
  4: { id: 4, title: 'Asia',     childIds: [5, 6] },
  5: { id: 5, title: 'Vietnam',  childIds: [] },
  6: { id: 6, title: 'Thailand', childIds: [] },
};

// Xóa 'Egypt' (id=3) — chỉ cần cập nhật 2 chỗ:
function handleDelete(parentId, childId) {
  const parent = plan[parentId];
  setPlan({
    ...plan,
    [parentId]: {
      ...parent,
      childIds: parent.childIds.filter(id => id !== childId)
    }
  });
  // Xong! Không cần đào sâu vào cây
}
```

**So sánh trước/sau:**

```
TRƯỚC (lồng nhau):
  Để xóa Egypt:
  1. Tìm Earth
  2. Tìm Africa trong childPlaces của Earth
  3. Lọc Egypt ra khỏi childPlaces của Africa
  4. Tạo Africa mới
  5. Tạo Earth mới với Africa mới
  6. setPlan(Earth mới)
  → 6 bước, nhiều spread lồng nhau

SAU (phẳng):
  Để xóa Egypt:
  1. Lấy parent (Africa) từ plan[parentId]
  2. Filter childIds của Africa
  3. setPlan với Africa mới
  → 3 bước, đơn giản và rõ ràng
```

### Render cấu trúc phẳng đệ quy

```jsx
function TravelPlan() {
  const [plan, setPlan] = useState(initialFlatPlan);

  function handleDelete(parentId, childId) {
    const parent = plan[parentId];
    setPlan({
      ...plan,
      [parentId]: {
        ...parent,
        childIds: parent.childIds.filter(id => id !== childId)
      }
    });
  }

  return (
    <PlaceTree
      id={0}            // bắt đầu từ root
      parentId={-1}
      placesById={plan}
      onDelete={handleDelete}
    />
  );
}

// Component đệ quy — render mỗi node và các con của nó
function PlaceTree({ id, parentId, placesById, onDelete }) {
  const place = placesById[id];
  return (
    <li>
      {place.title}
      {parentId !== -1 && (
        <button onClick={() => onDelete(parentId, id)}>✕ Xóa</button>
      )}
      {place.childIds.length > 0 && (
        <ul>
          {place.childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onDelete={onDelete}
            />
          ))}
        </ul>
      )}
    </li>
  );
}
```

---

## Tổng hợp: Checklist review state

Dùng checklist này mỗi khi thiết kế state cho một component mới:

```
□ 1. GOM: Hai state luôn thay đổi cùng lúc không?
         → Có: gom thành một object/array

□ 2. MÂU THUẪN: Có state nào không thể cùng true một lúc không?
              → Có: gộp thành enum (status: 'a' | 'b' | 'c')

□ 3. THỪA: State này có tính được từ state/props khác không?
          → Có: xóa state, tính trong render

□ 3b. MIRROR: State này có phải là copy của props không?
             → Có (và không cố ý): dùng prop trực tiếp

□ 4. TRÙNG LẶP: State này có lưu cùng object với nơi khác không?
               → Có: chỉ lưu ID, tính object khi cần

□ 5. LỒNG SÂU: State có lồng quá nhiều tầng không?
              → Có: làm phẳng (normalize) theo kiểu { id: {...} }
```

---

## Bài tập tự luyện

### Bài 1: Sửa component không cập nhật

```jsx
// Clock nhận prop 'color' nhưng không cập nhật khi prop thay đổi — tại sao?
export default function Clock({ color, time }) {
  const [currentColor, setCurrentColor] = useState(color); // ← lỗi ở đây
  return (
    <h1 style={{ color: currentColor }}>{time}</h1>
  );
}
```

<details>
<summary>👉 Xem đáp án</summary>

**Nguyên nhân:** `useState(color)` chỉ dùng `color` để khởi tạo lần đầu. Khi parent đổi prop `color`, state `currentColor` không tự cập nhật theo.

```jsx
// ✅ Dùng prop trực tiếp — không cần state
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color }}>{time}</h1>
  );
}
```
</details>

---

### Bài 2: Tìm và sửa state trùng lặp

```jsx
// Component có bug: đổi tên item → label "Bạn đã chọn" không cập nhật
const [items, setItems]               = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(items[0]);

function handleRename(id, newTitle) {
  setItems(items.map(item =>
    item.id === id ? { ...item, title: newTitle } : item
  ));
  // Bug: selectedItem vẫn giữ title cũ!
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
// ✅ Chỉ lưu ID — tính selectedItem từ items
const [items, setItems]       = useState(initialItems);
const [selectedId, setSelectedId] = useState(initialItems[0].id);

const selectedItem = items.find(item => item.id === selectedId);

function handleRename(id, newTitle) {
  setItems(items.map(item =>
    item.id === id ? { ...item, title: newTitle } : item
  ));
  // selectedItem tự cập nhật vì tính từ items!
}

function handleSelect(item) {
  setSelectedId(item.id); // chỉ lưu ID
}
```
</details>

---

### Bài 3: Thiết kế state cho ứng dụng Todo

Bạn cần thiết kế state cho app Todo có các tính năng: thêm/xóa/sửa task, đánh dấu hoàn thành, lọc theo trạng thái (All/Active/Completed).

```
Hỏi: Nên dùng những state nào?
```

<details>
<summary>👉 Xem gợi ý thiết kế tối ưu</summary>

```jsx
// ✅ Thiết kế state tối ưu
const [todos, setTodos] = useState([
  { id: 1, text: 'Học React', completed: false },
  { id: 2, text: 'Làm project', completed: true },
]);
const [filter, setFilter] = useState('all'); // 'all' | 'active' | 'completed'

// Tất cả các giá trị sau đây KHÔNG cần là state — tính trong render:
const activeTodos    = todos.filter(t => !t.completed);
const completedTodos = todos.filter(t => t.completed);
const visibleTodos   = filter === 'all'       ? todos
                     : filter === 'active'    ? activeTodos
                     : completedTodos;
const activeCount    = activeTodos.length;
const allCompleted   = todos.every(t => t.completed);

// KHÔNG cần:
// ❌ const [activeCount, setActiveCount] = useState(0)  → tính được
// ❌ const [visibleTodos, setVisibleTodos] = useState([]) → tính được
// ❌ const [allCompleted, setAllCompleted] = useState(false) → tính được
```
</details>

---

## Tóm tắt

```
5 nguyên tắc cấu trúc state:

1. GOM state liên quan
   Luôn cập nhật cùng lúc → gom vào một object/array

2. TRÁNH MÂU THUẪN
   Không thể cùng true → dùng enum: status = 'a'|'b'|'c'

3. TRÁNH STATE THỪA
   Tính được từ state/props khác → biến thường, không phải state
   Đừng copy props vào state (trừ khi cố ý với prefix initial/default)

4. TRÁNH TRÙNG LẶP
   Chỉ lưu ID thay vì toàn bộ object
   Tính object từ ID khi render

5. TRÁNH LỒNG SÂU
   Normalize: { id: { id, title, childIds: [] } }
   Cập nhật chỉ cần 1-2 tầng thay vì đào sâu
```

---
