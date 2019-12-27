---
title: 用 React Hook 配合 Provider 和 Consumer 模拟 Redux 的功能
date: 2019-12-27 15:13:42
tags:
---

自从React正式暴露 context给开发者后一直想实现下 Redux的功能，最近又在研究 React Hook， 发现 useReducer 配合 Provider和Consumer 很容易就实现了 Redux的功能。

其中 Connect 组件是用函数组件实现的，重高阶组件换成高阶函数而已。具体实现看注释。
```
// 全局 context 对象
const Context = React.createContext();


// 初始的state
let initialState = {
  count: 0,
  color: '#DC0073',
  name: 'edward'
};

// 应用的reducer
function Reducer(state = initialState, action) {
  if (action.type === 'increaseCount') return { ...state, count: state.count + 1 }
  if (action.type === 'toggleColor') return { ...state, color: state.color === '#DC0073' ? '#00A1E4' : '#DC0073' }
  if (action.type === 'CHANGEMODEL') return { ...state, ...action.payload }
  return state
}

// Provider 将 全局的 state 和 dispatch 放入 Context.Provider中
function Provider({ children, store = {}}) {
  const [state, dispatch] = useReducer(Reducer, store)
  console.log('Provider', state);
  const value = {
    state,
    dispatch
  }
  return <Context.Provider value={value}>{children}</Context.Provider>
}


// 发出事件的函数
const actions = {
  changeModel(dispatch) {
      return (params) => {
        console.log('changeModel', params);
        console.log('dispatch', dispatch);
        dispatch({
          type: 'CHANGEMODEL',
          payload: {
              ...params
          }
        });
      }
  },
  increaseCount(dispatch) {
    return () => {
      dispatch({
        type: 'increaseCount',
      });
    }
  },
  toggleColor(dispatch) {
    return () => {
      dispatch({
        type: 'toggleColor'
      });
    }
  }
}


// 1. 将 Context.Consumer 里拿出来的数据放到组件props中
// 2. 将actions和Context.Consumer里拿出的dispatch关联，并放入组件props中
function Connect(cb, actions) {
    return (Component) => {
      return (props) => {
        return (
          <Context.Consumer>
            {(value) => {
                let Mapedprops = cb && cb(value.state);
                // 这里需要把全局的传进的prop和组件传入的prop合并下
                let finalProps = {
                  ...props,
                  ...Mapedprops
                };

                let actions_obj = {};

                for (let f in actions) {
                  actions_obj[f] = actions[f](value.dispatch);
                }
                // console.log('actions_obj', actions_obj);

                // console.log('Connect---->', value, actions);
                return <Component {...finalProps} { ...actions_obj }/>;
              }
            }
          </Context.Consumer>
        )
      }
    }

}


// 页面组件
class App extends Component {
  render() {
    // 可以在页面的props拿到 store和 Actions 里的函数发起事件了
    let { store, changeModel, increaseCount, toggleColor } = this.props;
    let { name, color, count } = store;
    console.log('this.props', this.props);
    return (
      <div>
        <span style={{ color }}>Current count is: {count}</span>
        <div>{name}</div>
        <button onClick={() => {
          changeModel({name: 'elen'})
        }}>change</button>
        <button onClick={increaseCount}>Increase count</button>
        <button
            onClick={toggleColor}
        >
            Change Color
        </button>
      </div>
    )
  }
}

let Enhanced = Connect(state => {
  return { store: state };
}, actions)(App)

ReactDOM.render(
    <Provider
      store={initialState}
    >
      <Enhanced />
    </Provider>,
    document.getElementById('app')
);
```

每次发起dispatch都会触发状态的改变 Provider 函数会执行，Connect 的组件也会走一遍更新的周期函数, 如果是 Connect 的组件比较多的话，可以用PureComponent 或 shouldComponentUpdate 进行进一步优化比较好。
以上就是个简易版的Redux功能，主要帮助我理解状态管理库的实现方式，和优化组件的渲染策略。

以下面的代码为例

```
class Parent extends Component {
    render() {
        let { store,  increaseCount } = this.props;
        let {  count } = store;

        return (
            <div>
                <EnhancedChild count={count}></EnhancedChild>
            </div>
        )
    }
}

class Child extends Component {
    render() {
        let { store,  increaseCount } = this.props;
        let { count } = store;

        return (
            <div>
                <div>{count}</div>
                <button onClick={increaseCount}>Increase count</button>
            </div>
        )
    }
}


let EnhancedParent = Connect(state => {
  return { store: state };
}, actions)(Parent)

let EnhancedChild = Connect(state => {
  return { store: state };
}, actions)(Child)
```
总结渲染逻辑：
* 全局状态更新时先触发 `Parent` 组件的 render 然后才是 `Child` 的 render，这样就确保 `Child` render 时是拿到最新的 props； 故只渲染一次Child；
* 像Child 这样被全局 store Connect 的复杂组件, 每次store 变化都会 render。如果只依赖固定的字段跟新的话，可以优化下render策略去提升性能。

优化如下

```
class Child extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    if(nextProps.store.color != this.props.store.color) {
      return true;
    }
    return false;
  }
  render() {
      let { increaseCount } = this.props;
      let { color } = store;

      console.log('Child render', this.props);

      return (
          <div>
              <div>color: {color}</div>
              <button onClick={increaseCount}>Increase count</button>
          </div>
      )
  }
}
```