Formik 是 React 官方推荐的用来增强表单的功能的第三方模块，使用 Formik 之后我们只需要关注自己的业务逻辑而不用分心处理表单技术细节。

只能在函数组件中使用

# 1. 基本使用

## 安装

```bash
yarn add formik
```



## 使用

1. 首先通过 useFormik 生成一个 formik 对象，通过设置 initialValues 来设置表单的初始值对象，通过 onSubmit 来设置表单提交的响应事件。

2. 为 form 标签绑定 onSumbit 时间处理函数，绑定的值需要是 formik.handleSubmit
3. 为每个表单标签指定一个 name， 值必须和初始值对象的属性对应，然后绑定 value 和 onChange 事件。

```jsx
import logo from './logo.svg';
import './App.css';
import { useFormik } from 'formik'

function App() {
  const formik = useFormik({
    initialValues: {
      name: '123',
      password: 'sdfs'
    },
    onSubmit: values => {
      console.log(values);
    }
  })
  return (
    <div className="App">
      <form onSubmit={formik.handleSubmit}>
        <input name="name" type="text" value={formik.values.name} onChange={formik.handleChange} />
        <input name="password" value={formik.values.password} onChange={formik.handleChange} type="password" />
        <button type="submit">提交</button>
      </form>
    </div>
  );
}

export default App;
```



# 2. 表单验证

## 1. 基本验证功能

在生成 formik 对象时，传递 validate 属性，该属性是一个方法，方法接收一个 values 对象表示当前所有表单项的值，并需要返回一个错误对象，存储表单的错误信息。

```jsx
import logo from './logo.svg';
import './App.css';
import { useFormik } from 'formik'

function App() {
  const formik = useFormik({
    initialValues: {
      name: '',
      password: ''
    },
    validate: values => {
      let error = {}
      if (!values.name) error.name = '请输入用户名'
      if (!values.password) error.password = '请输入密码'
      return error
    },
    onSubmit: values => {
      console.log(values);
    }
  })
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <div>
          <form onSubmit={formik.handleSubmit}>
            <input name="name" type="text" value={formik.values.name} onChange={formik.handleChange}/>
            <p>{ formik.errors.name || '' }</p>
            <input name="password" value={formik.values.password} onChange={formik.handleChange} type="password"/>
            <p>{ formik.errors.password || '' }</p>
            <button type="submit">提交</button>
          </form>
        </div>
      </header>
    </div>
  );
}

export default App;

```



## 2. 限定只展示当前修改的表单项的错误信息

1. 为表单项绑定 onBlur 事件为 formik.handleBlur
2. 在展示错误信息的判断中添加 formik.touched.[name] 来判断是否是编辑当前表单项。

```jsx
import logo from './logo.svg';
import './App.css';
import { useFormik } from 'formik'

function App() {
  const formik = useFormik({
    initialValues: {
      name: '',
      password: ''
    },
    validate: values => {
      let error = {}
      if (!values.name) error.name = '请输入用户名'
      if (!values.password) error.password = '请输入密码'
      return error
    },
    onSubmit: values => {
      console.log(values);
    }
  })
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <div>
          <form onSubmit={formik.handleSubmit}>
            <input name="name" type="text" value={formik.values.name} onChange={formik.handleChange} onBlur={formik.handleBlur} />
            <p>{ (formik.touched.name && formik.errors.name) || '' }</p>
            <input name="password" value={formik.values.password} onChange={formik.handleChange} type="password"  onBlur={formik.handleBlur} />
            <p>{ (formik.touched.password && formik.errors.password) || '' }</p>
            <button type="submit">提交</button>
          </form>
        </div>
      </header>
    </div>
  );
}

export default App;
```



# 3. 和 Yup 一起使用

yup 是一个表单验证的第三方库

```jsx
import logo from './logo.svg';
import './App.css';
import { useFormik } from 'formik'
import * as Yup from 'yup'

function App() {
  const formik = useFormik({
    initialValues: {
      name: '',
      password: ''
    },
    // 自定义表单验证
    validationSchema: Yup.object({
      name: Yup.string().required('请输入用户名'),
      password: Yup.string().required('请输入密码').min(6, '请输入最少6位密码')
    }),
    onSubmit: values => {
      console.log(values);
    }
  })
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <div>
          <form onSubmit={formik.handleSubmit}>
            <input name="name" type="text" value={formik.values.name} onChange={formik.handleChange} onBlur={formik.handleBlur} />
            <p>{ (formik.touched.name && formik.errors.name) || '' }</p>
            <input name="password" value={formik.values.password} onChange={formik.handleChange} type="password"  onBlur={formik.handleBlur} />
            <p>{ (formik.touched.password && formik.errors.password) || '' }</p>
            <button type="submit">提交</button>
          </form>
        </div>
      </header>
    </div>
  );
}

export default App;
```



