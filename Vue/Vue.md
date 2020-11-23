# Vue结合axios

```javascript
axios.get("data.json").then(response=>(this.data = response.data))
```

# 计算属性

计算出来的结果保存在属性中，内存中运行：虚拟Dom。

```javascript
		// 计算属性 调用的时候不用加() 因为这个是一个属性，只是通过函数计算得到的，
        // 也可以写在methods里面，调用的时候需要加上() 是调用方法而不是计算属性
        // 计算属性中的函数必须要有返回值，在 浏览器中调用跟属性一样，方法的话需要加上()
        // 计算属性的值会缓存到内存中，随着浏览器刷新并不会刷新，只有当其中的数据刷新之后才会重新计算
        computed: {
            currenttime: function (){
                return Date.now();
            }
        },
```

# solt插槽

用于组件的嵌套。

```javascript
// 主要的组件，这里里面嵌套两个组件
    Vue.component("list",{
        template: "<div>\n" +
            "        <slot name='list-head'></slot>\n" +
            "        <ul>\n" +
            "            <slot name='list-body'></slot>\n" +
            "        </ul>\n" +
            "    </div>"
    });
    Vue.component("list-head",{
        template: "<div>{{title}}</div>",
        props:["title"]
    });
    Vue.component("list-body",{
        template: "<li>{{item.url}}</li>",
        props: ["item"]
    })
    var vue = new Vue({
        el:"#app",
        data:{
            data:{}
        },
        methods: {
        },
        mounted(){
            axios.get("data.json").then(response=>(this.data = response.data))
        },
    });
```

# element-ui

![步骤](http://oos.sfzzz.xyz/2020_08_02_20_17_53_1.png)