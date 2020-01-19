---
title: React 应用中的优化技巧
date: 2019-12-23 19:22:59
tags:
---

本文提供一些减少 React 重绘的小技巧

## 1. 使用纯组件

它与普通组件（Component）是一样的，只是 PureComponents 负责 shouldComponentUpdate——它对状态和 props 数据进行浅层比较（shallow comparison）。如果先前的状态( state )和 props 数据与下一个 props 或状态相同，则组件不会重新渲染。

```
export default class ApplicationComponent extends React.Component {
    constructor() {
        super();
        this.state = {
            name: "Mayank"
        }
    }

    updateState = () => {
        setInterval(() => {
            this.setState({
                name: "Mayank"
            })
        }, 1000)
    }

    componentDidMount() {
        this.updateState();
    }

    render() {

        console.log("Render Called Again")
        return (
            <div>
                <PureChildComponent name={this.state.name} />
            </div>
        )
    }
}


class PureChildComponent extends React.PureComponent {

    // Pure Components are the components that do not re-render if the State data or props data is still the same

    render() {
        console.log("Pure Component Rendered..")
        return <div>{this.props.name}</div>;
    }
}
```

## 2. 使用 React.memo 进行组件记忆

它很像 PureComponent，但 PureComponent 属于 Component 的类实现，而“memo”则用于创建函数组件。

```
function CustomisedComponen(props) {
    return (
        <div>
            <b>User name: {props.user.name}</b>
            <b>User age: {props.user.age}</b>
            <b>User designation: {props.user.designation}</b>
        </div>
    )
}

function userComparator(previosProps, nextProps) {
    if(previosProps.user.name !== nextProps.user.name ||
       previosProps.user.age !== nextProps.user.age ||
       previosProps.user.designation !== nextProps.user.designation) {
        return false
    } else {
        return true;
    }
}

var memoComponent = React.memo(CustomisedComponent, userComparator);
```

假设 props 值（user）是一个对象引用，包含特定用户的 name、age 和 designation。这种情况下需要进行深入比较。我们可以创建一个自定义函数，查找前后两个 props 值的 name、age 和 designation 的值，如果它们不相同则返回 false。


## 3. 使用 shouldComponentUpdate 生命周期事件

可以利用此事件来决定何时需要重新渲染组件。如果组件 props 更改或调用 setState，则此函数返回一个 Boolean 值。

```
export default class ShouldComponentUpdateUsage extends React.Component {

  constructor(props) {
    super(props);
    this.state = {
      name: "Mayank";
      age: 30,
      designation: "Architect";
    }
  }

  componentDidMount() {
    setTimeout(() => {
      this.setState({
        designation: "Senior Architect"
      });
    }
  }

  shouldComponentUpdate(nextProps, nextState) {
      if(nextState.age != this.state.age || netState.name = this.state.name) {
        return true;
      }
      return false;
  }

  render() {
    return (
      <div>
        <b>User Name:</b> {this.state.name}
        <b>User Age:</b> {this.state.age}
      </div>
    )
  }
}
```

## 4. 懒加载组件

像 webpack 这样的打包器就支持代码拆分，它可以为应用创建多个包，并在运行时动态加载，减少初始包的大小。为此我们使用 Suspense 和 lazy。

```
import React, { lazy, Suspense } from "react";

export default class CallingLazyComponents extends React.Component {
  render() {

    var ComponentToLazyLoad = null;

    if(this.props.name == "Mayank") {
      ComponentToLazyLoad = lazy(() => import("./mayankComponent"));
    } else if(this.props.name == "Anshul") {
      ComponentToLazyLoad = lazy(() => import("./anshulComponent"));
    }
    return (
        <div>
            <h1>This is the Base User: {this.state.name}</h1>
            <Suspense fallback={<div>Loading...</div>}>
                <ComponentToLazyLoad />
            </Suspense>
        </div>
    )
  }
}
```

> 这个方法的好处:
> 1. 主包体积变小，消耗的网络传输时间更少。
> 2. 动态单独加载的包比较小，可以迅速加载完成。

## 5. 使用 React Fragments 避免额外标记
片段不会向组件引入任何额外标记，但它仍然为两个相邻标记提供父级，因此满足在组件顶级具有单个父级的条件。

```
export default class NestedRoutingComponent extends React.Component {
    render() {
        return (
            <Fragment>
                <h1>This is the Header Component</h1>
                <h2>Welcome To Demo Page</h2>
            </Fragment>
        )
    }
}
```

## 6. 不要使用内联函数定义

如果我们使用内联函数，则每次调用“render”函数时都会创建一个新的函数实例。
当 React 进行虚拟 DOM diffing 时，它每次都会找到一个新的函数实例；因此在渲染阶段它会会绑定新函数并将旧实例扔给垃圾回收。
因此直接绑定内联函数就需要额外做垃圾回收和绑定到 DOM 的新函数的工作。

```
export default class InlineFunctionComponent extends React.Component {
  render() {
    return (
      <div>
        <h1>Welcome Guest</h1>
        <input type="button" onClick={(e) => { this.setState({inputValue: e.target.value}) }} value="Click For Inline Function" />
      </div>
    )
  }
}
```

所以不要用内联函数，而是在组件内部创建一个函数，并将事件绑定到该函数本身。这样每次调用 render 时就不会创建单独的函数实例了，参考组件如下。

```
export default class InlineFunctionComponent extends React.Component {
  constructor() {
    this.state = {}

    this.setNewStateData = this.setNewStateData.bind(this);
  }

  setNewStateData (event) {
    this.setState({
      inputValue: e.target.value
    })
  }

  render() {
    return (
      <div>
        <h1>Welcome Guest</h1>
        <input type="button" onClick={this.setNewStateData} value="Click For Inline Function" />
      </div>
    )
  }
}
```

