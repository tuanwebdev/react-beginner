# Truyền Dữ Liệu Sâu với Context

> Hướng dẫn từ [react.dev/learn/passing-data-deeply-with-context](https://react.dev/learn/passing-data-deeply-with-context)

Thông thường bạn truyền dữ liệu từ cha xuống con qua props. Nhưng khi cây component sâu và nhiều component cần cùng một dữ liệu, việc truyền props qua nhiều tầng trở nên rườm rà — gọi là **prop drilling**. **Context** giải quyết điều này bằng cách cho phép component cha "phát sóng" dữ liệu đến bất kỳ component con nào — dù sâu bao nhiêu — mà không cần truyền qua từng tầng.

---

## Vấn đề: Prop Drilling

Khi dữ liệu cần đi qua nhiều component trung gian không thực sự dùng nó:

```
App (có state: theme = 'dark')
  ↓ theme prop
  Layout
    ↓ theme prop (Layout không dùng, chỉ truyền tiếp)
    Sidebar
      ↓ theme prop (Sidebar không dùng, chỉ truyền tiếp)
      NavItem   ← component thực sự cần theme
      NavItem   ← component thực sự cần theme
```

```jsx
// ❌ Prop drilling — truyền theme qua 3 tầng dù chỉ NavItem dùng
function App() {
  const [theme, setTheme] = useState('dark');
  return <Layout theme={theme} />;      // Layout không dùng theme
}

function Layout({ theme }) {
  return <Sidebar theme={theme} />;     // Sidebar không dùng theme
}

function Sidebar({ theme }) {
  return (
    <>
      <NavItem theme={theme} />         // NavItem mới dùng
      <NavItem theme={theme} />
    </>
  );
}
```

Càng nhiều tầng, càng nhiều props thừa, code càng khó bảo trì.

---

## Giải pháp: Context — "Phát sóng" dữ liệu

Context giống như một **kênh radio**: component cha phát sóng, component con ở bất kỳ độ sâu nào có thể "bắt sóng" — mà không cần truyền qua các tầng trung gian.

```
App (Provider: theme = 'dark')
  Layout           ← không cần nhận theme prop
    Sidebar        ← không cần nhận theme prop
      NavItem      ← useContext(ThemeContext) → 'dark' ✓
      NavItem      ← useContext(ThemeContext) → 'dark' ✓
```

---

## 3 Bước dùng Context

---

### Bước 1: Tạo Context

Tạo file riêng cho context và export nó:

```jsx
// LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
//                                        ↑
//                              Giá trị mặc định (dùng khi không có Provider)
```

`createContext(defaultValue)` nhận vào giá trị mặc định — được dùng khi component đọc context nhưng không có Provider nào bao quanh nó.

---

### Bước 2: Đọc Context (useContext)

Component muốn nhận dữ liệu từ context dùng hook `useContext`:

```jsx
// Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

// Trước: nhận level qua props
export default function Heading({ level, children }) { ... }

// Sau: đọc level từ context — không cần prop nữa!
export default function Heading({ children }) {
  const level = useContext(LevelContext);  // ← đọc từ context

  switch (level) {
    case 1: return <h1>{children}</h1>;
    case 2: return <h2>{children}</h2>;
    case 3: return <h3>{children}</h3>;
    case 4: return <h4>{children}</h4>;
    default: throw Error('Unknown level: ' + level);
  }
}
```

`useContext` là Hook — chỉ được gọi ở đầu component, không trong vòng lặp hay điều kiện.

---

### Bước 3: Cung cấp Context (Provider)

Bọc component con trong `<Context value={...}>` để cung cấp giá trị:

```jsx
// Section.js
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section>
      <LevelContext value={level}>  {/* ← Provider */}
        {children}
      </LevelContext>
    </section>
  );
}
```

Bây giờ bất kỳ `<Heading>` nào bên trong `<Section level={3}>` sẽ tự động nhận `level = 3` — không cần truyền prop!

```jsx
// App.js — gọn hơn rất nhiều!
export default function Page() {
  return (
    <Section level={1}>
      <Heading>Tiêu đề</Heading>          {/* tự biết level=1 */}
      <Section level={2}>
        <Heading>Mục 2</Heading>          {/* tự biết level=2 */}
        <Section level={3}>
          <Heading>Mục con</Heading>      {/* tự biết level=3 */}
        </Section>
      </Section>
    </Section>
  );
}
```

---

## Nâng cao: Vừa đọc vừa cung cấp Context

Component có thể vừa đọc context từ cha, vừa cung cấp context mới (đã biến đổi) cho con. Điều này cho phép tự động tăng cấp độ lồng:

```jsx
// Section.js — tự động tăng level, không cần truyền prop level!
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);  // đọc level từ cha
  return (
    <section>
      <LevelContext value={level + 1}>    {/* cung cấp level+1 cho con */}
        {children}
      </LevelContext>
    </section>
  );
}
```

Kết quả: Không cần truyền prop `level` ở bất kỳ đâu!

```jsx
// App.js — cực kỳ gọn!
export default function Page() {
  return (
    <Section>                   {/* level=1 (default) */}
      <Heading>Tiêu đề</Heading>    {/* h1 */}
      <Section>               {/* level=2 */}
        <Heading>Mục</Heading>    {/* h2 */}
        <Section>           {/* level=3 */}
          <Heading>Mục con</Heading>  {/* h3 */}
        </Section>
      </Section>
    </Section>
  );
}
```

---

## Context "xuyên" qua các component trung gian

Context đi thẳng đến bất kỳ component nào cần nó — không quan tâm có bao nhiêu component ở giữa:

```jsx
// Dù Post nằm sâu đến đâu, Heading bên trong vẫn tự biết level đúng
function ProfilePage() {
  return (
    <Section>                    {/* level 1 */}
      <Heading>My Profile</Heading>
      <AllPosts />
    </Section>
  );
}

function AllPosts() {
  return (
    <Section>                    {/* level 2 */}
      <Heading>Posts</Heading>
      <RecentPosts />
    </Section>
  );
}

function RecentPosts() {
  return (
    <Section>                    {/* level 3 */}
      <Heading>Recent Posts</Heading>
      <Post title="Flavors of Lisbon" />  {/* Post chứa Section+Heading riêng */}
    </Section>
  );
}

function Post({ title }) {
  return (
    <Section>                    {/* level 4 — tự động! */}
      <Heading>{title}</Heading>  {/* h4 — không cần biết mình đang ở tầng nào */}
    </Section>
  );
}
```

Giống như CSS inheritance: đặt `color: blue` cho `<div>`, tất cả con cháu kế thừa — trừ khi bị ghi đè.

---

## Ví dụ thực tế: Theme + User + Routing

```jsx
// contexts/ThemeContext.js
import { createContext } from 'react';
export const ThemeContext = createContext('light');

// contexts/UserContext.js
import { createContext } from 'react';
export const UserContext = createContext(null);

// App.js — cung cấp nhiều context cùng lúc
export default function App() {
  const [theme, setTheme] = useState('light');
  const [user, setUser]   = useState(null);

  return (
    <ThemeContext value={theme}>
      <UserContext value={user}>
        <Page />
      </UserContext>
    </ThemeContext>
  );
}

// NavBar.js — đọc từ 2 context khác nhau
function NavBar() {
  const theme = useContext(ThemeContext);
  const user  = useContext(UserContext);

  return (
    <nav className={`nav nav--${theme}`}>
      {user ? <span>Xin chào, {user.name}!</span> : <LoginButton />}
    </nav>
  );
}

// DarkModeToggle.js — cập nhật context bằng cách lift state + truyền setter
function DarkModeToggle({ onToggle }) {
  const theme = useContext(ThemeContext);
  return (
    <button onClick={onToggle}>
      {theme === 'dark' ? '☀️ Sáng' : '🌙 Tối'}
    </button>
  );
}
```

---

## Context + State: Kết hợp hoàn hảo

Context thường kết hợp với state để dữ liệu có thể thay đổi:

```jsx
// ThemeProvider.js — tách thành component riêng cho gọn
import { createContext, useState, useContext } from 'react';

const ThemeContext = createContext('light');
const ThemeDispatchContext = createContext(null);

// Custom Provider component — gom state + context lại
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext value={theme}>
      <ThemeDispatchContext value={setTheme}>
        {children}
      </ThemeDispatchContext>
    </ThemeContext>
  );
}

// Custom Hooks để đọc context — tránh import nhiều nơi
export function useTheme() {
  return useContext(ThemeContext);
}

export function useSetTheme() {
  return useContext(ThemeDispatchContext);
}

// Dùng trong App:
export default function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

// Dùng trong component bất kỳ:
function Header() {
  const theme      = useTheme();
  const setTheme   = useSetTheme();

  return (
    <header className={theme}>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle theme
      </button>
    </header>
  );
}
```

---

## Trước khi dùng Context — Kiểm tra 2 lựa chọn khác

Context rất hấp dẫn nhưng **dễ bị dùng thừa**. Trước khi dùng, thử:

### Lựa chọn 1: Truyền props bình thường

```jsx
// Đôi khi props rõ ràng hơn context — dễ trace data flow
// Nếu chỉ qua 2-3 tầng → props vẫn ổn
<UserCard user={user} />
```

### Lựa chọn 2: Tách component + dùng children

Nếu dữ liệu đi qua nhiều component trung gian không dùng nó → có thể component chưa được tách đúng:

```jsx
// ❌ Layout nhận posts chỉ để truyền tiếp — không dùng
function App() {
  const posts = fetchPosts();
  return <Layout posts={posts} />;
}
function Layout({ posts }) {
  return <div><Posts posts={posts} /></div>;  // Layout không dùng posts
}

// ✅ Tách component đúng — dùng children
function App() {
  const posts = fetchPosts();
  return (
    <Layout>
      <Posts posts={posts} />    {/* App quyết định content, không phải Layout */}
    </Layout>
  );
}
function Layout({ children }) {
  return <div>{children}</div>;  // Layout gọn, không biết về posts
}
```

---

## Khi nào nên dùng Context

| Use case | Ví dụ |
|---|---|
| **Theme** | Dark/light mode, màu sắc app |
| **Người dùng hiện tại** | User đang đăng nhập, avatar, quyền |
| **Routing** | Route hiện tại, active link |
| **Ngôn ngữ/Locale** | i18n, timezone |
| **State toàn app** | Kết hợp với Reducer để quản lý state phức tạp |
| **Config** | Feature flags, API endpoints |

---

## Ghi đè Context ở cây con

Context gần nhất sẽ thắng — bạn có thể ghi đè context cho một phần của cây:

```jsx
// App cung cấp theme='light' cho toàn bộ
<ThemeContext value="light">
  <Header />
  <Main />
  {/* Ghi đè theme cho phần admin */}
  <ThemeContext value="dark">
    <AdminPanel />   {/* nhận 'dark' thay vì 'light' */}
  </ThemeContext>
</ThemeContext>
```

---

## Sơ đồ tổng quan

```
createContext(defaultValue)  ←  Tạo Context object
        ↓
<MyContext value={data}>     ←  Provider: cung cấp giá trị
  <ChildA />                     ChildA có thể đọc MyContext
  <ChildB>                       ChildB có thể đọc MyContext
    <GrandChild />               GrandChild có thể đọc MyContext
  </ChildB>                      (dù sâu bao nhiêu!)
</MyContext>
        ↓
useContext(MyContext)         ←  Consumer: đọc giá trị gần nhất

Quy tắc:
  • Đọc giá trị từ Provider GẦN NHẤT phía trên
  • Nếu không có Provider → dùng defaultValue
  • Nhiều context khác nhau không ảnh hưởng nhau
  • Context thay đổi → tất cả consumer re-render
```

---

## Bài tập tự luyện

### Bài 1: Thay prop drilling bằng context

```jsx
// App truyền imageSize → List → Place → PlaceImage (3 tầng!)
// Nhưng chỉ PlaceImage thực sự dùng imageSize
// Hãy dùng context để xóa prop imageSize khỏi List và Place

export default function App() {
  const [isLarge, setIsLarge] = useState(false);
  const imageSize = isLarge ? 150 : 100;
  return (
    <>
      <label>
        <input type="checkbox" checked={isLarge}
          onChange={e => setIsLarge(e.target.checked)} />
        Ảnh lớn
      </label>
      <List imageSize={imageSize} />    {/* ← bắt đầu prop drilling */}
    </>
  );
}

function List({ imageSize }) {
  return places.map(p => <Place key={p.id} place={p} imageSize={imageSize} />);
}

function Place({ place, imageSize }) {
  return <PlaceImage place={place} imageSize={imageSize} />;
}

function PlaceImage({ place, imageSize }) {
  return <img src={place.imageUrl} width={imageSize} height={imageSize} />;
}
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
// ImageSizeContext.js
import { createContext } from 'react';
export const ImageSizeContext = createContext(100);  // default 100px

// App.js
import { useState } from 'react';
import { ImageSizeContext } from './ImageSizeContext.js';

export default function App() {
  const [isLarge, setIsLarge] = useState(false);
  const imageSize = isLarge ? 150 : 100;
  return (
    <ImageSizeContext value={imageSize}>   {/* ← cung cấp context */}
      <label>
        <input type="checkbox" checked={isLarge}
          onChange={e => setIsLarge(e.target.checked)} />
        Ảnh lớn
      </label>
      <List />           {/* ← không cần imageSize prop nữa! */}
    </ImageSizeContext>
  );
}

function List() {                                    // ← không nhận imageSize
  return places.map(p => <Place key={p.id} place={p} />);
}

function Place({ place }) {                          // ← không nhận imageSize
  return <PlaceImage place={place} />;
}

function PlaceImage({ place }) {
  const imageSize = useContext(ImageSizeContext);    // ← đọc từ context
  return <img src={place.imageUrl} width={imageSize} height={imageSize} />;
}
```
</details>

---

### Bài 2: Tạo ThemeContext đầy đủ

Tạo một ThemeContext cho phép toggle dark/light mode từ bất kỳ đâu:

```jsx
// Yêu cầu:
// 1. Tạo ThemeContext với giá trị mặc định 'light'
// 2. Tạo ThemeProvider component
// 3. Tạo custom hook useTheme()
// 4. Dùng trong App và Button component
```

<details>
<summary>👉 Xem đáp án</summary>

```jsx
// ThemeContext.js
import { createContext, useState, useContext } from 'react';

const ThemeContext        = createContext('light');
const ThemeToggleContext = createContext(null);

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const toggle = () => setTheme(t => t === 'light' ? 'dark' : 'light');

  return (
    <ThemeContext value={theme}>
      <ThemeToggleContext value={toggle}>
        {children}
      </ThemeToggleContext>
    </ThemeContext>
  );
}

export function useTheme()       { return useContext(ThemeContext); }
export function useThemeToggle() { return useContext(ThemeToggleContext); }

// App.js
import { ThemeProvider, useTheme, useThemeToggle } from './ThemeContext.js';

function ToggleButton() {
  const theme  = useTheme();
  const toggle = useThemeToggle();
  return (
    <button onClick={toggle}>
      Chuyển sang {theme === 'light' ? 'tối' : 'sáng'}
    </button>
  );
}

function Page() {
  const theme = useTheme();
  return <div className={`page page--${theme}`}><ToggleButton /></div>;
}

export default function App() {
  return (
    <ThemeProvider>
      <Page />
    </ThemeProvider>
  );
}
```
</details>

---

## Tóm tắt

```
Context giải quyết: Prop Drilling (truyền props qua nhiều tầng không dùng đến)

3 bước:
  1. Tạo:    export const MyCtx = createContext(defaultValue)
  2. Đọc:    const value = useContext(MyCtx)   [trong component con]
  3. Cung cấp: <MyCtx value={data}>{children}</MyCtx>  [trong component cha]

Đặc điểm:
  • Đọc từ Provider GẦN NHẤT phía trên
  • Không có Provider → dùng defaultValue
  • Xuyên qua mọi component trung gian
  • Thay đổi giá trị → tất cả consumer re-render
  • Nhiều context độc lập → không ảnh hưởng nhau

Trước khi dùng — thử:
  1. Truyền props bình thường (rõ ràng hơn)
  2. Tách component + dùng children (giảm tầng trung gian)

Khi nào dùng:
  theme, current user, routing, i18n, global state (kết hợp Reducer)

Best practice:
  • Tạo custom hook useMyContext() để đóng gói logic đọc context
  • Tách Provider thành component riêng
  • Lưu cả value lẫn setter vào context riêng biệt
```
