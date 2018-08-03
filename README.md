## React16.4.1新特性

#### React.createRef()
> https://segmentfault.com/a/1190000015113359

> React V16版本新增一个API：React.createRef(); 通过这个API，我们可以先创建一个ref变量，然后再将这个变量赋值给组件声明中ref属性就好了。

```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // create a ref to store the textInput DOM element
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // Explicitly focus the text input using the raw DOM API
    // Note: we're accessing "current" to get the DOM node
    this.textInput.current.focus();
  }

  render() {
    // tell React that we want to associate the <input> ref
    // with the `textInput` that we created in the constructor
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} />

        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}

```
> 在上面的代码中，我们先通过 React.createRef();创建一个ref，并赋值给组件属性textInput（this.textInput），然后在render函数中通过ref={this.textInput}的方式将ref和input这个真实DOM联系起来， 这样就可以通过 this.textInput.current.focus();的方式来直接操作input元素的方法。

#### React.forwardRef()
> 我们想要父组件拿到子组件的ref，需要通过一些特殊的方法，V16版本之后，React提供了一种原生的方式来完成这种操作。
```
  const FancyButton = React.forwardRef((props, ref) => (
    <button ref={ref} className="FancyButton">
      {props.children}
    </button>
  ));

  // You can now get a ref directly to the DOM button:
  const ref = React.createRef();
  <FancyButton ref={ref}>Click me!</FancyButton>;

  1、首先我们通过React.createRef();创建一个ref变量，然后在FancyButton属性中通过 ref={ref}的方式把这个ref和组件关联起来。
  2、目前为止，如果FancyButton 是一个通过class或者函数声明的组件，那么就到此为止，我们可以说 ref变量的current属性持有对 FancyButton组件实例的引用。
  3、不幸的是，FancyButton经过了 React.forwardRef的处理， 这个API接受两个参数，第二个参数就是ref，然后通过 <button ref={ref}>把ref绑定到button元素上，这样ref.current的引用就是button元素这个DOM对象了。。。

```

#### 你不能在函数式组件上使用 ref 属性，因为它们没有实例,然而你可以 在函数式组件内部使用 ref 来引用一个 DOM 元素或者 类(class)组件：
```
function CustomTextInput(props) {
  // textInput必须在这里声明，所以 ref 回调可以引用它
  let textInput = null;

  function handleClick() {
    textInput.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={(input) => { textInput = input; }} />

      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );  
}

```
#### 你可以在组件间传递回调形式的 refs，就像你可以传递通过 React.createRef() 创建的对象 refs 一样。

```
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
    );
  }
}

```
#### 高阶组件和rest参数(不是新特性)
> https://segmentfault.com/a/1190000010371752

> HOC（higher-order components）高阶组件，简单的说，就是通过组件包裹的方式来提到代码复用，高阶组件就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件。

> rest参数：  Rest就是为解决传入的参数数量不一定， rest parameter(Rest 参数) 本身就是数组，数组的相关的方法都可以用。

```
function logProps(Component) {
  class LogProps extends React.Component {
    render() {
      const {forwardedRef, ...rest} = this.props;

      // Assign the custom prop "forwardedRef" as a ref
      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  // Note the second param "ref" provided by React.forwardRef.
  // We can pass it along to LogProps as a regular prop, e.g. "forwardedRef"
  // And it can then be attached to the Component.
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}

```
> logProps是函数，接受一个组件参数，返回一个包裹参数组件的logProps组件。

#### React.createContext()
> https://segmentfault.com/a/1190000013365874?utm_source=tag-newest

> Context 旨在共享一个组件树内可被视为 “全局” 的数据，例如当前经过身份验证的用户，主题或首选语言等
```
// Context lets us pass a value deep into the component tree
// without explicitly threading it through every component.
// Create a context for the current theme (with "light" as the default).
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // Use a Provider to pass the current theme to the tree below.
    // Any component can read it, no matter how deep it is.
    // In this example, we're passing "dark" as the current value.
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// A component in the middle doesn't have to
// pass the theme down explicitly anymore.
function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton(props) {
  // Use a Consumer to read the current theme context.
  // React will find the closest theme Provider above and use its value.
  // In this example, the current theme is "dark".
  return (
    <ThemeContext.Consumer>
      {theme => <Button {...props} theme={theme} />}
    </ThemeContext.Consumer>
  );
}

```