## 7. 避免 componentWillMount() 中的异步请求

这个函数用的不多，可用来配置组件的初始配置，但使用 constructor 方法也能做到。
一些开发人员认为这个函数可以用来做异步数据 API 调用, 但是也可以在componentDidMount 去做，其性能没有多大差别，所以即将在v17.0中被弃用

## 8. 箭头函数与构造函数中的绑定

```
export default class DelayedBinding extends React.Component {
  constructor() {
    this.state = {
      name: "Mayank"
    }
  }

  handleButtonClick = () => {
    alert("Button Clicked: " + this.state.name)
  }

  render() {
    return (
      <div>
        <input type="button" value="Click" onClick={this.handleButtonClick} />
      </div>
    )
  }
}
```

当我们添加箭头函数时，该函数被添加为对象实例，而不是类的原型属性。这意味着如果我们多次复用组件，那么在组件外创建的每个对象中都会有这些函数的多个实例。
因此箭头函数确实有其缺点。实现这些函数的最佳方法是在构造函数中绑定函数。


## 9.避免使用内联样式属性
使用内联样式时浏览器需要花费更多时间来处理脚本和渲染，因为它必须映射传递给实际 CSS 属性的所有样式规则。

```
export default class InlineStyledComponents extends React.Component {
  render() {
    return (
      <>
        <b style={{"backgroundColor": "blue"}}>Welcome to Sample Page</b>
      </>
    )
  }
}
```

上面样式 backgroundColor 需要转换为等效的 CSS 样式属性，然后才应用样式。这样就需要额外的脚本处理和 JS 执行工作。更好的办法是将 CSS 文件导入组件。

## 10.优化 React 中的条件渲染

安装和卸载 React 组件是昂贵的操作。为了提升性能，我们需要减少安装和卸载的操作。

```
export default class ConditionalRendering extends React.Component {
  constructor() {
    this.state = {
      name: "Mayank"
    }
  }

  render() {
    if(this.state.name == "Mayank") {
      return (
        <>
          <AdminHeaderComponent></AdminHeaderComponent>
          <HeaderComponent></HeaderComponent>
          <ContentComponent></ContentComponent>
        </>
      )
    } else {
      return (
        <>
          <HeaderComponent></HeaderComponent>
          <ContentComponent></ContentComponent>
        </>
      )
    }
  }
}
```

上面的代码可以优化为下面代码

```
export default class ConditionalRendering extends React.Component {
  constructor() {
    this.state = {
      name: "Mayank"
    }
  }

  render() {
    return (
      <>
        { this.state.name == "Mayank" && <AdminHeaderComponent></AdminHeaderComponent> }
        <HeaderComponent></HeaderComponent>
        <ContentComponent></ContentComponent>
      </>
    )
  }
}
```

在上面的代码中，当 name 不是 Mayank 时，React 在位置 1 处放置 null。

开始 DOM diffing 时，位置 1 的元素从 AdminHeaderComponent 变为 null，但位置 2 和位置 3 的组件保持不变。

由于元素没变，因此组件不会卸载，减少了不必要的操作。

## 11.为组件创建错误边界

组件渲染错误是很常见的情况。
在这种情况下，组件错误不应该破坏整个应用。创建错误边界可避免应用在特定组件发生错误时中断。
错误边界是一个 React 组件，可以捕获子组件中的 JavaScript 错误。我们可以包含错误、记录错误消息，并为 UI 组件故障提供回退机制。

```
class Home extends Component {
  constructor(props) {
    super(props);
    this.state = {
    };

  }

  componentDidMount() {
    console.log('componentDidMount');

  }

  render() {
    return (
      <div>
        <ErrorBoundary>
          <MyWidget />
        </ErrorBoundary>
      </div>
    )
  }
}


class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    console.log('getDerivedStateFromError', error);
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // You can also log the error to an error reporting service
    console.log('componentDidCatch', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}

class MyWidget extends Component {
  constructor(props) {
    super(props);

    this.state = {
      name: 'tom'
    };
  }

  render() {
    return (
      <div>
        <div>{this.state.name}</div>

        <div>
          {this.state.hello.map((item, idx) => {
            return <div key={idx}></div>
          })}
        </div>
      </div>
    )
  }

}
```

如上state上没有hello，故MyWidget内会报错，会被ErrorBoundary捕获并提供回退 UI。




## 下面部分为React里的一些技术细节

```
Q. React 如何区分Class组件和Function组件?


A.
// Inside React
class Component {}
Component.prototype.isReactComponent = {};

// We can check it like this
class Greeting extends Component {}
console.log(Greeting.prototype.isReactComponent); // ✅ Yes

Q. 为啥React Elements会有$$typeof属性？

A.
主要是服务端的安全问题，如果服务端有个漏洞，让用户存任意的JSON对象，而客户端期望的是个string类型变量，就会有问题

let expectedTextButGotJSON = {
  type: 'div',
  props: {
    dangerouslySetInnerHTML: {
      __html: '/* put your exploit here */'
    },
  },
  // ...
};
let message = { text: expectedTextButGotJSON };

// Dangerous in React 0.13
<p>
  {message.text}
</p>

如果加上$$typeof后变成

{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'),
}

原理是我们不能把Symbol 类型的数据放到自己定义的JSON上，因此即使服务器有安全漏洞返回JSON，
React也会检查返回的对象上的$$typeof属性，如果缺失就不会解析，并抛错。

```