---
layout: w
title: React Hooks 常见坑点解析
date: 2020-10-29 18:11:05
tags: react hooks
---

# React Hooks 常见坑点解析


通过简化状态重用和副作用管理，Hooks取代了基于类的组件。此外，咱们可以将重复的逻辑提取到自定义 Hook 中，以便在应用程序之间重用。但使用过程经常会有各种困惑，比如： 为什么有时候在effect里拿到的是旧的state或prop？我该在什么时候使用useLayoutEffect而不是useEffect？

详请👇:

---

##1. useEffect与过时闭包

Hooks依赖Javascript闭包, 但有时错误的用法会导致bug:
```
function ErrorDemo() {
  const [count, setCount] = useState(0);
  const dom = useRef(null);
  useEffect(function bind() {
    dom.current.addEventListener('click', () => setCount(count + 1));
  }, []);
  return <div ref={dom}>{count}</div>;
}
```
你可能会认为每当用户点击dom，count就加1。但实际上`count`加到1之后就不会再变化了。

表现：首先 `useEffect(fn, [])` 表明`bind()`只会在首次渲染时运行，即首次渲染会绑定事件。显然  `setCount(count + 1)` 会运行多次， 但渲染出 `count` 的值**始终为1**则表明每次 `setCount` 时count的=0。

> Any function inside a component, including event handlers and effects,
> “sees” the props and state from the render it was created in.

第一渲染时，bind()闭包捕获了值为0的`count`变量，导致click的回调获取的count始终等于0，意味着回调**始终从过时的闭包中获取数据**！

在任意一次渲染中，**props和state是始终保持不变的。**如果props和state在不同的渲染中是相互独立的，那么使用到它们的任何值也是独立的（包括事件处理函数）。它们都“属于”一次特定的渲染。即便是事件处理中的异步函数调用“看到”的也是这次渲染中的props和state。

**✅的做法：**
#### 消除依赖，使用函数方式更新状态
主要依靠setCount的[functional updates][1]用法，该方法接受previous value，返回updated value.
改成 `() => setCount(prevCount => ++prevCount)`, 就不用关心你的值是不是最新的。

#### 设置依赖项目，每次都拿到最新的值
```
useEffect(() => {
  const $dom = dom.current;
  const event = () => {
    console.log(count);
    setCount(prev => ++prev);
  };
  $dom.addEventListener('click', event);
  return () => $dom.removeEventListener('click', event);
}, [count]);
```
麻烦在类似处理事件的场景，可能需要重新绑定/解绑事件，开销较大

#### 将变量存入ref中缓存
useRef 不仅仅可以管理DOM ref，它还相当于 this , 可以存放任何变量.  
```
function ErrorDemo() {
  const [count, setCount] = useState(0);
  const dom = useRef(null);
  useEffect(() => {
        lastCount.current = count;
  }, []);
  useEffect(function bind() {
    dom.current.addEventListener('click', () => setCount(lastCount.current + 1));
  }, []);
  return <div ref={dom}>{count}</div>;
}
```

##2. Hooks与生命周期方法
我们写的有状态组件，通常会产生很多的副作用，比如ajax请求、事件处理、操作dom等等。我们之前都会逻辑分散写在 `CWM` 、`componentDidUpdate` 和`componentWillUnmountd`等钩子中。而使用useEffect则帮助你把相关逻辑**集合**在一个Effect中，代码变得更加清晰。

先梳理一下如何用`useEffect`模拟React class的生命周期函数：
```
function Counter() {
    const [count, setCount] = useState(0);
    useEffect(() => {
        // 方法体对应componentDidMount/componentDidUpdate
        // 在第一次渲染之后和每次更新之后都会执行
        document.title = `You clicked ${count} times`;
        return () => {
            // 对应componentWillUnmount
            document.title = `Finally you clicked ${count} times`;
        }
    });
    return ...;
}
```

#### **useLayoutEffect与useEffect的运行时机**
`useEffect` 在界面更新之后**异步**运行
```flow
st=>operation: 触发render(state变化、parent重渲染)
op1=>operation: react渲染组件
fin=>operation: 页面layout、painting结束
op3=>operation: useEffect异步运行
st->op1->fin->op3
```
`useLayoutEffect` 在render之后，界面更新之前**同步**执行
(和`componentDidMount`、`componentDidUpdate` 的调用阶段一致)
```flow
st=>operation: 触发render(state变化、parent重渲染)
op1=>operation: react渲染组件
op2=>operation: 页面layout
op3=>operation: useLayoutEffect同步运行，并阻塞painting
fin=>operation: 页面painting
st->op1->op2->op3->fin
```

鉴于 `useLayoutEffect` 运行会**阻塞页面渲染**，**当组件快速渲染两次并导致闪烁**时，是使用它的恰当时机。

#### 清除函数
与 `componentWillUnmount` 不同的是，清除函数会在组件卸载前执行，或者当组件多次渲染，在执行下一个effect之前执行，而 `componentWillUnmount` 仅仅在组件卸载时执行一次。

## 3. Hooks与Ref
在class时代，由于组件节点是class的实例，因而可以在实例上存放内容，这些内容随着实例化产生，随着componentWillUnmount销毁。但是在hook的范围下，函数组件并没有this和对应的实例，因此useRef作为这一能力的弥补，扮演着跨多次渲染存放内容的角色。

#### 条件型的useRef
最常见的用法是保存一个dom引用，然后在useEffect中引用
```javascript
const Foo = ({ text }) => {
    const [width, setWidth] = useState();
    const root = useRef(null);
    useEffect(
        () => {
            if (root.current) {
                setWidth(root.current.offsetWidth);
            }
        },
        []
    );

    return <span ref={root}>{text}</span>;
};
```
但如果首次渲染时带有ref属性的节点不存在呢？
```javascript
return visible ? <span ref={root}>{text}</span> : null;
```
此时首次渲染root.current没值，这段代码并不能按照预期工作。

✅的和上文类似，这里也可以通过传入额外的依赖来实现
```
useEffect(fn, [root.current]);
```
✅的更好的解决办法是使用回调ref来确保追踪到正确的dom

```
const Foo = ({text, visible}) => {
    const [width, setWidth] = useState();
    const ref = useCallback(
        element => element && setWidth(element.offsetWidth),
        []
    );

    return visible ? <span ref={ref}>{text}</span> : null;
};
```



  [1]: https://reactjs.org/docs/hooks-reference.html#functional-updates