##### Provider
> React组件允许 Consumer(使用者) 订阅 context 的改变。

> 接受一个 value 属性传递给 Provider(提供则) 的后代的 Consumer(使用者) 。 一个 Provider 可以连接到许多 Consumers 。 Providers 可以被嵌套以覆盖树中更深层次的值。

```
  const {Provider, Consumer} = React.createContext(defaultValue);
  <Provider value={/* some value */}>
```
##### Consumer
> 一个可以订阅 context 变化的 React 组件。

> 需要接收一个 函数作为子节点。 该函数接收当前 context 值并返回一个 React 节点。 传递给函数的 value 参数将等于组件树中上层这个 context 最接近的 Provider 的 value 属性。 如果上层没有提供这个 context 的 Provider ，value参数将等于传递给 createContext() 的 defaultValue 。

```
<Consumer>
  {value => /* render something based on the context value */}
</Consumer>
```
#####  高阶用法
```
const ThemeContext = React.createContext('light');

// This function takes a component...
export function withTheme(Component) {
  // ...and returns another component...
  return function ThemedComponent(props) {
    // ... and renders the wrapped component with the context theme!
    // Notice that we pass through any additional props as well
    return (
      <ThemeContext.Consumer>
        {theme => <Component {...props} theme={theme} />}
      </ThemeContext.Consumer>
    );
  };
}

```

#### create-react-app项目打包相关问题
> https://www.jianshu.com/p/f9535acd0462


####  create-react-app按需加载以及部署
> create-react-app按需加载以及部署:https://blog.csdn.net/hesongGG/article/details/78109631

> webpack4利用import动态加载的一些说明:https://blog.csdn.net/sunq1982/article/details/80540548

> 理解 JavaScript 的 async/await:https://segmentfault.com/a/1190000007535316

> async/await的使用以及注意事项:https://blog.csdn.net/juhaotian/article/details/78934097
```
import React, { Component } from "react";

export default function asyncComponent(importComponent) {
    class AsyncComponent extends Component {
        constructor(props) {
            super(props);

            this.state = {
                component: null
            };
        }

        async componentDidMount() {
            const { default: component } = await importComponent();

            this.setState({
                component: component
            });
        }

        render() {
            const Component = this.state.component;

            return Component ? <Component {...this.props} /> : null;
        }
    }

    return AsyncComponent;
}

```
> 这个是异步加载组件的方法.到时在需要的组件上加入引用就好.例如在路由里. 
```
const AsyncUser = asyncComponent(() => import("./User"));
...
<Route path="/users" component={AsyncUser}/>

```
#### 生命周期的变化
> https://github.com/lisong981174580/blogs.git

#### 错误边界(Error Boundaries)
> 如果一个类组件定义了一个名为 componentDidCatch(error, info): 的新生命周期方法，它将成为一个错误边界：error 是被抛出的错误,info 是一个含有 componentStack 属性的对象。这一属性包含了错误期间关于组件的堆栈信息。

错误边界 无法 捕获如下错误:
* 事件处理 （了解更多）
* 异步代码 （例如 setTimeout 或 requestAnimationFrame 回调函数）
* 服务端渲染
* 错误边界自身抛出来的错误 （而不是其子组件）

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    // Display fallback UI
    this.setState({ hasError: true });
    // You can also log the error to an error reporting service
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

```

而后你可以像一个普通的组件一样使用：

```
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>

```
> componentDidCatch() 方法机制类似于 JavaScript catch {}，但是针对组件。仅有类组件可以成为错误边界。实际上，大多数时间你仅想要定义一个错误边界组件并在你的整个应用中使用。

#### 包裹标签 React.Fragment

> 例如在 <tbody></tbody> 标签中，我们只能放置 <tr></tr>标签，假如我们同时有多个 <tr> 标签被赋值给一个 JSX 变量，那么在 React 里也有类似的功能：<React.Fragment> 标签。

  ```
  let DOM = (<React.Fragment>
    <tr>
        <td>1</td>
    </tr>
    <tr>
        <td>2</td>
    </tr>
  </React.Fragment>)

  let Component = <table>
      <tbody>
      {DOM}
      </tbody>
  </table>

