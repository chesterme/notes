### vue-router基本用法

1. 安装`vue-router`

```
npm install --save-dev vue-router
```

2. 在入口js文件中`main.js`使用`Vue.use()`加载插件

```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';
import App from './app.vue';

Vue.use(VueRouter);
```

3. 在views目录下创建两个文件`index.vue`和`about.vue`

```vue
// index.vue
<template>
    <div>首页</div>
</template>
<script>
export default {
    
}
</script>

```

```vue
// about.vue
<template>
    <div>介绍页</div>
</template>
<script>
export default {
    
}
</script>
```

```vue
// user.vue
<template>
    <div>
        {{$route.params.id}}
    </div>
</template>

<script>
export default {
    mounted: function(){
        console.log(this.$route.params.id)
    }
}
</script>
```

> ps:  `this.$route`可以访问当前路由很多信息

4. 在入口`main.js`文件里配置路由表

webpack会在每个路由都打包成一个js文件，在请求到该页面时，才去加载这个页面的js文件，即异步实现的懒加载（按需加载）。

```JavaScript
const Routers = [
    // webpack会把每一个路由都打包成一个js文件，在页面请求到达时才加载对应的文件
    {
    	// 指定当前匹配的路径
        path: '/index',
        // 映射组件
        component: (resolve) => require(['./views/index.vue'], resolve)
    },
    {
        path: '/about',
        component: (resolve) => require(['./views/about.vue'], resolve)
    }
];
```

ps: 重新命名块（chunk）

修改webpack配置文件，在项目输出文件中重新命名块（chunk）

使用异步路由后，编译出的每一个页面的js文件都叫做一个块（chunk）

```javascript
output:{
        publicPath: './dist/',
        // 将入口文件重命名为带有20位hash值的唯一文件
        filename: '[name].[hash].js',
        chunkFilename: '[name].chunk.js'
},
```

有了chunk后，在每一个页面（vue文件）中编写的css样式也需要配置后才能打包进`main.css`，否则仍然会通过js创建<style>的方式在html文件中插入css样式代码。

修改webpack配置文件：

```javascript
plugins:[
	new ExtractTextPlugin({
            // 提取css，并重命名为带有20位hash值的唯一文件
            filename: '[name].[hash].css',
            allChunks: true
        }),
]
```

5. 配置入口文件`main.js`：

```javascript
const RouterConfig = {
    // 使用html5的history模式设置前端路由
    model: 'history',
    // 设置路由规则
    routes: Routers
};

// 创建一个VueRouter实例
const router = new VueRouter(RouterConfig);

// 创建一个vue实例
new Vue({
    el: '#app',
    router: router,
    // 设置渲染函数
    render: h => {
        return h(App)
    }
});
```

6. 设置`webpack-dev-server`支持history路由，修改配置文件`package.json`：

```javascript
"scripts":{
	"dev": "webpack-dev-server --host 127.0.0.1 --port 8080 --open --history-api-fallback --config webpack.config.js",
}
```

增加了`--history-api-fallback`，所有的路由都会指向`index.html`

7. 在根实例`app.vue`中添加一个路由视图来挂载所有的路由组件：

```vue
<template>
    <div>
        <div>
            导航栏
    	</div>
        <router-view></router-view>
        <div>
        	底部版权信息    
    	</div>
    </div>
</template>

<script>
export default {
    
}
</script>
<style scoped>
    div{
        color: #f60;
        font-size: 24px;
    }
</style>

```

运行服务时，<router-view>会根据当前路由动态渲染页面组件。网页中的一些公共内容（比如顶部的导航栏、侧边导航栏、底部的版权等），这些内容可以写在<div>内，与<router-view>同级。当路由切换时，切换的是<router-view>挂载的组件，其他的内容不会发生变化。

8. 启动服务

```
npm run dev
```

![1565169899615](.\images\1565169899615.png)

![1565169950925](.\images\1565169950925.png)

### 跳转

1. 使用内置组件<router-link>，它会被渲染成一个<a>标签：

```vue
// index.vue
<template>
    <div>
        首页
        <router-link to="/about">跳转到 about</router-link>
    </div>
</template>
```

![1565170529609](.\images\1565170529609.png)

> ps: 使用<router-link>在html5的history模式下会拦截点击，避免浏览器重新加载页面

2. 使用js处理跳转页面，类似更改`window.location.href = newURL`

```vue
// about.vue
<template>
    <div>
        介绍页
        <button @click="handleClick">跳转到 index页面</button>
    </div>
</template>
<script>
export default {
    methods: {
        handleClick: function(){
            this.$router.push('/index');
        }
    }
}
</script>

```

### 更改标题内容

1. 为不同路由关联的页面设置标题内容：

这可以在创建`VueRouter`对象时设置

```javascript
const Routers = [
    {
        path: '/index',
        meta: {
            title: '首页'
        },
        component: (resolve) => require(['./views/index.vue'], resolve)
    },
    {
        path: '/about',
        meta: {
            title: '关于'
        },
        component: (resolve) => require(['./views/about.vue'], resolve)
    },
    {
        path: '/user/:id',
        meta: {
            title: '个人主页'
        },
        component: (resolve) => require(['./views/user.vue'], resolve)
    },
    {
        path: '*',
        redirect: '/index'
    }
];
const RouterConfig = {
    // 使用html5的history模式设置前端路由
    mode: 'history',
    routes: Routers
};

// 创建一个VueRouter实例
const router = new VueRouter(RouterConfig);
```

