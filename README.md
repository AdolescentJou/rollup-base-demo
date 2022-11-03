
## 关于本项目

本项目包含了rollup的基础打包配置


### 基础配置

```js
module.exports = {
  input: './src/main.js', //入口文件
  output: {
    file: './dist/bundle.js', //打包后的存放文件
    format: 'cjs', //输出格式 amd es6 iife umd cjs
    name: 'bundleName', //如果iife,umd需要指定一个全局变量
  },
};
```

- input
入口文件地址
- output
```
output:{
    file:'bundle.js', // 输出文件
    format: 'cjs,  //  五种输出格式：amd /  es6 / iife / umd / cjs
    name:'A',  //当format为iife和umd时必须提供，将作为全局变量挂在window(浏览器环境)下：window.A=...
    sourcemap:true  //生成bundle.map.js文件，方便调试
}
```
- plugins
各种插件使用的配置
- external
```
external:['lodash'] //告诉rollup不要将此lodash打包，而作为外部依赖
```
- global
```
global:{
    'jquery':'$' //告诉rollup 全局变量$即是jquery,类似于webpack的别名alias
}
```

### 引用node包

####  node-resolve 

在 Rollup 中，我们要想使用 node_modules 里面的包，必须使用 node-resolve 这个插件才行，这一点和 Webpack 很不一样，在 Webpack，我们可以无需任何配置，就能直接使用 node_modules 的包。
```js
import resolve from '@rollup/plugin-node-resolve';

export default {
  ...
  plugins: [resolve()],
  ...
}
```


#### rollup-plugin-commonjs

rollup默认是不支持CommonJS模块的，自己写的时候可以尽量避免使用CommonJS模块的语法，但有些外部库的是cjs或者umd（由webpack打包的），所以使用这些外部库就需要支持CommonJS模块。

添加支持common.js的插件
```
npm i rollup-plugin-commonjs --D
```

在plugin中添加
```
const commonjs = require('rollup-plugin-commonjs');
...
  plugins:[
    commonjs()
  ]
...
```
### 加载css

#### rollup-plugin-postcss

处理css需要用到的插件是rollup-plugin-postcss。它支持css文件的加载、css加前缀、css压缩、对scss/less的支持等等。
。

### 打包优化

#### rollup-plugin-delete
用来清理指定的目录或文件。

```

const del = require('rollup-plugin-delete');
...
plugins:[
    del({ targets: 'dist/*',  verbose: true }),
]
...
```

#### rollup-plugin-terser

在生产环境中，代码压缩是必不可少的。我们使用rollup-plugin-terser进行代码压缩。

>需要rollup版本2.x

`rollup-plugin-serve`、`rollup-plugin-livereload`

这两个插件常常一起使用，rollup-plugin-serve用于启动一个服务器，rollup-plugin-livereload用于文件变化时，实时刷新页面


### 编译es6,jsx,tsx,ts


#### babel插件

rollup打包出来的依赖默认是遵循es6语法，但是这样在有些地方不是很兼容

`需要rollup版本 < 3`

```
npm i rollup-plugin-babel @babel/core @babel/preset-env --D
```

配置
```js
 plugins:[
    babel({
        exclude: 'node_modules/**'
    })
  ]
```
新增babelrc，可以将这些配置项写到plugin的babel配置中，就可以不用再写一份babelrc文件
```js
{
  "presets": [
    [
      "@babel/env",
      {
        "modules": false // 设置为false,否则babel会在rollup有机会执行其操作之前导致我们的模块转化为commonjs
      }
    ]
  ]
}
```

#### 加载react

安装`@babel/preset-react`

配置
```js
plugin:[
    ...
    babel({
      exclude: 'node_modules/**',
      extensions: ['.js', '.jsx'],
      presets: ['@babel/preset-env','@babel/preset-react'],
    }),
    ...
]
```
### 加载typescript

安装
```
npm install @rollup/plugin-typescript typescript --save-dev 
```

配置
```
plugin:[
    ...
    typescript(),
    ...
]
```

更多配置项参考 https://www.npmjs.com/package/@rollup/plugin-typescript


### 使用别名替换相对路径

安装@rollup/plugin-alias
```
npm install @rollup/plugin-alias --save-dev
```

配置
```
plugins:[
    ...
    alias({
      entries: [{ find: '@', replacement: resolveDir(' src ') }],
    }),
    ...
]

```

### 启动本地服务和开启热替换

安装

```
npm install rollup-plugin-serve rollup-plugin-livereload --save-dev
```

配置
```
plugins:[
    ...
    serve({
      contentBase: '', //服务器启动的文件夹，默认是项目根目录，需要在该文件下创建index.html
      port: 8020, //端口号，默认10001
    }),

    livereload('dist'), //watch dist目录，当目录中的文件发生变化时，刷新页面
    ...
]
```