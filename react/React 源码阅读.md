# React 源码阅读

## 1. jsx转化为virtualDom

### 1.1 createElement

文件位置：react\packages\react\src\ReactElement.js

```javascript
/**
 * Create and return a new ReactElement of the given type.
 * See https://reactjs.org/docs/react-api.html#createelement
 * 1.从config中提取特殊属性
 * 2.将config中的非特殊属性存放到props中
 * 3.将子元素对象存放到props.children中
 * 4.为props的属性设置默认值
 * 5.创建并返回reactElement对象
 */
export function createElement(type, config, children) {
  let propName;

  // Reserved names are extracted
  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  // 从config中提取特殊属性，并将其他属性存放到props中
  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref;

      // __DEV__ 为true，表示开发环境
      if (__DEV__) {
        warnIfStringRefCannotBeAutoConverted(config);
      }
    }
    if (hasValidKey(config)) {
      key = '' + config.key;
    }

    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // Remaining properties are added to a new props object
    // 排除特殊属性 key ref __self __source 将其他属性保存到props中
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  // Children can be more than one argument, and those are transferred onto
  // the newly allocated props object.
  // 从第三个参数开始为子节点
  // 如果只有一个子节点，则props.children为一个对象
  // 如果有多个子节点，props.children为一个数组
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    if (__DEV__) {
      if (Object.freeze) {
        Object.freeze(childArray);
      }
    }
    props.children = childArray;
  }

  // Resolve default props
  // 为props中的属性设置默认值
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  // 开发环境下如果通过props去获取特殊属性的值，提示警告
  if (__DEV__) {
    if (key || ref) {
      const displayName =
        typeof type === 'function'
          ? type.displayName || type.name || 'Unknown'
          : type;
      if (key) {
        defineKeyPropWarningGetter(props, displayName);
      }
      if (ref) {
        defineRefPropWarningGetter(props, displayName);
      }
    }
  }
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```

### 1.2 hasValidRef

文件位置：react\packages\react\src\ReactElement.js

```javascript
// 判断ref是否合法
function hasValidRef(config) {
  if (__DEV__) {
    // 判断config中是否含有ref属性
    if (hasOwnProperty.call(config, 'ref')) {
      const getter = Object.getOwnPropertyDescriptor(config, 'ref').get;
      if (getter && getter.isReactWarning) {
        return false;
      }
    }
  }
  // ref 不为 undefined 即可
  return config.ref !== undefined;
}
```

### 1.3 warnIfStringRefCannotBeAutoConverted

文件位置：react\packages\react\src\ReactElement.js

```javascript
function warnIfStringRefCannotBeAutoConverted(config) {
  if (__DEV__) {
    if (
      typeof config.ref === 'string' &&
      ReactCurrentOwner.current &&
      config.__self &&
      ReactCurrentOwner.current.stateNode !== config.__self
    ) {
      const componentName = getComponentName(ReactCurrentOwner.current.type);

      if (!didWarnAboutStringRefs[componentName]) {
        console.error(
          'Component "%s" contains the string ref "%s". ' +
            'Support for string refs will be removed in a future major release. ' +
            'This case cannot be automatically converted to an arrow function. ' +
            'We ask you to manually fix this case by using useRef() or createRef() instead. ' +
            'Learn more about using refs safely here: ' +
            'https://fb.me/react-strict-mode-string-ref',
          getComponentName(ReactCurrentOwner.current.type),
          config.ref,
        );
        didWarnAboutStringRefs[componentName] = true;
      }
    }
  }
}
```

### 1.4 hasValidKey

文件位置：react\packages\react\src\ReactElement.js

```javascript
function hasValidKey(config) {
  if (__DEV__) {
    if (hasOwnProperty.call(config, 'key')) {
      const getter = Object.getOwnPropertyDescriptor(config, 'key').get;
      if (getter && getter.isReactWarning) {
        return false;
      }
    }
  }
  return config.key !== undefined;
}
```

### 1.5 RESERVED_PROPS

```javascript
const RESERVED_PROPS = {
  key: true,
  ref: true,
  __self: true,
  __source: true,
};
```



## 2. 虚拟Dom渲染

### 2.1 render