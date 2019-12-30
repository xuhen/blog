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

## 在 React Hook 发起请求和错误处理

```
function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'https://hn.algolia.com/api/v1/search?query=redux',
  );
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);


  useEffect(() => {
    console.log('App useEffect');
    async function fetchData() {
      setIsLoading(true);
      setIsError(false);

      try {
        const result = await axios(url);
        setData(result.data);
      } catch (error) {
        setIsError(true);
      }
      setIsLoading(false);
    }
    fetchData();
  }, [url]);

  console.log('App render', url, isLoading, isError);
  return (
    <Fragment>
      <form
        onSubmit={(event) => {
          setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`);
          event.preventDefault();
        }}
      >
        <input
          type="text"
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
        <button type="submit">Search</button>
      </form>

      {isError && <div>Something went wrong ...</div>}

      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>

  );
}
```

## Custom fetch Hook

```
const useApi = (initialUrl, initialData) => {
  const [data, setData] = useState(initialData);
  const [url, setUrl] = useState(initialUrl);
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);
  useEffect(() => {
    const fetchData = async () => {
      setIsError(false);
      setIsLoading(true);
      try {
        const result = await axios(url);
        setData(result.data);
      } catch (error) {
        setIsError(true);
      }
      setIsLoading(false);
    };
    fetchData();
  }, [url]);
  return [{ data, isLoading, isError }, setUrl];
}

function App() {
  const [query, setQuery] = useState('redux');
  const [{ data, isLoading, isError }, doFetch] = useApi(
    'https://hn.algolia.com/api/v1/search?query=redux',
    { hits: [] }
  );

  console.log('App render', isLoading, isError);
  return (
    <Fragment>
      <form
        onSubmit={(event) => {
          doFetch(`http://hn.algolia.com/api/v1/search?query=${query}`);
          event.preventDefault();
        }}
      >
        <input
          type="text"
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
        <button type="submit">Search</button>
      </form>

      {isError && <div>Something went wrong ...</div>}
      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>

  );
}
```

## Custom fetch Hook using useReducer

```
const dataFetchReducer = (state, action) => {
  switch (action.type) {
    case 'FETCH_INIT':
      return {
        ...state,
        isLoading: true,
        isError: false
      };
    case 'FETCH_SUCCESS':
      return {
        ...state,
        isLoading: false,
        isError: false,
        data: action.payload,
      };
    case 'FETCH_FAILURE':
      return {
        ...state,
        isLoading: false,
        isError: true
      };
    default:
      throw new Error();
  }
};
const useApi = (initialUrl, initialData) => {
  const [url, setUrl] = useState(initialUrl);

  const [state, dispatch] = useReducer(dataFetchReducer, {
    isLoading: false,
    isError: false,
    data: initialData,
  });

  useEffect(() => {
    // 这个参数的作用是切换输入框内容时，如果第一个请求还没回来，
    // 就取消掉第一个接口返回数据时的 dispatch
    let didCancel = false;

    const fetchData = async () => {
      dispatch({ type: 'FETCH_INIT' });
      try {
        const result = await axios(url);
        if (!didCancel) {
          dispatch({ type: 'FETCH_SUCCESS', payload: result.data });
        }
      } catch (error) {
        if (!didCancel) {
          dispatch({ type: 'FETCH_FAILURE' });
        }
      }
    };
    fetchData();

    return () => {
      didCancel = true;
    };
  }, [url]);
  return [state, setUrl];
}
```



## 使用React Hook 的问题

* 当请求的数据，依赖 url 变化时

```
function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'https://hn.algolia.com/api/v1/search?query=redux',
  );

  const [isLoading, setIsLoading] = useState(false);


  useEffect(() => {
    console.log('App useEffect');
    async function fetchData() {
      setIsLoading(true);
      const result = await axios(url);
      setData(result.data);
      setIsLoading(false);
    }
    fetchData();
  }, [url]);

  console.log('App render', data);
  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />

      <button type="button" onClick={() => setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)}>
        Search
      </button>

      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>

  );
}
```

上面的代码url不改变点击按钮是不会发起请求的，根据你的业务需求，这可能是一个优化的feature 或者一个 bug。