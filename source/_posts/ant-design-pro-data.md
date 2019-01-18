---
layout: '[_posts]'
title: ant-design-pro-data
date: 2019-01-18 17:27:02
tags:
---

# 对于 Ant-design Pro 中数据流的理解

相对于 Vue 的 vuex 来讲， React 里面的 Redux 可能稍微有些难以理解，尤其是当接触过 vuex 之后，再去理解redux 就会有些转不过弯来。

但总的来讲，起始无论是 vuex 还是 redux 都是类似的。确切来讲，vuex 是借鉴了 redux 的部分思想，在此之上的一个改进。 所谓的数据流其实可以通俗的理解为一个大的全局变量，只不过对这个全局变量进行了修改和获取的一个约束。而所谓的 SPA应用，虽然在 ES6时代进行了模块化的分隔，但实质上也是在一个大的作用域里面运行，（Build 之后是一个文件，或者多个文件加载在一个页面。）运行时没有开发的那种文件上的间隔，而文件区分只不过是方便于开发。所有代码是可以写在一个文件的。

## Redux 的概念理解

首先，要使用 redux 必先了解其都有那些组成。这些网上有很多的文章，包括官方文档介绍的也很清楚。这里有几个概念比较重要。

#### 1. state --> 数据本身。  
可能是一个字符串、数组或者是布尔值。可能是一个按钮的value,或者开关的开闭状态（所谓的状态管理，state 就是状态，其余的工作就是管理这个状态）
#### 2. reducer --> 状态改变器
按道理来讲，state 如果作为一个全局变量，是可以被任何程序代码修改的。但是为了防止有些代码乱改这个 state,所以对 state 添加了一个修改约束，那就是只能通过 reducer 修改。 需要注意的是，这里的 reducer 只能是『直来直往』的修改，意思就是不能有异步修改。再简单一点就是 reducer 中不能有 Ajax 网络请求，定时器等异步操作。

#### 3. Action 

这里先放官方文档
>Action 是把数据从应用（译者注：这里之所以不叫 view 是因为这些数据有可能是服务器响应，用户输入或其它非 view 的数据 ）传到 store 的有效载荷。它是 store 数据的唯一来源。一般来说你会通过 store.dispatch() 将 action 传到 store。

简直不说人话。

实质上，一个程序对于数据的要求，无非就是 怎删改查，而这里的 Action 无非就是增加前期的复杂度，可以说毫无卵用


## Ant-design Pro 中 数据流的走向

Ant-design 对 Redux 进行了封装，解决了其中的一些比较繁琐的问题。当然其本身来讲还是没有 vuex 使用起来更加的优雅

这个分装包叫 dva.js

dva.js 里面又有几个概念,其中比较重要的有

##### 1.  Effect 

一群翻译称其为 副作用，什么副作用长副作用短的。也是不说人话。
也不知道起名字的人怎么想的，瞎起名字，毫无意义。

实质上，这个东西是专门处理异步操作的。就是什么 Ajax ，fileReader 之类的放到里面。当然，其比较好的一点是 使用了 ES6 的 Generator。 使得可以用同步写法去写异步操作。

#####  2. connect

这是一个函数，有有些人叫高阶函数。可能是受函数式编程的影响，这个函数刚开始写起来也后写怪异。

其作用是 将 dva 管理的数据 绑定到 视图页面。类似于一个桥梁。如果没有这个东东，就会有个问题，那就是 视图如何去操作 dva 管理的数据，而数据又如何去影响界面。它就是解决了这个问题。


**那 Ant-design Pro 数据流到底怎么走呢？**

这里通过 登录 流程来分析一下


首先从点登录的按钮说起

