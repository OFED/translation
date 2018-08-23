> 原文链接：[React Context API: Managing State with Ease](https://auth0.com/blog/react-context-api-managing-state-with-ease/)
>
> 译者：[OFED](https://github.com/OFED)

# React Context API: 轻松管理状态

#### 使用最新的 React Context API 管理状态非常容易。现在就跟随我一起学习下它和 Redux 的区别以及它是如何使用的吧。

## 前言：

The React Context API 在 React 生态系统中并不是个新鲜事物。不过，在 React `16.3.0` 版本中做了一些[改进](https://auth0.com/blog/whats-new-in-react-16-3/)。这些改进是如此巨大，以至于大大减少了我们对 Redux 和其他高级状态管理库的需求。在本文中，你将通过一个实用教程了解到新的 React Context API 是如何取代 Redux 完成小型应用的状态管理的。

## Redux 快速回顾

在直奔主题之前，我们先来快速回顾下 Redux，以便我们更好的比较两者的区别。[redux 是一个便于状态管理的JavaScript库](https://redux.js.org/)。Redux 本身和 React 并没有关系。来自世界各地的众多开发者选择在流行的前端框架（比如 *React* 和 *Angular* ）中使用 Redux。

说明一点，在本文中，状态管理指的是处理单页面应用（SPA）中产生的基于特定事件而触发的状态变化。比如，一个按钮的点击事件或者一条来自服务器的异步信息等，都可以触发应用状态的变化。

在 Redux 中，你尤其需要注意下面几点：

1. 整个 app 的状态存储在单个对象中（该对象被称作数据源）。
2. 如果要改变状态，你需要通过 dispatch 方法触发`actions`，actions 描述了需要发生的事情。
3. 在 Redux 中，你不能更改对象的属性或更改现有数组，必须始终返回新对象或新数组。

如果你对 Redux 并不熟悉并且你想要了解更多，请移步 [Redux 的实用教程](https://auth0.com/blog/redux-practical-tutorial/)学习。

## React Context API 指南

The React Context API 提供了一种通过组件树传递数据的方法，而不必通过 `props` 属性一层层的传递。在React中，数据通常会作为一个属性从父组件传递到子组件。

使用最新的 React Context API 取决于三个关键步骤：

1. 将初始状态传递给 `React.createContext`。这个方法会返回一个带有 `Provider` 和 `Consumer` 的对象。
2. 使用 `Provider` 组件包裹在组件树的最外层，并接收一个 value 属性。value 属性可以是任何值。
3. 在 `Provider` 组件下的 `Consumer` 组件在组件树的任何位置都能接收到状态的子集。

如你所见，所涉及的概念实际上与 Redux 没有什么不同。事实上，甚至 Redux 也在其公共API的底层使用了 React Context API。然而，直到最近，Context API 才达到了足够独立使用的水平。

## 使用 Redux 创建 React 应用

如上所述，本文的目标是向你展示新的 Context API 如何在小型应用中替代 Redux 的。因此，你将首先用 Redux 创建一个简单的 React app，然后，你将学习如何删除这个状态管理库，以便可以使用 React Context API 来再次实现它。

你将构建的示例应用是一个处理一些流行食物及其来源的列表。这个应用还将包括一个搜索功能，使用户能够根据一些关键词过滤列表。

最终，你将创建一个类似下面所述的应用：

### 项目要求

由于本文仅使用 React 和一些 NPM 库，因此除了 Node.js 和 NPM 之外，你什么都不需要。如果你还没有安装 Node.js 和 NPM，请前往[官网](https://nodejs.org/en/download/)下载并安装。

安装这些依赖后，你还需要安装 `create-react-app` 工具。这个工具帮助开发人员创建 React 项目。打开一个终端并运行以下命令来安装:

```bash
npm i -g create-react-app
```

### 搭建 React 应用

安装完 `create-react-app` 后，进入项目所在目录，执行以下命令：

```bash
create-react-app redux-vs-context
```

几秒钟后，`create-react-app` 将完成应用程序的创建。在此之后，进入该工具创建的新目录，并安装 Redux：

```bash
# move into your project
cd redux-vs-context

# install Redux
npm i --save redux react-redux
```

> *Note:* `redux` 是主库，`react-redux` 是促进 React 和 Redux 之间交互的库。简而言之，后者充当 React 和 Redux 之间的代理。

### 使用 Redux 开发 React 应用

你已经搭建好了你的 React 应用，安装好了 Redux，现在，在你喜欢的开发工具中打开你的项目。然后在 `src` 文件夹中创建三个文件:

* `foods.json` ：保存食物及其来源信息的列表数据的文件
* `reducers.js`：管理应用中 Redux 状态的文件
* `actions.js`：保存应用中触发 Redux 状态改变的方法的文件

所以，首先，打开 `foods.json` 文件，添加如下内容：

```json
[
  {
    "name": "Chinese Rice",
    "origin": "China",
    "continent": "Asia"
  },
  {
    "name": "Amala",
    "origin": "Nigeria",
    "continent": "Africa"
  },
  {
    "name": "Banku",
    "origin": "Ghana",
    "continent": "Africa"
  },
  {
    "name": "Pão de Queijo",
    "origin": "Brazil",
    "continent": "South America"
  },
  {
    "name": "Ewa Agoyin",
    "origin": "Nigeria",
    "continent": "Africa"
  }
]
```

正如你所见，json文件存储的数据并没有什么特别的。仅仅是一个包含着不同食物及其来源的数组。

在定义了 `foods.json` 文件后，你可以专注于创建你的 Redux store 了。回顾一下，`store` 是保存你的应用中的真实状态的唯一来源。打开你的 `reducers.js` 文件，并向其中添加以下代码：

```js
import Food from './foods';

const initialState = {
  food: Food,
  searchTerm: '',
};

export default function reducer(state = initialState, action) {
  // switch between the action type
  switch (action.type) {
    case 'SEARCH_INPUT_CHANGED':
      const {searchTerm} = action.payload;
      return {
        ...state,
        searchTerm: searchTerm,
        food: searchTerm ? Food.filter(
          (food) => (food.name.toLowerCase().indexOf(searchTerm.toLowerCase()) > -1)
        ) : Food,
      };
    default:
      return state;
  }
}
```

在上面的代码中，你可以看到 `reducer` 方法接收两个参数：`state` 和 `action`。当你启动你的 React 应用，这个方法将获得它之前定义的初始状态，当你 dispatch 一个 action 的实例时，这个方法将获得当前状态(不再是初始状态)。然后，基于这些 actions 的内容，`reducer` 方法将为你的应用程序生成一个新的状态。

接下来，你必须定义这些 actions 做什么。实际上，为了简单起见，你将定义一个单一的 action ，当用户在你的应用中输入搜索词时，这个 action 会被触发。因此，打开 `actions.js` 文件，并在其中插入以下代码：

```js
function searchTermChanged(searchTerm) {
  return {
    type: 'SEARCH_INPUT_CHANGED',
    payload: {searchTerm},
  };
}

export default {
  searchTermChanged,
};
```

`action` 创建好之后，你需要做的下一件事就是将你的 `app` 组件包装到 `react-redux` 提供的 `Provider` 组件中。Provider 统一负责 React 应用的数据（即 `store`）传递。

要使用 provider ，首先，你将使用 `reducers.js` 中定义的 `initialState` 创建 `store`。然后，通过 `Provider` 组件，你将把 `store` 传给你的 `App`。要完成这些任务，你必须打开 `index.js` 文件，并将其内容替换为：

```js
import React from 'react';
import ReactDOM from 'react-dom';
import {Provider} from 'react-redux';
import {createStore} from 'redux';
import reducers from './reducers';
import App from './App';

// Creating the store using the reducers info.
// That's because reducers are the building blocks of a Redux Store.
const store = createStore(reducers);

ReactDOM.render(
  <Provider store={store}>
    <App/>
  </Provider>,
  document.getElementById('root')
);
```

就是这样！你刚刚在 React 应用中配置完 Redux。现在，你必须实现UI (用户界面)，这样你的用户就可以使用本节中实现的功能了。


## 创建UI界面

现在，你已经完成了应用中的核心代码，你可以专注于构建你的用户界面。为此，打开你的 `App.js` 文件，用下方代码替换它的内容：

```js

import React from 'react';
import {connect} from 'react-redux';
import actions from './actions';
import './App.css';

function App({food, searchTerm, searchTermChanged}) {
  return (
    <div>
      <div className="search">
        <input
          type="text"
          name="search"
          placeholder="Search"
          value={searchTerm}
          onChange={e => searchTermChanged(e.target.value)}
        />
      </div>
      <table>
        <thead>
        <tr>
          <th>Name</th>
          <th>Origin</th>
          <th>Continent</th>
        </tr>
        </thead>
        <tbody>
        {food.map(theFood => (
          <tr key={theFood.name}>
            <td>{theFood.name}</td>
            <td>{theFood.origin}</td>
            <td>{theFood.continent}</td>
          </tr>
        ))}
        </tbody>
      </table>
    </div>
  );
}

export default connect(store => store, actions)(App);

```

对于未用过 Redux 的人来说，他们唯一不熟悉的是用于封装 `App` 组件的 `connect` 方法。这个方法实际上是一个[高阶组件( HOC )](https://reactjs.org/docs/higher-order-components.html)，充当应用程序和Redux之间的粘合剂。

使用以下命令启动你的应用，你将能够在浏览器中访问你的应用：

```bash

npm run start

```

然而，正如你所看到的，这个应用现在很难看。因此，为了让它看起来更好一点，你可以打开 `App.css` 文件，用以下内容替换它的内容:

```css

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
  line-height: 25px;
}

th {
  background-color: #eee;
}

td, th {
  text-align: center;
}

td:first-child {
  text-align: left;
}

input {
  min-width: 300px;
  border: 1px solid #999;
  border-radius: 2px;
  line-height: 25px;
}

```

![Alt text](https://raw.githubusercontent.com/OFED/translation/master/react-context-api-managing-state-with-ease/img/react-app-with-redux.png)

完成了！你现在有了一个基本的 React 和 Redux 的应用，可以开始学习如何迁移到 Context API 上了。

## 用 React Context API 实现 React Apps

在本节，你将要学习如何将你的 Redux 应用迁移到 React Context API上。

幸运的是，你不需要在 Redux 和 the Context API 之间做很多的重构。

作为开始，你必须先从你的应用中移除Redux组件。为此，请打开终端，删除 `redux` 和 `react-redux` 库：

```bash

npm rm redux react-redux

```

之后，删除应用中对这些库的引用代码。打开 `App.js` 删除以下几行：

```js

import {connect} from 'react-redux';
import actions from './actions';

```

然后，在相同的文件中，用下方的代码替换最后一行（以 export default 开头的那一行）：

```js

export default App;

```

有了这些改变，你可以用 Context API 重写你的应用了。

## 从 Redux 迁移到 React Context API

要将之前的应用从 Redux 驱动的应用转换为使用 Context API，你需要一个 context 来存储应用的数据(该 context 将替换 Redux Store)。此外，你还需要一个 `Context.Provider` 组件，该组件包含 `state`、`props` 和正常的 React 组件生命周期。

为此，你需要在src目录中创建一个 `providers.js` 文件，并向其中添加以下代码：

```js

import React from 'react';
import Food from './foods';

const DEFAULT_STATE = { allFood: Food, searchTerm: '' };

export const ThemeContext = React.createContext(DEFAULT_STATE);

export default class Provider extends React.Component {
  state = DEFAULT_STATE;
  searchTermChanged = searchTerm => {
    this.setState({searchTerm});
  };

  render() {
    return (
      <ThemeContext.Provider value={{
        ...this.state,
        searchTermChanged: this.searchTermChanged,
      }}> {this.props.children} </ThemeContext.Provider>);
  }
}

```

上面代码中定义的 `Provider` 类负责将其他组件封装在 `ThemeContext.Provider` 中。通过这样做，你可以让这些组件访问应用中的 state 和 `searchTermChanged` 方法，该方法提供了更改该 state 的方法。

若要稍后在组件树中使用这些值，你需要启动一个 `ThemeContext.Consumer` 组件。这个组件将需要一个 render 渲染方法，该方法将接收上述值 `props` 作为参数。

因此，接下来，你需要在src目录中创建一个名为 `consumer.js` 的文件，并将以下代码写入其中：

```js

import React from 'react';
import {ThemeContext} from './providers';

export default class Consumer extends React.Component {
  render() {
    const {children} = this.props;

    return (
      <ThemeContext.Consumer>
        {({allFood, searchTerm, searchTermChanged}) => {
          const food = searchTerm
            ? allFood.filter(
              food =>
                food.name.toLowerCase().indexOf(searchTerm.toLowerCase()) > -1
            )
            : allFood;

          return React.Children.map(children, child =>
            React.cloneElement(child, {
              food,
              searchTerm,
              searchTermChanged,
            })
          );
        }}
      </ThemeContext.Consumer>
    );
  }
}

```

现在，为了完成迁移，你将打开 `index.js` 文件，并在 `render()` 函数中，用 `Consumer` 组件包装 `App` 组件。此外，你需要将 `Consumer` 包装在 `Provider` 组件中。代码如下所示：

```js

import React from 'react';
import ReactDOM from 'react-dom';
import Provider from './providers';
import Consumer from './consumer';
import App from './App';

ReactDOM.render(
  <Provider>
    <Consumer>
      <App />
    </Consumer>
  </Provider>,
  document.getElementById('root')
);

```

打完收工！你刚刚完成了从 Redux 到 React Context API 的迁移。如果你现在启动你的应用，你会发现整个app运行如常。唯一不同的是，你的应用不再使用 Redux 了。

## 题外话：使用 Auth0 使你的 React 应用更安全

原文中有关于Auth0使用的详细教程，但译者认为此处内容和本文主题关系不大，故不作翻译。感兴趣者可移步[原文](https://auth0.com/blog/react-context-api-managing-state-with-ease/)阅读该部分内容。

## 总结

Redux 是一个高级状态管理库，适合在构建大规模 React 应用时使用。另一方面，The Context API 可以用于字节大小级别数据更改的小规模 React 应用中。通过使用 Context API，你不必像 `reducers`、`actions` 等一样编写大量代码，就能完成状态变化的逻辑表现。

