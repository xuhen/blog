---
title: react hook 如何工作解析
date: 2019-12-27 11:26:03
tags:
---
## 基本示例

```
function Counter() {
    const [state, setState] = useState({
        count: 0,
        color: '#DC0073',
    })

    const { color, count } = state

    useEffect(
        () => { // 1. effect function
            console.log('effect')
            document.title = count

            return () => { // 2. cleanup function
                console.log('cleanup')
            }
        },
        [count], // 3. watch-array
    )

    return (
        <div>
            <span style={{ color }}>Current count is: {count}</span>
            <br />
            <button onClick={() => setState({ ...state, count: count + 1 })}>Increase count</button>
            <button
                onClick={() =>
                    setState({ ...state, color: color === '#DC0073' ? '#00A1E4' : '#DC0073' })
                }
            >
                Change Color
            </button>
        </div>
    )
}
```

## 用didMount式和setInterval的方法实现倒计时

```
// 有问题是实现方式
function Counter() {
    const [count, setCount] = useState(0)

    useEffect(() => {
        const interval = setInterval(() => {
            setCount(count + 1)
        }, 1000)

        return () => {
            clearInterval(interval)
        }
    }, [])

    return (
        <div>
            Current count is: {count}
        </div>
    )
}

// 正确的实现方式
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    color: '#DC0073',
  })

  const { color, count } = state


  console.log('Counter');

  useEffect(() => {
      const interval = setInterval(() => {
          console.log('setCount');
          dispatch({ type: 'increaseCount' })
      }, 1000)

      console.log('useEffect', interval, count);


      return () => {
          console.log('clear', interval);
          clearInterval(interval)
      }
  }, [])

  return (
      <div>
          Current count is: {count}
      </div>
  )
}

```

## 用didUpdate式和setTimeout的方法实现倒计时

```
// 下面两种都正确

function Counter() {
    const [count, setCount] = useState(0)

    useEffect(() => {
        const interval = setTimeout(() => {
            setCount(count + 1)
        }, 1000)

        return () => {
            clearInterval(interval)
        }
    }, [count])

    return (
        <div>
            Current count is: {count}
        </div>
    )
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    color: '#DC0073',
  })

  const { color, count } = state


  console.log('Counter');

  useEffect(() => {
      const interval = setTimeout(() => {
          console.log('setCount');
          dispatch({ type: 'increaseCount' })
      }, 1000)

      console.log('useEffect', interval, count);


      return () => {
          console.log('clear', interval);
          clearInterval(interval)
      }
  }, [count])

  return (
      <div>
          Current count is: {count}
      </div>
  )
}
```

## custom hook

下面是用自定义hook实现倒计时功能（推荐方式）

```
function counterReducer(state, action) {
  console.log('action', action);
  if (action.type === 'increaseCount')
    return { ...state, count: state.count + 1 }
  if (action.type === 'toggleColor')
    return { ...state, color: state.color === '#DC0073' ? '#00A1E4' : '#DC0073' }
  return state
}

function useInterval(callback, delay) {
  const savedCallback = useRef()
  console.log('useInterval');

  useEffect(() => {
      console.log('useEffect1');
      savedCallback.current = callback
  })

  useEffect(() => {
    console.log('useEffect2', delay);
      function tick() {
          savedCallback.current()
      }
      if (delay !== null) {
          let id = setInterval(tick, delay);

          return () => {
            console.log('clear', id);

            clearInterval(id)
          }
      }
  }, [delay])
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    color: '#DC0073',
  })

  const { color, count } = state

  useInterval(() => {
    dispatch({ type: 'increaseCount' })
  }, 1000)


  console.log('Counter', state)

  return (
    <div>
      <span style={{ color }}>Current count is: {count}</span>
      <br />
      <button onClick={() => dispatch({ type: 'increaseCount' })}>Increase count</button>
      <button onClick={() => dispatch({ type: 'toggleColor' })}>Change Color</button>
    </div>
  )
}
```

## useContext Hook

useContext 比Class 组件的 Consumer 更方便点

```
const ColorContext = React.createContext();

function Application() {
  return (
      <ColorContextProvider>
          <Workspace />
      </ColorContextProvider>
  )
}

function Workspace() {
  const { color, toggleColor } = useContext(ColorContext)

  return (
      <div>
          <h1 style={{ color }}>Making sense of React Hooks</h1>
          <Counter />
          <button onClick={toggleColor}>Toggle color</button>
      </div>
  )
}

function ColorContextProvider({ children }) {
  const [color, setColor] = useState('#35CE8D')

  const toggleColor = () => {
      color === '#35CE8D' ? setColor('#721817') : setColor('#35CE8D')
  }

  const value = { color, toggleColor }

  return <ColorContext.Provider value={value}>{children}</ColorContext.Provider>
}


function Counter() {
  const [state, setState] = useState({
      count: 0,
      // color: '#DC0073',
  })

  const { color, toggleColor } = useContext(ColorContext)

  const { count } = state

  return (
      <div>
          <span style={{ color }}>Current count is: {count}</span>
          <br />
          <button onClick={() => setState({ ...state, count: count + 1 })}>Increase count</button>
          <button
              onClick={toggleColor}
          >
              Change Color
          </button>
      </div>
  )
}
```