# 4. 使用 getFieldProps 简化代码

getFieldProps 会返回一个对象，对象中包含 value 、onChange、onBlur 等属性，用来简化代码

```jsx
<form onSubmit={formik.handleSubmit}>
  <input
    name="name"
    type="text"
    {...formik.getFieldProps('name')}
    />
  <p>{ (formik.touched.name && formik.errors.name) || '' }</p>
  <input
    name="password"
    type="password"
    {...formik.getFieldProps('password')}
    />
  <p>{ (formik.touched.password && formik.errors.password) || '' }</p>
  <button type="submit">提交</button>
</form>
```



# 5. 使用 formik 组件来构建表单

## 1. Formik 组件

使用 formik 组件构建表单的根组件，所有 formik 组件需要被包裹在 Formik 组件内。

组件接收 initialValues、onSubmit 以及 validationSchema 等来定义表单的初始值、提交事件响应以及表单验证信息。

## 2. Form

包裹表单项的组件

## 3. Field

表单项，默认是 text 文本输入框，可以通过 as 来指定文本的类型。必须指定 name 属性，对应初始值对象中的属性。

## 4. ErrorMessage

显示错误提示信息，需要指定 name 属性，和 Field 组件的 name 属性对应。

```jsx
<Formik
  initialValues={{
    name: '',
    desc: '',
    subject: 'jj'
  }}
  onSubmit={vals => { console.log(vals) }}
  validationSchema={Yup.object({
    name: Yup.string().required('请输入用户名'),
    desc: Yup.string().required('请输入描述'),
  })}
  >
  <Form>
    <Field name="name" />
    <p>
      <ErrorMessage name="name" />
    </p>
    <Field as="textarea" name="desc" />
    <p>
      <ErrorMessage name="desc" />
    </p>
    <Field as="select" name="subject" >
      <option value="nc">南昌</option>
      <option value="jj">九江</option>
    </Field>
    <input type="submit" />
  </Form>
</Formik>
```



# 6. 自定义表单控件

自定义表单控件实际就是组件

通过 useField 来构建自定义表单，useField 接收包含表单信息的属性对象（name、type等），并返回一个field 和 meta 对象组成的数组。

field: 包含表单的基本属性，包括 name、onChange、onBlur等

![image-20210714162423178](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210714162423178.png)

meta: 包含表单的验证信息，包括touched(是否修改值)，error(错误信息)等

![image-20210714162440468](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210714162440468.png)

## 1. 简单表单控件自定义

简单的表单控件可以直接使用 useField 返回的信息来完成。+

```jsx
function MyInputField({ label, ...props }) {
  // field 存储表单的信息，如 value、onChange、onBlur
  // meta 存储表单的验证相关的信息， 如果 touched、error
  const [field, meta] = useField(props)

  return (
    <div>
      <label htmlFor={props.id}>{label}</label>
      <input {...field} {...props} />
      {meta.touched && meta.error ? <p>{ meta.error }</p> : ''}
    </div>
  )
}
```

```jsx
<MyInputField id="myInput" label="密码" type="password" name="password" placeholder="请输入密码" />
```



## 2. 复杂表单控件的定义

复杂表单控件的定义需要自定义表单的 onChange 事件，按照自己的逻辑来改变 value。

不能直接修改 value 的值，需要通过 useField 返回的 helper 对象的 setValue 方法来重新设置 value 的值。

```jsx
function Checkbox({ label, ...props }) {
  const [field, meta, helper] = useField(props)
  const { value } = field
  const { setValue } = helper
  const handleChange = () => {
    const set = new Set(value)
    if (set.has(props.value)) { // 已存在
      set.delete(props.value)
    } else { // 未存在
      set.add(props.value)
    }
    setValue([...set])
  }
  return (
    <div>
      <input checked={value.includes(props.value)} onChange={handleChange} {...props} type="checkbox" />{ label }
    </div>
  )
}
```

```jsx
<Formik
  initialValues={{
    name: '',
      desc: '',
        passowrd: '',
          hobbies: ['篮球', '足球'],
            subject: 'jj'
  }}
  onSubmit={vals => { console.log(vals) }}
  validationSchema={Yup.object({
    name: Yup.string().required('请输入用户名'),
    password: Yup.string().required('请输入密码'),
    desc: Yup.string().required('请输入描述'),
  })}
  >
  <Form>
    <Checkbox name="hobbies" label="篮球" value="篮球" />
    <Checkbox name="hobbies" label="足球" value="足球" />
    <Checkbox name="hobbies" label="排球" value="排球" />
  </Form>
</Formik> 
```

