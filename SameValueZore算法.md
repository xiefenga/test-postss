---
title: SameValueZore算法
created: 2025-12-16T06:38:41.746Z
updated: 2025-12-16T06:38:41.746Z
slug: SameValueZore算法
tags: []
description: ""
---

---
title: SameValueZore算法
created: 2023-5-31 15:14:12
updated: 2023-5-31 15:35:13
---

[SameValueZero](https://tc39.es/ecma262/#sec-samevaluezero)（零值相等）是 ECMAScript 规范新增的相等性比较算法，类似于同值相等（但 `+0` 和 `-0` 被视为相等）

零值相等被许多内置运算使用，例如：`Map` 的 `key` 值比较、`Set` 的 `key` 值比较

```javascript
const set = new Set([0, NaN])
set.has(-0) // true
set.has(NaN) // true
```

该算法没有通过 JavaScript API 公开，但可以通过自定义代码实现

```typescript
const sameValueZero = (x: any, y: any) => {
  if (typeof x === 'number' && typeof y === '') {
    // x 和 y 相等（可能是 -0 和 0）或它们都是 NaN
    return x === y || (x !== x && y !== y)
  }
  return x === y
}
```

**其他算法** 

JavaScript 中能对两个值进行相等性比较的操作有：`==`、`===` 、`Object.is` 

这三种操作在进行相等性比较时有不同的行为，因此也对应三个 JavaScript 相等算法

- [IsLooselyEqual](https://tc39.es/ecma262/#sec-islooselyequal)（宽松相等）
  - `==` 进行比较时采用的算法
  - 比较时会进行类型转换，以及对一些类型进行特殊处理
  - 按照 IEEE 754 标准对 `NaN`、`+0` 和 `-0` 特殊处理（`NaN != NaN`、`-0 == +0`）
- [IsStrictlyEqual](https://tc39.es/ecma262/#sec-isstrictlyequal)（严格相等）
  - `===` 进行比较时采用的算法
  - 比较时不会进行类型转换，不同类型的操作数会返回 `false` 
  - 同样按照 IEEE 754 标准对 `NaN`、`+0` 和 `-0` 特殊处理（`NaN != NaN`、`-0 == +0`）
- [SameValue](https://tc39.es/ecma262/#sec-samevalue)（同值相等）
  - `Object.is` 采用的算法
  - 比较时不会进行类型转换
  - 对 `NaN`、`-0` 和 `+0` 不进行特殊处理

```javascript
NaN == NaN // false
0 == -0 // true

NaN === NaN // false
0 === -0 // true

Object.is(NaN, NaN) // true
Object.is(0, -0) // false
```

**参考**

- [JavaScript 中的相等性判断](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness#%E7%9B%B8%E7%AD%89%E6%80%A7%E6%96%B9%E6%B3%95%E6%AF%94%E8%BE%83) 
- 《JavaScript高级程序设计》第四版
