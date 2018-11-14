# 高级

## 主题
`styled-component`提供`<ThemeProvider>`包装组件以支持主题.`<ThemeProvider>`通过`context`API 为其后代组件提供主题.在其渲染树中的所有组件都能够访问主题.

下面的示例通过创建一个按钮组件来说明如何传递主题:
```jsx
// Define our button, but with the use of props.theme this time
const Button = styled.button`
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;

  /* Color the border and text with theme.main */
  color: ${props => props.theme.main};
  border: 2px solid ${props => props.theme.main};
`;

// We are passing a default theme for Buttons that arent wrapped in the ThemeProvider
Button.defaultProps = {
  theme: {
    main: "palevioletred"
  }
}

// Define what props.theme will look like
const theme = {
  main: "mediumseagreen"
};

render(
  <div>
    <Button>Normal</Button>
    <ThemeProvider theme={theme}>
      <Button>Themed</Button>
    </ThemeProvider>
  </div>
);
```

### 函数主题
theme prop 也可以传递一个函数.该函数接收渲染树上级`<ThemeProvider>`所传递的主题. 通过这种方式可以使 themes 形成上下文.

下面的示例说明了如何通过第二个`<ThemeProvider>`来交换 `background`和`foreground`的颜色. 函数`invertTheme` 接收上级 theme 后创建一个新的 theme.

```jsx
// Define our button, but with the use of props.theme this time
const Button = styled.button`
  color: ${props => props.theme.fg};
  border: 2px solid ${props => props.theme.fg};
  background: ${props => props.theme.bg};

  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;
`;

// Define our `fg` and `bg` on the theme
const theme = {
  fg: "palevioletred",
  bg: "white"
};

// This theme swaps `fg` and `bg`
const invertTheme = ({ fg, bg }) => ({
  fg: bg,
  bg: fg
});

render(
  <ThemeProvider theme={theme}>
    <div>
      <Button>Default Theme</Button>

      <ThemeProvider theme={invertTheme}>
        <Button>Inverted Theme</Button>
      </ThemeProvider>
    </div>
  </ThemeProvider>
);
```

### 在`styled-components`外使用主题
如果需要在`styled-components`外使用主题,可以使用高阶组件`withTheme`:
```jsx
import { withTheme } from 'styled-components'

class MyComponent extends React.Component {
  render() {
    console.log('Current theme: ', this.props.theme)
    // ...
  }
}

export default withTheme(MyComponent)
```

### theme prop
主题可以通过`theme prop`传递给组件.通过使用`theme prop`可以绕过或重写`ThemeProvider`所提供的主题.

```jsx
// Define our button
const Button = styled.button`
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;

  /* Color the border and text with theme.main */
  color: ${props => props.theme.main};
  border: 2px solid ${props => props.theme.main};
`;

// Define what main theme will look like
const theme = {
  main: "mediumseagreen"
};

render(
  <div>
    <Button theme={{ main: "royalblue" }}>Ad hoc theme</Button>
    <ThemeProvider theme={theme}>
      <div>
        <Button>Themed</Button>
        <Button theme={{ main: "darkorange" }}>Overidden</Button>
      </div>
    </ThemeProvider>
  </div>
);
```

## Refs
通过传递`ref prop`给 styled component 将获得:
- 底层 DOM 节点 (如果 styled 的对象是基本元素如 div)
- React 组件实例 (如果 styled 的对象是 React Component)
```jsx
const Input = styled.input`
  padding: 0.5em;
  margin: 0.5em;
  color: palevioletred;
  background: papayawhip;
  border: none;
  border-radius: 3px;
