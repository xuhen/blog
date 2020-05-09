---
title: 倒计时的实现（对比class组件和react hook）
date: 2020-05-09 16:12:24
tags:
    - react
---


## 用hook 实现

```
function useInterval(callback, delay) {
  const savedCallback = useRef();

  useEffect(() => {
    console.log('useEffect 1');
    savedCallback.current = callback;
  });

  useEffect(() => {
    console.log('useEffect 2');

    function tick() {
      savedCallback.current();
    }

    if (delay !== null) {
      let id = setInterval(tick, delay);
      return () => clearInterval(id);
    }

  }, [delay]);

  console.log('useInterval render');
}


function Counter() {
  const [count, setCount] = useState(0);
  const [delay, setDelay] = useState(1000);
  const [isRunning, setIsRunning] = useState(true);

  useInterval(() => {
    setCount(count + 1);
  }, isRunning ? delay : null);


  function handleDelayChange(e) {
    setDelay(Number(e.target.value));
  }

  function toggle() {
    setIsRunning(isRunning ? false : true);
  }
  console.log('Counter render');

  return (
    <div>
      <h1>{count}</h1>
      <input value={delay} onChange={handleDelayChange} />
      <div onClick={toggle}>toggle Timer</div>
    </div>
  )
}
```

这里需要理解的是 `savedCallback.current = callback;` 这句，每次 Counter 渲染都会向 `useInterval` 传入一个包含最新 count的新函数，然后被保存下来。


## 用类组件实现

```
class Counter extends React.Component {
  state = {
    count: 0,
    delay: 1000,
    isRunning: true,
  };

  componentDidMount() {
    this.interval = setInterval(this.tick, this.state.delay);
  }
  componentDidUpdate(prevProps, prevState) {
    
    if (prevState.delay !== this.state.delay || prevState.isRunning !== this.state.isRunning) {
      clearInterval(this.interval);

      if (this.state.isRunning) {
        this.interval = setInterval(this.tick, this.state.delay);
      }
    }
  }
  componentWillUnmount() {
    clearInterval(this.interval);
  }
  tick = () => {
    this.setState({
      count: this.state.count + 1
    });
  }

  handleDelayChange = (e) => {
    this.setState({ delay: Number(e.target.value) });
  }

  toggle = () => {
    this.setState((prevState) => {
      return {
        isRunning: !prevState.isRunning
      }
    });
  }
  render() {
    return (
      <>
        <h1>{this.state.count}</h1>
        <input value={this.state.delay } onChange={this.handleDelayChange} />
        <div onClick={this.toggle}>toggle Time</div>
      </>
    );
  }
}

```

这里我们看到 `componentDidUpdate` 周期函数里delay和isRunning都会决定我们要不要清除倒计时，并且逻辑有点耦合。


## 总结
根据上面的两个例子，我们能看出，调用自定义的hook逻辑比较清晰，而类组件逻辑比较分散且耦合。