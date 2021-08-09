# connect()

## 概要

`connect()`関数はReactコンポーネントをRedux storeに接続するために使います。

`connect()`関数(の戻り値)はstoreから必要なデータを抽出して接続済みコンポーネントに渡します。また、`connect()`関数(の戻り値)はactionをstoreにdispatchすることができる関数も接続済みコンポーネントに渡します。

`connect()`関数(の戻り値)は、渡されたコンポーネントクラスに変更を加えません。代わりに、渡されたコンポーネントクラスをラップしたstoreに接続した新しいコンポーネントクラスを返します。

```js
function connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)
```

`mapStateToProps`と`mapDispatchToProps`は、それぞれ、Redux storeの`state`と`dispatch`を扱います。`state`は`mapStateToProps`関数の第1引数として提供されます。`dispatch`は`mapDispatchToProps`関数の第1引数として提供されます。

`mapStateToProps`と`mapDispatchToProps`の戻り値をそれぞれ`stateProps`と`dispatchProps`とします。それらは`mergeProps`が定義されていれば、第1引数と第2引数になります。ちなみに第3引数は`ownProps`です。それらを統合した`mergeProps`の結果は`mergedProps`と言われます。それは接続済みコンポーネントに渡されます。

## `connect()`の引数

`connect`は以下の4つの引数を受け取ります。それらはすべてオプションです。

1. `mapStateToProps?: Function`
2. `mapDispatchToProps?: Function | Object`
3. `mergeProps?: Function`
4. `options?: Object`

### `mapStateToProps?: (state, ownProps?) => Object`

`mapStateToProps`関数が`connect`に渡されると新しく生成されるラップしているコンポーネントはRedux storeの更新をsubscribeします。これはstoreが更新される度、`mapStateToProps`が実行されることを意味します。`mapStateToProps`関数は素のobjectを返す必要があります。これはラップされたコンポーネントのpropsとマージされます。storeの更新をsubscribeしたくない場合は、`mapStateToProps`の所に`null`か`undefined`を渡します。

#### 引数

1. `state: Object`
2. `ownProps?: Object`

