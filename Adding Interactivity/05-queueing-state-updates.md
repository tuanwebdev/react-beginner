# Xếp Hàng Nhiều Lần Cập Nhật State

> Hướng dẫn từ [react.dev/learn/queueing-a-series-of-state-updates](https://react.dev/learn/queueing-a-series-of-state-updates)

Bài trước cho thấy `setNumber(number + 1)` ba lần vẫn chỉ tăng 1 — vì state là "bản chụp". Bài này giải thích cơ chế **batching** và cách dùng **updater function** để thực sự cộng dồn nhiều lần.

---

## Batching — React gom nhiều update lại rồi xử lý một lần

React **không** re-render ngay sau mỗi lần `setState`. Thay vào đó, nó đợi cho đến khi **toàn bộ event handler chạy xong**, rồi mới xử lý tất cả các update và thực hiện một lần re-render duy nhất.

Hành vi này gọi là **batching** (gom nhóm).

### Phép ẩn dụ: Bồi bàn nhà hàng

Giống như bồi bàn không chạy vào bếp sau mỗi món bạn gọi — họ chờ bạn gọi xong hết rồi mới đi một lần:

```
Không có batching (giả sử):        Có batching (thực tế):
─────────────────────────          ─────────────────────────
setA(1) → re-render ngay            setA(1) ─┐
setB(2) → re-render ngay            setB(2) ─┤→ 1 lần re-render duy nhất
setC(3) → re-render ngay            setC(3) ─┘
= 3 lần re-render                   = 1 lần re-render
```

**Lợi ích của batching:**
- App chạy nhanh hơn nhiều (ít re-render hơn)
- Tránh các trạng thái "nửa chừng" — tất cả update áp dụng cùng lúc
- React chỉ batch trong cùng một event handler, **không** batch giữa các click khác nhau

---

## Vấn đề: `setNumber(number + 1)` nhiều lần không cộng dồn

```jsx
// Từ bài trước — chỉ tăng 1 dù gọi 3 lần
<button onClick={() => {
  setNumber(number + 1);  // setNumber(0 + 1) → "thay bằng 1"
  setNumber(number + 1);  // setNumber(0 + 1) → "thay bằng 1"
  setNumber(number + 1);  // setNumber(0 + 1) → "thay bằng 1"
}}>+3</button>
// Kết quả: number = 1, không phải 3
```

Lý do: `number` trong bản chụp hiện tại luôn là `0`. Cả 3 lần đều ra lệnh "thay bằng 1".

---

## Giải pháp: Updater Function — Hàm cập nhật

Thay vì truyền **giá trị** vào setter, truyền một **hàm** — React sẽ gọi hàm đó với giá trị mới nhất trong hàng đợi:

```jsx
// ✅ Dùng updater function — tăng đúng 3 lần
<button onClick={() => {
  setNumber(n => n + 1);  // "lấy n hiện tại, cộng 1"
  setNumber(n => n + 1);  // "lấy n hiện tại, cộng 1"
  setNumber(n => n + 1);  // "lấy n hiện tại, cộng 1"
}}>+3</button>
// Kết quả: number = 3 ✓
```

**`n => n + 1`** là updater function. Cú pháp: nhận giá trị trước đó → trả về giá trị mới.

---

## Cách React xử lý hàng đợi (queue)

Khi event handler chạy xong, React duyệt qua hàng đợi từ đầu đến cuối, truyền kết quả của bước trước vào bước sau:

### Ví dụ 1: Ba updater function

```jsx
// number ban đầu = 0
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

| Thứ tự | Lệnh trong queue | `n` nhận vào | Trả về |
|--------|-----------------|-------------|--------|
| 1 | `n => n + 1` | `0` (state ban đầu) | `1` |
| 2 | `n => n + 1` | `1` (kết quả bước 1) | `2` |
| 3 | `n => n + 1` | `2` (kết quả bước 2) | `3` |

**Kết quả cuối:** `number = 3` ✓

---

### Ví dụ 2: Trộn giá trị thẳng và updater function

```jsx
// number ban đầu = 0
setNumber(number + 5);   // truyền giá trị: "thay bằng 5"
setNumber(n => n + 1);   // updater function: "lấy n, cộng 1"
```

| Thứ tự | Lệnh trong queue | `n` nhận vào | Trả về |
|--------|-----------------|-------------|--------|
| 1 | `"thay bằng 5"` | `0` (không dùng) | `5` |
| 2 | `n => n + 1` | `5` (kết quả bước 1) | `6` |

**Kết quả cuối:** `number = 6`

---

### Ví dụ 3: Updater rồi thay thẳng

```jsx
// number ban đầu = 0
setNumber(number + 5);   // "thay bằng 5"
setNumber(n => n + 1);   // "lấy n, cộng 1"
setNumber(42);           // "thay bằng 42"
```

| Thứ tự | Lệnh trong queue | `n` nhận vào | Trả về |
|--------|-----------------|-------------|--------|
| 1 | `"thay bằng 5"` | `0` (không dùng) | `5` |
| 2 | `n => n + 1` | `5` | `6` |
| 3 | `"thay bằng 42"` | `6` (không dùng) | `42` |

**Kết quả cuối:** `number = 42`

> 💡 Khi truyền giá trị thẳng (`set(42)`), React hiểu là "bỏ qua tất cả bước trước, thay bằng 42". Về mặt kỹ thuật, `set(5)` tương đương `set(n => 5)` — chỉ là `n` không được dùng đến.

---

## Tóm tắt: Hai cách dùng setter

| Cách dùng | Ví dụ | React hiểu là | Dùng khi |
|---|---|---|---|
| **Truyền giá trị** | `setN(5)` | "Thay bằng 5" | Đặt giá trị cố định |
| **Updater function** | `setN(n => n + 1)` | "Tính từ giá trị hiện tại" | Cần dựa vào state trước đó |

```
setNumber(5)          →  "replace with 5"        → bỏ qua queue cũ
setNumber(n => n + 1) →  "thêm vào queue, n = kết quả bước trước"
```

---

## Quy ước đặt tên updater function

Tên tham số của updater function thường là viết tắt của tên state:

```jsx
// Viết tắt (phổ biến nhất)
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);

