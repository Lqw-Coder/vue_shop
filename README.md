## vue_shop使用流程

1. 项目构建 

   ```shell
   npm install
   ```

2. 编译以及热部署

   ```shell
   npm run serve
   ```

3. 项目打包

   ```shell
   npm run build
   ```

4. 修复文件格式

   ```shell
   npm run lint
   ```

## vue项目优化

### 1.安装NProgress实现进度条效果

+ 使用vue ui进入到ui界面安装运行依赖NProgress
+ 在入口文件main.js中配置

```js
//导入NProgress包对应的JS和CSS,实现进度条效果
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'
//在request拦截器中展示进度条NProgress.start()
axios.interceptors.request.use(config => {
  NProgress.start()
  config.headers.Authorization = window.sessionStorage.getItem('token')
  //将请求放行
  return config
})
//在response拦截器中隐藏进度条NProgress.done()
axios.interceptors.response.use(config => {
  NProgress.done()
  return config
})
```

### 2.安装babel-plugin-transform-remove-console开发依赖清除所有的console语句

+ 在vue ui界面安装开发依赖
+ 在babel.config.js中设置配置，使其在生产阶段清除所有的console

```js
// 这是项目发布阶段需要用到的 babel 插件
const prodPlugins = []
if (process.env.NODE_ENV === 'production') {
  prodPlugins.push('transform-remove-console')
}
module.exports = {
  presets: [
    '@vue/cli-plugin-babel/preset'
  ],
  plugins: [
    [
      'component',
      {
        libraryName: 'element-ui',
        styleLibraryName: 'theme-chalk'
      }
    ],    
    // 发布产品时候的插件数组
    ...prodPlugins
  ]
}
```

### 3.使用vue.config.js设置webpack打包配置

通过 vue-cli 3.0 工具生成的项目，默认隐藏了所有 webpack 的配置项，如果有修改 webpack 默认配置的需求，可以在项目根目录中，按需创建 vue.config.js 这个配置文件

+ 在src同级下创建vue.config.js
+ 通过chainWebpack或者configureWebpack配置打包信息

```js
module.exports = {
    chainWebpack: config => {
        // 发布模式
        config.when(process.env.NODE_ENV === 'production', config => {
            config
                .entry('app')
                .clear()
                .add('./src/main-prod.js')
        })

        // 开发模式
        config.when(process.env.NODE_ENV === 'development', config => {
            config
                .entry('app')
                .clear()
                .add('./src/main-dev.js')

        })
    }
}

```

### 4.使用CDN加载模式优化打包体积

默认情况下，通过 import 语法导入的第三方依赖包，最终会被打包合并到同一个文件中，从而导致打包成功 

后，单文件体积过大的问题，为了解决上述问题，可以通过 webpack 的 externals 节点，来配置并加载外部的 CDN 资源。凡是声明在externals 中的第三方依赖包，都不会被打包

+ 通过externals加载外部资源(vue.config.js)

```js
module.exports = {
    chainWebpack: config => {
        // 发布模式
        config.when(process.env.NODE_ENV === 'production', config => {
            config
                .entry('app')
                .clear()
                .add('./src/main-prod.js')
                //配置CDN资源加载
            config.set('externals', {
                vue: 'Vue',
                'vue-router': 'VueRouter',
                axios: 'axios',
                lodash: '_',
                echarts: 'echarts',
                nprogress: 'NProgress',
                'vue-quill-editor': 'VueQuillEditor'
            })
        })

        // 开发模式
        config.when(process.env.NODE_ENV === 'development', config => {
            config
                .entry('app')
                .clear()
                .add('./src/main-dev.js')

        })
    }
}

```

+ 在 public/index.html 文件的头部，添加如下的 CDN 资源引用(在main.js中删除css文件导入)

