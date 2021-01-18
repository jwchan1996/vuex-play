## 核心概念

![image](https://s3.ax1x.com/2021/01/12/sGNkAx.png)

`Store` `State` `Getter` `Mutation` `Action` `Module`

### 基本结构

- 导入 Vuex
- 注册 Vuex
- 注入 $store 到 Vue 实例

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
    state: {},
    mutations: {},
    actions: {},
    modules: {}
})
```

```javascript
import store from './store'

new Vue({
    router,
    store,
    render: h => h(App)
}).$mount('#app')
```

### State

Vuex 使用单一状态树，用一个对象就包含了全部的应用层级状态。

使用 mapState 简化 State 在视图中的使用，mapState 返回计算属性。

```javascript
// store/index.js
export default new Vuex.Store ({
    state: {
        count: 0,
        msg: 'Hello Vuex'
    },
    mutations: {},
    actions: {},
    modules: {}
})
```

mapState 有两种使用的方式：

- 接收数组参数

```javascript
// 该方法是 vuex 提供的，所以使用前要先导入 
import { mapState } from 'vuex' 
// mapState 返回名称为 count 和 msg 的计算属性 
// 在模板中直接使用 count 和 msg 
computed: { 
    // count: state => state,count
    ...mapState(['count', 'msg'])
}
```

- 接收对象参数

如果当前视图中已经有了 count 和 msg，如果使用上述方式的话会有命名冲突，解决的方式：

```javascript
// 该方法是 vuex 提供的，所以使用前要先导入 
import { mapState } from 'vuex' 
// 通过传入对象，可以重命名返回的计算属性 
// 在模板中直接使用 num 和 message 
computed: { 
    // ...mapState({ num: 'count', message: 'msg' }) 也可以
    ...mapState({ 
        num: state => state.count, 
        message: state => state.msg 
    }) 
}
```

### Getter

Getter 就是 store 中的计算属性，使用 mapGetter 简化视图中的使用。

```diff
// store/index.js
export default new Vuex.Store ({
    state: {
        count: 0,
        msg: 'Hello Vuex'
    },
+   getters: {
+       reverseMsg(state) {
+           return state.msg.split('').reverse().join()
+       }
+   },
    mutations: {},
    actions: {},
    modules: {}
})
```

```javascript
import { mapGetter } from 'vuex' 
...
computed: { 
    ...mapGetter(['reverseMsg']), 
    // 改名，在模板中使用 reverse 
    ...mapGetter({ 
        reverse: 'reverseMsg' 
    }) 
}
```

### Mutation

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数。

使用 Mutation 改变状态的好处是，集中的一个位置对状态修改，不管在什么地方修改，都可以追踪到状态的修改。可以实现高级的 time-travel（时间旅行）调试功能，比如通过 vue-devtools 工具。

```diff
// store/index.js
export default new Vuex.Store ({
    state: {
        count: 0,
        msg: 'Hello Vuex'
    },
    getters: {
        reverseMsg(state) {
            return state.msg.split('').reverse().join()
        }
    },
+   mutations: {
+       increase(state, payload) {
+           state.count += payload
+       }
+   },
    actions: {},
    modules: {}
})
```

在 vue 组件中可以通过 `this.$store.commit('increase', 2)` 去调用 Mutation 对应的方法。

```javascript
import { mapMutations } from 'vuex' 
...
methods: { 
    ...mapMutations(['increase']), 
    // 传对象解决重名的问题 
    ...mapMutations({ 
        increaseMut: 'increase' 
    }) 
}
```

这种写法可以直接通过调用 `this.increase(2)` 触发 Mutation 。

### Action

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

注意：Action 的调用需要通过 dispatch ，就跟 Mutation 的调用需要通过 commit 一样。

```diff
// store/index.js
export default new Vuex.Store ({
    state: {
        count: 0,
        msg: 'Hello Vuex'
    },
    getters: {
        reverseMsg(state) {
            return state.msg.split('').reverse().join()
        }
    },
    mutations: {
        increase(state, payload) {
            state.count += payload
        }
    },
+   actions: {
+       increaseAsync(context, payload) {
+           setTimeout(() => {
+               // 异步操作的回调函数改变状态需要提交 Mutation 
+               context.commit('increase', payload)
+           })
+       }
+   },
    modules: {}
})
```

在 vue 组件中可以通过 `this.$store.dispatch('increaseAsync', 5)` 去调用 Action 对应的方法。

```javascript
import { mapActions } from 'vuex' 
...
methods: { 
    ...mapActions(['increaseAsync']), 
    // 传对象解决重名的问题 
    ...mapActions({ 
        increaseAsyncAction: 'increaseAsync' 
    }) 
}
```

这种写法可以直接通过调用 `this.increaseAsync(5)` 触发 Action ，通过 `...mapActions(['increaseAsync'])` 方法会给 vue 组件实例的 methods 属性增加 increaseAsync 方法，其中 increaseAsync 方法内部调用了 `this.$store.dispatch('increaseAsync', 5)` 。

### Module

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。

为了解决以上问题，Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块。

```diff
// store/index.js
export default new Vuex.Store ({
    state: {
        count: 0,
        msg: 'Hello Vuex'
    },
    getters: {
        reverseMsg(state) {
            return state.msg.split('').reverse().join()
        }
    },
    mutations: {
        increase(state, payload) {
            state.count += payload
        }
    },
    actions: {
        increaseAsync(context, payload) {
            setTimeout(() => {
                // 异步操作的回调函数改变状态需要提交 Mutation 
                context.commit('increase', payload)
            })
        }
    },
+   modules: {
+       products,
+       cart
+   }
})
```

```javascript
// store/modules/cart.js
const state = {}
const getters = {}
const mutations = {}
const actions = {}

export default {
  namespaced: true,
  state,
  getters,
  mutations,
  actions
}
```

```javascript
// store/modules/products.js
const state = {
  products: [
    { id: 1, title: 'iPhone 11', price: 8000 },
    { id: 2, title: 'iPhone 12', price: 10000 }
  ]
}
const getters = {}
const mutations = {
  setProducts (state, payload) {
    state.products = payload
  }
}
const actions = {}

export default {
  namespaced: true, // 指定命名空间
  state,
  getters,
  mutations,
  actions
}
```

modules 配置的模块会通过模块名挂载到全局 state 下，比如 products 模块的 state 状态的 products 属性可以通过 `this.$store.state.products.products` 来读取。

不使用 mapState、mapMutations 方法：

```html
<template>
    <div>
        products: {{ $store.state.products.products }} <br>
        <button @click="$store.commit('setProducts', [])">Mutation</button>
    </div>
</template>
```

使用 mapState、mapMutations 方法，且指定模块的命名空间：

```html
<template>
    <div>
        <!--这里因为 mapState 方法指定命名空间，所以 products 属性值会从命名空间的模块中找-->
        products: {{ products }} <br>
        <button @click="setProducts([])">Mutation</button>
    </div>
</template>
<script>
import { mapState, mapMutations } from 'vuex'
export default {
  computed: {
    ...mapState('products', ['products'])
  },
  methods: {
    ...mapMutations('products', ['setProducts'])
  }
}
</script>
```