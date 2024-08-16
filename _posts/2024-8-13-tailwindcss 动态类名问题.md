---
title: TailwindCSS 动态类名问题
date: 2024-08-13
categories: [Front-end]
tags: [frontend, TailwindCSS]
description: Tailwind 提取类名的方法最重要的一点是，它只能找到源文件中以完整字符串形式存在的类。 如果使用字符串插值或将部分类名连接在一起，Tailwind 将无法找到它们，因此也不会生成相应的 CSS 
---

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

### 不能构造动态类名的原因

Tailwind 本质是一个 PostCSS 插件，在构建过程中会处理你的 CSS 文件。它会根据你的配置文件生成大量的实用类，并将这些类插入到最终的 CSS 文件中。Tailwind 的构建过程依赖于对你的代码进行静态分析，以确定哪些类需要生成。如果类名是动态构造的，Tailwind 无法在构建阶段识别这些类，导致这些类不会被包含在最终的 CSS 文件中。

使用时必须是 TailwindCss 之前构造好的

```
const className = classNames({
  'bg-blue-500': isBlue,
  'bg-red-500': !isBlue,
})
```

### [安全列表类](https://tailwindcss.com/docs/content-configuration#safelisting-classes)

- 依靠 `content` 配置来告诉 Tailwind 尽可能多地生成哪些类

	```
	content: ['./pages/**/*.{ts,tsx}', 
						'./components/**/*.{ts,tsx}', 
						'./app/**/*.{ts,tsx}', 
						'./src/**/*.{ts,tsx}'],
	```

- 安全列表 `safelist` 是最后的手段，仅应在无法扫描特定内容以查找类名的情况下使用

  ```
  safelist: [
      'bg-red-500',
      'text-3xl',
      'lg:text-4xl',
  ]
  ```

- 如果需要将大量的类列入安全列表，可以使用正则表达式

  ```
  {
  	pattern: /bg-(red|green|blue)-(100|200|300)/,
  },
  ```







