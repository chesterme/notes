## webpack基础配置

1. 安装npm
2. 创建一个目录，使用npm初始化

```javascript
npm init
```

会在该目录下生成一个`package.json`配置文件，内容大概如下：

```
{
  "name": "example",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}

```

3. 在本地局部安装webpack

```
// --save-dev 会作为开发依赖来安装webpack
npm install webpack --save-dev
```

安装完成后，会在`package.json`文件中添加：

```
"devDependencies": {
    "webpack": "^4.39.1"
  }
```

4. 安装`webpack-dev-server`，它可以在开发环境中提供很多服务，比如启动一个服务器、热更新、接口代理等

```
npm install webpack-dev-server --save-dev
```

5. 在当前目录下，创建一个`webpack.config.js`的配置文件，用来初始化webpack的内容：

> 配置入口，告诉webpack从哪里开始寻找依赖，并且编译
>
> 配置出口，告诉webpack编译后的文件如何命名和如何存储
>
> 配置加载器，加载不同的模块（在webpack里，每一文件都是一个模块，不同的模块，需要使用不同的加载器来处理）
>
> 插件

```javascript
var path = require("path");


var config = {
    entry:{
        main: "./main"
    },
    output:{
        path: path.join(__dirname, "./dist"),
        publicPath: "/dist/",
        filename: "main.js"
    }
};

module.exports = config;
```

5. 在`package.json`配置文件中添加快速启动服务：

```javascript
"scripts": {
        "test": "echo\"Error: no test specified\" && exit 1",
        "dev": "webpack-dev-server --open --config webpack.config.js"
    },
```

当运行`npm run dev`命令时，就会执行`webpack-dev-server --open --config webpack.config.js`命令。其中`--config`是指向`webpack-dev-server`读取的配置文件路径，`--open`会在执行命令时自动在浏览器打开页面，默认地址是`127.0.0.1:8080`，不过IP和端口号可以自行配置。

```javascript
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server --host 127.0.0.1 --port 8080 --open --config webpack.config.js"
  },
```

6. 在目录下新建一个`index.html`文件作为SPA的入口：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div id="app">
        Hello World.
    </div>
    <script type="text/javascript" src="/dist/main.js"></script>
</body>
</html>
```

7. 启动spa

```
npm run dev
```

注意：

```
The CLI moved into a separate package: webpack-cli
Please install 'webpack-cli' in addition to webpack itself to use the CLI
-> When using npm: npm i -D webpack-cli
-> When using yarn: yarn add -D webpack-cli
internal/modules/cjs/loader.js:638
    throw err;