`mapStateToProps`関数は最大2つの引数を受け取ります。関数の定義に設定された引数の数によって実行されるタイミングが異なります。つまり、定義時にownPropsを受け取るようにしているかしていないかです。
詳しくは[こちら](#mapToProps関数の引数)を見てください。

##### `state`

`mapStateToProps`関数定義の引数の数が1つの場合、`mapStateToProps`関数はstoreのstateが変更される時に実行されます。その時、store stateのみ引数として受け取ります。

```js
const mapStateToProps = (state) => ({ todos: state.todos })
```

##### `ownProps`

`mapStateToProps`関数定義の引数の数が2つの場合、storeのstateが変更されるもしくは(浅い(shallow)比較に基づいた)新しいpropsを受け取った時に実行されます。それの第1引数はstoreのstateです。第2引数はラップしている(外側の)コンポーネントが受け取るpropsです。第2引数のことを`ownProps`と言います。

```js
const mapStateToProps = (state, ownProps) => ({
  todo: state.todos[ownProps.id],
})
```

#### 戻り値

`mapStateToProps`の戻り値は、(Factory Functionの場合を除いて)objectである必要があります。そのobjectは`stateProps`と言われます。`stateProps`は接続済みコンポーネントのpropsにマージ(merge)されます。`stateProps`は`mergeProps`の第1引数として渡されます。

`mapStateToProps`の戻り値によって接続済みコンポーネントが再レンダリングされるかが決まります。(詳しくは[こちら](https://react-redux.js.org/using-react-redux/connect-mapstate#return-values-determine-if-your-component-re-renders))

`mapStateToProps`を使いこなしたい場合は、[こちら](https://react-redux.js.org/using-react-redux/connect-mapstate)を見てください。

> `mapStateToProps`と`mapDispatchToProps`をfactory function(objectでなく関数を返す関数)として定義している場合、その戻り値の関数が`mapStateToProps`や`mapDispatchToProps`として扱われます。詳しくは[Factory Functions](#factory-functions)や[パフォーマンス最適化に関するガイド](https://react-redux.js.org/using-react-redux/connect-mapstate#only-perform-expensive-operations-when-data-changes)を見てください。

### `mapDispatchToProps?: Object | (dispatch, ownProps?) => Object`

`mapDispatchToProps`は`connect()`の第2引数です。これはobject、関数、渡されないかのいずれかです。

以下のように`connect()`の第2引数(`mapDispatchToProps`)を指定しない場合、コンポーネントはデフォルトで`dispatch`を受け取ります。

```js
// `mapDispatchToProps`を渡さない。
connect()(MyComponent)
connect(mapStateToProps)(MyComponent)
connect(mapStateToProps, null, mergeProps, options)(MyComponent)
```

`mapDispatchToProps`が関数の場合、それは最大で2つの引数を持つ関数として定義することができます。

#### 引数

1. `dispatch: Function`
2. `ownProps?: Object`

##### `dispatch`

`mapDispatchToProps`の引数が1つと定義されている場合、`store`の`dispatch`が渡されます。

```js
const mapDispatchToProps = (dispatch) => {
  return {
    // dispatching plain actions
    increment: () => dispatch({ type: 'INCREMENT' }),
    decrement: () => dispatch({ type: 'DECREMENT' }),
    reset: () => dispatch({ type: 'RESET' }),
  }
}
```

##### `ownProps`

`mapDispatchToProps`の引数が1つと定義されている場合、第1引数には`dispatch`が渡されます。第2引数にはラップしているコンポーネントに渡されたpropsが渡されます。新しいpropsが接続済みコンポーネントに渡される度、`mapDispatchToProps`関数が実行されます。


第2引数は`ownProps`と言われます。

```js
// binds on component re-rendering
<button onClick={() => this.props.toggleTodo(this.props.todoId)} />

// binds on `props` change
const mapDispatchToProps = (dispatch, ownProps) => ({
  toggleTodo: () => dispatch(toggleTodo(ownProps.todoId)),
})
```

`mapDispatchToProps`で定義されている引数の数によってownPropsが渡されるか決まります。詳しくは[これ](#mapToProps関数の引数)を見てください。

#### 戻り値

`mapDispatchToProps`関数はobjectを返す必要があります。そのobjectの各フィールドは関数でなければなりません。その関数が実行されるとactionがstoreにdispatchされます。`mapDispatchToProps`関数の戻り値は`dispatchProps`と言われます。接続済みコンポーネントのpropsにマージされます。`dispatchProps`は`mergeProps`の第2引数になります。

```js
const createMyAction = () => ({ type: 'MY_ACTION' })
const mapDispatchToProps = (dispatch, ownProps) => {
  const boundActions = bindActionCreators({ createMyAction }, dispatch)
  return {
    dispatchPlainObject: () => dispatch({ type: 'MY_ACTION' }),
    dispatchActionCreatedByActionCreator: () => dispatch(createMyAction()),
    ...boundActions,
    // こうやってdispatchを渡すこともできます。
    dispatch,
  }
}
```

mapDispatchToPropsを使いこなしたい場合は、[こちら](https://react-redux.js.org/using-react-redux/connect-mapdispatch)を見てください。

> `mapStateToProps`と`mapDispatchToProps`をfactory function(objectでなく関数を返す関数)として定義している場合、その戻り値の関数が`mapStateToProps`や`mapDispatchToProps`として扱われます。詳しくは[Factory Functions](#factory-functions)や[パフォーマンス最適化に関するガイド](https://react-redux.js.org/using-react-redux/connect-mapstate#only-perform-expensive-operations-when-data-changes)を見てください。

#### Object Shorthand Form

以下のように、`mapDispatchToProps`は[action creator](https://japanese-document.github.io/redux/glossary.html#action-creator)を値に持つobjectにすることもできます。

```js
import { addTodo, deleteTodo, toggleTodo } from './actionCreators'

const mapDispatchToProps = {
  addTodo,
  deleteTodo,
  toggleTodo,
}

export default connect(null, mapDispatchToProps)(TodoApp)
```

このケースではReact-Reduxは`bindActionCreators`を使って各action creatorをstoreの`dispatch`にbindします。`mapDispatchToProps`の戻り値は`dispatchProps`と言われます。接続済みコンポーネントのpropsにマージされるか`mergeProps`の第2引数になります。

```js
// internally, React-Redux calls bindActionCreators
// to bind the action creators to the dispatch of your store
bindActionCreators(mapDispatchToProps, dispatch)
```

`mapDispatchToProps`のObject Shorthand Formを使ったやり方について詳しく知りたい場合は[こちら](https://react-redux.js.org/using-react-redux/connect-mapdispatch#defining-mapdispatchtoprops-as-an-object)を見てください。

### `mergeProps?: (stateProps, dispatchProps, ownProps) => Object`

`mergeProps`はラップされたコンポーネントに最終的に渡るpropsを決定します。`mergeProps`がconnect関数に渡されてない場合、ラップされたコンポーネントにはデフォルトで`{ ...ownProps, ...stateProps, ...dispatchProps }` が渡されます。


#### 引数

`mergeProps`は最大で3つの引数を渡すことができます。それらの引数は、それぞれ、`mapStateToProps()`と`mapDispatchToProps()`の戻り値とラップしているコンポーネントの`props`です。


1. `stateProps`
2. `dispatchProps`
3. `ownProps`

`mergeProps`が返す素のobjectにある各値はラップされたコンポーネントのpropsになります。この関数でpropsに応じてstateの一部らから更に値を抽出したり、action creatorに特定のpropsの値をbindしたりすることができます。

#### 戻り値

`mergeProps`の戻り値は`mergedProps`と言われます。これの各値はラップされたコンポーネントのpropsになります。

### `options?: Object`

```js
{
  context?: Object,
  pure?: boolean,
  areStatesEqual?: Function,
  areOwnPropsEqual?: Function,
  areStatePropsEqual?: Function,
  areMergedPropsEqual?: Function,
  forwardRef?: boolean,
}
```

#### `context: Object`

> 注意: この変数は6.0以上でサポートされています。

React-Redux 6.0からReact-Redux内で使用できるカスタムcontextインスタンスを設定できるようになりました。contextインスタンスを`<Provider />`と接続済みコンポーネントに渡す必要があります。contextインスタンスを接続済みコンポーネントに渡す方法は、このオプションを使う方法とレンダリング時に接続済みコンポーネントのprops経由で渡す方法があります。

```js
// const MyContext = React.createContext();
connect(mapStateToProps, mapDispatchToProps, null, { context: MyContext })(
  MyComponent
)
```

#### `pure: boolean`

- default value: `true`

ラップされたコンポーネントは"pure" componentとみなされ、propsやRedux Storeのstate以外の入力やstateには依存しないとみなされます。

`options.pure`の場合、`connect`は不要な`mapStateToProps`、`mapDispatchToProps`、`mergeProps`、`render`の呼び出しを予防するために複数の等価チェックを行います。その等価チェックは`areStatesEqual`、`areOwnPropsEqual`、`areStatePropsEqual`、`areMergedPropsEqual`です。これらの関数はデフォルトの物で99%の場合で問題ありませんが、パフォーマンスや別の理由でこれらを置き換えたいかもしれません。

以下でそれらの例を示します。

#### `areStatesEqual: (next: Object, prev: Object) => boolean`

- default value: `strictEqual: (next, prev) => prev === next`

pureが有効な場合、現在のstore stateと1つ前のそれを比較します。

_例 1_

```js
const areStatesEqual = (next, prev) =>
  prev.entities.todos === next.entities.todos
```

`mapStateToProps`の処理コストが高い、かつ、それがstateのごく一部のみに依存している場合は`areStatesEqual`を上書きすると良いと思います。上の例では指定したstateの一部以外のstateの変化は無視されます。

_例 2_

store stateを変更するimpure reducersが存在している場合、`areStatesEqual` は常にfalseを返すと良いと思います。

```js
const areStatesEqual = () => false
```

おそらく、これは`mapStateToProps`関数によって他の等価チェックに影響を及ぼすと思います。

#### `areOwnPropsEqual: (next: Object, prev: Object) => boolean`

- default value: `shallowEqual: (objA, objB) => boolean`
  ( 各objectの各値が等しい場合、trueを返します。 )

pureが有効な場合、現在のpropsと1つ前のpropsを比較します。

現在のpropsを通過させたい場合、`areOwnPropsEqual`を上書きすると良いと思います。propsを通過させるには、`mapStateToProps`、`mapDispatchToProps`、`mergeProps`を実装する必要があります。([recomposeのmapProps](https://github.com/acdlite/recompose/blob/master/docs/API.md#mapprops)のような別の方法の方が簡単かもしれません。)

#### `areStatePropsEqual: (next: Object, prev: Object) => boolean`

- type: `function`
- default value: `shallowEqual`

pureが有効な場合、現在の`mapStateToProps`戻り値と1つ前のそれを比較します。

#### `areMergedPropsEqual: (next: Object, prev: Object) => boolean`

- default value: `shallowEqual`

pureが有効な場合、現在の`mergeProps`戻り値と1つ前のそれを比較します。

`mapStateToProps`が関連するpropsが変更された場合のみ新しいobjectを返すメモ化されたselectorを使っているなら、`areStatePropsEqual`を上書きして`strictEqual`にした方が良いと思います。`mapStateToProps`が実行される度、各propsの余計な等価性チェックを避けることができるので少しだけパフォーマンスが改善します。

selectorが複雑なpropsを生成する場合、`areMergedPropsEqual`を上書きして`deepEqual`な実装にすると良いと思います。(たぶん、deep equalは再レンダリングよりも速いと思います。)

#### `forwardRef: boolean`

> 注意: このパラメータはバージョン6.0からサポートされています。

`{forwardRef : true}`が`connect`に渡されると、`ref`をラップしている接続済みコンポーネントに渡すと`ref.current`は内側のラップされたコンポーネントを参照します。

## `connect()`の戻り値

`connect()`はコンポーネントを受け取って、注入される追加のpropsを持つそれをラップしているコンポーネントを返すラッパー関数です。

```js
import { login, logout } from './actionCreators'

const mapState = (state) => state.user
const mapDispatch = { login, logout }

// 1回目: コンポーネントをラップするために使うHOCを返します。
const connectUser = connect(mapState, mapDispatch)

// 2回目: mergedPropsを持つラップしているコンポーネントを返します。
// HOCを使うと異なるコンポーネントに同じ動作を追加することができます。
const ConnectedUserLogin = connectUser(Login)
const ConnectedUserProfile = connectUser(Profile)
```

通常、ラッパー関数は変数に格納されることなく、すぐに実行されます。

```js
import { login, logout } from './actionCreators'

const mapState = (state) => state.user
const mapDispatch = { login, logout }

// connect関数を実行して、すぐにラッパー関数を実行して最終的なラッパーコンポーネントを生成します。

export default connect(mapState, mapDispatch)(Login)
```

## 使用例

`connect`はとても柔軟なので、いろいろな使い方ができます。その使用例を見ていきましょう。

- 単に`dispatch`を注入してstoreをsubscribeしない。

```js
export default connect()(TodoApp)
```

- storeをsubscribeなしで、すべてのaction creatorを注入する。

```js
import * as actionCreators from './actionCreators'

export default connect(null, actionCreators)(TodoApp)
```

- `dispatch`とglobal stateの各フィールドを注入する。

> 以下のようにしないでください。`TodoApp`はstateが変更される度、再レンダリングされるのでパフォーマンスの最適化が犠牲になります。
> だから、個々のコンポーネントが必要なstaetの部分のみをsubscribeするようにした方が良いです。

```js
// これはダメ！
export default connect((state) => state)(TodoApp)
```

- `dispatch`と`todos`を注入します。

```js
function mapStateToProps(state) {
  return { todos: state.todos }
}

export default connect(mapStateToProps)(TodoApp)
```

- `todos`とすべてのaction creatorを注入します。

```js
import * as actionCreators from './actionCreators'

function mapStateToProps(state) {
  return { todos: state.todos }
}

export default connect(mapStateToProps, actionCreators)(TodoApp)
```

-　`todos`とすべてのaction creator(`addTodo`, `completeTodo`, ...)を`actions`として注入します。

```js
import * as actionCreators from './actionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return { actions: bindActionCreators(actionCreators, dispatch) }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- `todos`と特定のaction creator(`addTodo`)を注入します。

```js
import { addTodo } from './actionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators({ addTodo }, dispatch)
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- `todos`と特定のaction creator(`addTodo`と`deleteTodo`)を注入します。

```js
import { addTodo, deleteTodo } from './actionCreators'

function mapStateToProps(state) {
  return { todos: state.todos }
}

const mapDispatchToProps = {
  addTodo,
  deleteTodo,
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- `todos`と`todoActionCreators`と`todoActions`として、`counterActionCreators`を`counterActions`として注入します。

```js
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return {
    todoActions: bindActionCreators(todoActionCreators, dispatch),
    counterActions: bindActionCreators(counterActionCreators, dispatch),
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- `todos`と、todoActionCreatorsとcounterActionCreatorsを統合して`actions`として注入します。

```js
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return {
    actions: bindActionCreators(
      { ...todoActionCreators, ...counterActionCreators },
      dispatch
    ),
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- `todas`と、すべての`todoActionCreators`と`counterActionCreators`の各action creatorをpropsとして注入します。

```js
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators(
    { ...todoActionCreators, ...counterActionCreators },
    dispatch
  )
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- propsに応じて特定のuserの`todos`を`todos`として注入します。

```js
import * as actionCreators from './actionCreators'

function mapStateToProps(state, ownProps) {
  return { todos: state.todos[ownProps.userId] }
}

export default connect(mapStateToProps)(TodoApp)
```

- propsに応じて特定のuserの`todos`を`todos`として注入します。そして、`props.userId`をactionに注入します。

```js
import * as actionCreators from './actionCreators'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mergeProps(stateProps, dispatchProps, ownProps) {
  return Object.assign({}, ownProps, {
    todos: stateProps.todos[ownProps.userId],
    addTodo: (text) => dispatchProps.addTodo(ownProps.userId, text),
  })
}

export default connect(mapStateToProps, actionCreators, mergeProps)(TodoApp)
```

## 注意点

### mapToProps関数の引数

`mapStateToProps`と`mapDispatchToProps`関数定義の引数の数によって、引数に`ownProps`を受け取るか決まります。

> `mapStateToProps`と`mapDispatchToProps`関数定義関数定義で引数の数が1つ(func.lengthが1)の場合、`ownProps`は`mapStateToProps`と`mapDispatchToProps`関数に渡されません。例えば、以下のように定義された関数は第2引数に`ownProps`を受け取りません。`ownProps`が`undefined`の場合、デフォルトの値が使われます。

```js
function mapStateToProps(state) {
  console.log(state) // state
  console.log(arguments[1]) // undefined
}

const mapStateToProps = (state, ownProps = {}) => {
  console.log(state) // state
  console.log(ownProps) // {}
}
// console.log(mapStateToProps.length) => 1
```

関数が0個または2以上の個数の引数を受け取るように定義されている場合、ownPropsが渡されます。

```js
const mapStateToProps = (state, ownProps) => {
  console.log(state) // state
  console.log(ownProps) // ownProps
}

function mapStateToProps() {
  console.log(arguments[0]) // state
  console.log(arguments[1]) // ownProps
}

const mapStateToProps = (...args) => {
  console.log(args[0]) // state
  console.log(args[1]) // ownProps
}
```

### Factory Functions

`mapStateToProps`もしくは`mapDispatchToProps`関数が関数を返す場合、コンポーネントがインスタンス化する際に一度だけ実行されます。それらの戻り値は、それ以後、それぞれ`mapStateToProps`と`mapDispatchToProps`関数として取り扱われます。

factory functionsはメモ化されたselectorによく使われます。これによって、クロージャー内にコンポーネントインスタンスごとにselectorを作成することが可能になります。

```js
const makeUniqueSelectorInstance = () =>
  createSelector([selectItems, selectItemId], (items, itemId) => items[itemId])
const makeMapState = (state) => {
  const selectItemForThisComponent = makeUniqueSelectorInstance()
  return function realMapState(state, ownProps) {
    const item = selectItemForThisComponent(state, ownProps.itemId)
    return { item }
  }
}
export default connect(makeMapState)(SomeComponent)
```

## License

### Japanese part

Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)

Copyright (c) 2021 38elements

### Other

The MIT License (MIT)

Copyright (c) 2015-present Dan Abramov

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

<hr/>

* [Redux](https://japanese-document.github.io/redux/)

* [React Redux](https://japanese-document.github.io/react-redux/)

* [Redux Thunk](https://japanese-document.github.io/redux-thunk/)
