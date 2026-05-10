# Cập Nhật Object trong State

> Hướng dẫn từ [react.dev/learn/updating-objects-in-state](https://react.dev/learn/updating-objects-in-state)

State có thể chứa bất kỳ kiểu dữ liệu nào — kể cả object. Nhưng bạn **không được sửa trực tiếp** object đang có trong state. Thay vào đó, hãy tạo một object mới rồi set state bằng object đó.

---

## Mutation là gì?

Với kiểu dữ liệu nguyên thủy (number, string, boolean), bạn không thể thay đổi bản thân giá trị đó:

```js
// Số 0 không bao giờ "trở thành" 5
// setX(5) chỉ là "thay số 0 bằng số 5"
const [x, setX] = useState(0);
setX(5);
```

Nhưng với object, bạn **có thể** thay đổi nội dung bên trong mà không cần tạo object mới:

```js
const [position, setPosition] = useState({ x: 0, y: 0 });

// ❌ Đây là MUTATION — sửa thẳng vào object đang có
position.x = 5;
```

Về mặt kỹ thuật, JavaScript cho phép làm vậy. Nhưng trong React, bạn phải **coi object trong state như thể nó là read-only (chỉ đọc)** — dù JavaScript không cưỡng chế điều đó.

---

## Tại sao không được mutation?

### Vấn đề trực tiếp: React không biết để re-render

```jsx
// ❌ Code này KHÔNG hoạt động — chấm đỏ không di chuyển
export default function MovingDot() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <div onPointerMove={e => {
      position.x = e.clientX;  // ← mutation trực tiếp
      position.y = e.clientY;  // ← mutation trực tiếp
      // Không gọi setPosition → React không biết → không re-render!
    }}>
      <div style={{
        transform: `translate(${position.x}px, ${position.y}px)`
      }} />
    </div>
  );
}
```

```jsx
// ✅ ĐÚNG — tạo object mới, gọi setPosition → React re-render
export default function MovingDot() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <div onPointerMove={e => {
      setPosition({       // ← tạo object mới hoàn toàn
        x: e.clientX,
        y: e.clientY
      });
    }}>
      <div style={{
        transform: `translate(${position.x}px, ${position.y}px)`
      }} />
    </div>
  );
}
```

### Lý do sâu hơn: 5 lý do không nên mutation

| Lý do | Giải thích |
|---|---|
| **Debug dễ hơn** | `console.log` của render cũ không bị ghi đè — thấy rõ state thay đổi như thế nào |
| **Tối ưu hóa** | React dùng `prevObj === obj` để bỏ qua re-render không cần thiết. Mutation phá vỡ điều này |
| **Tính năng mới** | Các tính năng React mới (Concurrent Mode...) dựa vào state bất biến |
| **Undo/Redo dễ** | Giữ lịch sử state cũ trong bộ nhớ → hoàn tác dễ dàng |
| **Đơn giản hơn** | React không cần "theo dõi" object của bạn — không cần Proxy, getter/setter phức tạp |

---

## Cách cập nhật object đúng cách

### Trường hợp 1: Thay toàn bộ object

```jsx
// Khi tất cả trường đều thay đổi — tạo object hoàn toàn mới
setPosition({ x: 100, y: 200 });
```

### Trường hợp 2: Chỉ thay một trường — dùng Spread Syntax `...`

```jsx
const [person, setPerson] = useState({
  firstName: 'Barbara',
  lastName: 'Hepworth',
  email: 'bhepworth@sculpture.com'
});

// ❌ SAI — mutation trực tiếp, React không biết để re-render
function handleFirstNameChange(e) {
  person.firstName = e.target.value;
}

// ✅ ĐÚNG — copy object cũ, ghi đè trường cần thay
function handleFirstNameChange(e) {
  setPerson({
    ...person,              // copy tất cả trường cũ
    firstName: e.target.value  // ghi đè chỉ trường này
  });
}
```

**`...person` hoạt động như thế nào?**

```js
const person = { firstName: 'Barbara', lastName: 'Hepworth', email: 'b@h.com' };

// Spread copy tất cả key-value vào object mới
{ ...person, firstName: 'Bob' }
// = { firstName: 'Bob', lastName: 'Hepworth', email: 'b@h.com' }
//                ↑ trường sau ghi đè trường trước (nếu trùng key)
```

> ⚠️ **Spread chỉ là shallow copy (copy nông)** — chỉ copy 1 tầng. Nếu object có object lồng bên trong, phần lồng vẫn là cùng tham chiếu. Xem phần Object lồng bên dưới.

### Trường hợp 3: Một handler cho nhiều trường — dùng computed property name

```jsx
// Thay vì 3 handler riêng cho 3 input:
function handleFirstNameChange(e) { setPerson({ ...person, firstName: e.target.value }); }
function handleLastNameChange(e)  { setPerson({ ...person, lastName: e.target.value });  }
function handleEmailChange(e)     { setPerson({ ...person, email: e.target.value });     }

// ✅ Gộp thành 1 handler duy nhất dùng [e.target.name]
function handleChange(e) {
  setPerson({
    ...person,
    [e.target.name]: e.target.value  // key động từ thuộc tính 'name' của input
  });
}

// Trong JSX — thêm thuộc tính name cho mỗi input
<input name="firstName" value={person.firstName} onChange={handleChange} />
<input name="lastName"  value={person.lastName}  onChange={handleChange} />
<input name="email"     value={person.email}     onChange={handleChange} />
```

---

## Cập nhật Object lồng nhau (Nested Object)

### Vấn đề: Spread chỉ copy 1 tầng

```jsx
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {           // ← object lồng bên trong
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://example.com/blue-nana.jpg',
  }
});
```

**Muốn đổi city thành 'New Delhi':**

```jsx
// ❌ SAI — mutation trực tiếp object lồng
person.artwork.city = 'New Delhi';

// ❌ CŨNG SAI — spread chỉ copy tầng ngoài
// artwork bên trong vẫn là cùng object cũ!
setPerson({
  ...person,
  artwork: person.artwork  // ← vẫn là object cũ!
});
person.artwork.city = 'New Delhi'; // ← mutation vẫn xảy ra!
```

```jsx
// ✅ ĐÚNG — phải tạo object mới cho từng tầng
setPerson({
  ...person,           // copy tầng ngoài
  artwork: {           // tạo object artwork MỚI
    ...person.artwork, // copy các trường cũ của artwork
    city: 'New Delhi'  // ghi đè city
  }
});
```

**Hoặc tách ra 2 bước cho dễ đọc:**

```jsx
const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson  = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

### Sơ đồ: Tại sao phải tạo object mới từng tầng?

```
TRƯỚC (mutation):
person ──→ { name: 'Niki', artwork: ──→ { title: 'Blue Nana', city: 'Hamburg' } }
                                    ↑
                              React vẫn thấy cùng object này
                              → không biết city đã thay đổi
                              → không re-render!

SAU (đúng cách):
person ──→ { name: 'Niki', artwork: ──→ { title: 'Blue Nana', city: 'Hamburg' } }  (cũ)
                                    ↗ (object artwork MỚI)
newPerson → { name: 'Niki', artwork: ──→ { title: 'Blue Nana', city: 'New Delhi' } } (mới)
React thấy newPerson !== person → re-render! ✓
```

### Ví dụ thực tế: Form với object lồng

```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://example.com/blue-nana.jpg',
    }
  });

  function handleNameChange(e) {
    setPerson({ ...person, name: e.target.value });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: { ...person.artwork, title: e.target.value }
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: { ...person.artwork, city: e.target.value }
    });
  }

  return (
    <>
      <label>Tên:  <input value={person.name}           onChange={handleNameChange}  /></label>
      <label>Tác phẩm: <input value={person.artwork.title} onChange={handleTitleChange} /></label>
      <label>Thành phố: <input value={person.artwork.city}  onChange={handleCityChange}  /></label>
      <p>
        <i>{person.artwork.title}</i> by {person.name}
        <br />(tại {person.artwork.city})
      </p>
    </>
  );
}
```

---

## Giải pháp gọn hơn: Immer

Khi object lồng nhiều tầng, phải spread nhiều lần rất dài dòng. **Immer** là thư viện giúp bạn viết code như đang mutation, nhưng thực ra tạo object mới phía sau hậu trường.

### Cài đặt

```bash
npm install use-immer
```

### So sánh: Trước và sau khi dùng Immer

```jsx
// ❌ Không dùng Immer — spread nhiều tầng rất dài
function handleCityChange(e) {
  setPerson({
    ...person,
    artwork: {
      ...person.artwork,
      city: e.target.value
    }
  });
}