```

####  MySQL的简单查询语句

> https://www.cnblogs.com/cuikang/p/6054046.html


#### 利用shouldComponentUpdate钩子函数优化react性能以及引入immutable库的必要性

> https://www.cnblogs.com/penghuwan/p/6707254.html

#### 插槽(Portals)

> 本人感觉在做对话框或者冒泡用法的时候很有用

> Portals 提供了一种很好的方法，将子节点渲染到父组件 DOM 层次结构之外的 DOM 节点。
```
ReactDOM.createPortal(child, container)

```

> 第一个参数（child）是任何可渲染的 React 子元素，例如一个元素，字符串或 片段(fragment)。第二个参数（container）则是一个 DOM 元素。

```
通常来说，当你从组件的 render 方法返回一个元素时，它将被作为子元素被装载到最近父节点 DOM 中：

render() {
  // React 装载一个新的 div，并将 children 渲染到这个 div 中
  return (
    <div>
      {this.props.children}
    </div>
  );
}

然而，有时候将子元素插入到 DOM 节点的其他位置会有用的：

render() {
  // React *不* 会创建一个新的 div。 它把 children 渲染到 `domNode` 中。
  // `domNode` 可以是任何有效的 DOM 节点，不管它在 DOM 中的位置。
  return ReactDOM.createPortal(
    this.props.children,
    domNode,
  );
}

对于 portal 的一个典型用例是当父组件有 overflow: hidden 或 z-index 样式，但你需要子组件能够在视觉上 “跳出(break out)” 其容器。例如，对话框、hovercards以及提示框：
记住这点非常重要，当在使用 portals 时，你需要确保遵循合适的可访问指南。

```
#### 使用 PropTypes 进行类型检查

> 要在组件中进行类型检测，你可以赋值 propTypes 属性。

```
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};

```

> 使用 PropTypes.element ，你可以指定仅将单一子元素传递给组件，作为子节点。

```
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // 这里必须是一个元素，否则会发出警告。
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};

```

#### 默认的 prop 值

> 你可以通过赋值特定的 defaultProps 属性为 props 定义默认值：

```
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// 指定 props 的默认值：
Greeting.defaultProps = {
  name: 'Stranger'
};

// 渲染为 "Hello, Stranger":
ReactDOM.render(
  <Greeting />,
  document.getElementById('example')
);

```

#### static getDerivedStateFromProps()


```
class EmailInput extends Component {
  state = {
    email: this.props.defaultEmail,
    prevPropsUserID: this.props.userID
  };

  static getDerivedStateFromProps(props, state) {
    // Any time the current user changes,
    // Reset any parts of state that are tied to that user.
    // In this simple example, that's just the email.
    if (props.userID !== state.prevPropsUserID) {
      return {
        prevPropsUserID: props.userID,
        email: props.defaultEmail
      };
    }
    return null;
  }

  
}

```

#### getSnapshotBeforeUpdate()

> getSnapshotBeforeUpdate() 在最近一次的渲染输出被提交之前调用。它使您的组件能够在DOM发生潜在变化之前捕获一些信息（例如滚动位置）。此生命周期返回的任何值将作为参数传递给componentDidUpdate()。

这个用例并不常见， 但它可能会出现在需要以特殊方式处理滚动位置的聊天线程之类的UI中。

```
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Are we adding new items to the list?
    // Capture the scroll position so we can adjust scroll later.
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // If we have a snapshot value, we've just added new items.
    // Adjust scroll so these new items don't push the old ones out of view.
    // (snapshot here is the value returned from getSnapshotBeforeUpdate)
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}

```
> 在上面的例子中，读取 getSnapshotBeforeUpdate 中的scrollHeight属性是很重要的，因为在 “render” 阶段生命周期（比如render）和 “commit（提交）” 阶段生命周期之间可能存在延迟（比如getSnapshotBeforeUpdate和componentDidUpdate）。