```

提示需要安装`webpack-cli`，

```
npm install webpack-cli --save-dev
```

此时就可以启动spa程序了

## webpack进阶配置

在webpack中，每个文件就是一个模块，比如.css，.js，.html，.jpg等，对于不同的模块需要使用不同的加载器来处理。

### 处理css文件

1. 安装css加载器

```javascript
// 加载css样式
npm install css-loader --save-dev
npm install style-loader --save-dev
```

2. 安装完成后，需要在`webpack.config.js`文件中配置loader，增加对`.css`文件的处理

```javascript
var config = {
   // ...
    module: {
        rules: [
            {
                test: /.css$/,
                use:[
                    'style-loader',
                    'css-loader'
                ]
            }
        ]
    }
};
```

3. 在当前目录下，新建一个`style.css`文件

```css
#app {
    font-size: 24px;
    color: #f50;
}
```

4. 在`main.js`中导入：

```javascript
import './style.css';
document.getElementById("app").innerHTML = "Hello webpack";
```

此时，`css`是通过JavaScript动态创建style标签来写入，这意味着样式代码已经编译在`main.js`文件里。

### 安装插件

如果所有的样式代码都编译在`main.js`文件中，导致该文件变大，还不能做缓存。

1. 在目录下定义一个css文件：

```css
#app {
    font-size: 24px;
    color: #f50;
}
```

2. 安装`extract-text-webpack-plugin`插件

```javascript
// 将散落在各地的css提取出来，并生成一个main.css文件，最终在index.html里通过link元素加载它
npm install extract-text-webpack-plugin -- save-dev
```

3. 配置`webpack.config.js`文件，导入插件，改写loader的配置：

```javascript
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var config = {
    entry:{
        main: "./main"
    },
    output:{
        path: path.join(__dirname, "./dist"),
        publicPath: "/dist/",
        filename: "main.js"
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: ExtractTextPlugin.extract({
                    use: 'css-loader',
                    fallback: 'style-loader'
                })
            }
        ]
    },
    plugins:[
        // 重命名提取后的css文件
        new ExtractTextPlugin("main.css")
    ]
};
```

4. 注意：

```
// 在使用extract-text-wenpack-plugin时，可能会因为版本的原因出现一些问题：
Error: Chunk.entrypoints: Use Chunks.groupsIterable and filter by instanceof Entrypoint instead
// 解决办法是：
npm install --save-dev extract-text-webpack-plugin@next
```

5. 改正：

```
npm install --save-dev extract-text-webpack-plugin@next
```

6. 在SPA程序入口`main.js`中引入css文件

```javascript
import './style.css';
```

![1565080941250](.\images\1565080941250.png)

### 处理vue文件

可以在vue文件中填写template，不需要再字符串模板template选项中拼写DOM。

1. 安装vue文件加载器

安装`vue-loader`处理vue文件，一个vue文件一般包含3部分：

+ `<template>`
+ `<script>`
+ `<style>`

`vue-loader`在编译.vue文件时，会对<template>, <script>,<style>分别处理，所以在vue-loader选项里多了一项options来进一步对不同语言进行配置。

```javascirpt
// 加载vue文件
npm install --save vue
npm install --save-dev vue-loader
npm install --save-dev vue-style-loader
npm install --save-dev vue-hot-reload-api
// 处理ES6语法
npm install --save-dev babel
npm install --save-dev babel-loader
npm install --save-dev babel-core
npm install --save-dev babel-plugin-transform-runtime
npm install --save-dev babel-preset-es2015
npm install --save-dev babel-runtime
```

注意：

```JavaScript
由于版本不同，在使用babel功能时可能会出现问题：
Error: Cannot find module '@babel/core'
解决办法：安装兼容的版本
```

2. 在该目录下，新建一个`.babelrc`文件，并写入babel的配置，webpack会依赖此配置文件来使用babel编译es6代码：

```javascirpt
{
    "presets": ["es2015"],
    "plugins": ["transform-runtime"],
    "comments": false
}
```

配置好`.babelrc`文件之后，就可以使用.vue文件了，每个.vue文件就代表一个组件，组件之间可以相互依赖。

3. 修改`webpack.config.js`配置文件：

```javascript
var path = require("path");
// 导入插件
var ExtractTextPlugin = require("extract-text-webpack-plugin");

var config = {
    entry:{
        main: "./main"
    },
    output:{
        path: path.join(__dirname, "./dist"),
        publicPath: "/dist/",
        filename: "main.js"
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: ExtractTextPlugin.extract({
                    use: 'css-loader',
                    fallback: 'style-loader'
                })
            },
            {
                test: /\.vue$/,
                loader: 'vue-loader',
                options:{
                    laoders:{
                        css: ExtractTextPlugin.extract({
                            use: 'css-loader',
                            fallback: 'vue-style-loader'
                        })
                    }
                }
            },
            {
                test: /.js$/,
                loader: 'babel-loader',
                exclude: /node_modules/
            }
        ]
    },
    plugins:[
        // 重命名提取后的css文件
        new ExtractTextPlugin("main.css")
    ]
};
```

4. 在目录下新建一个`app.vue`文件

```vue
<template>
    <div>{{name}}</div>
</template>

<script>
export default {
    data(){
        return{
            name: 'vue.js'
        }
    }
}
</script>
<style scoped>
    div{
        color: #f60;
        font-size: 24px;
    }
</style>
```

> 在<template>内可以按照html的写法来写，不用加“\\"换行，webpack最终会把它编译为Render函数的形式。

> 写在style中的样式，因为已经用插件`extract-text-webpack-plugin`配置过了，最终会统一提取并打包在`main.css`里，又因为加了`scoped`属性，这部分样式只会对当前组件app.vue有效。

5. 在`main.js`中配置vue文件

```
import './style.css';

// 导入vue框架
import Vue from 'vue';
// 导入app.vue组件
import App from './app.vue';

// 创建Vue根实例
new Vue({
    el: '#app',
    render: h => h(App)
});

