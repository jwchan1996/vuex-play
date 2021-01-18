## 模拟 Vuex

### 基本结构

回顾基础示例，自己模拟实现一个 Vuex 实现同样的功能。

```javascript
import Vue from 'vue' 
import Vuex from 'vuex' 

Vue.use(Vuex) 

export default new Vuex.Store({ 
    state: { 
        count: 0, 
        msg: 'Hello World' 
    },
    getters: { 
        reverseMsg (state) { 
            return state.msg.split('').reverse().join('') 
        } 
    },
    mutations: { 
        increate (state, payload) { 
            state.count += payload.num 
        } 
    },
    actions: { 
        increate (context, payload) { 
            setTimeout(() => { 
                context.commit('increate', { num: 5 }) 
            }, 2000) 
        } 
    }
})
```

### 实现思路

- **实现 install 方法**
    - Vuex 是 Vue 的一个插件，所以和模拟 VueRouter 类似，先实现 Vue 插件约定的 install 方法
- **实现 Store 类**
    - 实现构造函数，接收 options
    - state 的响应化处理
    - getter 的实现
    - commit、dispatch 方法

### install 方法

```javascript
let _Vue = null
function install(Vue) {
    _vue = Vue
    _Vue.mixin({
        beforeCreate() {
            if (this.$options.store) {
                Vue.prototype.$store = this.$options.store
            }
        }
    })
}
```

### Store 类

```javascript
class Store {
    constructor(options) {
        const {
            state = {},
            getters = {},
            mutations = {},
            actions = {}
        } = options
        
        this.state = _Vue.observable(state)
        
        // 当访问 getters 的 key 的时候，实际上就是访问 this.getters 的 key 会触发 key 属性 的 getter
        this.getters = Object.create(null)
        
        // 为了给 getters 的方法传递 state 参数，采用 defineProperty 的方式返回带 state 参数 getters 方法的调用
        Object.keys(getters).forEach(key => {                           
            Object.defineProperty(this.getters, key, { 
                get: () => getters[key](this.state)
            }) 
        })
        
        this._mutations = mutations
        this._actions = actions
    }
    
    commit(type, payload) {
        this._mutations[type](this.state, payload)
    }
    
    dispatch(type, payload) {
        this._actions[type](this, payload)
    }
}
```

### 使用自己实现的 Vuex

src/store/index.js 中修改导入 Vuex 的路径，测试：

```javascript
import Vuex from '../myvuex' 
// 注册插件 
Vue.use(Vuex)
```