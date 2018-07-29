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

#### 高阶组件和rest参数(不是新特性)
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