2. `vue-router`提供导航钩子`beforeEach`和`afterEach`，它们会在路由发生改变之前或之后触发：

```
// 在路由发生变化之前触发
// to: 即将进入的目标的路由对象
// from：当前导航即将离开的对象
// next：调用该方法后，才能进入下一个钩子
router.beforeEach((to, from, next) => {
    window.document.title = to.meta.title;
    next();
})
```

### Vuex基本用法

1. 安装`vues`:

```
npm install --save-dev vuex
```

2. 在入口的js文件中使用：

```javascript
// vuex配置
Vue.use(Vuex);
const store = new Vuex.Store({
    // 保存应用数据和操作过程
    state: {
        count: 0
    }
});

// 创建一个vue实例
new Vue({
    el: '#app',
    router: router,
    // 设置渲染函数
    render: h => {
        return h(App)
    },
    // 使用vuex
    store: store,
    data: {
        name: 'vue'
    }
});
```

> ps: `store`对象就像是一个在多个组件间共享的一个内存区域，而且它是响应式的，当该区域的内容发生改变时，会更改对应的视图。

3. 更改该共享区域的内容

改变`store`对象中的数据的合法途径是显示提交`mutations`(突变)

```javascript
const store = new Vuex.Store({
    // 保存应用数据和操作过程
    state: {
        count: 0
    },
    mutations: {
        increment (state) {
            state.count++;
        },
        decrease (state) {
            state.count--;
        }
    }
});
```

提交`mutations`的一种方式是：

```JavaScript
const store = new Vuex.Store({
	state:{
		// 共享内容
	},
	mutations: {
		// 处理共享的内容
	}
})
$store.commit('处理函数')
```
提交`mutations`的另一种方式是：

```javascript
mutations:{
	decrease (state, params){
		state.count -= params.count;
	}
}

$store.commit({
	type: 'decrease',
	count: 10
})
```

```vue
<template>
    <div>
        介绍页
        <button @click="handleClick">跳转到 index页面</button>
        <div>
            <button @click="handleIncrement">+1</button> count: {{$store.state.count}} <button @click="handleDecrease">-1</button>
        </div>
    </div>
</template>
<script>
export default {
    methods:{
        handleClick: function(){
            this.$router.push('/index');
        },
         // 提交`mutation`的一种方式
        handleIncrement () {
            this.$store.commit('increment');
        },
        handleDecrease () {
            this.$store.commit('decrease');
        }
    }
}
</script>

```

### Vuex高级用法

#### 使用`getters`

```javascript
Vue.use(Vuex);
const store = new Vuex.Store({
    // 保存应用数据和操作过程
    state: {
        count: 0,
        list: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    },
    mutations: {
        increment (state) {
            state.count++;
        },
        decrease (state) {
            state.count--;
        }
    },
    getters:{
        filteredList : state => {
            return state.list.filter(item => item < 5);
        },
        listCount : (state, getters) => {
            return getters.filteredList.length;
        }
    }
});
```

```vue
// index.vue
<template>
    <div>
        首页
        <router-link to="/about">跳转到 about</router-link>
        <div>
            count: {{count}} <button @click="handlePlusOne">+</button>
            list: {{$store.state.list}}
            list less than 5: {{listCount}}
        </div>
    </div>
</template>
<script>
export default {
    computed: {
        count: function(){
            return this.$store.state.count;
        },
        listCount () {
            return this.$store.getters.filteredList;
        }
    },
    methods: {
        handlePlusOne: function(){
            this.$store.state.count++;
        }
    }
}
</script>

```

#### 使用`action`

作用与`mutations`相似，但`mutations`是以同步的方式工作的，而`action`中提交的是`mutation`并且可以处理异步操作

```javascript
const store = new Vuex.Store({
    // 保存应用数据和操作过程
    state: {
        count: 0,
        list: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    },
    mutations: {
        increment (state) {
            state.count++;
        },
        decrease (state) {
            state.count--;
        }
    },
    getters:{
        filteredList : state => {
            return state.list.filter(item => item < 5);
        },
        listCount : (state, getters) => {
            return getters.filteredList.length;
        }
    },
    actions: {
        asyncDecrease (context){
            return new Promise(resolve => {
                setTimeout(() => {
                    context.commit('decrease');
                    resolve();
                }, 3000)
            });
        },
        asyncIncrement (context){
            return new Promise(resolve => {
                setTimeout(() => {
                    context.commit('increment');
                    resolve();
                }, 3000)
            });
        }
    }
});
```

```javascript
methods: {
        handlePlusOne: function(){
            this.$store.state.count++;
        },
        handleAsyncDecrease (){
            this.$store.dispatch('asyncDecrease').then(() => {
                console.log('decrement: ' + this.$store.state.count);
            })
        },
        handleAsyncIncrement (){
            this.$store.dispatch('asyncIncrement').then(() => {
                console.log('increment' + this.$store.state.count);
            })
        }
    }
```

