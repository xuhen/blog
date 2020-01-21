---
title: babel转换jsx的5条规则
date: 2020-01-17 16:07:16
tags:
  - JSX
---


> 编者按：
>
>    曾几何时，感觉JSX这货高深莫测，用了React.createElement 和 createReactClass 写了段时间的代码，才发现原来babel是要把那些fancy的语法增强都一一转译到这些基础api
>
>    不只是JSX这样的奇怪语法，而且像es6的class语法糖也被转译成es5函数式继承语法，前端发明了这么多工具，只不过是为了提高开发效率和易于维护而已罢（一点拙见）




## 1. tags 变为 React.createElement

```
DOM 元素用 <lowercase /> 标签
<div />  -----> React.createElement('div', null)

自定义元素用 <Capitalized /> 标签
<Modal /> -----> React.createElement(Modal, null)
```

## 2. attributes 会变为 props

```
当props是string类型时，用""符号

<Modal title='edit' /> -----> React.createElement(Modal, { title: "edit" });

当props为字面量或变量时，用{}符号

<Modal
    title={`edit${name}`}
    onClick={this.click}/>

    |
    |
    |
   \|/

React.createElement(Modal, {
    title: "edit".concat(name),
    onClick: this.click
});
```

## 3. {...object} 表达式变为 Object.assign

```

<div className="default" {...this.props} />

            |
            |
            |
           \|/

React.createElement("div", Object.assign({
  className: "default"
}, this.props));
```

## 4. tag 子元素变为 props.children

```
子元素为普通文字

<div>hello world</div>   ----->  React.createElement("div", null, "hello world")

子元素为DOM元素

<Modal>
  <h1>hello world</h1>
</Modal>

        |
        |
        |
       \|/

React.createElement(Modal, null,
    React.createElement("h1", null,
        "hello world"
    )
);


子元素既有普通文字又有DOM元素

<div>
  Nice to meet you <em>Edward</em>
</div>

        |
        |
       \|/

React.createElement("div", null,
    "Nice to meet you ",
    React.createElement("em", null, "Edward")
);
```

## 5. 用 {} 表达式插入 children

```
插入的是普通文字变量

<div>
  Nice to meet you {this.props.name}
</div>

        |
        |
       \|/

React.createElement("div", null,
    "Nice to meet you ",
    this.props.name
);

插入数组

<ul>
  {
    steps.map((step) => {
      return <li key={step.id}>{step.content}</li>
    })
  }
</ul>

        |
        |
       \|/

React.createElement("ul", null, steps.map(function (step) {
  return React.createElement("li", {
    key: step.id
  }, step.content);
}));
```


结束： 转译的产物可能根据当前的babel版本和preset版本有所差别

可以拿着源码到babel官网看下转译产物


## 链接部分
[babeljs官网](https://babeljs.io/)
[Learn Raw React 系列](http://jamesknelson.com/learn-raw-react-no-jsx-flux-es6-webpack/)