// ✅ Dùng Immer — viết như mutation thông thường, ngắn gọn hơn
import { useImmer } from 'use-immer';

const [person, updatePerson] = useImmer({ name: '...', artwork: { city: '...' } });

function handleCityChange(e) {
  updatePerson(draft => {
    draft.artwork.city = e.target.value;  // ← trông như mutation, nhưng an toàn!
  });
}
```

**Immer hoạt động như thế nào?**

`draft` là một object đặc biệt (Proxy) — nó "ghi lại" tất cả những gì bạn làm với nó. Sau khi hàm chạy xong, Immer tạo ra một object mới hoàn toàn dựa trên những thay đổi đó. State gốc không bị ảnh hưởng.

```jsx
// Ví dụ đầy đủ với useImmer
import { useImmer } from 'use-immer';

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://example.com/blue-nana.jpg',
    }
  });

  function handleNameChange(e) {
    updatePerson(draft => {
      draft.name = e.target.value;  // đơn giản hơn nhiều!
    });
  }

  function handleCityChange(e) {
    updatePerson(draft => {
      draft.artwork.city = e.target.value;  // không cần spread nhiều tầng
    });
  }

  return (
    <>
      <label>Tên: <input value={person.name} onChange={handleNameChange} /></label>
      <label>Thành phố: <input value={person.artwork.city} onChange={handleCityChange} /></label>
      <p>{person.name} — {person.artwork.city}</p>
    </>
  );
}
```

---

## Local Mutation — Ngoại lệ được phép

Không phải mọi mutation đều là xấu. Mutation chỉ gây vấn đề khi bạn sửa **object đang tồn tại trong state**. Nếu bạn vừa tạo một object mới và chưa set vào state, mutation nó hoàn toàn ổn:

```jsx
// ✅ Local mutation — hoàn toàn ổn
function handleChange(e) {
  const nextPosition = {};    // tạo mới
  nextPosition.x = 100;       // mutation object mới — ổn!
  nextPosition.y = 200;       // mutation object mới — ổn!
  setPosition(nextPosition);  // set vào state
}

