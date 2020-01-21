---
title: 分析 redux and friends 的实现原理
date: 2020-01-08 17:27:00
tags:
  - redux
---

由于代码量很大，请看 comments。

## redux

```
const validateAction = action => {
    if (!action || typeof action !== 'object' || Array.isArray(action)) {
      throw new Error('Action must be an object!');
    }
    if (typeof action.type === 'undefined') {
      throw new Error('Action must have a type!');
    }
};

const createStore = (reducer, middleware) => {
    let state;
    const subscribers = [];
    const coreDispatch = action => {
        validateAction(action);
        state = reducer(state, action);
        subscribers.forEach(handler => handler());
    };
    const getState = () => state;
    const store = {
        dispatch: coreDispatch,
        getState,
        subscribe: handler => {
            subscribers.push(handler);
            return () => {
              const index = subscribers.indexOf(handler);
              if (index > 0) {
                subscribers.splice(index, 1);
              }
            };
        }
    };

    if (middleware) {
        const dispatch = action => store.dispatch(action);
        store.dispatch = middleware({
          dispatch,
          getState
        })(coreDispatch);
    }

    // 创建初始store
    coreDispatch({type: '@@redux/INIT'});

    return store;
};

const applyMiddleware = (...middlewares) => store => {
    if (middlewares.length === 0) {
      return dispatch => dispatch;
    }
    if (middlewares.length === 1) {
      return middlewares[0](store);
    }
    const boundMiddlewares = middlewares.map(middleware => middleware(store));

    // 这里是精华部分将 coreDispatch 透传到每个 Middleware中
    return boundMiddlewares.reduce((a, b) => {
        return next => a(b(next))
    });


};
```


## react-redux


```
// 将store变量和actions和props合并传入被连接的组件的props
const connect = (
    mapStateToProps = () => ({}),
    mapDispatchToProps = () => ({})
  ) => Component => {
    class Connected extends React.Component {
      constructor(props, context) {
        super(props, context);

        // 从context拿到store
        const {store} = context;
        const state = store.getState();
        // 将store属性拿取需要的字段
        const stateProps = mapStateToProps(state);
        const dispatchProps = mapDispatchToProps(store.dispatch);

        this.state = {
            stateProps,
            dispatchProps,
            props
        };

        this.unsubscribe = store.subscribe(() => {
            this.onStoreOrPropsChange(props)
        });
      }
      onStoreOrPropsChange(props) {

        const {store} = this.context;
        const state = store.getState();
        const stateProps = mapStateToProps(state, props);
        // 将store上的dispatch传入actions 对象的闭包内并返回actions 对象
        const dispatchProps = mapDispatchToProps(store.dispatch, props);


        this.setState({
          stateProps,
          dispatchProps
        });
      }

        // 更新state里的props字段
      static getDerivedStateFromProps(nextProps, prevState) {
          console.log('getDerivedStateFromProps', nextProps === prevState.props);
        if (nextProps !== prevState.props) {
            return {
                props: nextProps
            };
        }
        return null;
      }

      shallowDiffers (a, b) {
        for (let i in a) if (!(i in b)) return true
        for (let i in b) if (a[i] !== b[i]) return true
        return false
      }
    // 优化渲染性能
    shouldComponentUpdate(nextProps, nextState) {
        return (
            this.shallowDiffers(nextState.props, this.state.props) ||
            this.shallowDiffers(nextState.stateProps, this.state.stateProps) ||
            this.shallowDiffers(nextState.dispatchProps, this.state.dispatchProps)
        )
    }
      componentWillUnmount() {
        this.unsubscribe();
      }
      render() {
        return (
            <Component
                {...this.state.stateProps}
                {...this.state.dispatchProps}
                {...this.state.props}
            />
        );
      }

    }

    Connected.contextTypes = {
      store: PropTypes.object
    };

    return Connected;
}


class Provider extends React.Component {
    constructor(props) {
        super(props);
    }
    // 将store 放入 context 中
    getChildContext() {
      return {
        store: this.props.store
      };
    }
    render() {
        return this.props.children;
    }
}
Provider.childContextTypes = {
    store: PropTypes.object
};
```


## Middleware

```
const delayMiddleware = () => next => action => {
    setTimeout(() => {
      next(action);
    }, 1000);
};

const loggingMiddleware = ({getState}) => next => action => {
    console.info('before', getState());
    console.info('action', action);
    const result = next(action);
    console.info('after', getState());
    return result;
};

const thunkMiddleware = ({dispatch, getState}) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState);
    }
    return next(action);
};
```

## App

