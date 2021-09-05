##  webpack

> webpack: 用来为现代 JavaScript 应用提供静态模块打包
> 打包： 将不同类型的资源按照模块进行打包 
> 静态： 打包后产出的静态资源

#### webpack 初体验


```bash
 # 全局安装webpack webpack-cli

 npm i -g webpack webpack-cli

 # 查看版本

 webpack --version
```

#### webpack config 

项目下新建webpack 配置文件 `webpack.config.js` 配置webpack 

``` bash
# webpack.config.js

const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'build.js',
    path: path.resolve(__dirname,'dist')
  },
  mode: 'development' # production
}

```
 - 如果想根据不同环境执行不同的 webpack 进行打包，可以配置多个 webpack config文件 。 
 - 例如: webpack.dev.config.js webpack.prod.config.js

 ```
 # package.json
 
 "scripts" : {
   ...
   "build": "webpack --config webpack.dev.config.js"
 }
 ```

 #### 解析 css 文件
 > 默认 js 代码不能解析 css 文件，所以需要 loader 对 css 文件进行解析

``` js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'build.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/, // 一般写一个正则来匹配需要处理的文件
        use: ['css-loader'] // 需要使用的loader， 顺序从后向前依次执行
      }
    ]
  },
  mode: 'development'
}
```
> css-loader 只是用来对 css 进行解析，但是不能把处理好的 css 进行展示
> 需要使用 style-loader 对 处理好的 css 文件插入到文件中

```
npm i style-loader --save-D
```

``` js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'build.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/, // 一般写一个正则来匹配需要处理的文件
        use: ['style-loader','css-loader'] // 需要使用的loader, 顺序从右往左依次执行
      }
    ]
  },
  mode: 'development'
}
```
> 项目中使用 less 需要先解析为css 
> 所以使用 less-loader 对less 文件解析

 如果项目中使用了 less ， 需要安装less ，less-loader
```
  npm i less less-loader -D
```
配置webpack config

``` js
const path = require('path')

module.exports = {
...
  module: {
    rules: [
      ...
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader','less-loader']
      }
    ]
  }
...
}
```

#### Browserslistrc 工作流程
> 在不同前端工具之间共享目标浏览器和 Node.js 版本的配置。主要用于: Autoprefixer
Babel
postcss-preset-env
...


安装
``` bash
npm i browserslist
```


配置方式: package.json 或者 .browserslistrc 配置

``` bash
 # .browserslistrc

defaults
not IE 11
maintained node versions
last 2 version # 最新的两个版本
>1%   

```

#### 如何兼容 css js

##### postcss  
> 利用 JavaScript 转换样式的工具

安装 

```bash
npm i postcss-loader -D

npm i autoprefixer -D
```

默认 postcss-loader 是使用 postcss 只是将 css 进行拷贝，不能做兼容处理，需要设置option 才可以

```js
...
rules: [
      {
        test: /\.css$/,
        use: ['style-loader','css-loader', {
          loader: 'postcss-loader',
          options: {
            postcssOptions: {
              plugins: [
                require('autoprefixer')
              ]
            }
          }
        }]
      }
    ]
...
```

###### postcss-preset-env
> 预设 插件的集合
postcss-preset-env 中已经包含 autoprefixer 插件，所以可以不使用 autoprefixer

```js
...
rules: [
      {
        test: /\.css$/,
        use: ['style-loader','css-loader', {
          loader: 'postcss-loader',
          options: {
            postcssOptions: {
              plugins: ['postcss-preset-env']
            }
          }
        }]
      }
    ]
...
```

或者 创建 postcss.config.js 配置

```js
 // postcss.config.js
module.exports = {
  plugins: [
    require('postcss-preset-env')
  ]
}
```
这样在webpack 中就不需要写那么复杂了

```js
...
module: {
    rules: [
      {
        test: /\.css$/, // 一般写一个正则来匹配需要处理的文件
        use: ['style-loader','css-loader','postcss-loader'] // 需要使用的loader， 顺序从后向前依次执行
      },
      {
        test: /\.less$/, // 一般写一个正则来匹配需要处理的文件
        use: ['style-loader', 'css-loader','postcss-loader','less-loader'] // 需要使用的loader， 顺序从后向前依次执行
      }
    ]
  },
...
```