// Tương đương hoàn toàn với:
function handleChange(e) {
  setPosition({ x: 100, y: 200 });
}
```

---

## Tóm tắt

```
Quy tắc vàng:
  Object trong state → coi như READ-ONLY → không được sửa trực tiếp

Cách cập nhật đúng:
  1. Thay toàn bộ:   setObj({ newKey: newVal })
  2. Thay một trường: setObj({ ...obj, key: newVal })
  3. Lồng nhiều tầng: setObj({ ...obj, nested: { ...obj.nested, key: newVal } })
  4. Dùng Immer:      updateObj(draft => { draft.nested.key = newVal })

Lưu ý về spread:
  { ...obj }  →  chỉ copy TẦM NGOÀI (shallow)
  Object lồng bên trong vẫn là cùng tham chiếu
  → phải spread từng tầng nếu cần sửa bên trong

Local mutation (tạo mới rồi sửa):
  const next = {}; next.x = 1; setState(next);  ← ổn
  state.x = 1;                                   ← không ổn
```

---

## Bài tập tự luyện

### Bài 1: Sửa 3 lỗi trong Scoreboard

```jsx
// Code có 3 lỗi — tìm và sửa
export default function Scoreboard() {
  const [player, setPlayer] = useState({
    firstName: 'Ranjani',
    lastName: 'Shettar',
    score: 10,
  });

  function handlePlusClick() {
    player.score++;                    // Lỗi 1
  }

  function handleFirstNameChange(e) {
    setPlayer({ ...player, firstName: e.target.value });  // ← đúng rồi
  }

  function handleLastNameChange(e) {
    setPlayer({ lastName: e.target.value });  // Lỗi 2 — thiếu ...player
  }
}
```

<details>
<summary>👉 Xem đáp án và giải thích</summary>

```jsx
export default function Scoreboard() {
  const [player, setPlayer] = useState({
    firstName: 'Ranjani',
    lastName: 'Shettar',
    score: 10,
  });

  function handlePlusClick() {
    // ✅ Sửa lỗi 1: không mutation, dùng setter
    setPlayer({ ...player, score: player.score + 1 });
  }

  function handleFirstNameChange(e) {
    setPlayer({ ...player, firstName: e.target.value });
  }

  function handleLastNameChange(e) {
    // ✅ Sửa lỗi 2: thêm ...player để giữ các trường khác
    setPlayer({ ...player, lastName: e.target.value });
  }
}
```

**Giải thích 2 lỗi:**
1. `player.score++` — mutation trực tiếp. React không biết score thay đổi → không re-render.
2. `setPlayer({ lastName: e.target.value })` — chỉ set lastName, các trường khác bị mất! Sau khi gõ lastName, firstName và score sẽ thành `undefined`.
</details>

---

### Bài 2: Cập nhật object lồng

```jsx
// Hãy viết handler để cập nhật person.address.city
const [person, setPerson] = useState({
  name: 'Nguyen Van A',
  address: {
    city: 'Hanoi',
    country: 'Vietnam'
  }
});

function handleCityChange(newCity) {
  // Viết code ở đây
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
function handleCityChange(newCity) {
  setPerson({
    ...person,            // copy tầng ngoài
    address: {            // tạo object address mới
      ...person.address,  // copy các trường cũ của address
      city: newCity       // ghi đè city
    }
  });
}

// Hoặc dùng Immer:
updatePerson(draft => {
  draft.address.city = newCity;
});
```
</details>

---

### Bài 3: Tại sao code này bị lỗi?

```jsx
const [config, setConfig] = useState({ theme: 'light', lang: 'vi' });

// User A và User B đều dùng config này
const configA = config;
const configB = config;

// Thay đổi theme của configA
configA.theme = 'dark';

// Hỏi: configB.theme là gì?
```

<details>
<summary>👉 Xem đáp án</summary>

`configB.theme` cũng là `'dark'` — vì `configA` và `configB` đều **trỏ đến cùng một object**. Mutation một cái ảnh hưởng cả cái kia.

Đây chính xác là lý do React không cho mutation state: nếu nhiều thứ trỏ đến cùng object, bạn có thể vô tình thay đổi dữ liệu ở nơi mình không ngờ tới.

```jsx
// ✅ Cách đúng — tạo bản copy độc lập
const configA = { ...config };
const configB = { ...config };
configA.theme = 'dark';
// configB.theme vẫn là 'light' ✓
```
</details>
