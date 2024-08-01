---
title: Tiptap
date: 2024-07-31
categories: [Front-end, utils]
tags: [frontend, Tiptap]     # TAG names should always be lowercase
# comments: false 
pin: false
math: true
mermaid: true
toc: true
---

# Tiptap

## Core Concepts 核心

### Schema 结构描述

that defines how your content is structured

定义了内容是怎样被组织起来的，哪种类型的节点会被加载，节点上面又有哪些特性

e.g.

```javascript
// the underlying ProseMirror schema
{
  nodes: {
    doc: {
      content: 'block+',
    },
    paragraph: {
      content: 'inline*',
      group: 'block',
      parseDOM: [{ tag: 'p' }],
      toDOM: () => ['p', 0],
    },
    text: {
      group: 'inline',
    },
  },
}

```

use the schema API to create Node

```javascript
// the Tiptap schema API
import { Node } from '@tiptap/core'

const Document = Node.create({
  name: 'doc',
  topNode: true,
  content: 'block+',
})

const Paragraph = Node.create({
  name: 'paragraph',
  group: 'block',
  content: 'inline*',
  parseHTML() {
    return [{ tag: 'p' }]
  },
  renderHTML({ HTMLAttributes }) {
    return ['p', HTMLAttributes, 0]
  },
})

const Text = Node.create({
  name: 'text',
  group: 'inline',
})
```

- #### Content

  different content the node can have

  节点可以包含的内容

  ```javascript
  Node.create({
    // must have one or more blocks
    content: 'block+',
  
    // must have zero or more blocks
    content: 'block*',
  
    // allows all kinds of 'inline' content (text or hard breaks)
    content: 'inline*',
  
    // must not have anything else than 'text'
    content: 'text*',
  
    // can have one or more paragraphs, or lists (if lists are used)
    content: '(paragraph|list?)+',
  
    // must have exact one heading at the top, and one or more blocks below
    content: 'heading block+',
  })
  ```

- #### Marks

  which marks are allowed inside of a node

  可以被允许的标记

  ```javascript
  Node.create({
    // allows only the 'bold' mark
    marks: 'bold',
  
    // allows only the 'bold' and 'italic' marks
    marks: 'bold italic',
  
    // allows all marks
    marks: '_',
  
    // disallows all marks
    marks: '',
  })
  ```

- Group

  add this node to a group of extension, which can be referrend to in the content attribute of the schema

  （不是很理解）

  ```javascript
  Node.create({
    // add to 'block' group
    group: 'block',
  
    // add to 'inline' group
    group: 'inline',
  
    // add to 'block' and 'list' group
    group: 'block list',
  })
  ```

- other attributes

  | 属性       | 描述                                                         |
  | ---------- | ------------------------------------------------------------ |
  | inline     | `inline: true` nodes are rendered in line withe the text.    |
  | atom       | `atom: true` aren’t directly editable and should be treated as a single unit. |
  | selectable | `selectable: true` an invisible node selection you can configure |

  

### state `editor.state` 编辑器状态

The document is stored in a state.
All changes are applied as **transactions** to the state.

- current content
- current position
- current selection `state.selection`

### Extensions 扩展

Extensions add nodes, marks and/or functionalities to the editor.  [Extensions in Tiptap](https://tiptap.dev/docs/editor/extensions/overview)



## API

### Editor instance 编辑器实例



### Commands

### Node Positions



## Custom extensions

