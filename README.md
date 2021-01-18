# Vuex 状态管理

## 组件内状态管理流程

`Vue` 最核心的两个功能：数据驱动和组件化。

每个组件都有自己的状态、视图和行为等组成部分。

```javascript
new Vue({ 
    // state 
    data () { 
        return { 
          count: 0 
        } 
    },
    // view 
    template: ` <div>{{ count }}</div> `,
    // actions 
    methods: { 
        increment () { 
            this.count++ 
        } 
    } 
})
```

![image](https://s3.ax1x.com/2021/01/05/sA02Hs.png)

## 组件间通信方式

![image](https://s3.ax1x.com/2021/01/11/sG3TC4.png)

## Vuex 回顾

### [什么是 Vuex](https://vuex.vuejs.org/zh/)

> [Vuex](https://vuex.vuejs.org/zh/) 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件
的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 也集成到 Vue 的官方调试工具 [devtools extension](https://github.com/vuejs/vue-devtools)，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调
试功能。

- Vuex 是专门为 Vue.js 设计的状态管理库
- 它采用集中式的方式存储需要共享的数据
- 从使用角度，它就是一个 JavaScript 库
- 它的作用是进行状态管理，解决复杂组件通信，数据共享

### 什么情况下使用 Vuex

> 官方文档：  
Vuex 可以帮助我们管理共享状态，并附带了更多的概念和框架。这需要对短期和长期效益进行权衡。如果您不打算开发大型单页应用，使用 Vuex 可能是繁琐冗余的。确实是如此——如果您的应用
够简单，您最好不要使用 Vuex。一个简单的 [store 模式](https://cn.vuejs.org/v2/guide/state-management.html#%E7%AE%80%E5%8D%95%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86%E8%B5%B7%E6%AD%A5%E4%BD%BF%E7%94%A8)就足够您所需了。但是，如果您需要构建
一个中大型单页应用，您很可能会考虑如何更好地在组件外部管理状态，Vuex 将会成为自然而然的选择。引用 Redux 的作者 Dan Abramov 的话说就是：Flux 架构就像眼镜：您自会知道什么时候需要它。

当你的应用中具有以下需求场景的时候：

- 多个视图依赖于同一状态
- 来自不同视图的行为需要变更同一状态

建议符合这种场景的业务使用 Vuex 来进行数据管理，例如非常典型的场景：购物车。

**注意：Vuex 不要滥用，不符合以上需求的业务不要使用，反而会让你的应用变得更麻烦。**