document.getElementById("app").innerHTML = "Hello webpack";
```

6. 运行第一个vue程序

```javascript
npm run dev
```

7. 注意：

由于`babel-loader@8`需要`babel@7`，所以在运行时出现问题：

```
Error: Cannot find module '@babel/core'
 babel-loader@8 requires Babel 7.x (the package '@babel/core'). If you'd like to use Babel 6.x ('babel-core'), you should install 'babel-loader@7'.
```

![1565083160523](.\images\1565083160523.png)

查看项目中安装babel的版本：

![1565083382309](.\images\1565083382309.png)

改正：

```
npm install babel-loader@7 --save-dev
```

8. 一些问题：

![1565083820249](.\images\1565083820249.png)

![1565083881407](.\images\1565083881407.png)

![1565083907846](.\images\1565083907846.png)

原因是：

> Vue Loader v15现在需要配合一个webpack插件才能正确使用
>
> ```vue
> // webpack.config.js
> const VueLoaderPlugin = require('vue-loader/lib/plugin')
> 
> module.exports = {
>   // ...
>   plugins: [
>     new VueLoaderPlugin()
>   ]
> }
> ```

> 关于Vue Loader的详细内容，请查看：
>
> https://vue-loader.vuejs.org/zh/

9. 在目录下新建两个vue文件

```vue
// title.vue
<template>
    <h1>
        <a :href="'#' + title">{{title}}</a>
    </h1>
</template>

<script>
export default {
    props: {
        title: {
            type: String
        }
    }
}
</script>

<style scoped>
    h1 a{
        color: #3399ff;
        font-size: 24px;
    }
</style>
```

```vue
// button.vue
<template>
    <button @click="handleClick" :style="styles">
        <slot></slot>
    </button>
</template>

<script>
export default {
    props:{
        color:{
            type: String,
            default: '#00cc66'
        }
    },
    computed:{
        styles(){
            return {
                background: this.color
            }
        }
    },
    methods: {
        handleClick(e){
            this.$emit('click', e);
        }
    }
}
</script>

<style scoped>
    button{
        border: 0;
        outline: none;
        color: #fff;
        padding: 4px 8px;
    }
    button:active{
        position: relative;
        top: 1px;
        left: 1px;
    }
</style>
```

10. 修改根实例`app.vue`组件，把`title.vue`和`button.vue`导入进来：

```vue
// app.vue
<template>
    <div>
        <v-title title="Vue组件化"></v-title>
        <v-button @click="handleClick">点击按钮</v-button>
    </div>
</template>

<script>
// 导入组件
import vTitle form './title.vue';
import vButton form './button.vue';

export default{
    components:{
        vTitle,
        vButton
    },
    methods:{
        handleClick(e){
            console.log(e);
        }
    }
}
</script>
<style scoped>
    div{
        color: #f60;
        font-size: 24px;
    }
</style>
```

11. 存在的一些问题

![1565085545526](.\images\1565085545526.png)

原因：语法错误，写错单词 from写成form

### 处理图像、文字等文件

1. 安装`url-loader`和`file-loader`来支持图片、字体等文件：

```javascript
npm install --save-dev url-loader
npm install --save-dev file-loader
```

2. 配置`webpack.config.js`文件：

```
 // 配置加载器
        rules:[
            
            {
                test: /\.(gif|jpg|png|woff|svg|eot|ttf)\??.*$/,
                loader: 'url-loader?limit=1024'
            }
        ]
```

3. 测试，修改根vue文件`app.vue`

```vue
<template>
    <div>
        <v-title title="Vue组件化"></v-title>
        <v-button @click="handleClick">点击按钮</v-button>
        <p>
            <img src="./images/image.jpg" alt="" style="width: 200px; height: auto;">
        </p>
    </div>
</template>
```

### 打包

1. 打包需要使用以下两个依赖：

```
npm install --save-dev webpack-merge
npm install --save-dev html-webpack-plugin
```

2. 为了方便开发和生产环境的切换，在demo目录下再新建一个用于生产环境的配置文件`webpack.prod.config.js`

>  `webpack-merge`模块用于合并两个webpack的配置文件，所以prod的配置是在webpack.config.js基础上扩展的。
>
> 静态资源在大部分场景下都有缓存，更新上线后一般都希望用户能够及时地查看到内容，所以给打包后的css和js文件的名称都需要加上20为的hash值，这样文件名就为一类，只要不对html文件设缓存，上线后立即就可以加载最新的静态资源。

```javascript
var webpack = require('webpack');
var HtmlwebpackPlugin = require("html-webpack-plugin");
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var merge = require("webpack-merge");
var webpackBaseConfig = require("./webpack.config.js");
const TerserPlugin = require('terser-webpack-plugin');
const VueLoaderPlugin = require("vue-loader/lib/plugin");


