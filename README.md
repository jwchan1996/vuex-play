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