// Tên đầy đủ (rõ ràng hơn)
setEnabled(enabled => !enabled);

// Dùng tiền tố "prev" (rõ ý nghĩa nhất)
setEnabled(prevEnabled => !prevEnabled);
setCount(prevCount => prevCount + 1);
```

---

## ⚠️ Updater function phải thuần túy (pure)

Updater function **chạy trong lúc rendering** — vì vậy nó phải:

```jsx
// ✅ ĐÚNG — chỉ tính toán và trả về giá trị mới
setItems(items => [...items, newItem]);
setCount(n => n + 1);

// ❌ SAI — gọi API, console.log, setState khác trong updater
setCount(n => {
  console.log(n);       // ← side effect — không được!
  fetchData();          // ← side effect — không được!
  setOther(n * 2);      // ← setState khác — không được!
  return n + 1;
});
```

> 💡 Trong Strict Mode, React gọi mỗi updater function **2 lần** (bỏ kết quả lần 2) để phát hiện side effect. Nếu updater của bạn thuần túy, kết quả 2 lần sẽ giống nhau và không gây vấn đề.

---

## Ứng dụng thực tế: Khi nào dùng updater function?

### Trường hợp bắt buộc dùng updater: code bất đồng bộ

Đây là ví dụ lỗi kinh điển với `async/await`:

```jsx
// ❌ BUG — dùng giá trị thẳng trong code async
async function handleBuy() {
  setPending(pending + 1);     // dùng "bản chụp" pending = 0
  await delay(3000);           // đợi 3 giây
  setPending(pending - 1);     // vẫn dùng "bản chụp" pending = 0 !!!
  setCompleted(completed + 1); // vẫn dùng "bản chụp" completed = 0 !!!
}

// Nếu click 2 lần nhanh:
// Click 1: setPending(0+1)=1, sau 3s: setPending(0-1)=-1 ← BUG!
// Click 2: setPending(0+1)=1, sau 3s: setPending(0-1)=-1 ← BUG!
```

```jsx
// ✅ ĐÚNG — dùng updater function
async function handleBuy() {
  setPending(p => p + 1);      // "lấy p mới nhất, cộng 1"
  await delay(3000);
  setPending(p => p - 1);      // "lấy p mới nhất, trừ 1" ← luôn đúng!
  setCompleted(c => c + 1);    // "lấy c mới nhất, cộng 1"
}

