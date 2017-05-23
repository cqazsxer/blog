layout: w
title: sass 简明使用
date: 2016-08-24 15:29:07
tags: sass
---

### 核心原则
**尽可能让 Sass 代码保持简洁。除非是绝对需要，否则绝没有必要构建复杂的系统。**

<!-- more -->

### 一 **变量**
满足所有下述标准时方可创建新变量：

- 该值至少重复出现了两次；
- 该值至少可能会被更新一次；
- 该值所有的表现都与变量有关（非巧合）。

#### 使用
所有变量以$开头。
```css
$blue : #1875e7;　
```

如果变量需要镶嵌在字符串之中，就必须需要写在#{}之中。
```
$side : left;
.rounded {
　　border-#{$side}-radius: 5px;
}
```

#### 列表(数组)
列表就是 Sass 的数组，用于保存任意类型的数值（包括列表，从而产生嵌套列表）。

```css
// Yep
$font-stack: ('Helvetica', 'Arial', sans-serif);

// Yep
$font-stack: (
  'Helvetica',
  'Arial',
  sans-serif,
);

// Nope
$font-stack: 'Helvetica' 'Arial' sans-serif;

// Nope
$font-stack: 'Helvetica', 'Arial', sans-serif;

// Nope
$font-stack: ('Helvetica', 'Arial', sans-serif,);
```


#### 多变量或Maps:
在 Sass 中，可以使用 `map` 这种数据结构, 可以映射关联数组、哈希表甚至是 Javascript 对象。

使用 maps 比使用多个不同的变量有明显优势。最重要的优势就是 map 的遍历功能，这在多个不同变量中是不可能实现的。

另一个支持使用 map 的原因，是它可以创建 map-get() 函数以提供友好 API 的功能。比如，思考一下下述 Sass 代码：

```css
/// Z-indexes map, gathering all Z layers of the application
/// @access private
/// @type Map
/// @prop {String} key - Layer’s name
/// @prop {Number} value - Z value mapped to the key
$z-indexes: (
  'modal': 5000,
  'dropdown': 4000,
  'default': 1,
  'below': -1,
);

/// Get a z-index value from a layer name
/// @access public
/// @param {String} $layer - Layer’s name
/// @return {Number}
/// @require $z-indexes
@function z($layer) {
  @return map-get($z-indexes, $layer);
}
```

### 二 字符串
#### 单位
将一个单位添加给数字的时候，实际上是让该数值乘以1个单位。

```
$value: 42;

// Yep
$length: $value * 1px;

// Nope
$length: $value + px;

```

#### 计算
**最高级运算应该始终被包裹在括号中。**
这么做不仅是为了提高可读性，也是为了防止一些 Sass 强制要求对括号内内容计算的极端情况。

### 三 选择器嵌套

#### 父选择器的标识符&
一般用法：
```css
article a {
  color: blue;
  &:hover { color: red }
}
```
从 Sass3.3 开始，可以在同一行中使用最近选择器引用(&)来实现高级选择器:
```css
.foo {
  &-bar {
    color: red;
  }
}
```
生成的css
```ccs
.foo-bar {
  color: red;
}
```

### 四 注释
#### 注释风格
- 标准的CSS注释 /* comment */ ，会保留到编译后的文件。
- 单行注释 // comment，只保留在SASS源文件中，编译后被省略。

#### 文档
每一个旨在代码库中复用的变量、函数、混合宏和占位符，都应该使用 SassDoc 记录下来作为全部 API 的一部分。

这里有一个深入整合 SassDoc 生成文档的例子：
```css
/// Mixin helping defining both `width` and `height` simultaneously.
///
/// @author Hugo Giraudel
///
/// @access public
///
/// @param {Length} $width - Element’s `width`
/// @param {Length} $height [$width] - Element’s `height`
///
/// @example scss - Usage
///   .foo {
///     @include size(10em);
///   }
///
///   .bar {
///     @include size(100%, 10em);
///   }
///
/// @example css - CSS output
///   .foo {
///     width: 10em;
///     height: 10em;
///   }
///
///   .bar {
///     width: 100%;
///     height: 10em;
///   }
@mixin size($width, $height: $width) {
  width: $width;
  height: $height;
}
```

### 五 自定义函数

### 六 Mixin
混合宏是整个 Sass 语言中最常用的功能之一。这是重用和减少重复组件的关键。这么做有很棒的原因：混合宏允许开发者在样式表中定义可复用样式，**减少了对非语义类的需求**，比如`.float-left`。

#### 基础
如果你发现有一组 CSS 属性经常因同一个原因一起出现（非巧合），那么你就可以使用混合宏来代替。
#### 无参混合宏
不需要传递参数，则删除圆括号，使用 `@include` 关键字来表示当前行调用了混合宏。

```css
/// Helper to clear inner floats
/// @author Nicolas Gallagher
/// @link http://nicolasgallagher.com/micro-clearfix-hack/ Micro Clearfix
@mixin clearfix {
  &::after {
    content: '';
    display: table;
    clear: both;
  }
}
```

```css
// Yep
.foo {
  @include clearfix;
}

// Nope
.foo {
  @include clearfix();
}
```

#### 参数列表
@todo

### 循环

#### Each 

```css
@each $theme in $themes {
  .section-#{$theme} {
    background-color: map-get($colors, $theme);
  }
}
```

当迭代一个 `map` 时，通常使用 `$key` 和 `$value` 作为变量名称来确保一致性。
```css
@each $key, $value in $map {
  .section-#{$key} {
    background-color: $value;
  }
}
```

#### For
当需要聚合伪类 `:nth-*` 的时候，使用 @for 循环很有用。除了这些使用场景，如果必须迭代最好还是使用 `@each` 循环。
```css
@for $i from 1 through 10 {
  .foo:nth-of-type(#{$i}) {
    border-color: hsl($i * 36, 50%, 50%);
  }
}
```

