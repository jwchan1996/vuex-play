## 组件间通信方式

![image](https://s3.ax1x.com/2021/01/11/sG3TC4.png)

### 父组件给子组件传值

- 子组件中通过 props 接收数据
- 父组件中给子组件通过相应属性传值

[父传子：Props Down](https://cn.vuejs.org/v2/guide/components.html#%E9%80%9A%E8%BF%87-Prop-%E5%90%91%E5%AD%90%E7%BB%84%E4%BB%B6%E4%BC%A0%E9%80%92%E6%95%B0%E6%8D%AE)

```html
<blog-post title="My journey with Vue"></blog-post>
```

```javascript
Vue.component('blog-post', { 
    props: ['title'], 
    template: '<h3>{{ title }}</h3>'
})
```

### 子组件给父组件传值

[子传父：Event Up](https://cn.vuejs.org/v2/guide/components.html#%E7%9B%91%E5%90%AC%E5%AD%90%E7%BB%84%E4%BB%B6%E4%BA%8B%E4%BB%B6)

在子组件中使用 $emit 发布一个自定义事件：

```html
<button @click="$emit('enlargeText', 0.1)"> Enlarge text </button>
```

在 vue 对象方法中是调用 `this.$emit('enlargeText', 0.1)`。

在使用这个组件的时候，监听这个自定义事件。

```html
<blog-post @enlargeText="hFontSize += $event"></blog-post>
```

### 非父子组件传值

[非父子组件：Event Bus](https://cn.vuejs.org/v2/guide/migration.html#dispatch-%E5%92%8C-broadcast-%E6%9B%BF%E6%8D%A2)

我们可以使用一个非常简单的 Event Bus 来解决这个问题：

`eventbus.js :`

```javascript
export default new Vue()
```

然后在需要通信的两端：

使用 `$on` 订阅：

```javascript
// 没有参数 
bus.$on('自定义事件名称', () => { 
    // 执行操作 
})

// 有参数 
bus.$on('自定义事件名称', data => { 
    // 执行操作 
})
```

使用 `$emit` 发布：

```javascript
// 没有自定义传参 
bus.$emit('自定义事件名称'); 

// 有自定义传参 
bus.$emit('自定义事件名称', '数据');
```

### 父直接访问子组件：通过 ref 获取子组件

其他常见方式：

`$root` `$parent` `$children` `$refs`

下面演示一下 $refs 的获取子组件（不到万不得已不要使用，会导致数据紊乱）。

ref 两个作用：

- 在普通 HTML 标签上使用 ref ，获取到的是 DOM
- 在组件标签上使用 ref ，获取到的是组件实例

创建 base-input 组件

```html
<template> 
    <input ref="input"> 
</template> 

<script> 
export default { 
    methods: { 
        // 用来从父级组件聚焦输入框 
        focus: function () { 
            this.$refs.input.focus() 
        } 
    } 
} 
</script>
```

在使用子组件的时候，添加 ref 属性：

```html
<base-input ref="usernameInput"></base-input>
```

然后在父组件等渲染完毕后使用 `$refs` 访问：

```javascript
mounted () { 
    this.$refs.usernameInput.focus() 
}
```

> `$refs` 只会在组件渲染完成之后生效，并且它们不是响应式的。这仅作为一个用于直接操作子组
件的“逃生舱”—— 你应该避免在模板或计算属性中访问 `$refs` 。

## [简易的状态管理方案](https://cn.vuejs.org/v2/guide/state-management.html)

如果多个组件之间要共享状态(数据)，使用上面的方式虽然可以实现，但是比较麻烦，而且多个组件之间互相传值很难跟踪数据的变化，如果出现问题很难定位问题。

当遇到多个组件需要共享状态的时候，典型的场景：购物车。我们如果使用上述的方案都不合适，我们会遇到以下的问题。

- 多个视图依赖于同一状态。
- 来自不同视图的行为需要变更同一状态。

对于问题一，传参的方法对于多层嵌套的组件将会非常繁琐，并且对于兄弟组件间的状态传递无能为力。

对于问题二，我们经常会采用父子组件直接引用或者通过事件来变更和同步状态的多份拷贝。以上的这些模式非常脆弱，通常会导致无法维护的代码。

因此，我们为什么不把组件的共享状态抽取出来，以一个全局单例模式管理呢？在这种模式下，我们的组件树构成了一个巨大的“视图”，不管在树的哪个位置，任何组件都能获取状态或者触发行为！

我们可以把多个组件的状态，或者整个程序的状态放到一个集中的位置存储，并且可以检测到数据的更改。你可能已经想到了 Vuex 。

这里我们先以一种简单的方式来实现。

- 首先创建一个共享的仓库 store 对象

```javascript
export default { 
    debug: true, 
    state: { 
        user: { 
            name: 'xiaomao', 
            age: 18, 
            sex: '男' 
        } 
    },
    setUserNameAction (name) { 
        if (this.debug) { 
            console.log('setUserNameAction triggered：', name) 
        }
        this.state.user.name = name 
    } 
}
```

- 把共享的仓库 store 对象，存储到需要共享状态的组件的 data 中

```javascript
import store from './store' 
export default { 
    methods: { 
        // 点击按钮的时候通过 action 修改状态 
        change () { 
            store.setUserNameAction('componentB') 
        } 
    },
    data () { 
        return { 
            privateState: {}, 
            sharedState: store.state
        } 
    }
}
```

接着我们继续延伸约定，组件不允许直接变更属于 store 对象的 state，而应执行 action 来分发 (dispatch) 事件通知 store 去改变，这样最终的样子跟 Vuex 的结构就类似了。这样约定的好处是，我
们能够记录所有 store 中发生的 state 变更，同时实现能做到记录变更、保存状态快照、历史回滚/时光旅行的先进的调试工具。