在 saga 中通过 takeEvery 订阅 action 时，takeEvery 第一个参数可以是string，array 和 function。当参数为function时，将匹配 `pattern(action)` 为 true 的 action。（例如，`take(action => action.entities)` 将匹配哪些 `entities` 字段为真的 action）。

但是在定义下列的saga函数是，如果 `incrementSaga`是一个自定义函数，将一直触发 action。

```js
export const counterIncrementSaga = function * () {
  console.log(incrementSaga)
  yield takeEvery(incrementSaga, asyncIncrement)
}
```

自定义 `incrementSaga`

```js
export const incrementSaga = payload => {
	console.log(payload)
  const action = {
    type: INCREMENT_SAGA
  }
  if (payload) action.payload = payload
  return action
}
```

![image-20210629091533796](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210629091533796.png)

但如果使用 redux-action 的 createAction 来生成 action 时，就不会有这种现象

```js
export default function createAction(type, payloadCreator, metaCreator) {
  if (payloadCreator === void 0) {
    payloadCreator = identity;
  }

  invariant(isFunction(payloadCreator) || isNull(payloadCreator), 'Expected payloadCreator to be a function, undefined or null');
  var finalPayloadCreator = isNull(payloadCreator) || payloadCreator === identity ? identity : function (head) {
    for (var _len = arguments.length, args = new Array(_len > 1 ? _len - 1 : 0), _key = 1; _key < _len; _key++) {
      args[_key - 1] = arguments[_key];
    }

    return head instanceof Error ? head : payloadCreator.apply(void 0, [head].concat(args));
  };
  var hasMeta = isFunction(metaCreator);
  var typeString = type.toString();

  var actionCreator = function actionCreator() {
    var payload = finalPayloadCreator.apply(void 0, arguments);
    var action = {
      type: type
    };

    if (payload instanceof Error) {
      action.error = true;
    }

    if (payload !== undefined) {
      action.payload = payload;
    }

    if (hasMeta) {
      action.meta = metaCreator.apply(void 0, arguments);
    }

    return action;
  };

  actionCreator.toString = function () {
    return typeString;
  };

  return actionCreator;
```

原因分析：

> 注意: 如果 pattern 函数上定义了 `toString`，`action.type` 将改用 `pattern.toString` 来测试。这个设定在你使用 action 创建函数库（如 redux-act 或 redux-actions）时非常有用。

所以只要给自定义的`incrementSaga`加上 toString 函数，使其返回正确的 type 即可。

```js
export const incrementSaga = payload => {
  const action = {
    type: INCREMENT_SAGA
  }
  if (payload) action.payload = payload
  return action
}
incrementSaga.toString = () => INCREMENT_SAGA
```