`;

class Form extends React.Component {
  constructor(props) {
    super(props);
    this.inputRef = React.createRef();
  }

  render() {
    return (
      <Input
        ref={this.inputRef}
        placeholder="Hover to focus!"
        onMouseEnter={() => {
          this.inputRef.current.focus()
        }}
      />
    );
  }
}

render(
  <Form />
);
```
>注意
>
>v3 或更低的版本请使用 [innerRef prop](https://www.styled-components.com/docs/api#innerref-prop) instead.

## 安全性
因为 styled-components 允许使用任意的输入作为插值使用,我们必须谨慎处理输入使其无害.使用用户输入作为样式可能导致用户浏览器中的CSS文件被攻击者替换.

以下这个例子展示了糟糕的输入导致的 API 被攻击:

```jsx
// Oh no! The user has given us a bad URL!
const userInput = '/api/withdraw-funds'

const ArbitraryComponent = styled.div`
  background: url(${userInput});
  /* More styles here... */
`
```
请一定谨慎处理!这虽然是一个明显的例子,但是CSS注入可能隐式的发生并且产生不良影响.有些旧版本的 IE 甚至会在 url 声明中执行 JavaScript.

There is an upcoming standard to sanitize CSS from JavaScript有一个即将推出的标准,可以用于无害化 JavaScript 中的 CSS, [`CSS.escape`](https://developer.mozilla.org/en-US/docs/Web/API/CSS/escape). 这个标准还没有被浏览器很好的支持,因此建议使用 [`polyfill by Mathias Bynens`](https://github.com/mathiasbynens/CSS.escape) .

## Existing CSS
如果想将 styled-components 和现有的 CSS 共同使用,有很多实现的细节必须注意到.

styled-components 通过类生成实际的样式表,并通过`className prop`将这些类附加到响应的 DOM 节点. 运行时它会被注入到 document 的 head 末尾.

### Styling normal React components
使用`styled(MyComponent)` 声明,  `MyComponent` 却不接收传入的 `className` prop, 则样式并不会被呈现. 为避免这个问题,请确保组件接收 `className` 并传递给 DOM 节点:
```jsx
class MyComponent extends React.Component {
  render() {
    // Attach the passed-in className to the DOM node
    return <div className={this.props.className} />
  }
}
```
对于已存在类名的组件,可以将其余传入的类合并:
```jsx
class MyComponent extends React.Component {
  render() {
    // Attach the passed-in className to the DOM node
    return <div className={`some-global-class ${this.props.className}`} />
  }
}
```

### Issues with specificity
将`styled-components`类与全局类混用,可能会导致出乎意料的结果.如果一个`property`在两个类中被定义且两个类的优先级相同,则后者会覆盖前者.
```jsx
// MyComponent.js
const MyComponent = styled.div`background-color: green;`;

// my-component.css
.red-bg {
  background-color: red;
}

// For some reason this component still has a green background,
// even though you're trying to override it with the "red-bg" class!
<MyComponent className="red-bg" />
```
上述例子中`styled-components`类的样式覆盖了全局类,因为`styled-components`在运行时向`<head>`末尾注入样式.

一种解决方式是提高全局样式的优先级:
```jsx
/* my-component.css */
.red-bg.red-bg {
  background-color: red;
}
```

### 避免与第三方样式和脚本的冲突
如果在一个不能完全控制的页面上部署`styled-components`,可能需要采取措施确保 component styles 不与 host page 上其他样式冲突.

常见的问题是优先级相同,例如 host page 上持有如下样式:
```jsx
body.my-body button {
  padding: 24px;
}
```
因为其包含一个类名和两个标签名,它的优先级要高于 styled component 生成的一个类名的选择器:
```jsx
styled.button`
  padding: 16px;
`
```
没有让 styled component 完全不受 host page 样式影响的办法.但是可以通过[`babel-plugin-styled-components-css-namespace`](https://github.com/QuickBase/babel-plugin-styled-components-css-namespace)来提高样式的优先级, 通过它可以为 styled components 的类指定一个命名空间. 一个好的命名空间,譬如`#my-widget`,可以实现styled-components 在 一个 `id="my-widget"`的容器中渲染, 因为 id 选择器的优先级总是高于类选择器.

一个罕见的问题是同一页面上两个`styled-components`实例的冲突.通过在 code bundle 中定义 `process.env.SC_ATTR` 可以避免这个问题. 它将覆盖 `<style> `标签的`data-styled`属性,  (v3 及以下版本使用 `data-styled-components`), allowing each styled-components instance to recognize its own tags.

## 媒体模板
开发响应式 web app 时媒体查询是不可或缺的工具.

以下是一个非常简单的示例,展示了当屏宽小于700px时,组件如何改变背景色:
```jsx
const Content = styled.div`
  background: papayawhip;
  height: 3em;
  width: 3em;

  @media (max-width: 700px) {
    background: palevioletred;
  }
`;

render(
  <Content />
);
```
由于媒体查询很长,并且常常在应用中重复出现,因此有必要为其创建模板.

由于 JavaScript 的函数式特性,我们可以轻松的定义自己的标记模板字符串用于包装媒体查询中的样式.我们重写一下上个例子来试试:
```jsx
const sizes = {
  desktop: 992,
  tablet: 768,
  phone: 576,
}

// Iterate through the sizes and create a media template
const media = Object.keys(sizes).reduce((acc, label) => {
  acc[label] = (...args) => css`
    @media (max-width: ${sizes[label] / 16}em) {
      ${css(...args)}
    }
  `

  return acc
}, {})

const Content = styled.div`
  height: 3em;
  width: 3em;
  background: papayawhip;

  /* Now we have our methods on media and can use them instead of raw queries */
  ${media.desktop`background: dodgerblue;`}
  ${media.tablet`background: mediumseagreen;`}
  ${media.phone`background: palevioletred;`}
`;

render(
  <Content />
);
```

## 标记模板字符串
模板字符串是 ES6 的新功能.它允许我们自定义字符串插值规则--styled components 正是基于此功能实现.

如果没有传递插值,则函数接收的一个参数是包含一个字符串的数组:
```jsx
// These are equivalent:
fn`some string here`
fn(['some string here'])
```
如果传递了插值,则数组中包含了传递的字符串, split at the positions of the interpolations.其余参数将按顺序进行插值.

```jsx
const aVar = 'good'

// These are equivalent:
fn`this is a ${aVar} day`
fn(['this is a ', ' day'], aVar)
```
这用起来有点笨重,但是这意味着我们可以在 styled components 中接收变量,函数或是 mixins ,并且可以将它们转换成纯 CSS.

想了解有关标记模板字符串的更多信息, 请参阅 Max Stoiber 的文章: [The magic behind 💅 styled-components](https://mxstbr.blog/2016/11/styled-components-magic-explained/)

## 服务端渲染(v2+)

styled-components 支持并发服务端渲染, with stylesheet rehydration. 其核心思想是,每当在服务器上渲染应用时, 为 React 树创建一个`ServerStyleSheet` 和一个 `provider` ,通过 context API 来接收样式. 

这不会影响全局样式,例如 `keyframes` 或者 `createGlobalStyle` ,并且允 styled-components 与 React DOM 的 SSR API 共同使用.

### 设置
为了可靠的执行 SSR,正确的生成客户端 bundle,请使用 [babel plugin](https://www.styled-components.com/docs/tooling#babel-plugin). 
它通过为每个 styled component 添加确定的 ID 来防止校验错误. 更多信息请参考 [tooling documentation](https://www.styled-components.com/docs/tooling#serverside-rendering) .

对于 TypeScript 用户, TS 大师 Igor Oleinikov 整合了webpack ts-loader / awesome-typescript-loader 工具链 [TypeScript plugin](https://www.styled-components.com/docs/tooling#typescript-plugin)  来完成类似的任务.

If possible, we definitely recommend using the babel plugin though because it is updated the most frequently. It's now possible to [compile TypeScript using Babel](https://babeljs.io/docs/en/babel-preset-typescript), so it may be worth switching off TS loader and onto a pure Babel implementation to reap the ecosystem benefits.

### 示例
基本 API 的使用如下:
```js
import { renderToString } from 'react-dom/server'
import { ServerStyleSheet } from 'styled-components'

const sheet = new ServerStyleSheet()
const html = renderToString(sheet.collectStyles(<YourApp />))
const styleTags = sheet.getStyleTags() // or sheet.getStyleElement();
```

`collectStyles` 方法将元素包装进了 provider.也可以选择直接使用 `StyleSheetManager` provider.确保不要再客户端使用即可.

```jsx
import { renderToString } from 'react-dom/server'
import { ServerStyleSheet, StyleSheetManager } from 'styled-components'

const sheet = new ServerStyleSheet()
const html = renderToString(
  <StyleSheetManager sheet={sheet.instance}>
    <YourApp />
  </StyleSheetManager>
)

const styleTags = sheet.getStyleTags() // or sheet.getStyleElement();
```

`sheet.getStyleTags()` 方法返回多个字符串的 `<style>` 标签. 当向 HTML 输出增加 CSS 时需要考虑这一点.

作为另一种选择,` ServerStyleSheet` 实例也提供 `getStyleElement()` 方法,返回一个 React 元素的数组.

>注意
>
>`sheet.getStyleTags()` 和`sheet.getStyleElement()` 只能在元素渲染和调用. 所以`sheet.getStyleElement()`中的组件不能与`<YourApp /> `合并为一个更大的组件.

### Next.js
首先添加一个自定义的 ` pages/_document.js `. 然后 [复制这段逻辑](https://github.com/zeit/next.js/blob/master/examples/with-styled-components/pages/_document.js) 将服务端你渲染的样式注入 `<head>`.

参考 [our example](https://github.com/zeit/next.js/tree/master/examples/with-styled-components) 中的 Next.js repo .

### Streaming Rendering