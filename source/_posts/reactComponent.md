---
title: React组件化
categories: react javascript
tags: react 组件化
---

## 组件化概念

将一个复杂的页面分割成若干个独立的组件，每个组件包含自己的逻辑和样式，再将这些独立组件组合成完成一个复杂的页面，这样既减少了逻辑复杂度，又实现了代码的重用， 特点

1. 可组合：一个组件可以和其它的组件一起使用，或者可以直接嵌套再另外一个组件内部

2. 可重用：每个组件都是具有独立功能的，可以被使用在多个场景中

3. 可维护：每个小的组件仅仅包含自身的逻辑，更加容易被理解和维护

## 组件设计原则

- 单一功能
- 扁平化的数据结构
- 更加纯粹的 State 变化
- 低耦合
- 集中/统一的状态管理

## 维度

- 业务逻辑
- 页面 UI 交互

## 容器组件和可视化组件

- 可视化组件简单来说是我们用来渲染到页面上可以被看见的组件，它只负责根据父组件传来的 props 渲染视图
- 容器组件，它总是作为可视化组件的父级组件出现，通常作用是给可视化组件准备数据，充当支架

比如上面的 Form 可以理解成一个容器组件，下面的其他元素就是可视化组件, 同时如果 Form 作为其他组件的一个单独功能， 也可以理解成可视化组件， 整个 Form 根据父级状态去渲染。

## react hooks

react hook 解决了以下几个问题，使组件化更规范

- 在组件之间复用状态逻辑很难
- 复杂组件变得难以理解（生命周期内有不相关的逻辑）
- 类组件中 this 指向问题不容易理解

## 示例 - 表格验证

需求：

1. 完成一个表格,表格内容包括用户名, 电话, 验证码, 账号类型，密码， 确认密码
2. 错误时右侧给错误提示
3. 提交表格时验证错误， 并将所有数据传递给后台

```js
// 原始版 - 有完整的功能
const Form = () => {
  const [formState, useFormState] = useState({
    phone: "",
    password: "",
    code: "",
    confirmPassword: "",
    account: "",
    accountType: "",
    accountTypes: ["personal", "enterprise"],
  });

  function validatePhone() {
    // 验证格式
    return true;
  }
  function validatePassword() {
    return true;
  }
  function validateConfirmPassword() {
    return true;
  }
  function validateCode() {
    return true;
  }
  function validateAccount() {
    return true;
  }

  const onConfirm = useCallback(() => {
    if (!validatePhone()) {
    }
    if (!validatePassword()) {
    }
    if (!validateConfirmPassword()) {
    }
    if (!validateAccount()) {
    }
    if (!validateCode()) {
    }
    // 提交数据
    // post(formState)
  }, [formState]);
  return (
    <div className="form">
      <div className="form-item">
        <label>电话： </label>
        <input
          placeholder={"请输入电话号码"}
          value={formState.phone}
          type="text"
        />
        <span>获取验证码</span>
      </div>
      <div className="form-item">
        <label>验证码： </label>
        <input value={formState.code} type="text" />
      </div>
      <div className="form-item">
        <label>账户类型：</label>
        <select value={"personal"}>
          {formState.accountTypes.map((type) => (
            <option value={type}></option>
          ))}
        </select>
      </div>
      <div className="form-item">
        <label>密码：</label>
        <input
          placeholder={"请输入密码"}
          value={formState.password}
          type="password"
        />
      </div>
      <div className="form-item">
        <label>重复密码：</label>
        <input
          placeholder={"请输入再次输入密码"}
          value={formState.confirmPassword}
          type="password"
        />
      </div>
      <div onClick={onConfirm}>确认</div>
    </div>
  );
};
```

```js
// 改良版 组件拆分
enum FormValidateType {
  phone = 'phone',
  password = 'password',
  account = 'account',
  code = 'code',
  confirmPassword = 'confirmPassword',
  accountType = 'accountType',
}

type FormState = Record<FormValidateType, string>

function validateForm(type: FormValidateType, state: FormState) {
  switch (type) {
    case FormValidateType.phone:
      return validatePhone(state)
    case FormValidateType.password:
      return validatePassword(state)
    case FormValidateType.confirmPassword:
      return validateConfirmPassword(state)
    case FormValidateType.code:
      return validateCode(state)
    case FormValidateType.account:
      return validateAccount(state)
    default:
      return false
  }
  function validatePhone(state: FormState) {
    // 验证格式
    return true
  }
  function validatePassword(state: FormState) {
    return true
  }
  function validateConfirmPassword(state: FormState) {
    return true
  }
  function validateCode(state: FormState) {
    return true
  }
  function validateAccount(state: FormState) {
    return true
  }
}

const Form = () => {
  const [formState, setFormState] = useState<Record<FormValidateType, string>>({
    phone: '',
    password: '',
    code: '',
    confirmPassword: '',
    account: '',
    accountType: '',
  })
  const accountTypes = ['personal', 'enterprise']
  const onStateChange = useCallback((type: FormValidateType, value: string) => {
    setFormState((state) => ({
      ...state,
      [type]: value,
    }))
  }, [])

  const onBlur = useCallback(
    (type: FormValidateType) => {
      validateForm(type, formState)
    },
    [formState],
  )

  const onConfirm = useCallback(() => {
    // validateForm
    // 提交数据
    // post(formState)
  }, [formState])
  return (
    <div className="form">
      <div className="form-item">
        <label>电话： </label>
        <input
          placeholder={'请输入电话号码'}
          value={formState.phone}
          // 传递箭头函数是会有性能问题的， 这里暂时不展开了
          onChange={(e) =>
            onStateChange(FormValidateType.phone, e.target.value)
          }
          onBlur={(e) => onBlur(FormValidateType.phone)}
          type="text"
        />
        <span>获取验证码</span>
      </div>
      <div className="form-item">
        <label>验证码： </label>
        <input
          onChange={(e) => onStateChange(FormValidateType.code, e.target.value)}
          onBlur={(e) => onBlur(FormValidateType.code)}
          value={formState.code}
          type="text"
        />
      </div>
      <div className="form-item">
        <label>账户类型：</label>
        <select
          onChange={(e) =>
            onStateChange(FormValidateType.accountType, e.target.value)
          }
          onBlur={(e) => onBlur(FormValidateType.accountType)}
          value={'personal'}
        >
          {accountTypes.map((type) => (
            <option value={type}></option>
          ))}
        </select>
      </div>
      <div className="form-item">
        <label>密码：</label>
        <input
          onChange={(e) =>
            onStateChange(FormValidateType.password, e.target.value)
          }
          onBlur={(e) => onBlur(FormValidateType.password)}
          placeholder={'请输入密码'}
          value={formState.password}
          type="password"
        />
      </div>
      <div className="form-item">
        <label>重复密码：</label>
        <input
          onChange={(e) =>
            onStateChange(FormValidateType.confirmPassword, e.target.value)
          }
          placeholder={'请输入再次输入密码'}
          value={formState.confirmPassword}
          type="password"
        />
      </div>
      <div onClick={onConfirm}>确认</div>
    </div>
  )
}
```

## z
