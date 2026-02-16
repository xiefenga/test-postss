---
title: 协变与逆变——以TypeScript为例
created: 2026-01-13T13:19:58.684Z
updated: 2026-01-13T13:19:58.684Z
slug: 协变与逆变——以TypeScript为例
tags: []
description: ""
---

---
title: 协变与逆变——以TypeScript为例
created: 2023-7-19 23:56:09
updated: 2023-12-27 09:48:57
---

## 子类型

根据**里氏替换原则**（Liskov substitution principle），如果在期望类型 `T` 的实例的任何地方，都可以安全地使用类型 `S` 的实例，那么称类型 `S` 是类型 `T` 的子类型

我们使用这样的表达式定义子类型：对于 `A` 和 `B` 两个类型，如果 `A` 是 `B` 的子类型则可以表达为 `A <: B`

在类型系统中有一些概念用于描述类型关系中的子类型关系，同时也定义了类型的可赋值性和替代性

- 协变性（Covariance）：如果一个类型保留其底层类型的子类型关系，就称该类型具有协变性
- 逆变性（Contravariance）：如果一个类型颠倒了其底层类型的子类型关系，则称该类型具有逆变性
- 自反性（Reflexivity）：一个类型自己就是自己的子类型
- 转换性（Transitivity）：如果 `A` 是 `B` 的子类型，`B` 是 `C` 的子类型，那么 `A` 是 `C` 的子类型

如下的类型使用表达式可以表示为：`Square <: Rectangle <: Shape`

```typescript
class Shape {}

class Rectangle extends Shape {}

class Square extends Rectangle {}

declare const rectangle: Rectangle;

// 任何使用父类型的地方都可以传递子类型变量
const shape: Shape = rectangle;
```

## 协变

通常来说**数组**具有**协变性**，由于 `Rectangle <: Shape`，所以 `Rectangle[] <: Shape[]`

```typescript
declare const rectangles: Rectangle[];

const shapes: Shape[] = rectangles;
```

**函数的返回值**一般也具有**协变性**，所以 `() => Rectangle <: () => Shape`

```typescript
declare function makeRectangle(): Rectangle;
declare function makeShape(): Shape;

function useFactory(factory: () => Shape): Shape {
  return factory();
}

const shape1: Shape = useFactory(makeShape);
const shape2: Shape = useFactory(makeRectangle);
```

TypeScript 中**非函数签名包装类型**通常也具有**协变性**

例如有一个包装类型 `List<T>`，`TypeScript` 依旧会认为 `List<Rectangle> <: List<Shape>`

```typescript
interface List<T> {}

declare function func(box: List<Shape>): void;

declare const shapeBox: List<Shape>;

declare const rectangleBox: List<Rectangle>;

func(shapeBox);

func(rectangleBox);
```

## 逆变

和协变相反，逆变则是颠倒了父子类型关系，假设数组具有逆变性，则 `Triangle[]` 是 `Shape[]` 的父类型

在大部分编程语言中，**函数的实参**具有**逆变性**

```typescript
declare function drawShape(shape: Shape): void;
declare function drawRectangle(rectangle: Rectangle): void;

declare function render(drawFunc: (rectangle: Rectangle) => void): void;

render(drawRectangle);
render(drawShape);
```

此时 `(_: Shape) => void <: (_: Rectangle) => void`，与 `Rectangle <: Shape` 相反

这在逻辑上是合理的：

1. `render` 函数调用时会按照 `Rectangle` 来约束的类型
2. 而 `drawShape ` 实际上按照 `Shape` 来约束自己的调用，不会使用 `Rectangle` 的其他数据

对于传递的函数的参数而言，只要保证使用的是期望类型的父类型就能够安全的调用

## 双变

大部分编程语言中**函数的实参**具有**逆变性**，但是 TypeScript 中函数可以同时接收实参为子类型/父类型的函数

也就是**双变**：类型的底层类型的子类型关系决定了它们互为子类型

所以 TypeScript 中 `(_: Shape) => void` 和 `(_: Rectangle) => void` 互为子类型

> 这是故意做出的设计决策，目的是方便实现常见的 JavaScript 编程模式。不过，这可能导致运行时问题

TypeScript 2.6 版本引入了 `strictFunctionTypes` 配置用于开启更严格的函数类型检查（启用参数逆变）

## 不变

除了协变、逆变、双变，类型系统中还具有**不变性**

如果一个类型**不考虑其底层类型的子类型关系**，就称该类型具有不变性

比如 C# 中的 `List<T>` 具有不变性，`List<Shape>` 和 `List<Rectangle>` 之间不存在父子类型关系

## strictFunctionTypes

在开启 `strictFunctionTypes` 配置之后就可以开启**函数参数逆变类型检查**，但是 TypeScript 中并不是所有的函数/方法能够被检查

对象方法的类型声明有两种形式：`property` 和 `method`

```typescript
// method 声明
interface T1 {
  func(arg: string): number;
}

// property 声明
interface T2 {
  func: (arg: string) => number;
}
```

除了一般的函数声明，只有 `property` 形式的声明能享受到**基于逆变的参数类型检查** ，`method` 声明（以及构造函数声明）是无法享受到逆变的参数类型检查的

究其原因是为了兼容已有的内置类型，或者说需要保证 `Rectangle[] <: Shape[]` 成立

可以举个包装类型的例子来表达：

```typescript
// strictFunctionTypes: true
interface Box<T> {
  push(...item: T[]): void;
}

function boxShapeDemo(box: Box<Shape>) {
  box.push(new Shape());
}

declare const shapeBox: Box<Shape>;

declare const rectangleBox: Box<Rectangle>;

boxShapeDemo(shapeBox);

boxShapeDemo(rectangleBox);
```

因为 `Rectangle <: Shape`，所以 `Box<Rectangle> <: Box<Shape>`

这里使用了 `method` 形式去声明方法，如果 TypeScript 会对 `method` 形式的声明进行逆变类型检查

要模拟这种情况只需要将 `method` 形式改为 `property` 形式

```typescript
interface Box<T> {
  push: (...item: T[]) => void;
}

boxShapeDemo(triangleBox);
```

![](https://xf-blog-imgs.oss-cn-hangzhou.aliyuncs.com/images/20230719233337.png)

此时 `Box<Rectangle> <: Box<Shape>` 就会不成立，内置的 `Array` 等类型就是这样的情况

```typescript
interface Array<T> {
  push(...items: T[]): number;
}
```

在父子类型兼容性比较时要将它们视为两个完整的类型比较，也就是 `Rectangle[]` 的每一个成员（属性、方法）是否都能对应的赋值给 `Shape[]`

以 `push` 方法为例也就是 `Rectangle -> void <: Shape -> void` (忽略多个参数情况)

`Rectangle -> void <: Shape -> void` 在逆变的情况下表明 `Shape <: Rectangle`

这样就产生了前后矛盾，所以此时 TypeScript 仍然强制使用参数逆变的规则进行检查以兼容内置类型

对于一般的方法为了能够接受**逆变参数类型检查**，可以使用 [typescript-eslint/method-signature-style](https://github.com/typescript-eslint/typescript-eslint/blob/main/packages/eslint-plugin/docs/rules/method-signature-style.md) 规则
