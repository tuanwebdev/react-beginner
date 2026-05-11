# Cập Nhật Array trong State

> Hướng dẫn từ [react.dev/learn/updating-arrays-in-state](https://react.dev/learn/updating-arrays-in-state)

Array trong JavaScript là object, và giống như object, bạn phải **coi array trong state là read-only**. Không được sửa trực tiếp — hãy tạo array mới rồi set state bằng array đó.

---

## Bảng tra cứu nhanh: Dùng gì, tránh gì

Đây là bảng quan trọng nhất — hãy bookmark lại:

| Thao tác | ❌ Tránh (mutation) | ✅ Dùng (tạo array mới) |
|---|---|---|
| **Thêm phần tử** | `push()`, `unshift()` | `[...arr, newItem]`, `[newItem, ...arr]` |
| **Xóa phần tử** | `pop()`, `shift()`, `splice()` | `filter()` |
| **Thay thế phần tử** | `splice()`, `arr[i] = ...` | `map()` |
| **Sắp xếp / Đảo ngược** | `sort()`, `reverse()` | Copy trước rồi mới sort/reverse |
| **Chèn vào giữa** | `splice()` | `slice()` + spread |

> ⚠️ **`slice` vs `splice` — hay nhầm nhất:**
> - `slice` (không có **p**) → **KHÔNG** mutation, trả về mảng con mới → **dùng được trong React**
> - `splice` (có **p**) → **CÓ** mutation, sửa thẳng mảng gốc → **không dùng trong React**

---

## 1. Thêm phần tử vào array

### ❌ Sai — dùng `push()` (mutation)

```jsx
// push() thay đổi trực tiếp array gốc — React không biết để re-render
<button onClick={() => {
  artists.push({ id: nextId++, name: name });
}}>Thêm</button>
```

### ✅ Đúng — dùng spread syntax `...`

```jsx
// Tạo array MỚI chứa tất cả phần tử cũ + phần tử mới
<button onClick={() => {
  setArtists([
    ...artists,              // copy tất cả phần tử cũ
    { id: nextId++, name: name }  // thêm phần tử mới ở CUỐI
  ]);
}}>Thêm cuối</button>

// Thêm vào ĐẦU — đặt phần tử mới trước ...artists
<button onClick={() => {
  setArtists([
    { id: nextId++, name: name },  // phần tử mới ở ĐẦU
    ...artists                     // copy tất cả phần tử cũ
  ]);
}}>Thêm đầu</button>
```

**Ví dụ đầy đủ:**

```jsx
import { useState } from 'react';

let nextId = 0;

export default function ArtistList() {
  const [name, setName]       = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <input value={name} onChange={e => setName(e.target.value)} />
      <button onClick={() => {
        setArtists([...artists, { id: nextId++, name }]);
        setName('');  // reset input sau khi thêm
      }}>
        + Thêm
      </button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

---

## 2. Xóa phần tử khỏi array

Dùng `filter()` để tạo array mới **không chứa** phần tử cần xóa:

```jsx
// Xóa artist có id khớp với artist.id
setArtists(
  artists.filter(a => a.id !== artist.id)
  // → trả về array mới chỉ chứa các a mà a.id !== artist.id
  // → phần tử có id khớp bị loại ra
);
```

**Ví dụ đầy đủ:**

```jsx
export default function ArtistList() {
  const [artists, setArtists] = useState([
    { id: 0, name: 'Marta Colvin Andrade' },
    { id: 1, name: 'Lamidi Olonade Fakeye' },
    { id: 2, name: 'Louise Nevelson' },
  ]);

  return (
    <ul>
      {artists.map(artist => (
        <li key={artist.id}>
          {artist.name}
          <button onClick={() => {
            setArtists(artists.filter(a => a.id !== artist.id));
          }}>
            Xóa
          </button>
        </li>
      ))}
    </ul>
  );
}
```

**Cách filter hoạt động:**

```
artists = [
  { id: 0, name: 'Marta' },
  { id: 1, name: 'Lamidi' },  ← cần xóa (artist.id = 1)
  { id: 2, name: 'Louise' },
]

filter(a => a.id !== 1):
  { id: 0 } → 0 !== 1 → true  → GIỮ LẠI
  { id: 1 } → 1 !== 1 → false → LOẠI RA
  { id: 2 } → 2 !== 1 → true  → GIỮ LẠI

Kết quả: [{ id: 0, name: 'Marta' }, { id: 2, name: 'Louise' }]
```

---

## 3. Biến đổi phần tử trong array

Dùng `map()` để tạo array mới — mỗi phần tử có thể được giữ nguyên hoặc thay thế:

```jsx
// Di chuyển tất cả hình tròn xuống 50px, giữ nguyên hình vuông
function handleClick() {
  const nextShapes = shapes.map(shape => {
    if (shape.type === 'square') {
      return shape;                      // giữ nguyên
    } else {
      return { ...shape, y: shape.y + 50 };  // tạo object mới với y thay đổi
    }
  });
  setShapes(nextShapes);
}
```

---

## 4. Thay thế phần tử tại vị trí cụ thể

`arr[i] = value` là mutation — dùng `map()` với index thay thế:

```jsx
// Tăng counter tại vị trí index được click
function handleIncrementClick(index) {
  const nextCounters = counters.map((c, i) => {
    if (i === index) {
      return c + 1;  // tăng phần tử được click
    } else {
      return c;      // giữ nguyên phần tử khác
    }
  });
  setCounters(nextCounters);
}

// Rút gọn hơn với ternary:
const nextCounters = counters.map((c, i) =>
  i === index ? c + 1 : c
);
```

---

## 5. Chèn phần tử vào giữa array

Dùng `slice()` (không phải splice!) kết hợp spread:

```jsx
function handleInsert() {
  const insertAt = 1;  // vị trí muốn chèn vào
  const nextArtists = [
    ...artists.slice(0, insertAt),    // phần tử từ đầu đến trước vị trí chèn
    { id: nextId++, name: name },     // phần tử mới
    ...artists.slice(insertAt)        // phần tử từ vị trí chèn đến cuối
  ];
  setArtists(nextArtists);
}
```

**Cách slice hoạt động:**

```
artists = ['Marta', 'Lamidi', 'Louise']  (index: 0, 1, 2)
insertAt = 1

slice(0, 1) → ['Marta']           (từ 0 đến trước 1)
newItem    → ['Alice']
slice(1)   → ['Lamidi', 'Louise'] (từ 1 đến cuối)

Kết quả: ['Marta', 'Alice', 'Lamidi', 'Louise']
```

---

## 6. Sắp xếp và Đảo ngược array

`sort()` và `reverse()` là mutation — phải **copy trước** rồi mới thao tác:

```jsx
function handleReverse() {
  const nextList = [...list];   // copy array trước
  nextList.reverse();           // reverse trên bản copy — ổn!
  setList(nextList);
}

function handleSort() {
  const nextList = [...list];
  nextList.sort((a, b) => a.name.localeCompare(b.name));
  setList(nextList);
}
```

> ⚠️ **Bẫy quan trọng:** Dù copy array nhưng các **object bên trong vẫn là cùng tham chiếu**! Copy array chỉ là shallow copy.
>
> ```jsx
> const nextList = [...list];
> nextList[0].seen = true;  // ❌ vẫn là mutation! nextList[0] === list[0]
> setList(nextList);
> ```

---

## 7. Cập nhật Object bên trong Array

Đây là trường hợp **hay gây bug nhất** vì copy array là shallow — các object bên trong vẫn dùng chung tham chiếu.

### Vấn đề: Copy array nhưng vẫn mutation object bên trong

```jsx
// ❌ Bug — 2 danh sách dùng chung object, checkbox một cái ảnh hưởng cái kia!
function handleToggleMyList(artworkId, nextSeen) {
  const myNextList = [...myList];           // copy array — ổn
  const artwork = myNextList.find(          // tìm phần tử
    a => a.id === artworkId
  );
  artwork.seen = nextSeen;                  // ❌ mutation object! artwork === myList[i]
  setMyList(myNextList);
}
```

```
myList    = [artwork0, artwork1, artwork2]   (tham chiếu)
yourList  = [artwork0, artwork1, artwork2]   (CÙNG tham chiếu!)

Sau [...myList]:
myNextList = [artwork0, artwork1, artwork2]  (array mới, nhưng chứa cùng object!)

artwork.seen = true → thay đổi artwork0 gốc
→ myList[0].seen = true    (dự kiến)
→ yourList[0].seen = true  (ngoài ý muốn — BUG!)
```

### ✅ Giải pháp: Dùng `map()` + spread để tạo object mới

```jsx
function handleToggleMyList(artworkId, nextSeen) {
  setMyList(myList.map(artwork => {
    if (artwork.id === artworkId) {
      // Tạo object MỚI với seen được cập nhật
      return { ...artwork, seen: nextSeen };
    } else {
      return artwork;  // giữ nguyên các phần tử khác
    }
  }));
}
```

**Cách rút gọn với ternary:**

```jsx
setMyList(myList.map(artwork =>
  artwork.id === artworkId
    ? { ...artwork, seen: nextSeen }  // object mới
    : artwork                          // giữ nguyên
));
```

---

## 8. Dùng Immer để code ngắn gọn hơn

Khi thao tác array phức tạp, Immer giúp viết code như mutation thông thường:

```bash
npm install use-immer
```

```jsx
import { useImmer } from 'use-immer';

const [myList, updateMyList] = useImmer(initialList);

// Với Immer — viết như mutation, ngắn gọn hơn nhiều:
function handleToggleMyList(artworkId, nextSeen) {
  updateMyList(draft => {
    const artwork = draft.find(a => a.id === artworkId);
    artwork.seen = nextSeen;  // ← trông như mutation nhưng an toàn!
  });
}

// Thêm phần tử — dùng push được!
function handleAdd(name) {
  updateMyList(draft => {
    draft.push({ id: nextId++, name });  // ← push được với Immer!
  });
}

// Xóa phần tử
function handleDelete(id) {
  updateMyList(draft => {
    const index = draft.findIndex(a => a.id === id);
    draft.splice(index, 1);  // ← splice được với Immer!
  });
}
```

### So sánh: Không Immer vs Có Immer

```jsx
// ❌ Không Immer — dài dòng, dễ nhầm
function handleToggle(artworkId, nextSeen) {
  setMyList(myList.map(artwork =>
    artwork.id === artworkId
      ? { ...artwork, seen: nextSeen }
      : artwork
  ));
}

// ✅ Có Immer — ngắn gọn, dễ đọc
function handleToggle(artworkId, nextSeen) {
  updateMyList(draft => {
    draft.find(a => a.id === artworkId).seen = nextSeen;
  });
}
```

---

## Tổng hợp: Cheatsheet đầy đủ

```jsx
const [items, setItems] = useState([...]);

// THÊM vào cuối
setItems([...items, newItem]);

// THÊM vào đầu
setItems([newItem, ...items]);

// THÊM vào giữa (vị trí i)
setItems([...items.slice(0, i), newItem, ...items.slice(i)]);

// XÓA theo điều kiện
setItems(items.filter(item => item.id !== targetId));

// XÓA theo index
setItems(items.filter((_, i) => i !== targetIndex));

// THAY THẾ theo điều kiện
setItems(items.map(item =>
  item.id === targetId ? { ...item, ...changes } : item
));

// THAY THẾ theo index
setItems(items.map((item, i) =>
  i === targetIndex ? { ...item, ...changes } : item
));

// ĐẢO NGƯỢC
const next = [...items];
next.reverse();
setItems(next);

// SẮP XẾP
const next = [...items];
next.sort((a, b) => a.name.localeCompare(b.name));
setItems(next);

// CẬP NHẬT object bên trong array
setItems(items.map(item =>
  item.id === targetId
    ? { ...item, field: newValue }
    : item
));
```

---

## Bài tập tự luyện

### Bài 1: Thêm chức năng tăng số lượng trong giỏ hàng

```jsx
// Viết handleIncreaseClick để tăng count của sản phẩm được click
const [products, setProducts] = useState([
  { id: 0, name: 'Baklava', count: 1 },
  { id: 1, name: 'Cheese', count: 5 },
  { id: 2, name: 'Spaghetti', count: 2 },
]);

function handleIncreaseClick(productId) {
  // Viết code ở đây
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
function handleIncreaseClick(productId) {
  setProducts(products.map(product =>
    product.id === productId
      ? { ...product, count: product.count + 1 }
      : product
  ));
}
```
</details>

---

### Bài 2: Xóa sản phẩm khi count = 0

```jsx
// Thêm nút "-" và xóa sản phẩm khi count về 0
function handleDecreaseClick(productId) {
  // Viết code ở đây
  // Gợi ý: giảm count, nếu count = 0 thì xóa khỏi danh sách
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
function handleDecreaseClick(productId) {
  // Bước 1: giảm count
  const nextProducts = products.map(product =>
    product.id === productId
      ? { ...product, count: product.count - 1 }
      : product
  );
  // Bước 2: lọc bỏ sản phẩm có count = 0
  setProducts(nextProducts.filter(p => p.count > 0));
}
```
</details>

---

### Bài 3: Tìm lỗi — tại sao 2 danh sách bị liên kết với nhau?

```jsx
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
];

const [myList, setMyList]     = useState(initialList);
const [yourList, setYourList] = useState(initialList);

// ❌ Code có bug — tick checkbox ở myList cũng ảnh hưởng yourList
function handleToggleMyList(id, nextSeen) {
  const myNextList = [...myList];
  const artwork = myNextList.find(a => a.id === id);
  artwork.seen = nextSeen;   // ← lỗi ở đây
  setMyList(myNextList);
}
```

<details>
<summary>👉 Xem giải thích và đáp án</summary>

**Nguyên nhân:** `initialList` là một array, và cả `myList` lẫn `yourList` đều trỏ đến **cùng các object** bên trong. Khi `[...myList]` — array mới được tạo, nhưng các object bên trong vẫn là cùng tham chiếu. Mutation `artwork.seen` thay đổi object gốc, ảnh hưởng cả `yourList`.

```jsx
// ✅ Đúng — dùng map + spread để tạo object mới
function handleToggleMyList(id, nextSeen) {
  setMyList(myList.map(artwork =>
    artwork.id === id
      ? { ...artwork, seen: nextSeen }  // object MỚI
      : artwork
  ));
}
```
</details>

---

## Tóm tắt

```
Nguyên tắc:
  Array trong state → READ-ONLY → không mutation

Các thao tác:
  Thêm   → [...arr, newItem] hoặc [newItem, ...arr]
  Xóa    → arr.filter(item => điều_kiện_giữ_lại)
  Thay   → arr.map(item => item.id === id ? mới : item)
  Chèn   → [...arr.slice(0, i), newItem, ...arr.slice(i)]
  Sort   → copy trước: [...arr].sort(...)

Object bên trong array:
  Copy array (shallow) KHÔNG đủ để tránh mutation bên trong
  Phải dùng map + spread: { ...item, field: newValue }

Immer:
  Cho phép dùng push, splice, direct assignment
  Phù hợp khi thao tác phức tạp, nhiều tầng lồng
```