```html
<!-- nprogress 的样式表文件 --> <link rel="stylesheet" href="https://cdn.staticfile.org/nprogress/0.2.0/nprogress.min.css" />
<!-- 富文本编辑器 的样式表文件 -->
<link rel="stylesheet" href="https://cdn.staticfile.org/quill/1.3.4/quill.core.min.css" />
<link rel="stylesheet" href="https://cdn.staticfile.org/quill/1.3.4/quill.snow.min.css" />
<link rel="stylesheet" href="https://cdn.staticfile.org/quill/1.3.4/quill.bubble.min.css" />
<script src="https://cdn.staticfile.org/vue/2.5.22/vue.min.js"></script>
<script src="https://cdn.staticfile.org/vue-router/3.0.1/vue-router.min.js"></script>
<script src="https://cdn.staticfile.org/axios/0.18.0/axios.min.js"></script>
<script src="https://cdn.staticfile.org/lodash.js/4.17.11/lodash.min.js"></script>
<script src="https://cdn.staticfile.org/echarts/4.1.0/echarts.min.js"></script>
<script src="https://cdn.staticfile.org/nprogress/0.2.0/nprogress.min.js"></script>
<!-- 富文本编辑器的 js 文件 -->
<script src="https://cdn.staticfile.org/quill/1.3.4/quill.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue-quill-editor@3.0.4/dist/vue-quill-editor.js"></script>
```

```html
<!--使用cdn加载element-ui时可在main.js中删除import element-ui.js-->
<!-- element-ui 的样式表文件 --> <link rel="stylesheet" href="https://cdn.staticfile.org/element-ui/2.8.2/themechalk/index.css" />
<!-- element-ui 的 js 文件 -->
<script src="https://cdn.staticfile.org/element-ui/2.8.2/index.js"></script>
```

### 5.使用路由懒加载的方式异步加载组件

当打包构建项目时，JavaScript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成 

不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。

+ 安装 @babel/plugin-syntax-dynamic-import 包（开发依赖）

+ 在 babel.config.js 配置文件中声明该插件

  ```js
  // 这是项目发布阶段需要用到的 babel 插件
  const prodPlugins = []
  if (process.env.NODE_ENV === 'production') {
    prodPlugins.push('transform-remove-console')
  }
  module.exports = {
    presets: [
      '@vue/cli-plugin-babel/preset'
    ],
    plugins: [
      [
        'component',
        {
          libraryName: 'element-ui',
          styleLibraryName: 'theme-chalk'
        }
      ],    // 发布产品时候的插件数组
      ...prodPlugins,
        //该插件
      ['@babel/plugin-syntax-dynamic-import']
    ]
  }
  
  ```

  

+ 将路由改为按需加载的形式，示例代码如下： 

```js
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-boo" */ './Baz.vue')
```

## vue项目上线

### 1.通过node创建web服务器

```shell
npm init -y
npm i express -S
```

创建入口文件app.js

```js
const express = require('express')
// 创建 web 服务器
const app = express()
// 托管静态资源
app.use(express.static('./dist'))
// 启动 web 服务器
app.listen(80, () => {
console.log('web server running at http://127.0.0.1')
})
```

### 2.开启gzip配置，使用 gzip 可以减小文件体积，使传输速度更快。

```shell
npm install compression -S
```

```js
const express = require('express')
//导入包
const compression = require('compression')
const app = express()

// 一定要把这一行代码，写到 静态资源托管之前
app.use(compression())
app.use(express.static('./dist'))

app.listen(8080,()=>{
    console.log('serve running')
})
```

### 3.配置Https服务

1. 下载SSL证书

2. 在后台项目中导入

   ```js
   const https = require('https');
   const fs = require('fs');
   const options = {
   cert: fs.readFileSync('./full_chain.pem'),
   key: fs.readFileSync('./private.key')
   }
   https.createServer(options, app).listen(443);//https默认时443端口
   ```

### 4.使用pm2来后台管理项目

1. 在服务器中安装 pm2：npm i pm2 -g
2. 启动项目：pm2 start 脚本 --name 自定义名称
3. 查看运行项目：pm2 ls
4. 重启项目：pm2 restart 自定义名称
5. 停止项目：pm2 stop 自定义名称
6. 删除项目：pm2 delete 自定义名称

## 注意事项

### 1.使用cdn时最好保持开发时的版本与引入的cdn版本一致

[CDN库](http://www.staticfile.org/)

### 2.vue ui前端项目工程化流程以及注意事项

1. 使用vue ui界面安装功能时，需安装Router，babel，Linter/Formatter，且使用配置文件等功能
2. 安装vue-cli-plugin-element插件时注意使用按需导入
3. 在运行依赖中安装axios

4. 该项目后期运行依赖 echarts(图表)，lodash(拷贝)，nprogress(进度条)，vue-table-with-tree-grid(树形组件，可参考Roles组件)，vue-quill-editor(富文本编辑器)