```
// pages/User/login.js

import React, { Component } from 'react';
import { connect } from 'dva';
import { formatMessage, FormattedMessage } from 'umi/locale';
import Link from 'umi/link';
import { Checkbox, Alert, Icon } from 'antd';
import Login from '@/components/Login';
import styles from './Login.less';

const { Tab, UserName, Password, Mobile, Captcha, Submit } = Login;


/**
 * 此处的 loading 是 dva-loading 插件的 state .
 * 而 login 是 models 中 login 的。因为 connect 会自动注入到函数里面
 */
@connect(({ login, loading }) => ({
  login,
  submitting: loading.effects['login/login'],
}))

class LoginPage extends Component {
  state = {
    type: 'account',
    autoLogin: true,
  };

  onTabChange = type => {
    this.setState({ type });
  };

  onGetCaptcha = () =>
    new Promise((resolve, reject) => {
      this.loginForm.validateFields(['mobile'], {}, (err, values) => {
        if (err) {
          reject(err);
        } else {
          const { dispatch } = this.props;
          dispatch({
            type: 'login/getCaptcha',
            payload: values.mobile,
          })
            .then(resolve)
            .catch(reject);
        }
      });
    });

  handleSubmit = (err, values) => {
    const { type } = this.state;
    if (!err) {
      const { dispatch } = this.props;
      dispatch({
        type: 'login/login',
        payload: {
          ...values,
          type,
        },
      });
    }
  };

  changeAutoLogin = e => {
    this.setState({
      autoLogin: e.target.checked,
    });
  };

  renderMessage = content => (
    <Alert style={{ marginBottom: 24 }} message={content} type="error" showIcon />
  );

  render() {
    const { login, submitting } = this.props; //通过 connect 已经将 state 绑定到 this.props
    const { type, autoLogin } = this.state;
         
    return (
      <div className={styles.main}>
        <Login
          defaultActiveKey={type}
          onTabChange={this.onTabChange}
          onSubmit={this.handleSubmit}
          ref={form => {
            this.loginForm = form;
          }}
        >
          <Tab key="account" tab={formatMessage({ id: 'app.login.tab-login-credentials' })}>
            {login.status === 'error' &&
              login.type === 'account' &&
              !submitting &&
              this.renderMessage(formatMessage({ id: 'app.login.message-invalid-credentials' }))}
            <UserName
              name="userName"
              placeholder={`${formatMessage({ id: 'app.login.userName' })}: admin or user`}
              rules={[
                {
                  required: true,
                  message: formatMessage({ id: 'validation.userName.required' }),
                },
              ]}
            />
            <Password
              name="password"
              placeholder={`${formatMessage({ id: 'app.login.password' })}: ant.design`}
              rules={[
                {
                  required: true,
                  message: formatMessage({ id: 'validation.password.required' }),
                },
              ]}
              onPressEnter={() => this.loginForm.validateFields(this.handleSubmit)}
            />
          </Tab>
          <Tab key="mobile" tab={formatMessage({ id: 'app.login.tab-login-mobile' })}>
            {login.status === 'error' &&
              login.type === 'mobile' &&
              !submitting &&
              this.renderMessage(
                formatMessage({ id: 'app.login.message-invalid-verification-code' })
              )}
            <Mobile
              name="mobile"
              placeholder={formatMessage({ id: 'form.phone-number.placeholder' })}
              rules={[
                {
                  required: true,
                  message: formatMessage({ id: 'validation.phone-number.required' }),
                },
                {
                  pattern: /^1\d{10}$/,
                  message: formatMessage({ id: 'validation.phone-number.wrong-format' }),
                },
              ]}
            />
            <Captcha
              name="captcha"
              placeholder={formatMessage({ id: 'form.verification-code.placeholder' })}
              countDown={120}
              onGetCaptcha={this.onGetCaptcha}
              getCaptchaButtonText={formatMessage({ id: 'form.get-captcha' })}
              getCaptchaSecondText={formatMessage({ id: 'form.captcha.second' })}
              rules={[
                {
                  required: true,
                  message: formatMessage({ id: 'validation.verification-code.required' }),
                },
              ]}
            />
          </Tab>
          <div>
            <Checkbox checked={autoLogin} onChange={this.changeAutoLogin}>
              <FormattedMessage id="app.login.remember-me" />
            </Checkbox>
            <a style={{ float: 'right' }} href="">
              <FormattedMessage id="app.login.forgot-password" />
            </a>
          </div>
          <Submit loading={submitting}>
            <FormattedMessage id="app.login.login" />
          </Submit>
          <div className={styles.other}>
            <FormattedMessage id="app.login.sign-in-with" />
            <Icon type="alipay-circle" className={styles.icon} theme="outlined" />
            <Icon type="taobao-circle" className={styles.icon} theme="outlined" />
            <Icon type="weibo-circle" className={styles.icon} theme="outlined" />
            <Link className={styles.register} to="/user/register">
              <FormattedMessage id="app.login.signup" />
            </Link>
          </div>
        </Login>
      </div>
    );
  }
}

export default LoginPage;

```

首先是 connect

```

//这里将 Model 中的 Login 以及 dva-loading 插件的 loading 绑定到当前页面
@connect(({ login, loading }) => ({
  login,
  submitting: loading.effects['login/login'],
}))

```
这里的 @connect 的写法也是个迷。实质上是

```
Export default connect( ({ login, loading }) => ({
  login,
  submitting: loading.effects['login/login'],
}) )(LoginPage)
```
的简写。

```
connect(A)(B) //将 A 和 B 绑定起来
```