```
const CREATE_NOTE = 'CREATE_NOTE';
const UPDATE_NOTE = 'UPDATE_NOTE';
const OPEN_NOTE = 'OPEN_NOTE';
const CLOSE_NOTE = 'CLOSE_NOTE';

const initialState = {
    nextNoteId: 1,
    openNoteId: null,
    notes: {},
    isLoading: false
};

const reducer = (state = initialState, action) => {

  switch (action.type) {
    case CREATE_NOTE: {
        if (!action.id) {
            return {
              ...state,
              isLoading: true
            };
        }

        const newNote = {
            id: action.id,
            content: ''
        };
        return {
            ...state,
            isLoading: false,
            openNoteId: action.id,
            notes: {
              ...state.notes,
              [action.id]: newNote
            }
        };

        // const id = state.nextNoteId;
        // const {content} = action;

        // const newNote = {
        //   id,
        //   content: content || ''
        // };
        // return {
        //   ...state,
        //   nextNoteId: id + 1,
        //   notes: {
        //     ...state.notes,
        //     [id]: newNote
        //   }
        // };
    }

    case UPDATE_NOTE: {
        const {id, content} = action;
        const editedNote = {
          ...state.notes[id],
          content
        };
        return {
          ...state,
          notes: {
            ...state.notes,
            [id]: editedNote
          }
        };
    }

    case OPEN_NOTE: {
        return {
          ...state,
          openNoteId: action.id
        };
    }

    case CLOSE_NOTE: {
        return {
          ...state,
          openNoteId: null
        };
    }

    default:
      return state
  }
};


const store = createStore(reducer, applyMiddleware(
    // delayMiddleware,
    loggingMiddleware,
    thunkMiddleware
));


const NoteEditor = ({note, onChangeNote, onCloseNote}) => (
  <div>
    <div>
      <textarea
        className="editor-content"
        autoFocus
        value={note.content}
        onChange={event => onChangeNote(note.id, event.target.value)}
        rows={10} cols={80}
      />
    </div>
    <button className="editor-button" onClick={onCloseNote}>Close</button>
  </div>
);

const NoteTitle = ({note}) => {
  const title = note.content.split('\n')[0].replace(/^\s+|\s+$/g, '');
  if (title === '') {
    return <i>Untitled</i>;
  }
  return <span>{title}</span>;
};

const NoteLink = ({note, onOpenNote}) => (
  <li className="note-list-item">
    <a href="#" onClick={() => onOpenNote(note.id)}>
      <NoteTitle note={note}/>
    </a>
  </li>
);

const NoteList = ({notes, onOpenNote}) => (
  <ul className="note-list">
    {
      Object.keys(notes).map(id =>
        <NoteLink
          key={id}
          note={notes[id]}
          onOpenNote={onOpenNote}
        />
      )
    }
  </ul>
);

const NoteApp = ({
  notes, openNoteId, onAddNote, onChangeNote,
  onOpenNote, onCloseNote,
  name
}) => {

    return (
        <div>
            {name}
          {
            openNoteId ?
              <NoteEditor
                note={notes[openNoteId]} onChangeNote={onChangeNote}
                onCloseNote={onCloseNote}
              /> :
              <div>
                <NoteList notes={notes} onOpenNote={onOpenNote}/>
                <button className="editor-button" onClick={onAddNote}>New Note</button>
              </div>
          }
        </div>
    );
}


const createFakeApi = () => {
    let _id = 0;
    const createNote = () => new Promise(resolve => setTimeout(() => {
        _id++
        resolve({
            id: `${_id}`
        })
    }, 1000));
    return {
        createNote
    };
};

const api = createFakeApi()
const mapStateToProps = state => ({
    notes: state.notes,
    openNoteId: state.openNoteId
  });

const mapDispatchToProps = dispatch => ({
    // onAddNote: () => {
    //     console.log('onAddNote');
    //     dispatch({
    //         type: CREATE_NOTE
    //     })
    // },
    onAddNote: () => {
        dispatch(
        (dispatch) => {
        //   dispatch({
        //     type: CREATE_NOTE
        //   });
          api.createNote()
            .then(({id}) => {
              dispatch({
                type: CREATE_NOTE,
                id
              });
            });
        })
    },
    onChangeNote: (id, content) => dispatch({
      type: UPDATE_NOTE,
      id,
      content
    }),
    onOpenNote: id => dispatch({
      type: OPEN_NOTE,
      id
    }),
    onCloseNote: () => dispatch({
      type: CLOSE_NOTE
    })
});

const NoteAppContainer = connect(
    mapStateToProps,
    mapDispatchToProps
)(NoteApp);


class App1 extends React.Component {
    constructor(props, context) {
        console.log('App-->', context);
        super(props);
        this.state = {
            name: 'tom'
        };

        this.changeName = this.changeName.bind(this);
    }

    changeName() {
        this.setState({
            age: 'edward'
        });
    }

    render() {
        return (
            <div>
                <NoteAppContainer name={this.state.name}/>
                <div onClick={this.changeName}>change name</div>
            </div>
        )
    }
}

ReactDOM.render(
    <Provider store={store}>
        <App1></App1>
    </Provider>,
    document.getElementById('root')
);

```