// Click 2 lần nhanh:
// Click 1: pending: 0→1, sau 3s: pending: 1→0, completed: 0→1 ✓
// Click 2: pending: 1→2, sau 3s: pending: 0→1, completed: 1→2 ✓
//          (cộng dồn đúng cách)
```

### Khi nào KHÔNG cần updater:

```jsx
// Không cần updater khi giá trị không phụ thuộc vào state trước
setName('Bob');           // giá trị cố định → dùng thẳng
setIsOpen(false);         // giá trị cố định → dùng thẳng
setColor(selectedColor);  // từ biến ngoài → dùng thẳng

// Cần updater khi giá trị phụ thuộc vào state trước
setCount(c => c + 1);     // phụ thuộc vào count trước đó
setItems(i => [...i, newItem]); // phụ thuộc vào items trước đó
setToggle(t => !t);       // phụ thuộc vào toggle trước đó
```

---

## Kiểm tra hiểu bài — Tính kết quả

### Câu hỏi 1

```jsx
// number ban đầu = 0. Sau khi click, number = ?
setNumber(number + 1);
setNumber(number + 1);
```

<details>
<summary>👉 Xem đáp án</summary>

**number = 1**

Cả hai đều là `setNumber(0 + 1)` — "thay bằng 1". Gọi 2 lần vẫn chỉ ra 1.
</details>

---

### Câu hỏi 2

```jsx
// number ban đầu = 0. Sau khi click, number = ?
setNumber(n => n + 1);
setNumber(n => n + 1);
```

<details>
<summary>👉 Xem đáp án</summary>

**number = 2**

| Queue | `n` | Trả về |
|-------|-----|--------|
| `n => n + 1` | 0 | 1 |
| `n => n + 1` | 1 | 2 |
</details>

---

### Câu hỏi 3

```jsx
// number ban đầu = 0. Sau khi click, number = ?
setNumber(number + 5);
setNumber(n => n + 1);
setNumber(number + 5);
```

<details>
<summary>👉 Xem đáp án</summary>

**number = 5**

| Queue | `n` | Trả về |
|-------|-----|--------|
| `"thay bằng 5"` | 0 (không dùng) | 5 |
| `n => n + 1` | 5 | 6 |
| `"thay bằng 5"` (vì `number=0`, `0+5=5`) | 6 (không dùng) | 5 |

Lần cuối cùng dùng `number + 5 = 0 + 5 = 5` (giá trị từ bản chụp) — ghi đè lên kết quả trước.
</details>

---

### Câu hỏi 4 (thực tế)

```jsx
// Sửa code bị lỗi này — pending bị âm khi click nhanh
async function handleClick() {
  setPending(pending + 1);
  await delay(3000);
  setPending(pending - 1);
  setCompleted(completed + 1);
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
async function handleClick() {
  setPending(p => p + 1);      // updater: dùng giá trị mới nhất
  await delay(3000);
  setPending(p => p - 1);      // updater: dùng giá trị mới nhất
  setCompleted(c => c + 1);    // updater: dùng giá trị mới nhất
}
```

Vấn đề: `pending` và `completed` trong code gốc đều dùng giá trị từ "bản chụp" lúc click — không phản ánh các lần click khác xảy ra trong 3 giây chờ. Dùng updater function để luôn tính từ giá trị mới nhất.
</details>

---

## Tóm tắt

```
Batching:
  React gom tất cả setState trong một event handler
  → xử lý sau khi handler chạy xong
  → chỉ re-render 1 lần duy nhất

Hai loại đối số cho setter:
  setN(5)          → "thay bằng 5" (bỏ qua kết quả trước trong queue)
  setN(n => n + 1) → updater function (nhận kết quả trước, tính giá trị mới)

Dùng updater function khi:
  • Cần gọi setter nhiều lần trong 1 handler và muốn cộng dồn
  • Code bất đồng bộ (async/await, setTimeout) — giá trị bản chụp có thể lỗi thời
  • Muốn chắc chắn luôn tính từ state mới nhất

Updater function phải thuần túy:
  • Chỉ tính toán và trả về giá trị mới
  • Không được có side effect bên trong
```