// 清空基本配置的插件列表
webpackBaseConfig.plugins = {};

// module.exportes = merge(webpackBaseConfig, {
//     mode: 'production',
//     output: {
//         publicPath: './dist/',
//         // 将入口文件重命名为带有20位   hash值的唯一文件
//         filename: '[name].[hash].js'
//     },
//     plugins:[
//         new ExtractTextPlugin({
//             // 提取css，并重命名为带有20位hash值的唯一文件
//             filename: '[name].[hash].css',
//             allChunks: true
//         }),
//         // 定义当前node环境为生产环境
//         new webpack.DefinePlugin({
//             'process.env': {
//                 NODE_ENV: "production"
//             }
//         }),
//         // 压缩js
//         // new webpack.optimize.UglifyJsPlugin({
//         //     compress:{
//         //         warnings: false
//         //     }
//         // }),
//         // 提取模块，并保存入口html文件
//         new HtmlwebpackPlugin({
//             filename: '../index_prod.html',
//             template: './index.ejs',
//             inject: false
//         })
//     ],
//     optimization: {
//         minimizer: [new TerserPlugin()],
//     }
// });

module.exports = {
    mode: 'production',
    entry:{
        main: "./main"
    },
    output:{
        publicPath: './dist/',
        // 将入口文件重命名为带有20位   hash值的唯一文件
        filename: '[name].[hash].js'
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ExtractTextPlugin.extract({
                    use: 'css-loader',
                    fallback: 'style-loader'
                })
            },
            {
                test: /\.vue$/,
                loader: 'vue-loader',
                options:{
                    laoders:{
                        css: ExtractTextPlugin.extract({
                            use: 'css-loader',
                            fallback: 'vue-style-loader'
                        })
                    }
                }
            },
            {
                test: /\.js$/,
                loader: 'babel-loader',
                exclude: /node_modules/
            },
            {
                test:/.(gif|jpg|png|woff|svg|eot|ttf)\??.*/,
                loader:'url-loader?limit=1024'
            }
        ]
    },
    plugins:[
        // 重命名提取后的css文件
        new ExtractTextPlugin("main.css"),
        new VueLoaderPlugin(),
        new ExtractTextPlugin({
            // 提取css，并重命名为带有20位hash值的唯一文件
            filename: '[name].[hash].css',
            allChunks: true
        }),
        // 定义当前node环境为生产环境
        new webpack.DefinePlugin({
            'process.env': {
                NODE_ENV: "production"
            }
        }),
        // 压缩js
        // new webpack.optimize.UglifyJsPlugin({
        //     compress:{
        //         warnings: false
        //     }
        // }),
        // 提取模块，并保存入口html文件
        new HtmlwebpackPlugin({
            filename: '../index_prod.html',
            template: './index.ejs',
            inject: false
        })
    ],
    optimization: {
        minimizer: [new TerserPlugin()],
    }
}

```

3. 在`package.json`中，加入一个`build`的快捷脚本打包：

```javascript
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server --host 127.0.0.1 --port 8080 --open --config webpack.config.js",
    "build": "webpack --progress --hide-modules --config webpack.prod.config.js"
  }
```

4. 设置输入文件，`index_prod.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="<%= htmlwebpackPlugin.files.css[0] %">
</head>
<body>
    <div id="app"></div>
    <script type="text/javascript" src="<%= htmlwebpackPlugin.files.js[0] %"></script>
</body>
</html>
```



4. 编译打包

```
npm run build
```

5. 出现的一些问题：

![1565138845482](.\images\1565138845482.png)

解决：

导入一个`html-loader`

```
npm install html-loader --save-dev
```

在`webpack.prod.config.js`中进行配置：

```javascript
module: {
        rules: [
           // ...
            {
                test: /\.html$/,
                loader: 'html-loader',
            }
        ]
    },
```

