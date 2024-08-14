---
title: TailwindCSS 动态类名问题
date: 2024-08-13
categories: [Front-end]
tags: [frontend, TailwindCSS]  
---

# TailwindCSS 动态类名问题

### 动态设置类名

- `top-[*px]` 这样的动态类名是通过 `JIT` (Just-In-Time) 模式生成的，但这些类名必须在编译时被识别，如果在运行时生成类名，可能无法识别

https://tailwindcss.com/docs/content-configuration#dynamic-class-names

Tailwind 提取类名的方法最重要的一点是，它只能找到源文件中以完整字符串形式存在的类。 如果使用字符串插值或将部分类名连接在一起，Tailwind 将无法找到它们，因此也不会生成相应的 CSS，❌不能使用动态拼接字符串的方式

- 字符串模板

```tsx
X // 通过字符串模板的方式，动态设置类名
const topClass = `top-[${selectionHeight + 56}px]`
```

- classnames 第三方库

```jsx
import classNames from 'classnames'

const dynamicClass = classNames({
  'text-white p-4': true,
  [`top-${topValue}`]: topValue !== undefined
});
```

